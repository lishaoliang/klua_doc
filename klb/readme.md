# klb 文档

> `klua_doc/klb/` — 代码: [klb/](https://gitee.com/klua/klb). 文档真源 (Obsidian vault 子树); [klb/bin/klbdocs/](https://gitee.com/klua/klb/tree/trunk/bin/klbdocs/) 仅留跳转.

## 代码路径与 Gitee

源码路径统一 **[klb/inc/...](https://gitee.com/klua/klb/tree/trunk/inc)**、**[klb/src_c/...](https://gitee.com/klua/klb/tree/trunk/src_c)**、**[klb/bin/klbcore/...](https://gitee.com/klua/klb/tree/trunk/bin/klbcore)** 并链 Gitee **`trunk`**. 旧 C++ (`src_cpp/`、`inc_hpp/`) 已迁工作区 `backup/`, 不编入现行 klb. 细则: [_meta/code-path-gitee.md](../_meta/code-path-gitee.md).

## 导航

| 类别 | 入口 |
|------|------|
| **Lua 脚本文档** | [../lua/readme.md](../lua/readme.md) |
| 编译 / 裁剪 | [build.md](build.md), [build_clip.md](build_clip.md) |
| app (klbapp) | [klbapp/readme.md](klbapp/readme.md) |
| GUI (klbgui) | [klbgui/readme.md](klbgui/readme.md) |
| wlua (桌面 GUI 宿主) | [../wlua/readme.md](../wlua/readme.md) |
| klbcore 脚本库 | [../lua/klbcore/readme.md](../lua/klbcore/readme.md) |
| klua | [klua/readme.md](klua/readme.md) |
| 第三方协议副本 | [licenses/readme.md](licenses/readme.md) |

## 目录结构

```
klua_doc/klb/
  readme.md
  build.md          ← 编译入口 (Make / make win / 产物)
  build_clip.md     ← MY_CLIP 裁剪 (no-* / use-* / min-core / wui)
  klbapp/           ← 应用壳 (readme + design/ + api/)
  klbgui/           ← C klbgui (readme + design/ + api/)
  klua/             ← klua C 机制 (readme + design/ + api/; Lua k* → ../lua/klua/)
  licenses/         ← 第三方协议副本
```

C/C++ 模块文档格式 (readme / design / api): **doc-writing** § klb C/C++ 模块文档; 样板 [klbapp/readme.md](klbapp/readme.md).

## 查阅顺序 (ai)

1. 本目录用户文档
2. [klb/inc/<模块>/](https://gitee.com/klua/klb/tree/trunk/inc/<模块>/)
3. [klb/src_c/<模块>/](https://gitee.com/klua/klb/tree/trunk/src_c/<模块>/)（C 实现; C++ 包装归档见工作区 `backup/src_cpp/`、`backup/inc_hpp/`）
