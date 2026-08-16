# 第 1 章 klb (k*)

> `klua_doc/lua/lua_test/klb/` — klb k* C→Lua 手测; 源码 `bin/lua_test/klb/`

总入口 [../readme.md](../readme.md). 约定 **klua-test-design**.

## 1.1 kco

API: [kco.md](../../klua/kco.md)

### 1.1.1 klb.kco.fork — kco.fork / kco.timeout 烟测

| 项 | 值 |
|----|-----|
| doc_id | `1.1.1` |
| CLI | `1.1.1` / `klb.kco.fork` / `klb.kco_fork` |
| 模块 | `lua_test.klb.kco_fork` |
| 状态 | **已实现** |

步骤:

1. `kco.fork` 打印 fork 参数.
2. `kco.timeout(500, ...)` 延迟后输出 PASS 并 `ksys.exit()`.

预期: 输出 `lua_test.klb.kco_fork start` 与 `lua_test.klb.kco_fork PASS`.

## 新增用例

1. 在本章下增 `## 1.x <模块>` 与 `### 1.x.y` 节.
2. `bin/lua_test/klb/` 实现 `run(...)`.
3. `registry.lua` 登记 `doc_id` 与 `ids`.
4. 更新根 [readme.md](../readme.md) § 已实现用例.
