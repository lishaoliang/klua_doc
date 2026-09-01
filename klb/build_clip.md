# klb 编译裁剪

> `klua_doc/klb/build_clip.md` — 代码: `klb/clip.mk`, `klb/clip-build.sh`, `build/scripts/lib/klb-clip.sh`
>
> 编译入口 → [build.md](build.md)

`clip.mk` 把两种入参 **归一成一套 `no-*`** (`MY_CLIP_TAG`, 全集的子集; 空 = 全量再编). 后面各段只认 `MY_CLIP_TAG`, 不认 `min-core` / `use-*`.

**VS2015 (`make win`) 不走本文件逻辑.**

## 两种入参

| 写法 | 含义 | 基准 |
|------|------|------|
| 减法 `no-*` / `--disable-*` | 从全量裁掉 | 空 (全量) |
| 加回 `min-core` + `use-*` / `--enable-*` | 最小集上加库 | 全集 10 项 `no-*` |

仅写 `use-*` (无 `min-core`) 时 **隐含 min-core**. `no-all` 同 `min-core`.

同条中 `use-*` **覆盖** 对应 `no-*` (enable 优先). **`min-core no-zlib` 不能加回 zlib**, 须写 `use-zlib`.

含空格时:

```bash
make 'MY_CLIP=min-core use-zlib'
make MY_CLIP="no-gui no-zlib"
make info   # 核对 MY_CLIP_TAG
```

## min-core 全集

`no-pcre2 no-lpeg no-sqlite no-zlib no-wui no-cpp no-gui no-format no-qrencode no-net-proto`

`no-wui-sim` **不在** 全集里 (只编 embed 时另加).

## 可选项 (负 / 正 / 宏)

| 负 (裁) | 正 (加回) | 宏 | 去掉 / 恢复 |
|---------|-----------|-----|-------------|
| `no-pcre2` | `use-pcre2` | `__KLB_NO_PCRE2__` | pcre2 |
| `no-lpeg` | `use-lpeg` | `__KLB_NO_LPEG__` | lpeg |
| `no-sqlite` | `use-sqlite` | `__KLB_NO_SQLITE__` | lsqlite3 |
| `no-zlib` | `use-zlib` | `__KLB_NO_ZLIB__` | zlib, lua-zlib |
| `no-cpp` | `use-cpp` | `__KLB_NO_CPP__` | `src_cpp/**` |
| `no-gui` | `use-gui` | `__KLB_NO_GUI__` | klbgui, kgui |
| `no-format` | `use-format` | `__KLB_NO_FORMAT__` | klbformat, kh26x |
| `no-qrencode` | `use-qrencode` | `__KLB_NO_QRENCODE__` | qrencode |
| `no-net-proto` | `use-net-proto` | `__KLB_NO_NET_PROTO__` | mnp/rtsp/smp 等; 仍留 kurl |
| `no-wui` | `use-wui` / `--enable-wui` | `__KLB_NO_WUI__` | 整包 wui; **正项连带 gui**, 且含 sim |
| `no-wui-sim` | (全量减 sim; 加回见下) | `__KLB_NO_WUI_SIM__` | 不编 `sim_*` |

`--disable-X` 同 `no-X`; `--enable-X` 同 `use-X`.

## wui (`src_packages/klbwui`)

> 代码: `klb/clip.mk` (wui 段), `klb/src_packages/klbwui/`

依赖 klbgui: `no-gui` 时整包跳过. **sim 含 embed**.

| 意图 | `MY_CLIP` | 编入 |
|------|-----------|------|
| 全量 (默认) | (空) | `core` + `embed_*` + `sim_*` |
| 整包关掉 | `no-wui` | 无; `-D__KLB_NO_WUI__` |
| 只要 embed | `no-wui-sim` | `core` + `embed_*`; `-D__KLB_NO_WUI_SIM__` |
| 最小 + 全量 wui | `min-core use-wui` | 去掉 `no-wui` 与 `no-gui` |
| 最小 + 仅 embed | `min-core use-wui-embed` | 同上, 再并上 `no-wui-sim` |

`use-wui-embed` / `--enable-wui-embed` **只作正向**. 裁 sim 用 `no-wui-sim` / `--disable-wui-sim`.

## 示例

```bash
# 最小
make MY_CLIP="min-core" -j8

# 最小 + zlib + lpeg
make 'MY_CLIP=min-core use-zlib use-lpeg' -j8

# 最小 + wui 仅 embed
make 'MY_CLIP=min-core use-wui-embed' -j8

# 全量减 GUI / 全量只留 wui embed
make MY_CLIP="no-gui" -j8
make MY_CLIP="no-wui-sim" -j8
```

脚本 (`klb/clip-build.sh`):

```bash
./clip-build.sh --min-core --enable zlib --enable lpeg --print
./clip-build.sh --min-core --enable wui-embed -j8
./clip-build.sh --full --disable-gui --disable-zlib -j8
```

## 相关

| 主题 | 入口 |
|------|------|
| 编译命令 / 产物 | [build.md](build.md) |
| 真源 | `klb/clip.mk` |
| GUI | [klbgui/readme.md](klbgui/readme.md) |
| 预加载与裁剪宏 | [klua/design/preload.md](klua/design/preload.md) |
