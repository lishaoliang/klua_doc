# k* C→Lua 绑定 (L3)

> `klua_doc/klb/klua/design/k-bindings.md` — 实现: `klb/src_c/klua/**/klua_k*.c` — **本文: C 注册机制**. Lua `require` API 真源: [lua/klua/readme.md](../../../lua/klua/readme.md).

## 结论

**k\*** = klua **L3**: C 经 `klua_loadlib` 注册到 `registry._PRELOAD`, Lua 侧 `require("kxxx")`. **非** L1/L2 C 产品 API; **非** L6 `klbcore.*`.

## 与相邻层

| 层 | 形态 | 文档 |
|----|------|------|
| L1/L2 C 产品 | `klua_env_*`, `klua_ex_*` | [../api/klua_env.md](../api/klua_env.md) |
| **L3 k\*** | `require("krtsp")` 等 | [lua/klua/readme.md](../../../lua/klua/readme.md) |
| L6 klbcore | `require("klbcore.klbrtsp")` 等 | [lua/klbcore/readme.md](../../../lua/klbcore/readme.md) |

## 注册机制

1. `klua_env_doinit` → `cb_pre_load` 通常调 `klua_loadlib_all`
2. `klua_loadlib(L, klua_open_kxxx, "kxxx")` 写入 `_PRELOAD`
3. 首次 `require("kxxx")` 调 `klua_open_kxxx`, 返回模块表

详 [preload.md](preload.md), 全量清单 [require-guide.md](require-guide.md) §2.

## 编码约定 (ai)

| 项 | 约定 |
|----|------|
| C 文件 | `klua_kxxx.c`; 入口 `int klua_open_kxxx(lua_State* L)` |
| Lua 名 | 与 `klua_loadlib` 第三参数一致 |
| 网络 IO | 优先 `co_*` + **kco**; 见 [lua/klua/guide/coroutine.md](../../../lua/klua/guide/coroutine.md) |
| 对象 | 常用 userdata + `klb_obj_t` 桥 (`klua_object.h`) |
| C 编码 | **klb-coding-c**; 本层 glob `coding-klua` |
| 文档 | 每模块 `lua/klua/kxxx.md`; 结构见下节 § **Lua API 文档** |

## Lua API 文档 (k* / kpfs)

**适用范围**:

| 路径 | 模块 |
|------|------|
| `klua_doc/lua/klua/kxxx.md` | L3 `require("kxxx")` |
| `klua_doc/lua/kpfs/kpfs*.md` | `require("kpfs")` 及子模块 |

**真源**: C `luaL_Reg` / userdata 方法表与实现; k* 伪代码宜对照 `klb/bin/klbcore/help/k/kxxx.lua` (若有).

**枢纽** (非 API 正文): `lua/kpfs/readme.md` — 总览、加载、返回值、CLI 对照、完整流程; **勿** 塞入 `kpfs.md` 等 API 页.

### 固定四层 (顺序冻结)

| 序 | `###` 标题 | 内容 |
|----|------------|------|
| 文头 | `>` 引用块 | `require` 名; C 源路径; 链 [k-bindings.md](k-bindings.md) § Lua API 文档; kpfs 子模块链 [readme.md](../../../lua/kpfs/readme.md) 枢纽 |
| 1 | **导出 API** | **简略**: 函数/方法一览表; 单行说明; 模块级 `rc`/`force`/返回值约定一句带过. **细节放伪代码**, 勿在导出 API 展开参数形态/opts 字段/行为步骤 |
| 2 | **伪代码** | Lua 桩; **每函数** `@brief`/`@param`/`@return`/`@note`; **opts 与返回表内联字段注释**、**固定返回值** → **coding-lua** § API 伪代码 |
| 3 | **示例** | 可运行片段; `require("…")` 带括号 (**coding-lua**); 主路径 + 常见变体 |
| 4 | **注意** | 前置条件、分层边界、与邻模块/CLI 对照链 |

**四层之后** (可选, 布局冻结后只增补): `## info 表字段`、`## CLI 对照`、`## 盘符路径语法` 等; **不** 插入四层之间.

### 导出 API vs 伪代码 (分工)

| 导出 API | 伪代码 |
|----------|--------|
| 函数名 + 返回类型 + 一句话 | `@param` 逐项; **opts/返回表内联注释** (**coding-lua** § API 伪代码) |
| 模块级约定 (如 `rc, msg`, `force`) 一段 | 行为步骤、边界、错误码、示例调用 |
| 函数极多时可 `####` 分子表, **仍单行说明** | userdata 方法亦逐条注释 |
| **禁止** 与伪代码双真源矛盾 | 与 C / help 桩一致 |

**样板**: [kgui.md](../../../lua/klua/kgui.md) (历史分组表较细, **新建文档** 导出 API 以更简表为准, 如 [kpfs.md](../../../lua/kpfs/kpfs.md)); 伪代码 `@param` 以 [kpfs.md](../../../lua/kpfs/kpfs.md) `mount` 为准; **opts/返回表内联 + 固定 `info, rc, msg`** 以 [kpfs_probe.md](../../../lua/kpfs/kpfs_probe.md) `probe.all` 为准.

### ai 维护

| 须做 | 禁止 |
|------|------|
| 新建/大补时对照四层与上表分工 | 仅为对齐而重排既有正文 (文档布局冻结) |
| API 变更先改 C, 再同步导出 API 与伪代码 | 导出 API 写长文而伪代码无 `@param` |
| 示例遵守 **coding-lua** § require | 把 `klbcore.*` 目录索引结构混入 API 四层 |

**其它 Lua 文档**: `klbcore/` 根目录平铺 + `css/` 子目录 (klbui 控件/CSS, 见 [lua/klbcore/readme.md](../../../lua/klbcore/readme.md)); **非** API 四层; `klbcore` 模块 API 待按源码补充.

## 域技能对照

| k* 分组 | 域 C 技能 | L6 脚本封装 |
|---------|-----------|-------------|
| `krtsp`, `ksmp`, `kurl`… | **klb-net-design** | **klbcore-net-design** |
| `kgui` | **klb-gui-design** | **klbcore-design** § klbui |
| `ksmp`, `kmnp`, `khttp_mnp`… | **klb-mnp-smp-design** + **klb-net-design** | **klb-mnp-smp-design** § klbsmp |
| `kco`, `klpc` | **klb-klua-env-design** | — |

## 新增 k* 流程

1. 实现 `klua_kxxx.c` + `klua_open_kxxx` (域 C 技能约束)
2. 注册到 `klua_loadlib_all` 或产品 `cb_pre_load`
3. 新建 `lua/klua/kxxx.md`
4. 更新 [lua/klua/readme.md](../../../lua/klua/readme.md), [require-guide.md](require-guide.md) §2

用户说「记录 net 设计」: **C klbnet** → **klb-net-design**; **k\*** → `lua/klua/kxxx.md` + [readme.md](../../../lua/klua/readme.md); **klbcore** → **klbcore-net-design**.

## 相关

| 文档 | 内容 |
|------|------|
| [layers.md](layers.md) | L3 在八层中的位置 |
| [lua/klua/readme.md](../../../lua/klua/readme.md) | k* Lua API 索引 |

设计技能: **klb-klua-design** § L3 台账; 正文 API 以 `lua/klua/*.md` 为准.
