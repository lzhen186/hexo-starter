---
title: SpringBoot 03.spring boot配置原理实战 3.9.配置文件敏感字段加密
date: 2022-07-03T01:07:50.841Z
tags: [springboot]
---
# 一、`Jasypt`是什么

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200421145055.png)
[官网](http://www.jasypt.org/)

> [Jasypt](http://jasypt.org/)是一个Java库，允许开发人员以很简单的方式添加基本加密功能，而无需深入研究加密原理。利用它可以实现高安全性的，基于标准的加密技术，无论是单向和双向加密。加密密码，文本，数字，二进制文件。

1. 高安全性的，基于标准的加密技术，无论是单向和双向加密。加密密码，文本，数字，二进制文件…
2. 集成Hibernate的。
3. 可集成到Spring应用程序中，与Spring Security集成。
4. 集成的能力，用于加密的应用程序（即数据源）的配置。
5. 特定功能的高性能加密的multi-processor/multi-core系统。
6. 与任何JCE（Java Cryptography Extension）提供者使用开放的API

# 二、使用bat脚本生成加密串和盐值(密钥)

为了方便，简单编写了一个bat脚本方便使用。

```bat
@echo off
set/p input=待加密的明文字符串：
set/p password=加密密钥(盐值)：
echo 加密中......
java -cp jasypt-1.9.2.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI  ^
input=%input% password=%password% ^
algorithm=PBEWithMD5AndDES
pause
```

- 使用 `jasypt-1.9.2.jar`中的`org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI`类进行加密
- input参数是待加密的字符串，password参数是加密的密钥(盐值)
- 使用PBEWithMD5AndDES算法进行加密

**注意：`jasypt-1.9.2.jar` 文件需要和bat脚本放在相同目录下。

使用示例：
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200421145514.png)

**注意：相同的盐值(密钥)，每次加密的结果是不同的。**

# 三、`Jasypt`与spring boot整合

首先引入`Jasypt`的maven坐标

```xml
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>1.18</version>
</dependency>
```

在`properties`或`yml`文件中需要对明文进行加密的地方的地方，使用ENC()包裹，如原值："happy family"，加密后使用`ENC(密文)`替换。

为了方便测试，在`properties`或`yml`文件中，做如下配置

```yaml
# 设置盐值（加密解密密钥），我们配置在这里只是为了测试方便
# 生产环境中，切记不要这样直接进行设置，可通过环境变量、命令行等形式进行设置。下文会讲
jasypt:
  encryptor:
    password: 123456
```

简单来说，就是在需要加密的值使用`ENC(`和`)`进行包裹，即：`ENC(密文)`。之后像往常一样使用`@Value("${}")`获取该配置即可，获取的是解密之后的值。

# 四、如何存储盐值(密钥)更安全

> 本身加解密过程都是通过`盐值`进行处理的，所以正常情况下`盐值`和`加密串`是分开存储的。**`盐值`应该放在`系统属性`、`命令行`或是`环境变量`来使用，而不是放在同一个配置文件里面。**

## 4.1 命令行存储方式示例

```
java -jar xxx.jar --jasypt.encryptor.password=xxx &;
```

## 4.2 环境变量存储方式示例

设置环境变量(`linux`)：

```
# 打开/etc/profile文件
vim /etc/profile
# 文件末尾插入
export JASYPT_PASSWORD = xxxx
```

启动命令：

```
java -jar xxx.jar --jasypt.encryptor.password=${JASYPT_PASSWORD} &;
```

# 五、这样真的安全么？

**有的同学会问这样的问题：如果的`linux`主机被攻陷了怎么办，黑客不就知道了密钥？**

对于这个问题：我只能这么说，如果你的应用从内部被攻陷，在这个世界上没有一种加密方法是绝对安全的。这种加密方法只能做到：防君子不防小人。大家可能都听说过，某著名互联网公司将明文数据库密码上传到了`github`上面，导致用户信息被泄露的问题。这种加密方式，无非是将密钥与加密结果分开存放，减少个人疏忽导致的意外，增加破解难度。

如果密钥被从内部渗透暴露了，任何加密都是不安全的。就像你的组织内部有离心离德的人，无论你如何加密都不安全，你需要做的是把他找出来干掉，或者防范他加入你的组织！