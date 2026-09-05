## zlib 压缩 (lua-zlib)

> **require**: `zlib` | 代码: [klb/src_c/klua/lua-zlib-1.3/lua_zlib.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/lua-zlib-1.3/lua_zlib.c)
> **文档样板**: bundled Lua API 四层 — 同 [ksys.md](../klua/ksys.md)
> **上游**: [lua-zlib](https://github.com/brimworks/lua-zlib) (MIT); 链接 klb `zlib-1.2.11`

### 导出 API

**模块**

| 符号 | 返回 | 说明 |
|------|------|------|
| `version()` | major, minor, patch | 链接的 zlib 版本号 |
| `deflate([level [, window]])` | function | 压缩流函数 |
| `inflate([window])` | function | 解压流函数 |
| `adler32(data [, adler])` | integer | Adler-32 |
| `crc32(data [, crc])` | integer | CRC-32 |
| `BEST_SPEED` | integer | 1 |
| `BEST_COMPRESSION` | integer | 9 |

**流函数** `stream = zlib.deflate(level)` / `zlib.inflate(window)`

| 调用 | 返回 | 说明 |
|------|------|------|
| `stream(input)` | out, eof, bytes_in, bytes_out | 增量压缩/解压 |
| `stream(input, "sync")` | 同上 | 同步 flush |
| `stream(input, "full")` | 同上 | 完全 flush (可重建状态) |
| `stream(input, "finish")` | 同上 | 结束流 (deflate 必用) |
| `stream:close()` | — | 提前释放 (亦可 GC) |

**inflate `window`**: 默认 `MAX_WBITS + 32` (自动 gzip/zlib 头检测); 纯 deflate 可传 `15`.

### 伪代码

```lua
--[[
-- @file   zlib.lua
-- @brief  require("zlib") — lua-zlib
--   \n C: ./klb/src_c/klua/lua-zlib-1.3/lua_zlib.c
--]]

local zlib = {}

zlib.BEST_SPEED = 1
zlib.BEST_COMPRESSION = 9


-- @brief 链接的 zlib 版本
-- @return [number(int)] major, [number(int)] minor, [number(int)] patch
zlib.version = function ()
	return 1, 2, 11
end


-- @brief 创建 deflate 流
-- @param [in] level[number(int)]	[可选] 1..9 或默认 6
-- @param [in] window[number(int)]	[可选] windowBits
-- @return [function] stream(input [, flush])
-- @note stream 返回 out[string], eof[boolean], bytes_in[int], bytes_out[int]
--   flush: nil | "sync" | "full" | "finish" (deflate 结束须 "finish")
zlib.deflate = function (level, window)
	local stream = function (input, flush)
		return "", true, 0, 0
	end

	-- @brief 提前释放流 (亦可依赖 GC)
	stream.close = function ()
		return
	end

	return stream
end


-- @brief 创建 inflate 流
-- @param [in] window[number(int)]	[可选] 默认 MAX_WBITS+32
-- @return [function] stream(input [, flush])
zlib.inflate = function (window)
	local stream = function (input, flush)
		return "", true, 0, 0
	end

	stream.close = function ()
		return
	end

	return stream
end


-- @brief Adler-32 / CRC-32 校验
zlib.adler32 = function (data, adler)
	return 0
end

zlib.crc32 = function (data, crc)
	return 0
end


return zlib
```

### 示例

```lua
local zlib = require("zlib")

local maj, min, patch = zlib.version()
print(string.format("zlib %d.%d.%d", maj, min, patch))

-- 一次性 gzip 风格压缩
local def = zlib.deflate(zlib.BEST_COMPRESSION)
local compressed, eof = def("hello world", "finish")
assert(eof)

local inf = zlib.inflate()
local plain, eof2 = inf(compressed)
print(plain)	-- hello world

-- 分块
local d = zlib.deflate()
local out1 = d("part1")
local out2, done = d("part2", "finish")
local all = out1 .. out2
```

### 注意

- 裁剪: `__KLB_NO_ZLIB__` (`no-zlib`)
- 与 klb 根级 C 库 `zlib-1.2.11/` **不同路径**; 本模块为 Lua 绑定
- deflate 结束前须对 stream 调用 `"finish"`, 否则输出不完整
- klb 构建未启用 `LZLIB_COMPAT`; 无 `zlib.compress` / `zlib.decompress` 一键函数
- 技能 **klb-vendor-subagents** — zlib 形态说明
