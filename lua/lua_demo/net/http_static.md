# 2.2 HTTP/HTTPS 静态服务

> `klua_doc/lua/lua_demo/net/http_static.md` — 源码: `klua_run/lua_demo/net/http_static/main.lua` → `lua_demo.net.http_static.main`
> 协议: `require("khttp")`, `require("klbcore.klbhttp")` | 枢纽: [readme.md](../readme.md)

长期开着的静态站点. **非** lua test `3.2.x` 环回烟测.

---

## 启动

```bash
cd klua_run
./klua demo.lua 2.2
./klua demo.lua net.http.static
./klua demo.lua 2.2 8000
./klua demo.lua 2.2 8000 0
```

Windows: `klua.exe demo.lua 2.2`.

| 项 | 值 |
|----|-----|
| id | `2.2` |
| CLI | `2.2` / `net.http.static` |
| 模块 | `lua_demo.net.http_static.main` |
| 宿主 | **klua** |
| 状态 | **已实现** |
| 结束 | 人工停 (Ctrl+C) |

参数: `[port]` 默认 **8000**; 同端口 HTTP+HTTPS 一个监听. 第2参 **`0`** 关闭 TLS. 端口越界则退出.

`khttp` / `klbcore.klbhttp` 未加载 (`no-http`) 则退出.

---

## 挂载

| URL 前缀 | 目录 | 缺目录 |
|----------|------|--------|
| `/` | `klua_run/demores/html` | **必须存在**, 否则退出 |
| `/lua_demo` | `klua_run/lua_demo/` | skip, 打印 `mount skip` |
| `/lua_test` | `klua_run/lua_test/` | skip, 打印 `mount skip` |

HTTPS: `klua_run/demores/tls/cert.pem` + `key.pem` (演示自签; 浏览器告警正常). 缺证书或 `no-ssl` 则只开 HTTP, 不退出.

浏览示例: `http://127.0.0.1:8000/` 、`https://127.0.0.1:8000/` 、`/lua_demo/` 、`/lua_test/`.

---

## 行为

- 仅 **GET**; 其它方法 **405**
- 目录有 `index.html` 则返回该页, 否则列目录 (过滤 `.svn`)
- 源码扩展 (`lua` `md` `txt` `c` `h` `cpp` `hpp` `json` `mdc`) 按 `text/plain; charset=utf-8` 浏览; 其余走 `klbcore.util.http_mime`
- URL 过滤 `~`、`../`、残留 `..`、`.svn`、空字节, 防越权; 非法路径 **400**

---

## 相关

- 脚本 http: [klbcore.klbhttp](../../klbcore/readme.md) / `require("khttp")`
- 同内容走 klbweb: [2.3](web_static.md)
- lua test 环回烟测 (对照, **不要**当本场景): [lua_test/klb/klbhttp.md](../../lua_test/klb/klbhttp.md)
