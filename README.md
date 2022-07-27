# Learning-about-https
This is a learning about the use of the certificate generation tool certmaker

这是一个关于证书生成软件certmaker的使用学习

## Requierment - 必要条件
* certmaker
* 提前准备一个本地后端服务

## Running - 运行

### 1、签发根证书
```
./certmaker -gen root -cn "Test Root Ca"
```
生成根证书私钥rootCA.key和证书文件rootCA.pem

### 2、中间CA证书
```
./certmaker -gen intermediate -cn "TestIntermediateCA" -issuer rootCA.pem -issuerkey rootCA.key
```
生成中间CA证书私钥TestIntermediateCA.key和证书文件TestIntermediateCA.pem

### 3、签发域名证书
```
./certmaker -gen cert -cn "*.example.com" -issuer TestIntermediateCA.pem -issuerkey TestIntermediateCA.key
```
生成域名证书私钥_.example.com.key和证书文件_.example.com.pem

### 4、修改nginx配置
```
upstream hello_proxy_ssl {
    server 10.227.89.58:8443;
}
server {
    listen 443;
    proxy_connect_timeout 1s;
    proxy_timeout 300s;
    proxy_pass hello_proxy_ssl;
}

server {
    listen  880;
    listen  8443 ssl;
    server_name  www.example.com;
    # 引用中间CA证书
    ssl_certificate      /etc/nginx/ssl/TestIntermediateCA.pem;
    ssl_certificate_key  /etc/nginx/ssl/TestIntermediateCA.key;
    # 引用域名证书
    ssl_certificate      /etc/nginx/ssl/_.example.com.pem;
    ssl_certificate_key  /etc/nginx/ssl/_.example.com.key;
    # 自动跳转到HTTPS
    if ($server_port = 880) {
        rewrite ^(.*)$ https://$host$1 permanent;
    }
    ...
}
```
### 配置完成，接下来访问本地后端 www.example.com 就能看见https和小锁头啦！