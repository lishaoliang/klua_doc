# cjson

> 类别: ② bundled | `require("cjson")`, `require("cjson.safe")` | 代码: `klb/src_c/klua/lua-cjson-2.1.0/`

[lua-cjson](https://github.com/openresty/lua-cjson) JSON 编解码. klb 内嵌 vendor, 经 `klua_loadlib` 预加载.

## 加载

```lua
local cjson = require("cjson")
local cjson_safe = require("cjson.safe")  -- 遇错返回 nil+err, 不抛异常
```

## klb 注意

- 默认编入; 无 `__KLB_NO_*__` 裁剪宏
- 上游 API 以 vendor 源码与官方文档为准

## 相关

| 文档 | 内容 |
|------|------|
| [bundled/readme.md](readme.md) | bundled 索引 |
| 技能 **klb-vendor-subagents** | vendor 形态 A |
| `klua_doc/klb/licenses/LICENSE-lua-cjson-2.1.0` | 协议 |

函数级 API 待按需从 vendor 摘录.
