# k* C→Lua 绑定 (L3)

> `klua_doc/klb/klua/design/k-bindings.md` — 实现: `klb/src_c/klua/**/klua_k*.c`

## 结论

**k\*** = klua **L3**: C 经 `klua_loadlib` 注册到 `registry._PRELOAD`, Lua 侧 `require("kxxx")`. **非** L1/L2 C 产品 API; **非** L6 `klbcore.*`.

## 与相邻层

| 层 | 形态 | 文档 |
|----|------|------|
| L1/L2 C 产品 | `klua_env_*`, `klua_ex_*` | [c-env-api.md](c-env-api.md) |
| **L3 k\*** | `require("krtsp")` 等 | [k/readme.md](../k/readme.md) |
| L6 klbcore | `require("klbcore.klbrtsp")` 等 | [klbcore/design/net-rtsp.md](../../klbcore/design/net-rtsp.md) |

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
| 网络 IO | 优先 `co_*` + **kco**; 见 [coroutine.md](coroutine.md) |
| 对象 | 常用 userdata + `klb_obj_t` 桥 (`klua_object.h`) |
| C 编码 | **klb-coding-c**; 本层 glob `coding-klua` |
| 文档 | 每模块 `k/kxxx.lua.md` 或 `k/net/kxxx.lua.md` |

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
3. 新建 `klua/k/kxxx.lua.md` (或 `k/net/`)
4. 更新 [k/readme.md](../k/readme.md), [require-guide.md](require-guide.md) §2

用户说「记录 net 设计」: **C klbnet** → **klb-net-design**; **k\*** → 本目录 + `k/net/`; **klbcore** → **klbcore-net-design**.

## 相关

| 文档 | 内容 |
|------|------|
| [layers.md](layers.md) | L3 在八层中的位置 |
| [k/readme.md](../k/readme.md) | k* 文档索引 |

设计技能: **klb-klua-design** § L3 台账; 正文 API 以 `k/*.lua.md` 为准.
