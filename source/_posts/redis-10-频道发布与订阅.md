---
title: Redis 10.频道发布与订阅
date: 2022-07-01T23:36:01.126Z
tags: [redis]
---
- [1. publish 发布频道](#1-publish-发布频道)
- [2. subscribe 订阅指定频道](#2-subscribe-订阅指定频道)
- [3. psubscribe 订阅已匹配频道](#3-psubscribe-订阅已匹配频道)

- 发布端：publish
- 订阅端：subscribe，psubscribe

# 1. publish 发布频道

```bash
publish 频道名称 发布内容

# 示例
127.0.0.1:6379> publish music_2 "It's Not Goodbye"
(integer) 1
127.0.0.1:6379> publish music "just one last dance"
(integer) 2
127.0.0.1:6379> publish music "stay"
(integer) 2

127.0.0.1:6379> publish music_2 "It's Not Goodbye"
(integer) 1
```

# 2. subscribe 订阅指定频道

```bash
subscribe 频道名称

# 示例
127.0.0.1:6379> subscribe music
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "music"
3) (integer) 1
1) "message"
2) "music"
3) "just one last dance"
1) "message"
2) "music"
3) "stay" "music"
3) (integer) 1
1) "message"
2) "music"
3) "just one last dance"
1) "message"
2) "music"
3) "stay"
```

# 3. psubscribe 订阅已匹配频道

```bash
psubscribe 匹配频道名称

# 示例
127.0.0.1:6379> psubscribe music*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "music*"
3) (integer) 1
1) "pmessage"
2) "music*"
3) "music"
4) "just one last dance"
1) "pmessage"
2) "music*"
3) "music"
4) "stay"

1) "pmessage"
2) "music*"
3) "music_2"
4) "It's Not Goodbye"
```