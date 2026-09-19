# 第 3 章 klb (Lua 手测)

> `klua_doc/lua/lua_test/klb/` | 源码: `klua_run/lua_test/`klb/`
> 约定 **klua-test-design**

记名 **第 3 章** = **klb Lua 面**: **3.1** 内置 k*; **3.2+** 可裁剪协议包与对应 `klbcore.klb*` 脚本 **同节** (对齐第 1 章 `kgui`+`klbui`). **非** 历史 k* 桩烟测 (`backup/klbcore/help/k_test/`).

章号 **3.x** 与第 1/2 章、`pfs_test` **编号空间独立**. **冻结**: 1=klbui / 2=kpfs / 3=klb; 章内只追加节, 禁止再插入.

**源码目录** (`klua_run/lua_test/`klb/`): 节子目录 `builtin` `http` (3.3+ 预留 `ws` `rtsp` `smp` `mnp`); 用例 `ch3_s{M}_{z}.lua`（`3.M.z`）. `3.1.1` 历史文件 `kco_fork.lua` **不**迁入 `builtin/`. **已登记** `3.1.1` 与 `3.2.1`–`3.2.6`; 其余 3.1.x 条文+桩 **待实现**, 不登记.

---

## 运行

```bash
cd bin
./klua test.lua 3.1.1
./klua test.lua klb.kco.fork
```

Windows: `klua.exe test.lua 3.1.1`. 双入口: doc_id 或语义 id (`klb.*` / `klbhttp.*`).

## 批量 (第 3 章)

| 命令 | 说明 |
|------|------|
| `3.x` | 第 3 章已登记条 (`3.1.1`, `3.2.1`–`3.2.6`) |
| `3.1.x` / `3.1` | 节 3.1 (现行 `3.1.1`; `3.1.2+` 未登记) |
| `3.2.x` / `3.2` | 节 3.2 (`3.2.1`–`3.2.6`; `3.2.5`/`3.2.6` 公网, `list` `[SINGLE_ONLY]`) |
| `a` / `all` | 含本章已登记条 |

## 流程与 API 对照

| 本节 | 手测文档 | 阶段 | 主要 API |
|------|----------|------|----------|
| 3.1 | 本页 | 内置 k* | [`kco`](../../klua/kco.md) 等; **不含** `kgui` / `khttp` |
| 3.2 | [klbhttp.md](klbhttp.md) | HTTP | `khttp` + `klbcore.klbhttp` |
| 3.3+ | — | 预留 | `klbws` / `klbrtsp` / `klbsmp` / `klbmnp` (只追加节) |

`kgui` 见第 1 章. `khttp` 与脚本封装同在 3.2.

## 公共约定

| 项 | 约定 |
|----|------|
| 协程 | IO 型条须在 **kco** 协程内 (`kco.fork` / `kco.timeout` + `ksys.exit`) |
| 裁剪 | `require("khttp")` 等失败 → skip (`no-http` 等) |
| 网络 | `3.2.1`–`3.2.4` 环回 listen+client; `3.2.5`–`3.2.6` 公共站点 (不通 skip) |
| 工作目录 | `paths.case_dir(doc_id)` |
| 清理 | 条末 `disconnect` / `close`; 单条进程退出即可 |

---

## 3.1 内置 k*

API: [kco.md](../../klua/kco.md) 等. 源码: `3.1.1` → `klb/kco_fork.lua`; `3.1.2+` → `klb/builtin/ch3_s1_{z}.lua` → `lua_test.klb.builtin.ch3_s1_{z}`.

`3.1.1` **已实现**. 其余条为桩: `M.run` 返回 `nil`, **待实现**, 不登记. 其余内置 k* (`kos` / `ktime` / `kmcache` / `klist` / `krand` / `kkpa` / `klpc` …) 后续只追加条, 不新开节.

### lua 对照

| doc_id | 语义 id | lua `@brief` | 文件 |
|--------|---------|--------------|------|
| `3.1.1` | `klb.kco.fork` | kco.fork / kco.timeout 烟测 | `kco_fork.lua` |
| `3.1.2` | `klb.kurl.parse` | kurl.parse stub | `builtin/ch3_s1_2.lua` |
| `3.1.3` | `klb.ksys` | ksys stub | `builtin/ch3_s1_3.lua` |
| `3.1.4` | `klb.kenv` | kenv stub | `builtin/ch3_s1_4.lua` |
| `3.1.5` | `klb.kthread` | kthread stub | `builtin/ch3_s1_5.lua` |

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

### 3.1.2 kurl.parse

- `klb.kurl.parse`

### 3.1.3 ksys

- `klb.ksys`

### 3.1.4 kenv

- `klb.kenv`

### 3.1.5 kthread

- `klb.kthread`

---

## 3.2 klbhttp

手测 [klbhttp.md](klbhttp.md). `3.2.1`–`3.2.6` **已实现**.

---

## 新增用例

1. 在对应节 md 增 `### 3.x.y` (只追加, 禁止插到 3.1 前; 协议包不插到 3.2 前).
2. `klua_run/lua_test/`klb/`<节>/` 实现 `run(...)`（`ch3_s{M}_{z}.lua`）.
3. **已实现** 才 `registry_ch3.lua` 登记 `doc_id` 与 `ids`.
4. 更新根 [readme.md](../readme.md) § 已实现用例.
