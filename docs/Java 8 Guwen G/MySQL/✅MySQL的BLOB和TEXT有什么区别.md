# 典型回答


在 MySQL 中，`BLOB` 和 `TEXT` 数据类型都用于存储大量数据。BLOB的全称是Binary Large Object，所以他主要被设计出来是存储二进制数据的，而TEXT主要是用于存储文本数据。



所以，**当我们要在数据库中存储二进制数据，比如图像、音频、视频等等，就可以把他们的二进制的对象存储到BLOB中。而我们如要存长文本，如文章、扩展信息等等，则通常使用TEXT。**

****

因为BLOB 数据类型按二进制方式处理数据，所以他不会对存储的数据进行字符集转换，并且不支持排序。而TEXT 数据类型则是以字符的方式存储文本数据，并且可以对数据进行字符集转换和排序等操作。



BLOB 和 TEXT 类型都有不同的变种，分别支持不同的存储大小：

+ TINYBLOB / TINYTEXT: 存储最大长度为 255 字节
+ BLOB / TEXT: 存储最大长度为 65,535 字节（64 KB）
+ MEDIUMBLOB / MEDIUMTEXT: 存储最大长度为 16,777,215 字节（16 MB）
+ LONGBLOB / LONGTEXT: 存储最大长度为 4,294,967,295 字节（4 GB）



| | 存储内容 | 字符集转换 | 排序 | 类型 |
| --- | --- | --- | --- | --- |
| BLOB | 二进制（如视频、音频） | 不支持 | 不支持 | TINYBLOB、BLOB、MEDIUMBLOB、LONGBLOB |
| TEXT | 文本（如文章内容） | 支持 | 支持 | TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT |






