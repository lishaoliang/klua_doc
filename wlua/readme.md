# wlua 文档

> `klua_doc/wlua/` — 代码: [wlua/](https://gitee.com/klua/wlua) — **klbgui P0a 桌面画布后端**（SDL + wsdl）

## 代码路径与 Gitee

源码路径 **[wlua/...](https://gitee.com/klua/wlua)**; 关联 klb 产物见 [klb/bin/wlua.exe](https://gitee.com/klua/klb/blob/trunk/bin/wlua.exe). 细则: [_meta/code-path-gitee.md](../_meta/code-path-gitee.md).

**wlua** 是带 SDL 窗口的 Lua 进程，为 klbgui 脚本提供桌面运行环境。GUI 框架设计见 [klb/klbgui/design/layers.md](../klb/klbgui/design/layers.md)；画布契约 [klb/klbgui/design/canvas.md](../klb/klbgui/design/canvas.md).

## 子目录

| 路径 | 内容 |
|------|------|
| （本 readme） | 定位、对接、构建指针 |

设计技能（ai）：**wlua-design**（`.cursor/skills/wlua-design/`）.

## 定位

| 对比 | `klua`（klb 默认） | `wlua` |
|------|-------------------|--------|
| 窗口 | 无（产品自备） | SDL `SDL_Window` + Renderer |
| GUI 画布 | 产品实现 P0 后端 | `wsdl` 桥接 `klb_canvas` ↔ SDL Texture |
| 预加载 | `klua_loadlib_all` | 同上 + `require("wsdl")` |
| 产物 | [klb/bin/klua.exe](https://gitee.com/klua/klb/blob/trunk/bin/klua.exe) | [klb/bin/wlua.exe](https://gitee.com/klua/klb/blob/trunk/bin/wlua.exe) |

## 与 klbgui 对接（P0a）

```
wlua.exe
  → klua_env + WSDL_EXTENSION
  → wsdl.open_wnd(w, h, title)
  → klbui.load_font(path)        // 或 kgui.load_font；canvas vtable.load_font
  → wsdl_wnd + wsdl_surface_canvas   // klb_canvas vtable 实现
  → klb_gui_attach_canvas(p_gui, p_canvas)
  → klb_gui_loop_once / klua_env_loop_once
```

| 模块 | 路径 | 职责 |
|------|------|------|
| `wsdl_wnd` | [wlua/wsdl/](https://gitee.com/klua/wlua/tree/trunk/wsdl/) | SDL 窗口与呈现 |
| `wsdl_surface_canvas` | [wlua/wsdl/wsdl_surface_canvas.cpp](https://gitee.com/klua/wlua/blob/trunk/wsdl/wsdl_surface_canvas.cpp) | `klb_canvas_t` 软/硬缓冲 |
| `wsdl_surface_canvas_opt` | 同上 | `draw_opt` / FreeType 等 |
| `ft_cache` | [wlua/wsdl/ft_cache.cpp](https://gitee.com/klua/wlua/blob/trunk/wsdl/ft_cache.cpp) | 字体光栅 |

`klb_wnd_ex.h` 中 draw_opt 100～512 由 wsdl 后端实现或转发。

## 目录与产物

```
wlua/
  wlua.cpp              # main
  wsdl/                 # wsdl 模块 + env 扩展
  ff_dec/               # FFmpeg 解码（媒体控件）
  SDL2 / freetype / ffmpeg  # bundled
  proj/msvc2015/        # VS 工程
```

## 关联

| 入口 | 说明 |
|------|------|
| [klb/klbgui/readme.md](../klb/klbgui/readme.md) | C klbgui 文档 |
| [klb/klbgui/api/klb_canvas.md](../klb/klbgui/api/klb_canvas.md) | 画布 API |
| [lua/klua/kgui.md](../lua/klua/kgui.md) | `require("kgui")` |
| [lua/klbcore/readme.md](../lua/klbcore/readme.md) | 脚本 UI |

## 构建与调试

- VS2015 工程：[wlua/proj/msvc2015/](https://gitee.com/klua/wlua/tree/trunk/proj/msvc2015/)（ai 禁止读写作源码目录，见 **core-workflow**）
- 产物默认：[klb/bin/wlua.exe](https://gitee.com/klua/klb/blob/trunk/bin/wlua.exe)
- 编译验证：子项目 `proj/` MSBuild；运行目录 `klua_run/`（**workspace-bin-env**）

## 查阅顺序 (ai)

1. 本 readme
2. **wlua-design** 技能
3. [wlua/wsdl/](https://gitee.com/klua/wlua/tree/trunk/wsdl/)（实现细节）
4. [klb/klbgui/design/canvas.md](../klb/klbgui/design/canvas.md)
