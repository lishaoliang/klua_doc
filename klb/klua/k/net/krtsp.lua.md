# krtsp

> `require("krtsp")` — 代码: `klb/src_c/klua/klua_net/klua_krtsp.c`

RTSP 客户端与服务端. IO 须在 **kco** 内. C **klb-net-design**; L6 `klbcore.klbrtsp`.

## 库函数

| 函数 | 说明 |
|------|------|
| `connect(...)` | RTSP 客户端 |
| `listen(...)` | 监听 |
| `new_serve(...)` | 新建服务侧连接 |

## client userdata (节选)

| 方法 | 说明 |
|------|------|
| `disconnect()` | 断开 |
| `send_text(...)` | 发送文本信令 |
| `co_recv(...)` | 协程收 |
| `free_media(...)` / `dump_media(...)` | 媒体相关 |

serve 侧方法见 `klua_krtsp_serve.c` (随 `krtsp` 一并注册元表).
