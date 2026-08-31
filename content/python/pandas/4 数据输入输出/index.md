---
title: 4 数据输入输出
---

> [!abstract] 章节导读
> pandas 支持大量数据格式的读写。本章按数据组织形式分类介绍：文本文件、JSON、HTML/XML、Excel、二进制与列式格式、SQL，以及其他统计软件与云平台格式。

本章包含：

- [[4.1 文本文件]] – CSV 等分隔符文本文件
- [[4.2 JSON]] – JSON 与 JSON Lines
- [[4.3 HTML 与 XML]] – 网页表格与 XML 数据
- [[4.4 Excel]] – 电子表格读写
- [[4.5 二进制与列式格式]] – 高效存储格式
- [[4.6 SQL]] – 数据库交互
- [[4.7 其他数据格式]] – Stata / SAS / SPSS / BigQuery / Markdown

---

## 通用读写函数

所有读写函数均遵循：`read_*` 读取，`to_*` 写出。

| 格式 | 读取函数 | 写出函数 |
|------|---------|---------|
| CSV | `read_csv()` | `to_csv()` |
| JSON | `read_json()` | `to_json()` |
| HTML | `read_html()` | `to_html()` |
| XML | `read_xml()` | `to_xml()` |
| Excel | `read_excel()` | `to_excel()` |
| Pickle | `read_pickle()` | `to_pickle()` |
| Parquet | `read_parquet()` | `to_parquet()` |
| Feather | `read_feather()` | `to_feather()` |
| ORC | `read_orc()` | `to_orc()` |
| HDF5 | `read_hdf()` | `to_hdf()` |
| SQL | `read_sql()` | `to_sql()` |
| Stata | `read_stata()` | `to_stata()` |
| SAS | `read_sas()` | 仅读取 |
| SPSS | `read_spss()` | 仅读取 |
| GBQ | `read_gbq()` | `to_gbq()` |
| Markdown | `read_markdown()` | `to_markdown()` |

> [!tip] 路径与存储选项
> - 所有函数支持本地路径、URL、文件对象、字节流。
> - 云存储路径（s3://、gs://、azure://）需安装对应驱动并传入 `storage_options` 参数。
> - 压缩文件（.gz、.bz2、.zip、.xz）可通过 `compression` 参数自动或手动解压。


> [!success] 本章小结
> - **文本文件**：`read_csv` / `to_csv` 最常用，注意编码、分隔符、缺失值与分块读取。
> - **JSON**：灵活支持多种 `orient` 结构，`json_normalize` 处理嵌套数据。
> - **HTML/XML**：`read_html` 提取网页表格，`read_xml` / `to_xml` 处理标记语言。
> - **Excel**：`read_excel` / `to_excel` 支持多工作表，配合 `ExcelWriter` 灵活写入。
> - **二进制与列式**：Pickle 快速但危险；Parquet / Feather / ORC 高性能且跨语言。
> - **SQL**：`read_sql` / `to_sql` 通过 SQLAlchemy 连接主流数据库。
> - **其他格式**：Stata / SAS / SPSS 面向统计软件，Markdown 方便文档展示。
