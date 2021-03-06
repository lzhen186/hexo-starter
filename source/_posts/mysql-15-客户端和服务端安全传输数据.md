---
title: MySQL 15.客户端和服务端安全传输数据
date: 2022-07-01T12:19:06.807Z
tags: [mysql]
---
- [1. 为什么需要安全连接](#1-为什么需要安全连接)
- [2. 客户端安全连接](#2-客户端安全连接)
  - [**(1) 没有强制使用ssl安全连接情况**](#1-没有强制使用ssl安全连接情况)
  - [**(2) 强制所有用户使用ssl安全连接**](#2-强制所有用户使用ssl安全连接)
  - [**(3) 未使用SS和使用SSL安全性对比**](#3-未使用ss和使用ssl安全性对比)
  - [**(4) 使用SSL前后性能对比**](#4-使用ssl前后性能对比)

# 1. 为什么需要安全连接

SSL(Secure Socket Layer) ：安全套接字层，利用数据加密、身份验证和消息完整性验证机制，为基于TCP等可靠连接的应用层协议提供安全性保证，SSL协议提供的功能主要有：

- 数据传输的机密性：利用对称密钥算法对传输的数据进行加密。
- 身份验证机制：基于证书利用数字签名方法对服务器和客户端进行身份验证，其中客户端的身份验证是可选的。
- 消息完整性验证：消息传输过程中使用MAC算法来检验消息的完整性。

数据传输如果不是通过SSL的方式，那么其在网络中数据都是以明文进行传输的，如果传输的是敏感信息(账号、密码)，通过抓包就可以获取到敏感信息。所以在数据库方面，如果客户端连接服务器获取数据需要使用SSL连接，那么在传输过程中，经过加密的数据很难被窃取。

# 2. 客户端安全连接

从mysql5.7之后的版本，在其data目录下有很多后缀为.pem类型的文件，这些.pem文件是用来SSL加密连接的，也就是说mysql5.7之后默认是开启了ssl安全连接功能。

```bash
# 查看是否开启了SSL/TLS安全连接功能
SHOW VARIABLES LIKE '%ssl%';

# 如果未开启，使用命令mysql_ssl_rsa_setup生成*.pem文件，默认存放在目录/var/lib/mysql下
```

从mysql安装目录/var/lib/mysql下复制出来3个客户端安全连接时需要的文件：**client-cert.pem、client-key.pem、ca.pem**。



## **(1) 没有强制使用ssl安全连接情况**

默认情况下，mysql配置文件my.cnf的[mysqld]下require_secure_transport = OFF，说明不强制要求ssl连接，但mysql客户端连接时添加选项–ssl-mode也可以使用ssl安全连接，–ssl-mode选项值有以下5种类型：

- DISABLED：不使用安全连接，如果require_secure_transport = ON时则无效。
- PREFERRED, REQUIRED：使用SSL加密，加密算法没差别。
- VERIFY_CA, VERIFY_IDENTITY：客户端连接时需附加–ssl-ca等选项。

设置指定用户必须使用ssl传输数据示例：

```bash
# 使用x509连接
mysql -h 192.168.101.88 -u zhuyasen -P 33060 --ssl-cert=./client-cert.pem --ssl-key=./client-key.pem --ssl-mode=REQUIRED -p

# 使用x509连接和mysql服务颁发证书ca.pem连接
mysql -h 192.168.101.88 -u zhuyasen -P 33060 --ssl-mode=VERIFY_CA --ssl-cert=./client-cert.pem --ssl-key=./client-key.pem --ssl-ca=./ca.pem -p
```



在require_secure_transport = OFF(不需要ssl连接)，可以设置指定用户必须使用ssl传输数据。

```bash
# 创建新用户指定该用户使用连接方式：ssl或x509
CREATE USER 'krislin'@'%' IDENTIFIED WITH mysql_native_password BY '123456' REQUIRE ssl;
# 授权数据库和表的权限
GRANT ALL ON *.* TO 'krislin'@'%';
# 刷新
FLUSH PRIVILEGES;

# 对于已存在用户也可以是修改连接方式：ssl或x509
ALTER USER 'zhuyasen'@'%' REQUIRE x509;

# 查看用户表信息
select host,user,ssl_type,ssl_cipher,x509_issuer,x509_subject,plugin from mysql.user;

# 查看当前用户密文状态
show status like 'ssl_cipher';

# 如果用户不使用证书连接，返回连接失败(非本机连接)
mysql -h 192.168.101.88 -u vison -p

# 指定证书连接，连接成功
mysql -h 192.168.101.88 -u vison --ssl-cert=./client-cert.pem --ssl-key=./client-key.pem -p

# 查看上一条命令状态详情
\s
```



## **(2) 强制所有用户使用ssl安全连接**

在配置文件my.cnf的[mysqld]下添加参数，强制所有用户传输使用ssl安全传输，

> require_secure_transport = ON

重启mysql后，远程登陆必须使用证书

```bash
mysql -h 192.168.101.88 -u root -P 33060 --ssl-cert=./client-cert.pem --ssl-key=./client-key.pem -p

mysql -h 192.168.101.88 -u vison -P 33060 --ssl-cert=./client-cert.pem --ssl-key=./client-key.pem --ssl-ca=./ca.pem --ssl-mode=VERIFY_CA -p
```



## **(3) 未使用SS和使用SSL安全性对比**

- 未使用SSL情况下，在数据库服务器端可以通过抓包的方式获取数据，安全性不高。
- 没有抓到该语句，采用SSL加密后，tshark抓不到数据，安全性高。



## **(4) 使用SSL前后性能对比**

- 虽然SSL方式使得安全性提高了，但是相对地使得QPS也降低23%左右，使用连接池或者长连接可能会好许多。
- 对于非常敏感核心的数据，或者QPS本来就不高的核心数据，可以采用SSL方式保障数据安全性；
- 对于采用短链接、要求高性能的应用，或者不产生核心敏感数据的应用，性能和可用性才是首要，建议不要采用SSL方式，除非使用云服务商数据强制要求除外。



**注：如果用户是采用本地localhost或者sock连接数据库，不会使用SSL方式的。**