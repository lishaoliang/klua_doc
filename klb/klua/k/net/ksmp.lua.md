# ksmp

> `require("ksmp")` — 代码: `klb/src_c/klua/klua_net/klua_ksmp.c`

SMP 客户端/服务端与 RPC. IO 须在 **kco** 内. 协议 **klb-mnp-smp-design**; C 连接 **klb-net-design**; L6 `klbcore.klbsmp`.

## 库函数

| 函数 | 说明 |
|------|------|
| `connect(...)` | SMP 客户端 |
| `connect_rpc(...)` | SMP RPC 客户端 |
| `listen(...)` | SMP 服务端监听 |
| `listen_rpc(...)` | SMP RPC 监听 |

## client userdata (节选)

| 方法 | 说明 |
|------|------|
| `disconnect()` | 断开 |
| `send_text(...)` / `post(...)` / `notify(...)` | 发送 |
| `co_recv(...)` / `co_call(...)` / `co_wait(...)` | 协程 IO / RPC |
| `status()` | 状态 |

serve / RPC serve 元表见 `klua_ksmp_serve.c`.
