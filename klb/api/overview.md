# klb API 总览

> `klua_doc/klb/api/overview.md` — 代码: `klb/inc/`

待补充: 模块索引与对外头文件导航.

## 模块入口 (规划)

| 模块 | 头文件路径 |
|------|------------|
| 公共类型 | `klb/inc/klb_type.h` |
| platform | `klb/inc/klbplatform/` |
| mem | `klb/inc/klbmem/` |
| util | `klb/inc/klbutil/` |
| base | `klb/inc/klbbase/` |
| net | `klb/inc/klbnet/` |
| gui | `klb/inc/klbgui/` |
| app | `klb/inc/klbapp/` — [klbapp 文档](../klbapp/readme.md) |
| lua | `klb/inc/klua/` — [klua 文档](../klua/readme.md) |

## klua (Lua 运行时)

> 代码: `klb/inc/klua/`, `klb/src_c/klua/`

| 类别 | 入口 |
|------|------|
| 环境 C API | `klb/inc/klua/klua_env.h`, `klua.h` |
| 设计 | [lifecycle](../klua/design/lifecycle.md), [preload](../klua/design/preload.md), [require-guide](../klua/design/require-guide.md), [coroutine](../klua/design/coroutine.md) |
| k* Lua API | [klua/k/readme.md](../klua/k/readme.md); 网络 [k/net/](../klua/k/net/) |
| 脚本库 klbcore | `klb/bin/klbcore/` — [klbcore](../klbcore/readme.md); ui [klbui](../klbui/readme.md); net/rtsp [design/net-rtsp](../klbcore/design/net-rtsp.md) |
