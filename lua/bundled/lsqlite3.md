## SQLite3 绑定 (lsqlite3)

> **require**: `lsqlite3` | 代码: `klb/src_c/klua/lsqlite3/lsqlite3.c`
> **文档样板**: bundled Lua API 四层 — 同 [ksys.md](../klua/ksys.md)
> **上游**: [lsqlite3](http://lua.sqlite.org/index.cgi/doc/tip/doc/lsqlite3.wiki) (MIT); 内嵌 SQLite3

### 导出 API

**模块** (`local sqlite3 = require("lsqlite3")`)

| 函数 | 返回 | 说明 |
|------|------|------|
| `open(path [, flags])` | db | 打开数据库文件 |
| `open_memory([name])` | db | 内存库 |
| `open_ptr(ptr [, name])` | db | 已有 sqlite3* |
| `version()` | string | SQLite 库版本 |
| `lversion()` | string | lsqlite3 绑定版本 |
| `complete(sql)` | boolean | SQL 是否完整语句 |
| `backup_init(dest, dest_name, src, src_name)` | bu | 在线备份句柄 |
| `temp_directory()` | string | 临时目录 (非 Windows) |
| `SQLITE_*` 等 | integer | 常量 (OK, ROW, DONE, …) |

**db 对象** (metatable)

| 方法 | 说明 |
|------|------|
| `exec(sql [, params])` / `execute` | 执行 SQL |
| `prepare(sql)` | 预编译语句 → stmt |
| `rows` / `urows` / `nrows` | 迭代查询 |
| `last_insert_rowid()` | 末次 INSERT rowid |
| `changes()` / `total_changes()` | 影响行数 |
| `errcode()` / `errmsg()` | 错误码/信息 |
| `create_function` / `create_aggregate` / `create_collation` | 扩展 SQL 函数 |
| `trace` / `progress_handler` / `busy_handler` / `busy_timeout` | 回调与超时 |
| `update_hook` / `commit_hook` / `rollback_hook` | 事务钩子 |
| `load_extension` | 加载扩展 (平台相关) |
| `close()` | 关闭连接 |
| `isopen()` | 是否仍打开 |

**stmt 对象** (metatable)

| 方法 | 说明 |
|------|------|
| `bind` / `bind_values` / `bind_names` / `bind_blob` | 绑定参数 |
| `step()` | 执行一步; `SQLITE_ROW` / `SQLITE_DONE` |
| `reset()` / `finalize()` | 重置 / 释放 |
| `get_value(s)` / `get_values()` / `get_names()` / `get_types()` | 列访问 |
| `get_named_values()` / `get_named_types()` | 命名列 |
| `rows` / `urows` / `nrows` | 结果迭代 |
| `columns()` | 列数 |
| `isopen()` | 是否有效 |

**backup 对象**: `step()`, `remaining()`, `pagecount()`, `finish()`.

### 伪代码

```lua
--[[
-- @file   lsqlite3.lua
-- @brief  require("lsqlite3")
--   \n C: ./klb/src_c/klua/lsqlite3/lsqlite3.c
--]]

local sqlite3 = {}

sqlite3.SQLITE_OK = 0
sqlite3.SQLITE_ROW = 100
sqlite3.SQLITE_DONE = 101
-- … 其余 SQLITE_* 常量见 sqlitelib / 上游 wiki


-- @brief 打开数据库文件
-- @param [in] path[string]			文件路径; ":memory:" 同 open_memory
-- @param [in] flags[number(int)]		[可选] SQLITE_OPEN_* 组合
-- @return [userdata] db				成功
-- @return nil, err					失败
sqlite3.open = function (path, flags)
	return {}
end


-- @brief 打开内存数据库
-- @param [in] name[string]			[可选] 内存库名
-- @return [userdata] db
sqlite3.open_memory = function (name)
	return {}
end


-- @brief 包装已有 sqlite3* 指针
-- @param [in] ptr[lightuserdata]		sqlite3*
-- @param [in] name[string]			[可选] 库名
-- @return [userdata] db
sqlite3.open_ptr = function (ptr, name)
	return {}
end


-- @brief 链接的 SQLite 库版本
-- @return [string] 版本串
sqlite3.version = function ()
	return "3.x.x"
end


-- @brief lsqlite3 绑定版本
-- @return [string] 版本串
sqlite3.lversion = function ()
	return "unknown"
end


-- @brief SQL 语句是否语法完整
-- @param [in] sql[string]
-- @return [boolean] true 完整
sqlite3.complete = function (sql)
	return true
end


-- @brief 在线备份句柄 (非 Windows 另有 temp_directory)
-- @param [in] dest_db[userdata]		目标 db
-- @param [in] dest_name[string]		目标库名, eg. "main"
-- @param [in] src_db[userdata]		源 db
-- @param [in] src_name[string]		源库名
-- @return [userdata] backup			成功; 失败无返回值
sqlite3.backup_init = function (dest_db, dest_name, src_db, src_name)
	return {}
end


-- db 对象 (metatable `lsqlite3.db*`) — open/open_memory/open_ptr 返回
local db = {}


-- @brief 执行 SQL (无结果集或忽略结果)
-- @param [in] sql[string]
-- @param [in] params[table]			[可选] 绑定参数
-- @return [number(int)]				SQLITE_* ; 0/SQLITE_OK 成功
db.exec = function (sql, params)
	return sqlite3.SQLITE_OK
end

db.execute = db.exec


-- @brief 预编译语句
-- @param [in] sql[string]
-- @return [userdata] stmt
db.prepare = function (sql)
	return {}
end


-- @brief 迭代查询 (命名列)
-- @param [in] sql[string]
-- @param [in] params[table]			[可选]
-- @return [function] iterator(row_table)
db.nrows = function (sql, params)
	return function () return nil end
end


-- @brief 迭代查询 (数组列)
db.urows = function (sql, params)
	return function () return nil end
end


-- @brief 迭代查询 (混合)
db.rows = function (sql, params)
	return function () return nil end
end


-- @brief 末次 INSERT rowid
-- @return [number(int)]
db.last_insert_rowid = function ()
	return 0
end


-- @brief 上条语句影响行数
-- @return [number(int)]
db.changes = function ()
	return 0
end


-- @brief 连接累计影响行数
-- @return [number(int)]
db.total_changes = function ()
	return 0
end


-- @brief 错误码
-- @return [number(int)] SQLITE_*
db.errcode = function ()
	return sqlite3.SQLITE_OK
end


-- @brief 错误信息
-- @return [string]
db.errmsg = function ()
	return ""
end


-- @brief 注册 SQL 标量函数 / 聚合 / 排序规则
db.create_function = function (name, narg, func, ud)
	return
end

db.create_aggregate = function (name, narg, step, final, ud)
	return
end

db.create_collation = function (name, func, ud)
	return
end


-- @brief 跟踪 / 进度 / 忙等待回调
db.trace = function (func, ud)
	return
end

db.progress_handler = function (n, func, ud)
	return
end

db.busy_timeout = function (ms)
	return
end

db.busy_handler = function (func, ud)
	return
end


-- @brief 事务/更新钩子
db.update_hook = function (func, ud)
	return
end

db.commit_hook = function (func, ud)
	return
end

db.rollback_hook = function (func, ud)
	return
end


-- @brief 加载 SQLite 扩展 (平台/裁剪相关)
db.load_extension = function (path, entry)
	return
end


-- @brief 关闭连接
db.close = function ()
	return
end


-- @brief 连接是否仍打开
-- @return [boolean]
db.isopen = function ()
	return true
end


-- stmt 对象 (metatable `lsqlite3.vm*`) — db:prepare 返回
local stmt = {}


-- @brief 绑定第 index 个参数 (1-based)
stmt.bind = function (index, value)
	return sqlite3.SQLITE_OK
end


-- @brief 按序绑定多个参数
stmt.bind_values = function (...)
	return sqlite3.SQLITE_OK
end


-- @brief 按名绑定 (table 键为参数名)
stmt.bind_names = function (t)
	return sqlite3.SQLITE_OK
end


-- @brief 绑定 BLOB
stmt.bind_blob = function (index, blob)
	return sqlite3.SQLITE_OK
end


-- @brief 执行一步
-- @return [number(int)] SQLITE_ROW / SQLITE_DONE / 错误码
stmt.step = function ()
	return sqlite3.SQLITE_DONE
end


-- @brief 重置语句 (可再次 bind/step)
stmt.reset = function ()
	return sqlite3.SQLITE_OK
end


-- @brief 释放语句
stmt.finalize = function ()
	return sqlite3.SQLITE_OK
end


-- @brief 结果列数
-- @return [number(int)]
stmt.columns = function ()
	return 0
end


-- @brief 取第 i 列值 / 全部列值 / 列名 / 类型
stmt.get_value = function (i)
	return nil
end

stmt.get_values = function ()
	return {}
end

stmt.get_names = function ()
	return {}
end

stmt.get_types = function ()
	return {}
end


-- @brief 命名列结果
stmt.get_named_values = function ()
	return {}
end

stmt.get_named_types = function ()
	return {}
end


-- @brief 结果迭代 (同 db:rows 族)
stmt.rows = function ()
	return function () return nil end
end

stmt.urows = function ()
	return function () return nil end
end

stmt.nrows = function ()
	return function () return nil end
end


-- @brief 语句是否仍有效
-- @return [boolean]
stmt.isopen = function ()
	return true
end


-- backup 对象 — backup_init 返回
local bu = {}


-- @brief 备份一步 (nPage 页)
-- @param [in] nPage[number(int)]
-- @return [number(int)] SQLITE_* 状态
bu.step = function (nPage)
	return sqlite3.SQLITE_OK
end


-- @brief 剩余待备份页数
-- @return [number(int)]
bu.remaining = function ()
	return 0
end


-- @brief 总页数
-- @return [number(int)]
bu.pagecount = function ()
	return 0
end


-- @brief 结束备份
-- @return [number(int)] SQLITE_* 状态
bu.finish = function ()
	return sqlite3.SQLITE_OK
end


return sqlite3
```

db / stmt / backup 方法挂于 C metatable; 上表为伪代码示意. 兼容别名 `execute`、`error_code`/`error_message` 等同名见 `dblib[]`.

### 示例

```lua
local sqlite3 = require("lsqlite3")

local db = sqlite3.open(":memory:")
assert(db)

db:exec([[
	CREATE TABLE t (id INTEGER PRIMARY KEY, name TEXT);
	INSERT INTO t VALUES (1, 'alpha');
]])

for row in db:nrows("SELECT id, name FROM t") do
	print(row.id, row.name)
end

-- 预编译
local stmt = db:prepare("INSERT INTO t VALUES (?, ?)")
stmt:bind_values(2, "beta")
stmt:step()
stmt:finalize()

print(db:changes())
db:close()
```

### 注意

- 裁剪: `__KLB_NO_SQLITE__` (`no-sqlite`)
- 模块变量名惯例 `sqlite3`, require 名为 **`lsqlite3`**
- 嵌入式场景须评估存储、并发与 `busy_timeout`; 多线程访问须自行同步
- `load_extension` 在部分平台/裁剪构建中可能不可用
- 详 API 与常量以 vendor `lsqlite3.c` 与 [上游 wiki](http://lua.sqlite.org/index.cgi/doc/tip/doc/lsqlite3.wiki) 为准
