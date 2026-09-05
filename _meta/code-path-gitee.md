# 代码路径与 Gitee 链接

> `klua_doc/_meta/code-path-gitee.md` — **klua_doc 自有项目文档** 引用 C/H/Lua 等源码时的统一写法. ai 细则见 **doc-writing** 技能 § 代码路径与 Gitee.

## 适用范围

| 包含 | 不含 |
|------|------|
| `klua_doc/klb/`、`lua/`、`pfs/`、`wlua/` 正文 | `ref/` 外部参照、`licenses/`、`_meta/`、`_assets/` |
| 文首 `>`、正文、表格、查阅顺序中的 **源码路径** | 函数名/API 标识符（`klb_gui_create`） |
| `.c` / `.h` / `.lua` / `.mk` / 目录 | 外部规范 URL（Microsoft NTFS PDF 等） |

## 路径写法（显示）

相对 **opendemo 工作区根**，带子项目前缀：

| 仓 | 显示前缀 | 示例 |
|----|----------|------|
| klb | `klb/` | `klb/src_c/klbgui/extensions/klbuiex_wndhash.h` |
| portfs | `portfs/` | `portfs/src/pfs.h` |
| wlua | `wlua/` | `wlua/wsdl/wsdl_wnd.c` |

脚本/UI：`klb/bin/klbcore/klbui/parser.lua`（**须**含 `klb/` 前缀，勿写裸 `bin/klbcore/`）。

内部头仅存在于 `src_c` 时仍写完整路径，如 `klb/src_c/klbgui/klb_gui_in.h`。

## Gitee URL

对外 Web：[gitee.com/klua](https://gitee.com/klua) — 默认分支 **`trunk`**（非 `master`）。

| 显示路径 | Gitee URL |
|----------|-----------|
| 文件 `klb/src_c/klbgui/klb_wnd.c` | `https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_wnd.c` |
| 目录 `klb/inc/klbgui/` | `https://gitee.com/klua/klb/tree/trunk/inc/klbgui` |
| 文件 `portfs/src/pfs.h` | `https://gitee.com/klua/portfs/blob/trunk/src/pfs.h` |
| 文件 `wlua/wsdl/wsdl_wnd.c` | `https://gitee.com/klua/wlua/blob/trunk/wsdl/wsdl_wnd.c` |

规则：**URL 路径 = 显示路径去掉 leading `klb/`、`portfs/`、`wlua/`**。

## Markdown 链接格式

```markdown
[klb/src_c/klbgui/klb_wnd.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_wnd.c)
[klb/inc/klbgui/](https://gitee.com/klua/klb/tree/trunk/inc/klbgui)
```

文首 blockquote 示例：

```markdown
> `klua_doc/klb/klbgui/design/wnd.md` — 头文件: [klb/inc/klbgui/klb_wnd.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_wnd.h); 实现: [klb/src_c/klbgui/klb_wnd.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_wnd.c)
```

## 子项目枢纽

各子树 `readme.md` 须链回本页或 **doc-writing** § 代码路径与 Gitee. **klbgui** 样板：[klb/klbgui/readme.md](../klb/klbgui/readme.md).

## 例外

| 项 | 处理 |
|----|------|
| `pfs/api/*.md` 带 `AUTO-GENERATED` | 由 `gen-api-docs` 脚本生成；改生成器而非手改 |
| 已是 `[text](https://gitee.com/klua/...)` | 不重复包裹 |
| 发布禁写 | 禁止 opendemo、ai 技能名（**gitee-publish-design**） |

@history 2026-09-05: 初版；klbgui 全量链 trunk.
@history 2026-09-05: 批量 linkify klua_doc/klb|lua|pfs|wlua 及 pfs/std_*；枢纽 readme 加本节.
