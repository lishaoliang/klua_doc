# 第 3 章 § 3.2 klbhttp

> API: `require("khttp")`, `require("klbcore.klbhttp")` | 枢纽: [readme.md](readme.md)
> 源码: `klua_run/lua_test/`klb/http/ch3_s2_{z}.lua` → `lua_test.klb.http.ch3_s2_{z}`

`khttp` 与脚本封装 **同节**. `no-http` 时 skip. IO 须在 **kco** 协程内. `3.2.1`–`3.2.4` 环回, **禁止**公网; `3.2.5`–`3.2.6` 访问公共站点, 不通则 skip. `3.2.1`–`3.2.6` **已实现** 并登记.

---

## lua 对照

| doc_id | 语义 id | lua `@brief` | 文件 |
|--------|---------|--------------|------|
| `3.2.1` | `klbhttp.co_get` | klbhttp.co_get 环回烟测 | `http/ch3_s2_1.lua` |
| `3.2.2` | `klbhttp.co_post` | klbhttp.co_post 环回烟测 | `http/ch3_s2_2.lua` |
| `3.2.3` | `klbhttp.connect` | klbhttp.connect 环回烟测 | `http/ch3_s2_3.lua` |
| `3.2.4` | `klbhttp.listen` | klbhttp.listen / co_accept 环回烟测 | `http/ch3_s2_4.lua` |
| `3.2.5` | `klbhttp.co_get.public` | klbhttp.co_get 公网站点 | `http/ch3_s2_5.lua` |
| `3.2.6` | `klbhttp.co_post.public` | klbhttp.co_post 公网站点 | `http/ch3_s2_6.lua` |

---

## 3.2 klbhttp

每条: CLI = `doc_id` / 语义 id; 模块 = `lua_test.klb.http.ch3_s2_{z}`; 状态 **已实现**.

### 3.2.1 klbhttp.co_get

| 项 | 值 |
|----|-----|
| doc_id | `3.2.1` |
| CLI | `3.2.1` / `klbhttp.co_get` |
| 模块 | `lua_test.klb.http.ch3_s2_1` |
| 状态 | **已实现** |

步骤:

1. 环回 `klbhttp.listen` 后 fork `serve_once` 回固定 body.
2. `klbhttp.co_get("http://127.0.0.1:<port>/ping")`.
3. 断言 `msg=="text"`, body 一致, `#body` 与 `Content-Length` 一致.

预期: PASS.

### 3.2.2 klbhttp.co_post

| 项 | 值 |
|----|-----|
| doc_id | `3.2.2` |
| CLI | `3.2.2` / `klbhttp.co_post` |
| 模块 | `lua_test.klb.http.ch3_s2_2` |
| 状态 | **已实现** |

步骤:

1. 环回 listen; 服务端把请求 body 原样回写.
2. `klbhttp.co_post(url, data, { content_type = "text/plain" })`.
3. 断言回应 body 等于 POST data, `#body` 与 `Content-Length` 一致.

预期: PASS.

### 3.2.3 klbhttp.connect

| 项 | 值 |
|----|-----|
| doc_id | `3.2.3` |
| CLI | `3.2.3` / `klbhttp.connect` |
| 模块 | `lua_test.klb.http.ch3_s2_3` |
| 状态 | **已实现** |

步骤:

1. 环回 listen.
2. `klbhttp.connect(host, port)` 后 `send` GET, `co_recv_text`.
3. 断言 `msg=="text"`, body 一致, `#body` 与 `Content-Length` 一致.

预期: PASS.

### 3.2.4 klbhttp.listen

| 项 | 值 |
|----|-----|
| doc_id | `3.2.4` |
| CLI | `3.2.4` / `klbhttp.listen` |
| 模块 | `lua_test.klb.http.ch3_s2_4` |
| 状态 | **已实现** |

步骤:

1. `klbhttp.new_listen()` + `open(port)` (不用 `listen` 快捷).
2. `co_accept` 后检查请求行含 `GET /ping`.
3. 客户端 `co_get`; 断言 body 与长度 (`Content-Length`).

预期: PASS.

### 3.2.5 klbhttp.co_get 公网站点

| 项 | 值 |
|----|-----|
| doc_id | `3.2.5` |
| CLI | `3.2.5` / `klbhttp.co_get.public` |
| 模块 | `lua_test.klb.http.ch3_s2_5` |
| 状态 | **已实现** |

步骤:

1. 依次 `klbhttp.co_get` 公共站点 (`example.com` / `neverssl.com` / `baidu.com`, 含 https).
2. 任一 **2xx**: 有 `Content-Length` 则等于 `#body`, 否则 `#body >= 64`.
3. 3xx 不记长度, 试下一条. `Content-Length` 与 `#body` 不一致则 FAIL.
4. 全部失败 (DNS/连接/超时/无合格 2xx) 则 skip.

预期: 有网 PASS; 离线 skip.

### 3.2.6 klbhttp.co_post 公网站点

| 项 | 值 |
|----|-----|
| doc_id | `3.2.6` |
| CLI | `3.2.6` / `klbhttp.co_post.public` |
| 模块 | `lua_test.klb.http.ch3_s2_6` |
| 状态 | **已实现** |

步骤:

1. 依次 `klbhttp.co_post` 到 echo 站点 (`httpbin.org` / `postman-echo.com`).
2. 任一 **2xx** 且 body 含 POST data, 并按 3.2.5 同样核对长度.
3. 全部失败则 skip.

预期: 有网且 echo 可达 PASS; 否则 skip.
