# 第 3 章 klb (k*)

> `klua_doc/lua/lua_test/klb/` — klb k* C→Lua 手测; 源码 `klua_run/lua_test/`klb/`

总入口 [../readme.md](../readme.md). 约定 **klua-test-design**.

**源码目录**: `klua_run/lua_test/`klb/`（第 3 章根）; 新增用例推荐 `ch3_s{M}_{z}.lua`（`3.M.z`）; 现行 `kco_fork.lua` = `3.1.1`（历史文件名）.

章号冻结: 1=klbui / 2=kpfs / 3=klb; 新 k* 模块只追加 **3.2+**.

## 3.1 kco

API: [kco.md](../../klua/kco.md)

### 3.1.1 klb.kco.fork — kco.fork / kco.timeout 烟测

| 项 | 值 |
|----|-----|
| doc_id | `3.1.1` |
| CLI | `3.1.1` / `klb.kco.fork` / `klb.kco_fork` |
| 模块 | `lua_test.klb.kco_fork` |
| 状态 | **已实现** |

步骤:

1. `kco.fork` 打印 fork 参数.
2. `kco.timeout(500, ...)` 延迟后输出 PASS 并 `ksys.exit()`.

预期: 输出 `lua_test.klb.kco_fork start` 与 `lua_test.klb.kco_fork PASS`.

## 新增用例

1. 在本章下增 `## 3.x <模块>` 与 `### 3.x.y` 节 (只追加, 禁止插到 3.1 前).
2. `klua_run/lua_test/`klb/` 实现 `run(...)`（推荐 `ch3_s{M}_{z}.lua`）.
3. `registry_ch3.lua` 登记 `doc_id` 与 `ids`.
4. 更新根 [readme.md](../readme.md) § 已实现用例.
