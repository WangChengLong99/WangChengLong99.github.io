---
tags:
  - mysql
  - python
title: pymysql
---
# 主要用法

以下是对 `pymysql` 库中主要函数、方法和属性的系统总结。`pymysql` 遵循 [PEP 249](https://peps.python.org/pep-0249/)（Python 数据库 API v2.0），因此其接口与 `sqlite3`、`psycopg2` 等类似。

---

```python
# 查询数据
with con_data_center.cursor() as cursor:
    cursor.execute(sql.format(date_start,date_end))
    rows = cursor.fetchall()
headers = [i[0] for i in cursor.description]
data = pd.DataFrame(rows,columns=headers)
```

# 一、模块级函数

| 函数                                      | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| `pymysql.`**`connect(参数...)`**          | 创建数据库连接，返回 `Connection` 对象。常用参数：`host`, `port`, `user`, `password`, `database`, `charset`, `cursorclass` 等。 |
| `pymysql.`**`install_as_MySQLdb()`**      | 将 `pymysql` 伪装成 `MySQLdb` 模块，便于兼容旧代码。         |
| `pymysql.`**`escape_string(s)`**          | 对字符串进行转义，防止 SQL 注入（一般建议使用参数化查询而非手动转义）。 |
| `pymysql.`**`Binary(data)`**              | 包装二进制数据，用于插入 `BLOB` 类型字段。                   |
| `pymysql.`**`NULL`**                      | 表示为 SQL `NULL` 的常量。                                  |
| `pymysql.`**`version_info`**              | 模块版本信息的元组（如 `(1, 0, 2, 'final', 0)`）。           |
| `pymysql.`**`__version__`**               | 版本字符串。                                                 |

---

# 二、Connection 对象（连接类）

通过 `connect()` 返回。

## 2.1 常用方法

| 方法                                      | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| `conn.`**`cursor(cursor=None)`**          | 返回一个新的游标对象。可选的 `cursor` 参数指定游标类型（例如 `pymysql.cursors.DictCursor`）。 |
| `conn.`**`begin()`**                      | 显式开启事务（默认 autocommit 为 False 时自动开启）。        |
| `conn.`**`commit()`**                     | 提交当前事务，使所有更改永久生效。                           |
| `conn.`**`rollback()`**                   | 回滚当前事务，撤销未提交的更改。                             |
| `conn.`**`close()`**                      | 关闭连接。关闭后不能再进行任何数据库操作。                   |
| `conn.`**`ping(reconnect=True)`**         | 检查与服务器的连接是否存活。若 `reconnect=True` 且连接断开，则尝试重连。 |
| `conn.`**`select_db(db)`**                | 切换当前使用的数据库。                                       |
| `conn.`**`get_host_info()`**              | 返回连接的主机信息。                                         |
| `conn.`**`get_server_info()`**            | 返回 MySQL 服务器版本字符串。                                |
| `conn.`**`get_proto_info()`**             | 返回使用的协议版本号。                                       |
| `conn.`**`kill_thread(thread_id)`**       | 强制终止指定的线程（连接）。                                 |
| `conn.`**`set_charset(charset)`**         | 设置连接的字符集。                                           |
| `conn.`**`thread_id()`**                  | 返回当前连接在 MySQL 服务端的线程 ID。                       |
| `conn.`**`warning_count()`**              | 返回最近执行的 SQL 语句产生的警告数量。                      |

## 2.2 属性（读/写）

| 属性                                      | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| `conn.`**`autocommit`**                   | 布尔值，表示是否自动提交。可设置：`conn.autocommit = True`。 |
| `conn.`**`database`**                     | 当前使用的数据库名（只读）。                                 |
| `conn.`**`host`**                         | 连接的主机地址。                                             |
| `conn.`**`port`**                         | 连接的服务端口。                                             |
| `conn.`**`user`**                         | 连接的用户名。                                               |
| `conn.`**`db`**                           | 同 `database`。                                              |
| `conn.`**`charset`**                      | 字符集编码（如 `'utf8mb4'`）。                               |
| `conn.`**`server_status`**                | 服务器当前的状态位（整数，常用于判断事务状态）。             |
| `conn.`**`open`**                         | 布尔值，表示连接是否仍处于打开状态。                         |

---

# 三、Cursor 对象（游标类）

通过 `conn.cursor()` 获得。默认返回普通 `Cursor`，但也可指定 `DictCursor`、`SSCursor`（服务端游标）等。

## 3.1 常用方法（由 PEP 249 定义）

| 方法                                      | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| `cur.`**`execute(query, args=None)`**     | 执行单条 SQL 语句。`args` 可以是元组、列表或字典，用于参数化查询。返回受影响的行数（对于 `SELECT` 则返回行数）。 |
| `cur.`**`executemany(query, seq_of_args)`** | 对序列中的每组参数重复执行同一条 SQL，常用于批量插入。        |
| `cur.`**`fetchone()`**                    | 获取结果集的下一行，返回元组（或字典，若游标类型为 `DictCursor`）。无数据时返回 `None`。 |
| `cur.`**`fetchmany(size=None)`**          | 获取结果集的下 `size` 行（默认 `arraysize`），返回列表。       |
| `cur.`**`fetchall()`**                    | 获取结果集的所有剩余行，返回列表。                           |
| `cur.`**`nextset()`**                     | 跳至下一个可用结果集（如存储过程多结果），返回 `True` 或 `None`。 |
| `cur.`**`close()`**                       | 关闭游标。关闭后不能再使用。                                 |
| `cur.`**`setinputsizes(sizes)`**          | 预留，无实质效果。                                           |
| `cur.`**`setoutputsize(size, column=None)`** | 预留，无实质效果。                                           |
| `cur.`**`scroll(value, mode='relative')`** | 在结果集中移动位置。`mode` 可以是 `'relative'`（相对当前行移动）或 `'absolute'`（移动到绝对行号）。 |
| `cur.`**`callproc(procname, args=())`**   | 调用存储过程，返回参数元组。                                 |

## 3.2 属性（只读）

| 属性                                      | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| `cur.`**`description`**                   | 结果集列的描述信息，每个元素为 `(name, type_code, ...)` 元组。  |
| `cur.`**`rowcount`**                      | 最近一次 `execute` 影响或选择的行数。对 `SELECT` 在某些数据库下可能不准确（pymysql 默认返回实际行数）。 |
| `cur.`**`arraysize`**                     | `fetchmany` 默认抓取的行数，可读可写。                       |
| `cur.`**`lastrowid`**                     | 最近插入行自增字段的 ID（若存在）。                          |
| `cur.`**`rownumber`**                     | 当前光标在结果集中的位置（0 起始）。                         |
| `cur.`**`connection`**                    | 返回创建此游标的连接对象。                                   |

## 3.3 常用游标类型（通过 `cursorclass` 参数指定）

| 类名                            | 说明                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| `pymysql.cursors.`**`Cursor`**  | 默认游标，返回结果为元组。                                   |
| `pymysql.cursors.`**`DictCursor`** | 返回结果为字典（列名→值）。                                  |
| `pymysql.cursors.`**`SSCursor`**  | 服务端游标（无缓冲），用于大数据集，取一行网络请求一次。       |
| `pymysql.cursors.`**`SSDictCursor`** | 服务端+字典形式。                                            |

---

# 四、异常类（定义在 `pymysql.err` 中）

| 异常类                                    | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| `pymysql.`**`Warning`**                   | 基础警告类。                                                 |
| `pymysql.`**`Error`**                     | 所有异常的基类。                                             |
| `pymysql.`**`InterfaceError`**            | 与数据库接口相关的错误（如连接断开）。                       |
| `pymysql.`**`DatabaseError`**             | 与数据库相关的通用错误。                                     |
| `pymysql.`**`DataError`**                 | 数据处理错误（如数据超出范围）。                             |
| `pymysql.`**`OperationalError`**          | 操作错误（如连接超时、表不存在）。                           |
| `pymysql.`**`IntegrityError`**            | 完整性约束错误（如外键、唯一键冲突）。                       |
| `pymysql.`**`InternalError`**             | 数据库内部错误（如存储过程出错）。                           |
| `pymysql.`**`ProgrammingError`**          | SQL 语法错误、权限问题或表不存在等。                         |
| `pymysql.`**`NotSupportedError`**         | 不支持的操作（如事务中调用 commit 但数据库不允许）。         |

---

# 五、常用示例（快速参考）

```python
import pymysql

# 连接
conn = pymysql.connect(
    host='localhost', user='root', password='123',
    database='test', charset='utf8mb4', autocommit=False
)

# 获取普通游标（元组结果）
cur = conn.cursor()
cur.execute("SELECT id, name FROM users WHERE age > %s", (18,))
rows = cur.fetchall()
for row in rows:
    print(row[0], row[1])

# 使用字典游标
from pymysql.cursors import DictCursor
cur2 = conn.cursor(cursor=DictCursor)
cur2.execute("SELECT * FROM users")
row_dict = cur2.fetchone()
print(row_dict['name'])

# 批量插入
data = [('Alice', 25), ('Bob', 30)]
cur2.executemany("INSERT INTO users(name, age) VALUES (%s, %s)", data)
conn.commit()

# 关闭资源
cur.close()
cur2.close()
conn.close()
```

---

# 六、注意事项

1. **事务**：默认 `autocommit=False`，必须在执行数据修改语句后调用 `commit()`，否则更改不会保存。
2. **占位符**：使用 `%s` 作为占位符（无论数据类型），不要使用 `?` 或命名占位符（除非通过 `format` 参数修改）。
3. **SQL 注入**：永远通过 `execute` 的第二个参数传递动态值，避免拼接字符串。
4. **连接管理**：可以用 `with` 语句管理连接和游标（`Connection` 实现了上下文管理器，自动提交/回滚？实际上 `pymysql` 的 `__exit__` 会调用 `close`，但不自动 commit，需谨慎）。更推荐显式 `try-finally` 或使用连接池。

以上是对 `pymysql` 主要功能的一次完整总结，覆盖了开发中常用的 90% 以上的接口。更详细的文档可参考 [pymysql 官方文档](https://pymysql.readthedocs.io/)。