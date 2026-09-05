# P0 平台画布

> `klua_doc/klb/klbgui/design/canvas.md` — 头文件: [klb/inc/klbutil/klb_canvas.h](https://gitee.com/klua/klb/blob/trunk/inc/klbutil/klb_canvas.h), [klb/inc/klbgui/klb_wnd_ex.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_wnd_ex.h); 实现: [klb/src_c/klbutil/klb_canvas.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbutil/klb_canvas.c), [klb/src_c/klbutil/klb_canvas_argb8888.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbutil/klb_canvas_argb8888.c)

分层总览 [layers.md](layers.md). API 导读 [../api/klb_canvas.md](../api/klb_canvas.md).

## 结论

`klb_canvas_t` 是 klbgui **L0 / P0** 平台对接契约：上层只通过 `klb_canvas_*` 包装调用 `vtable`，不直接操作 fb/GPU/SDL。平台后端填充 vtable；软实现见 `klb_canvas_create()` + `klb_canvas_argb8888.c`。

## 栈位

```
P1  klbgui (klb_gui / klb_wnd / klbuiex_render)
      ↓ 绘制 / 刷新
P0  klb_canvas_t + vtable
      ↓
P0a 桌面 SDL     wlua/wsdl (wsdl_surface_canvas)
P0b 嵌入式 fb    规划
软实现           klb_canvas.c / klb_canvas_argb8888.c
```

## 边界

| 侧 | 职责 |
|----|------|
| 上层 | `klb_gui_attach_canvas` 挂主画布；`klb_wnd` 绘制；`klb_wnd_ex` 扩充 draw_opt |
| 契约 | `klb_canvas_vtable_t` — 绘制、资源、刷新、子层生命周期 |
| 平台 | 显存地址、`refresh*` 到屏幕、`malloc` 硬件子层 |

## 内存模型 (`klb_canvas_t`)

| 字段 | 说明 |
|------|------|
| `p_addr` / `phy_addr` | 虚拟 / 物理地址 |
| `bpp` / `pitch` / `color_fmt` / `mem_len` | 像素格式与行跨距 |
| `rect` | 画布在屏幕坐标中的区域 |
| `vtable` | 平台函数表 |
| `p_obj` | 平台私有对象 |

约束：`resize(w,h)` 在 **总 `mem_len` 不变** 前提下调整宽高，满足 `(bpp * w + padding) * h <= mem_len`。

## 多图层

### 画布图层类型 (`klb_canvas_layer_type_e`)

| 值 | 类型 | 数量 |
|----|------|------|
| `KLB_CANVAS_LAYER_main` | 主层 | 1（必须） |
| `KLB_CANVAS_LAYER_popup` | popup | 最多 4 |
| `KLB_CANVAS_LAYER_msgbox` | messagebox | 1 |
| `KLB_CANVAS_LAYER_udata` | 用户自定义 | 1 |
| `KLB_CANVAS_LAYER_wait` | 等待遮罩 | 1 |
| `KLB_CANVAS_LAYER_tip` | tip 浮层 | 1 |

单图层方案：main 显示 modal/popup/msgbox；tip 独立层。  
多图层方案：main / popup / msgbox / tip 各用独立画布，由 `refresh_layer` 合并刷新。

### 刷新方式 (`klb_canvas_refresh_opt_e`)

| 模式 | 用途 |
|------|------|
| `KLB_CANVAS_REFRESH_copy` | 全量重绘：依次拷贝各层 |
| `KLB_CANVAS_REFRESH_copy_bubble` | 局部刷新：拷贝后处理重叠区域冒泡 |

框架侧：`klbuiex_render` 在合成时调用 `klb_canvas_refresh_layer()`。查询是否多图层：`klb_gui_is_multi_canvas_layer()`。

### 子层申请

仅 **主画布** 可 `malloc(idx, rsv, layer_type)` 派生子层；子层 bpp 与主画布一致。图形适配决定初始尺寸；与 UI 期望不符时由框架 `resize`。

## 扩展槽

| 机制 | 说明 |
|------|------|
| `draw_opt1`～`draw_opt8` | 平台/控件私有绘图；窗口侧 opt 编号见 [klb/inc/klbgui/klb_wnd_ex.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_wnd_ex.h) 100～512 |
| `ioctrl_opt8` | 设备直通；GUI `klb_gui_canvas_ioctrl_opt8` |
| 字库/图集 | `vtable.load_font` / `load_image` 等（非 draw_opt） |

## 后端对照

| 后端 | 路径 | 文档 |
|------|------|------|
| P0a 桌面 SDL | [wlua/wsdl/](https://gitee.com/klua/wlua/tree/trunk/wsdl/) | [wlua/readme.md](../../../wlua/readme.md) |
| 软 ARGB8888 | [klb/src_c/klbutil/klb_canvas_argb8888.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbutil/klb_canvas_argb8888.c) | 本节 |
| P0b fb | 规划 | **klb-gui-design** |

## 关联

| 主题 | 入口 |
|------|------|
| 窗口图层栈 | [layer.md](layer.md) |
| 渲染管线 | [render.md](render.md) |
| 窗口绘制 | [draw.md](draw.md) |
| 渲染扩展 | [extension.md](extension.md) § render |
| 窗口扩充绘图 | [../api/klb_canvas.md](../api/klb_canvas.md) § klb_wnd_ex |
