# excel字符串操作

在 Excel 中，字符串操作是数据清洗、拆分重组和文本分析的核心能力。下面从基础函数、查找定位、替换清理、转换合并，到 Excel 365 新函数和高级实战技巧，进行系统详解。

---

## 一、字符串提取与长度计算

| 函数 | 作用 | 语法 |
|------|------|------|
| **LEN** | 返回字符个数 | `=LEN(text)` |
| **LENB** | 返回字节数（汉字2字节） | `=LENB(text)` |
| **LEFT** | 从左截取N个字符 | `=LEFT(text, [num_chars])` |
| **RIGHT** | 从右截取N个字符 | `=RIGHT(text, [num_chars])` |
| **MID** | 从指定位置截取N个字符 | `=MID(text, start_num, num_chars)` |

### 经典实例
假设 A1 为 `"Excel 2021 函数指南"`  
- `=LEFT(A1, 5)` → `"Excel"`  
- `=RIGHT(A1, 4)` → `"函数指南"`  
- `=MID(A1, 7, 4)` → `"2021"`  
- `=LEN(A1)` → `13`（含空格）

**提取身份证出生日期**（18位，第7-14位为日期）  
`=TEXT(MID(A2,7,8),"0000-00-00")*1` 并设为日期格式，返回 `1990-12-25` 这类真实日期。

---

## 二、字符串查找与定位

| 函数 | 作用 | 区别 |
|------|------|------|
| **FIND** | 查找子串位置（**区分大小写**） | `=FIND(find_text, within_text, [start_num])` |
| **SEARCH** | 查找子串位置（**不区分大小写**，支持通配符 `?` `*`） | `=SEARCH(find_text, within_text, [start_num])` |

- 找到时返回数字（从1开始），未找到返回 `#VALUE!`。
- 常与 `IFERROR` 或 `ISNUMBER` 配合判断包含关系。

**实例：提取邮箱域名（@ 后面的部分）**  
`=RIGHT(A1, LEN(A1)-FIND("@", A1))`

**实例：判断 A1 是否包含“利润”一词（不区分大小写）**  
`=IF(ISNUMBER(SEARCH("利润", A1)), "包含", "不包含")`

**查找第二个分隔符位置**  
`=FIND("-", A1, FIND("-", A1)+1)` 跳过第一个分隔符继续找。

---

## 三、字符串替换与清理

| 函数 | 作用 | 说明 |
|------|------|------|
| **SUBSTITUTE** | 将指定字符替换为新字符 | `=SUBSTITUTE(text, old_text, new_text, [instance_num])`，可指定替换第几次出现 |
| **REPLACE** | 从指定位置开始替换固定长度 | `=REPLACE(old_text, start_num, num_chars, new_text)` 按位置替换，非按内容 |
| **TRIM** | 删除多余空格 | 只保留单词间单个空格，去掉首尾和中间多余空格 |
| **CLEAN** | 删除所有不可打印字符 | 常用于从网页、系统导出的数据（换行、制表符等） |

**实例：隐藏手机号中间四位**  
`=REPLACE(A1, 4, 4, "****")` 例如 `13812345678` → `138****5678`

**实例：删除文本中所有空格**  
`=SUBSTITUTE(A1, " ", "")`

**实例：只替换第二个“-”为“/”**  
`=SUBSTITUTE(A1, "-", "/", 2)`

**配合使用：删除换行符（CHAR(10) 是换行，CHAR(13) 是回车）**  
`=SUBSTITUTE(SUBSTITUTE(A1, CHAR(10), " "), CHAR(13), " ")`

---

## 四、文本转换与格式化

| 函数 | 作用 |
|------|------|
| **UPPER** | 全部大写 |
| **LOWER** | 全部小写 |
| **PROPER** | 首字母大写 |
| **VALUE** / **NUMBERVALUE** | 文本型数字转数值（VALUE简单转换，NUMBERVALUE可指定分隔符） |
| **TEXT** | 数值/日期按指定格式转为文本 |
| **T** | 若参数为文本返回其本身，否则返回空文本（常用于公式中过滤非文本） |
| **CHAR** / **CODE** | CHAR(数字)返回对应字符，CODE(字符)返回ASCII/Unicode码 |
| **UNICHAR/UNICODE** | 处理Unicode字符（如 emoji） |

**实例：数字金额加千分位并保留两位小数转为文本**  
`=TEXT(B2, "#,##0.00")`

**实例：将文本日期 `20231215` 转为标准日期**  
`=DATE(LEFT(B2,4), MID(B2,5,2), RIGHT(B2,2))`

**实例：强制将字符串转换为数字并容错**  
`=IFERROR(VALUE(A1), 0)`

---

## 五、字符串合并（功能强大）

### 1. 传统合并：`&` 和 CONCATENATE（不推荐）
`=A2 & " - " & B2`

### 2. CONCAT（Excel 2019/365）
连接区域或文本列表，可忽略空单元格。  
`=CONCAT(A1:A10)` 将区域所有文本连在一起。

### 3. TEXTJOIN（Excel 2019/365，强烈推荐）
指定分隔符，并可选择是否忽略空单元格。  
`=TEXTJOIN(", ", TRUE, A1:A10)`  
- 第一参数：分隔符  
- 第二参数：TRUE 忽略空单元格  
- 后续：要连接的一个或多个区域/文本

**实战：将筛选后的数据合并为一句话**  
`=TEXTJOIN("、", TRUE, FILTER(B2:B100, C2:C100="销售部"))`

---

## 六、Excel 365 专属字符串函数（颠覆传统）

如果使用 Microsoft 365 或 Excel 2021，以下新函数极大简化拆分提取操作。

| 函数 | 作用 | 示例 |
|------|------|------|
| **TEXTSPLIT** | 按分隔符将文本拆分为多行或多列 | `=TEXTSPLIT(A1, ",")` |
| **TEXTBEFORE** | 返回分隔符之前的文本 | `=TEXTBEFORE(A1, "@")` |
| **TEXTAFTER** | 返回分隔符之后的文本 | `=TEXTAFTER(A1, "@")` |

**TEXTSPLIT 详细语法**  
`=TEXTSPLIT(text, col_delimiter, [row_delimiter], [ignore_empty], [pad_with])`  
支持按行、按列同时拆分，复杂分割一步到位。

**实例：将“北京,上海;广州,深圳”拆分为2行2列**  
`=TEXTSPLIT(A1, ",", ";")` 产生一个2×2数组。

**实例：提取文件名不含扩展名**  
`=TEXTBEFORE(A1, ".")` 例如 `report.xlsx` → `report`

**实例：提取URL中最后一个斜杠后的部分**  
`=TEXTAFTER(A1, "/", -1)` 负号代表从末尾开始找。

---

## 七、高级组合技巧（不分版本通用）

### 1. 提取第N个分隔符之间的内容

比如数据 `A-B-C-D`，提取第三个字段（C）：

**旧版通用：**  

```excel
=TRIM(MID(SUBSTITUTE(A1, "-", REPT(" ", 99)), (3-1)*99+1, 99))
```
原理：将分隔符替换为大量空格，然后截取对应“块”，最后去空格。

**新版：**  
`=INDEX(TEXTSPLIT(A1, "-"), , 3)`

### 2. 提取字符串中连续数字
假设 A1 为 `"合同金额5800元"`，提取5800：
```excel
=TEXTJOIN("", TRUE, IF(ISNUMBER(--MID(A1, ROW(INDIRECT("1:"&LEN(A1))), 1)), MID(A1, ROW(INDIRECT("1:"&LEN(A1))), 1), ""))
```
（数组公式，旧版需按 Ctrl+Shift+Enter）  
或简化成基于 SEQUENCE 的公式（365）：  
```excel
=CONCAT(IF(ISNUMBER(--MID(A1, SEQUENCE(LEN(A1)), 1)), MID(A1, SEQUENCE(LEN(A1)), 1), ""))
```

### 3. 提取括号内的文字
`=MID(A1, FIND("(", A1)+1, FIND(")", A1)-FIND("(", A1)-1)`  
若括号可能缺失，外套 `IFERROR`。

### 4. 条件组合文本（多条件合并）
用 TEXTJOIN + IF 实现类 FILTER 的文本连接：  
`=TEXTJOIN(", ", TRUE, IF(B2:B100="优秀", A2:A100, ""))`  
在365中直接回车，旧版按 Ctrl+Shift+Enter。

### 5. 生成重复字符
`=REPT("★", 5)` → `"★★★★★"`  
常用于进度条、评分显示。

---

## 八、常见问题与注意事项

- **文本型数字的坑**：用 ISTEXT/ISNUMBER 检测，用 VALUE 或 `*1` 转换。  
- **FIND 大小写敏感**，SEARCH 不敏感且支持通配符，按需选用。  
- **TRIM 无法删除不间断空格**（CHAR(160)），可用 `=SUBSTITUTE(A1, CHAR(160), " ")` 预处理。  
- **TEXTJOIN/CONCAT 数组用法** 在旧版中需要三键，365 自动溢出。  
- **长公式性能**：避免整列引用（如 A:A），尽量用明确范围。  
- **旧版拆分文本**（无 TEXTSPLIT）仍可用分列功能或 MID+SUBSTITUTE 组合。

---

## 九、函数速查表

| 目的 | 推荐函数 |
|------|----------|
| 取左右子串 | LEFT, RIGHT, MID |
| 测量长度 | LEN (字符), LENB (字节) |
| 查找位置 | FIND (大小写), SEARCH (忽略大小写, 通配) |
| 替换内容 | SUBSTITUTE (按字符), REPLACE (按位置) |
| 去空格/清洗 | TRIM, CLEAN |
| 大小写转换 | UPPER, LOWER, PROPER |
| 数值 ↔ 文本 | TEXT, VALUE, NUMBERVALUE |
| 合并文本 | TEXTJOIN (推荐), CONCAT, & |
| 拆分文本 | TEXTSPLIT, TEXTBEFORE, TEXTAFTER (365)；或分列功能 |
| 生成重复字符 | REPT |
| 字符与代码 | CHAR, CODE, UNICHAR, UNICODE |

掌握上述函数和组合逻辑，你就能在 Excel 中应对绝大部分字符串处理需求，从简单的姓名提取到复杂的不规则地址解析，都能找到高效解法。
