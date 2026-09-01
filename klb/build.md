# klb 编译

> `klua_doc/klb/build.md` — 代码: `klb/Makefile`, `klb/clip.mk`, `klb/clip-build.sh`
>
> 裁剪参数与 `MY_CLIP` → [build_clip.md](build_clip.md)

GNU Make (WSL / Linux) 编 `lib/libklb.{a,so}` 与 `lib/klua`. Windows 日常调试默认 **VS2015** (`make win`), **不走** `clip.mk`.

## 入口

| 方式 | 命令 | 说明 |
|------|------|------|
| GNU Make | `cd klb && make -j8` | 全量 (空 `MY_CLIP`) |
| 裁剪 | `make 'MY_CLIP=…'` | 见 [build_clip.md](build_clip.md); 含空格须整段引起来 |
| 脚本 | `./clip-build.sh --min-core --enable zlib -j8` | `klb/clip-build.sh`; `--print` 只打印归一后的 `no-*` |
| 查看归一 | `make 'MY_CLIP=…' info` | 看 `MY_CLIP_TAG`, `MY_CLIP_FLAGS`, `MY_DIRS` |
| VS2015 | `make win` | `klb.sln` Debug x86; **裁剪未同步** |

交叉: `MY_TOOL_CHAIN=arm-linux-gnueabi-`. 版本: `MY_VERSION=release` / `debug`.

## 产物

| 产物 | 路径 |
|------|------|
| 静态 / 动态库 | `klb/lib/libklb.a`, `klb/lib/libklb.so` |
| klua 可执行 | `klb/lib/klua` (目标 `make klua`) |
| 工作区调试目录 | 根 `klua_run/` (与子项目 `klb/lib` 区分; 见工作区 **klua_run**) |

`bin/klbcore/` Lua 脚本 **不**由 `MY_CLIP` 裁剪, 部署侧选择.

## 相关

| 主题 | 入口 |
|------|------|
| 裁剪 (`no-*` / `use-*` / min-core / wui) | [build_clip.md](build_clip.md) |
| 真源 | `klb/clip.mk`, `klb/Makefile` |
| GUI / wui | [klbgui/readme.md](klbgui/readme.md) |
