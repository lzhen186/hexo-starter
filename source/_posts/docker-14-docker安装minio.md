---
title: " Docker 14.Docker安装Minio"
date: 2022-09-11T08:36:02.464Z
---
1. 下载旧版本

   ```dockerfile
   docker pull minio/minio:RELEASE.2021-06-17T00-10-46Z
   ```
2. MinIO后台的方式启动
   如果要后台运行 加入 -d 参数


   ```dockerfile
   1. docker run -d -p 9000:9000 --name minio\
        -e "MINIO_ACCESS_KEY=admin" \
        -e "MINIO_SECRET_KEY=11111111" \
        -v /usr/local/minio/data:/data \
        -v /usr/local/minio/config:/root/.minio \
        minio/minio:RELEASE.2021-06-17T00-10-46Z server /data
   ```