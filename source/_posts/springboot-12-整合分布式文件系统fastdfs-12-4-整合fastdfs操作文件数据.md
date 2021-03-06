---
title: SpringBoot 12.整合分布式文件系统fastdfs 12.4.整合fastdfs操作文件数据
date: 2022-07-03T14:18:07.765Z
tags: [springboot]
---
# 一、整合fastdfs

### 引入maven依赖坐标

```xml
        <dependency>
            <groupId>club.krislin.spring</groupId>
            <artifactId>krislin-fastdfs-spring-boot-starter</artifactId>
            <version>1.0.0</version>
        </dependency>
```

### 在application.yml里面加上如下配置:

```yaml
krislin:
  fastdfs:
    httpserver: http://192.168.161.3:8888/ #这个不是fastdfs属性，但是填上之后，在使用FastDFSClientUtil会得到完整的http文件访问路径
    connect_timeout: 5
    network_timeout: 30
    charset: UTF-8
    tracker_server: # tracker_server 可以配置成数组
      - 192.168.161.3:22122
    max_total: 50
    http_anti_steal_token: false # 如果有防盗链的话，这里true
    http_secret_key: # 有防盗链，这里填secret_key
```

### 使用方式

```java
 // 使用fastDFSClientUtil提供的方法上传、下载、删除
    @Resource
    FastDFSClientUtil fastDFSClientUtil;
```

# 二、测试一下

```java
@Controller
@RequestMapping("fastdfs")
public class FastdfsController {

    @Resource
    private FastDFSClientUtil fastDFSClientUtil;

    @PostMapping("/upload")
    @ResponseBody
    public AjaxResponse upload(@RequestParam("file") MultipartFile file) {

        String fileId;
        try {
            String originalfileName = file.getOriginalFilename();
            fileId = fastDFSClientUtil.uploadFile(file.getBytes(),originalfileName.substring(originalfileName.lastIndexOf(".")));
            return AjaxResponse.success(fastDFSClientUtil.getSourceUrl(fileId));
        } catch (Exception e) {
            throw new CustomException(CustomExceptionType.SYSTEM_ERROR,"文件上传图片服务器失败");
        }
    }


    @DeleteMapping("/delete")
    @ResponseBody
    public AjaxResponse upload(@RequestParam String fileid) {
        try {
            fastDFSClientUtil.delete(fileid);
        } catch (Exception e) {
            throw new CustomException(CustomExceptionType.SYSTEM_ERROR,"文件删除失败");
        }
       return  AjaxResponse.success();
    }


}
```

postman文件上传配置:

![img](https://box.kancloud.cn/4278bfc49f2090f24c7d5de59ed7a461_1465x534.png)