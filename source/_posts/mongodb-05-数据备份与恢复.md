---
title: MongoDB 05.数据备份与恢复
date: 2022-07-02T00:16:28.459Z
tags: [mongodb]
---
# 1. mongoexport导出json/csv结构化数据

mongoexport命令导出的只有数据，不包括索引，mongoexport参数说明：

- -h：主机ip或域名 (默认localhost)
- –port：mongodb使用端口 (默认27107)
- -u：认证用户名 (当需要认证时用)
- -p：认证密码 (当需要认证时用)
- -d：指定导出的库名
- -c：指定导出的表名
- -f：指定导出的列名
- -q：查询条件，例如：’{sn:{“$lte”:100}}’
- -o：保存导出数据文件位置
- –csv：指定导出csv格式 (便于和传统数据库交换数据)，默认导出的json格式

```bash
# 导出json数据
mongoexport -h 192.168.8.200 --port 27017 -u vison -p 123456 -d test -c stu -f sn,name,email -q '{sn:{"$lte":100}}' -o /home/vison/src/test.stu.json

# 导出csv数据
mongoexport -h 192.168.8.200 --port 27017 -u vison -p 123456 -d test -c stu -f sn,name,email -q '{sn:{"$lte":100}}' --csv -o /home/vison/src/test.stu.csv
```

# 2. mongoimport导入json/csv结构化数据

mongoimport命令导入的只有数据，不包括索引，mongoimport参数说明：

- -h：主机ip或域名 (默认localhost)
- –port：mongodb使用端口 (默认27107)
- -u：认证用户名 (当需要认证时用)
- -p：认证密码 (当需要认证时用)
- -d：指定导入的库名
- -c：指定导入的表名(不存在会自己创建)
- –type：csv/json(默认json)
- –headline：当导入csv文件时，需要跳过第一行列名
- –file：导入数据文件的位置

```bash
# 导入json数据
mongoimport -h 192.168.8.200 --port 27017 -u vison -p 123456 -d test -c stu_json --type json --file /home/vison/src/test.stu.json

# 导入csv数据
mongoimport -h 192.168.8.200 --port 27017 -u vison -p 123456 -d test -c stu_csv --type csv --headerline --file /home/vison/src/test.stu.csv

# 注：老版本需要指定-fields参数
```

# 3. mongodump导出二进制数据

mongodump导出数据是包括索引的，mongodump的参数说明：

- -h：主机ip或域名 (默认localhost)
- –port：mongodb使用端口 (默认27107)
- -u：认证用户名 (当需要认证时用)
- -p：认证密码 (当需要认证时用)
- -d：指定导出的库名
- -c：指定导出的表名 (可选)
- -q：查询条件(可选)，例如：’{sn:{“$lte”:100}}’
- -o：保存导出数据文件位置(默认是导出到mongo下的dump目录)
- –gzip：导出并压缩

示例：

```bash
mongodump -h 192.168.8.200 –port 27017 -u vison -p 123456 -d test –gzip -o /home/vison/src/mongoDump
```

注：可以写脚本每天凌晨访问少的时候备份一次数据

# 4. mongorestore导入二进制数据

mongorestore导入数据是包括索引的，mongorestore的参数说明：

- -h：主机ip或域名 (默认localhost)
- –port：mongodb使用端口 (默认27107)
- -u：认证用户名 (当需要认证时用)
- -p：认证密码 (当需要认证时用)
- -d：指定导出的库名
- -c：指定导出的表名 (可选)
- –dir：保存导入数据文件位置
- –gzip：导出并压缩

示例：

```bash
mongorestore -h 192.168.8.200 –port 27017 -u vison -p 123456 -d test –gzip –dir /home/vison/src/mongoDump/test
```

