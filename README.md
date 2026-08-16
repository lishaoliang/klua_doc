# klua_doc

klua 生态集中文档库 (Obsidian vault). 与 `klb/`、`portfs/` 等代码仓并列; 用户文档逐步自各子项目迁入.

## 用途

| 放这里 | 不放这里 |
|--------|----------|
| klb / pfs 用户文档, API 说明, 规范镜像, 教程笔记 | 源码, 编译产物 |
| Obsidian 笔记与附件 (`_assets/`) | `.cursor/skills/*-design` 实现设计 |

## Obsidian

- Vault 根: 本目录 `klua_doc/`
- 默认附件: `_assets/` (见 `.obsidian/app.json`)
- 模板: `_meta/templates/`

## 协议

文档正文见 [LICENSE](LICENSE) (CC BY-SA 4.0). 源码协议见各代码仓 `LICENSE` (klb/pfs 为 LGPL-3.0).

---

# 导航

> Vault 总入口. 链接用相对路径 Markdown, 便于 Obsidian / Git / ai 共用.

## 子项目

| 子项目 | 入口 | 代码仓 |
|--------|------|--------|
| **lua** | [lua/readme.md](lua/readme.md) | `klb/` + `portfs/src_klua/` (脚本侧) |
| klb | [klb/readme.md](klb/readme.md) | `klb/` |
| pfs | [pfs/readme.md](pfs/readme.md) | `portfs/` |
| 其它 | [misc/readme.md](misc/readme.md) | — |

## 基础设施

| 路径 | 用途 |
|------|------|
| [_meta/inbox/](_meta/inbox/readme.md) | 速记收件箱 |
| [_meta/templates/](_meta/templates/) | Obsidian 笔记模板 |
| [_meta/tags.md](_meta/tags.md) | 标签约定 |
| [_assets/](_assets/readme.md) | 附件根 |
| [licenses/](licenses/readme.md) | 协议副本与索引 |

## 迁移状态

| 来源 | 目标 | 状态 |
|------|------|------|
| `klb/bin/klbdocs/` | `klua_doc/klb/` | 已迁入 (`klbdocs/` 仅跳转) |
| `portfs/docs/` | `klua_doc/pfs/` | 已迁入 |
| `klb/klua/k/` 等 | `lua/klua/` | 已迁入 (stub 已删) |
| `pfs/kpfs.md` | `lua/kpfs/` | 已迁入 (stub 已删) |
| `klb/klbcore/` (已删), `klb/klbui/widgets/` | `lua/klbcore/` | 已迁入; `klb/klbcore/` 目录已删除 |
| `klb/klbui/` (C klbgui 文档) | `klb/klbgui/` | 已重命名; 与源码 `klbgui/` 对齐 |
