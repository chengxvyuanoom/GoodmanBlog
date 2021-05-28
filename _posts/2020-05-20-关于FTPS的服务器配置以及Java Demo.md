---
layout: post
title:  "关于FTPS的服务器配置以及Java Demo"
date:   2020-05-20 12:03:12
tags: Elasticsearch Java Config
date: 2020-05-20 12:51:12
categories: FTP LINUX

---

* TOC
{:toc}
#### 关于FTPS的服务器配置以及Java Demo

#### 一、FTP服务器开启SSL认证(不携带证书)

相关配置如下:

```properties
## FTP证书的位置(需要CA来签发)
rsa_cert_file=/etc/vsftpd/ssl/vsftpd.crt
## FTP私钥的位置
rsa_private_key_file=/etc/vsftpd/ssl/vsftpd.key
## 客户端访问是否需要携带证书(该参数不校验证书内容，携带任意证书即可)
require_cert=NO
## 是否对客户端携带的证书进行校验
validate_cert=NO
## 是否开启SSL
ssl_enable=YES
ssl_sslv2=YES
ssl_sslv3=YES
## 是否开启TLS
ssl_tlsv1=YES
## 是否开启可重用SSL通道(经测试，该参数在主动模式并且带有SSL认证下，需要关闭)
require_ssl_reuse=NO
```

关于 **rsa_cert_file** 和 **rsa_private_key_file** 参数内容的生成(如何用自建CA给vsftpd服务器签发证书)

首先确认服务器有openssl，如果没有，需要手动安装

openssl目录详解: 

```
/etc/pki/CA
certs——存放已颁发的证书

newcerts——存放CA指令生成的新证书

private——存放私钥

crl——存放已吊销的证书

index.txt——OpenSSL定义的已签发证书的文本数据库文件，这个文件通常在初始化的时候是空的

serial——证书签发时使用的序列号参考文件，该文件的序列号是以16进制格式进行存放的，该文件必须提供并且包含一个有效的序列号
```

番外篇-如何生成自建CA 

签发ca证书和签发客户端证书过程大致类似，但是ca证书是自签名的
客户端证书是使用ca证书签名的

 配置文件可以修改证书的匹配规则，以及默认CA的路径(这个路径可以自建，也可以用默认,最好用默认)

> vim /etc/pki/tls/openssl.cnf

```
1.生成 ca私钥
openssl genrsa -out private/cakey.pem 2048
2. 生成ca 请求证书，会输入一些参数 (有效期3650天)
openssl req -new -days 3650 -key private/cakey.pem -out careq.pem
3. 对ca 证书进行自签名 根证书自签名
openssl ca -selfsign -in careq.pem -out cacert.pem
```

 **rsa_cert_file** 和 **rsa_private_key_file** (为服务器端签售证书)

```
1.生成服务器端私钥(key文件);
openssl genrsa -des3 -out server.key 1024
2.生成服务器端证书签名请求文件(csr文件),按照步骤一步步填写和自建证书时候填写的保持一致;
openssl req -new -key server.key -out server.
3.利用CA证书签售
openssl ca -in ../server.csr -out ../server.crt -cert ca.crt -keyfile ca.key 
```

#### 二、FTP服务器开启SSL认证(携带证书但不校验)

```properties
## 其他参数保持不变
require_cert=YES
```

随便生成个证书即可

#### 三、FTP服务器开启SSL认证(携带证书并且校验)

```properties
## 其他参数保持不变
require_cert=YES
## 是否对客户端携带的证书进行校验
validate_cert=YES
## 客户端信任列表
ca_certs_file=/etc/pki/CA/cacert.pem
```

  需要用第一步的CA签售一个客户端证书   **-certfile /etc/pki/CA/cacert.pem**

  步骤如下：

```
openssl genrsa  -out client_private.pem 2048
openssl rsa -in client_private.pem -pubout -out client_public_key.pem
openssl req -new -days 3650 -key client_private.pem -out client_req.pem
openssl ca -in client_req.pem -out client_public_key.pem -days 3650
openssl pkcs12 -export -in client_public_key.pem -inkey client_private.pem -out client.p12 -certfile /etc/pki/CA/cacert.pem
```

#### 四、证书常用命令

```
1. keytool（jdk生成p12）
keytool -storepass 123456 -storetype PKCS12 -keystore file.p12 -genkey -alias client -keyalg RSA
2. keytool 根据p12 生成 jks
keytool -importkeystore -deststorepass yourJKSpass -destkeypass yourKeyPass -destkeystore MyDSKeyStore.jks -srckeystore fullchain_and_key.p12 -srcstoretype PKCS12 -srcstorepass yourPKCS12pass -alias client
```

#### 五、总结和备注

。。 

