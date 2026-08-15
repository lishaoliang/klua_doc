# kkpa

> `require("kkpa")` — 代码: `klb/src_c/klua/klua_base/klua_kpackage.c`

klb **package** 文件读写的 k* 封装 (与 `klbbase` package 对应).

## 库函数

| 函数 | 说明 |
|------|------|
| `open_w(...)` | 打开可写 package, 返回 writer userdata |
| `open_r(...)` | 打开只读 package, 返回 reader userdata |

userdata 方法见源码 `klua_kpackage.c`.
