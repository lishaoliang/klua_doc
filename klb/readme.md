# klb 文档

> `klua_doc/klb/` — 代码: `klb/`. 文档真源 (Obsidian vault 子树); `klb/bin/klbdocs/` 仅留跳转.

## 导航

| 类别 | 入口 |
|------|------|
| API | [api/readme.md](api/readme.md) |
| app (klbapp) | [klbapp/readme.md](klbapp/readme.md) |
| GUI (klbui) | [klbui/readme.md](klbui/readme.md) |
| klbcore 脚本库 | [klbcore/readme.md](klbcore/readme.md) |
| klua | [klua/readme.md](klua/readme.md) |
| 第三方协议副本 | [licenses/readme.md](licenses/readme.md) |

## 目录结构

```
klua_doc/klb/
  readme.md
  api/              ← C API (手写 overview + 规划 _gen/)
  klbapp/           ← 应用壳, 启动, 扩展, plugins
  klbui/            ← GUI 控件, CSS, 设计
  klbcore/          ← 纯 Lua 运行时库 (net/rtsp/ui…)
  klua/             ← Lua API
  licenses/         ← 第三方协议副本
```

## 查阅顺序 (ai)

1. 本目录用户文档
2. `klb/inc/<模块>/`
3. `klb/src_c/<模块>/` 或 `klb/src_cpp/<模块>/`
