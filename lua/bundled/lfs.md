# lfs (LuaFileSystem)

> 类别: ② bundled | `require("lfs")` | 代码: `klb/src_c/klua/luafilesystem-2.0/`

[LuaFileSystem](https://lunarmodules.github.io/luafilesystem/) 目录/属性/链接等文件系统操作.

## 加载

```lua
local lfs = require("lfs")
```

## klb 注意

- 默认编入; 无裁剪宏
- 嵌入式场景注意路径权限与 `lfs.chdir` 副作用

## 相关

| 文档 | 内容 |
|------|------|
| [bundled/readme.md](readme.md) | bundled 索引 |
| `klua_doc/klb/licenses/LICENSE-luafilesystem-2.0` | 协议 |

函数级 API 见上游文档.
