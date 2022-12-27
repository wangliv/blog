---
title: openssl生成https证书
---
> 环境准备：
Linux 已安装了openssl 

###  openssl配置文件修改

``` bash
cp /etc/ssl/openssl.cnf  ./
```
在 v3_req最后增加 subjectAltName = @alt_names，同时后面增加配置。这样生成的证书有SAN (Subject Alternative Name) ，否则在chrome会提示证书无效。
[alt_names]   #可配置多个可选的名称
DNS.1 = www.wangli.com

``` bash
[ v3_req ]

# Extensions to add to a certificate request

basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = www.wangli.com
```

### 生成证书

``` bash
openssl req -sha256 -newkey rsa:2048 -nodes -keyout ssl.key -x509 -days 3650 -out ssl.crt -config ./openssl.cnf -extensions v3_req
```
命令最后指定了，使用当前目录下的openssl.cnf配置文件，同时使用v3版本的证书 ( -extensions v3_req )

执行完上面的命令会提示输入证书信息，注意：Common Name填写你的域名。

### Nginx配置证书

``` bash
server {
    listen 443 ssl;
    server_name www.wangli.com;
    ssl_certificate  /root/prod/ssl.crt;
    ssl_certificate_key /root/prod/ssl.key;
    location / {
        proxy_pass  http://0.0.0.0:80;
     }
}
```
启动服务后即可使用https访问。