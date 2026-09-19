# lua_demo 大型场景

> `klua_doc/lua/lua_demo/` — 记名 **lua demo** = 本体系; **非** **lua test**

## 代码路径与 Gitee

场景源码路径带 `klua_run/` 前缀. 脚本库: [klb/bin/klbcore/](https://gitee.com/klua/klb/tree/trunk/bin/klbcore). 细则: [_meta/code-path-gitee.md](../../_meta/code-path-gitee.md).

## 运行

```bash
cd klua_run
./klua demo.lua list
./klua demo.lua 2
./klua demo.lua 2.2
./klua demo.lua net.http.static
./klua demo.lua 2.3
./klua demo.lua net.web.static
```

Windows: `klua.exe demo.lua list`.

**双入口** (等价): 两层编号 `N.M` 或语义 id `net.http.static` / `net.web.static` / `ui.widgets`. `demo.lua 1` / `demo.lua 2` **只列出该章**, **不**启动.

**禁止**批量 (`a` / `N.x` / `N.M.x`): 场景是长期进程, 不能串跑. **禁止**用 `klua test.lua` 启动本目录.

参数经 `ksys.get_args()` 传递: `[1]` 宿主路径, `[2]` `demo.lua`, `[3]` 场景 id, `[4]…` 场景参数.

## 章索引

| 章 | 域 | 文档 | 源码 | 宿主 |
|----|----|------|------|------|
| **1** | ui | 见下表 (规划) | `klua_run/lua_demo/ui/` | **wlua** |
| **2** | net | 见下表 | `klua_run/lua_demo/net/` | **klua** |

章号 **冻结**: 1=ui / 2=net; 新域从 **3** 追加; 章内只追加条号, 禁止插入.

### 第 1 章 ui

| id | 语义 id | 体量 | 说明 | 状态 |
|----|---------|------|------|------|
| `1.1` | `ui.widgets` | 小型 | 单窗控件橱窗, 一直开着 | 规划, 未登记 |
| `1.2` | `ui.desktop` | 大型 | 多页壳 / 导航 / pref | 规划, 未登记 |

UI 图根: `klua_run/demores/images/<appearance>/`; **禁止** lua test 的 `tmpimage`.

### 第 2 章 net

| id | 语义 id | 文档 | 说明 | 状态 |
|----|---------|------|------|------|
| `2.1` | (待定) | — | 大型 net 场景 | 预留, 未登记 |
| `2.2` | `net.http.static` | [net/http_static.md](net/http_static.md) | HTTP/HTTPS 静态服务 (手写 `klbhttp`) | **已实现** |
| `2.3` | `net.web.static` | [net/web_static.md](net/web_static.md) | 同 2.2 内容, 走 `klbweb` | **已实现** |

`2.1` 条号已占用; **禁止**把静态服务改回 `2.1`.

## 已登记

| id | 语义 id | 宿主 | 文档 |
|----|---------|------|------|
| `2.2` | `net.http.static` | klua | [net/http_static.md](net/http_static.md) |
| `2.3` | `net.web.static` | klua | [net/web_static.md](net/web_static.md) |

## 目录

| 路径 | 说明 |
|------|------|
| `klua_run/demo.lua` | **唯一入口** |
| `klua_run/lua_demo/bootstrap.lua` | CLI 参数 |
| `klua_run/lua_demo/registry.lua` | 场景表、双入口、`list` |
| `klua_run/lua_demo/ui/` | **第 1 章** (规划, 尚无已登记条) |
| `klua_run/lua_demo/net/` | **第 2 章** (`http_static/` = `2.2`; `web_static/` = `2.3`) |
| `klua_run/demores/` | 共用资源 (html / media / tls / 皮肤图) |

入口 `main.lua` 导出 `run(...)`. 桩场景 **不**登记.

## 与 lua test 边界

| | lua test | lua demo |
|--|----------|----------|
| 目的 | 单 API / 单控件契约 | 可长期开着的场景 |
| 入口 | `klua_run/test.lua` | `klua_run/demo.lua` |
| 库 | `lua_test/` | `lua_demo/` |
| 编号 | 三层 `1.1.1` | 两层 `1.1` / `2.2` / `2.3` |
| 结束 | 断言 + `ksys.exit` | 人工停 |
| 批量 | `a` / `N.x` | **禁止** |

同一能力可两边都有: test 证接口, demo 证场景. 手测文档: [lua_test/readme.md](../lua_test/readme.md).

## 相关

- 仓说明: `klua_run/README.md`
- 起步: `klua_run/lua_demo/readme.md`
