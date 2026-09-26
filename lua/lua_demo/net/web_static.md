# 2.3 klbweb HTTP/HTTPS 静态服务

> `klua_doc/lua/lua_demo/net/web_static.md` — 源码: `klua_run/lua_demo/net/web_static/main.lua` → `lua_demo.net.web_static.main`
> 应用层: `require("klbcore.klbweb")` | 协议: `require("khttp")` | 枢纽: [readme.md](../readme.md)

与 [2.2](http_static.md) **同一套内容** (静态根 + `/lua_demo` `/lua_test` 浏览 + TLS), 走 [klb/bin/klbcore/klbweb/](https://gitee.com/klua/klb/tree/trunk/bin/klbcore/klbweb) 而非手写 `klbhttp` 循环.

---

## 启动

```bash
cd klua_run
./klua demo.lua 2.3
./klua demo.lua net.web.static
./klua demo.lua 2.3 8000
./klua demo.lua 2.3 8000 0
```

Windows: `klua.exe demo.lua 2.3`.

| 项 | 值 |
|----|-----|
| id | `2.3` |
| CLI | `2.3` / `net.web.static` |
| 模块 | `lua_demo.net.web_static.main` |
| 宿主 | **klua** |
| 状态 | **已实现** |
| 结束 | 人工停 (Ctrl+C) |

参数: `[port]` 默认 **8000**; 同端口 HTTP+HTTPS 一个监听. 第2参 **`0`** 关闭 TLS. 端口越界则退出.

`khttp` / `klbcore.klbweb` 未加载 (`no-http`) 则退出.

---

## 挂载

与 [2.2](http_static.md) 相同:

| URL 前缀 | 目录 | 缺目录 |
|----------|------|--------|
| `/` | `klua_run/demores/html` | **必须存在**, 否则退出 |
| `/lua_demo` | `klua_run/lua_demo/` | skip, 打印 `mount skip` |
| `/lua_test` | `klua_run/lua_test/` | skip, 打印 `mount skip` |

HTTPS: `setup.listen` 一项 `{ port, tls=true, plain=true, cert, key }` (klbweb 识别路径或 PEM 原文). 演示自签, 浏览器告警正常. 缺 PEM 文件则 skip HTTPS; 混用 bind 失败 (`no-ssl`) 回退仅 HTTP, 不退出.

浏览示例: `http://127.0.0.1:8000/` 、`https://127.0.0.1:8000/` 、`/lua_demo/` 、`/lua_test/`.

---

## 与 2.2 的差别

| | `2.2` | `2.3` |
|--|-------|-------|
| 库 | 手写 `klbhttp.listen` + 自管 GET | 内置 `klbweb.setup` + `klbweb.static` + `klbweb.serve` |
| `Server` | `lua_demo/2.2` | `lua_demo/2.3` |

内置站点: `klbweb.setup({ server = "lua_demo/2.3", listen = { port, 可选 tls+plain } })`; 每个挂载 `klbweb.static(prefix, root)`; 一次 `klbweb.serve` 自行 bind. 同端口 HTTP+HTTPS **一个监听**.

路径过滤、列目录、`index.html`、源码 `text/plain` 等由 **klbweb** 静态模块处理 (对齐 2.2 能力).

---

## 相关

- 脚本 web: `require("klbcore.klbweb")` — [klb/bin/klbcore/klbweb/](https://gitee.com/klua/klb/tree/trunk/bin/klbcore/klbweb)
- 对照手写协议层: [2.2 HTTP 静态](http_static.md)
