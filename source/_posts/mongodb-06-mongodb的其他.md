---
title: MongoDB 06.mongoDB的其他
date: 2022-07-02T00:18:56.904Z
tags: [mongodb]
---
# 1. 获取mongodb状态信息

首先在admin下创建一个新能够获取mongodb状态权限的用户

```bash
use admin
db.createUser(
  {
    user: "stat",
    pwd: "123456",
    roles: [{role: "mongostatRole", db: "admin"}]
  }
)
```

每隔2秒采集一次信息保存到文件stat.log

```bash
mongostat -u stat -p 123456 –authenticationDatabase admin -n 2 –json >> stat.log
```

每一条统计信息如下：

```json
JSON{
    "localhost:27017":{
        "arw":"1|0",
        "command":"2|0",
        "conn":"4",
        "delete":"*0",
        "dirty":"0.0%",
        "flushes":"0",
        "getmore":"0",
        "insert":"*0",
        "net_in":"18.9k",
        "net_out":"79.0k",
        "qrw":"0|0",
        "query":"94",
        "res":"49.0M",
        "time":"14:41:32",
        "update":"*0",
        "used":"0.1%",
        "vsize":"938M"
    }
}
```

更多mongostat命令说明看官网 https://docs.mongodb.com/manual/reference/program/mongostat/

# 2. 非正常关闭mongodb导致无法启动的解决方法

非正常关闭包括断电或强制关闭，造成文件mongod.lock锁住了，所以无法正常启动，解决方法：

```bash
(1) 删除mongod.lock文件，文件存放一般在数据库文件夹里
rm /data/mongodb/db/mongod.lock

(2) repair的模式启动
mongod -f /usr/local/mongodb/mongodb.conf –repair

(3) 启动mongodb
mongod -f /usr/local/mongodb/mongodb.conf
```