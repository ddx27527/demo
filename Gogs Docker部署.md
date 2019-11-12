# 	Gogs docker部署

Gogs 是 一个能够简单自建Git托管服务的开源项目，用 go 语言实现。

[Gogs 官方地址](https://link.jianshu.com/?t=https://gogs.io/)

## MySQL 、Gogs镜像的下载

```sh
# 下载镜像
$ docker pull mysql:5.7
$ docker pull gogs/gogs
$ docker pull nginx

# 查看镜像
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
nginx               latest              8daf737d1a4a        10 days ago         125.9 MB
mysql               5.7                 b0ce4a10ad68        11 days ago         373.3 MB
gogs/gogs           latest              83ce75a0822d        2 weeks ago         100.6 MB


```

## 启动MySQL，设置root密码为 ksgj72IvxIz6 

```sh
# 启动MySQL，设置root密码为 ksgj72IvxIz6 
mkdir -p /var/docker/mysql

docker run -d \
--name gogs-mysql \
--restart=always \
-p 13306:3306 \
-v /var/docker/mysql:/var/lib/mysql \
--privileged \
-e MYSQL_ROOT_PASSWORD='ksgj72IvxIz6' \
mysql:5.7

```



#### gogs数据库的初始化脚本需要提前准备好，[github官方mysql初始化脚](https://github.com/gogs/gogs/blob/master/scripts/mysql.sql)



```sh
# 内容如下：
SET GLOBAL innodb_file_per_table = ON,
           innodb_file_format = Barracuda,
           innodb_large_prefix = ON;
DROP DATABASE IF EXISTS gogs;
CREATE DATABASE IF NOT EXISTS gogs CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

# 将该脚本保存为mysql.sql 导入到容器的/tmp目录下下
$ docker cp mysql.sql mysql:/tmp
# 进入容器
$ docker exec -it mysql bash
# 登录容器，导入数据脚本
$ mysql --default-character-set=utf8 -uroot -pksgj72IvxIz6 -hlocalhost -f < /tmp/mysql.sql;

```



## 启动gogs容器

```sh
# 启动gogs容器
mkdir -p /var/docker/gogs

docker run -d \
--name gogs \
-p 10022:22 \
-p 10080:3000 \
--privileged \
-v /var/docker/gogs:/data \
gogs/gogs 


```

​	打开`http://IP:10080`就会重定向到`http://IP:10080/install`页面进行初始化安装

简要说明，不使用Docker安装Gogs，访问`http://IP:3000`是HTTP页面的地址，容器绑定了外部主机的10080端口到容器内部的3000，所以访问的是10080端口，同理，使用SSH进行git操作使用的是10022端口

如果已经安装了，可以删除/var/gogs/目录下的文件，刷新一下页面可以重新初始化

先不用安装，继续进行下面的操作



## 安装/启动nginx

```sh
# 启动nginx服务
docker run -d --name=gogs-nginx -p 80:80 nginx:latest


```





## 申请SSL证书

​	在使用`acme.sh`申请Let's Encrypt证书之前，首先需要解析域名，比如使用`https://git.wangkx.top`而不是`http://IP:10080`浏览项目

`acme.sh`很方便，可以自动根据Nginx配置文件来验证域名所有者，生成证书，所以申请证书之前，需要保证`http://git.wangkx.top`是可以访问服务器的

在`/etc/nginx/conf.d/`目录下创建`git.wangkx.top.conf`配置文件，内容如下：

```sh
server {
    listen       80;
    server_name  git.wangkx.top;
    root         /usr/share/nginx/html;

    # Load configuration files for the default server block.

    location / {
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

#### 重新加载Nginx配置

```sh

nginx -t
nginx -s reload

# 访问http://git.wangkx.top可以看到Nginx安装后的默认页面
```



#### 申请证书

```sh
curl https://get.acme.sh | sh

# 重新 重新 重新 ！！！ 登入系统 执行
acme.sh --issue --nginx -d git.wangkx.top

```

输出如下：

```sh
# 执行如下命令：

[root@host ~]# acme.sh --issue --nginx -d git.wangkx.top
[Fri Nov  2 02:56:56 EDT 2018] Registering account
[Fri Nov  2 02:56:57 EDT 2018] Registered
[Fri Nov  2 02:56:57 EDT 2018] ACCOUNT_THUMBPRINT='d4rTbIKhIebxLviyioVfI6EfU6PxzC-JoLFbLi54V_M'
[Fri Nov  2 02:56:57 EDT 2018] Creating domain key
[Fri Nov  2 02:56:57 EDT 2018] The domain key is here: /root/.acme.sh/git.wangkx.top/git.wangkx.top.key
[Fri Nov  2 02:56:57 EDT 2018] Single domain='git.wangkx.top'
[Fri Nov  2 02:56:57 EDT 2018] Getting domain auth token for each domain
[Fri Nov  2 02:56:57 EDT 2018] Getting webroot for domain='git.wangkx.top'
[Fri Nov  2 02:56:57 EDT 2018] Getting new-authz for domain='git.wangkx.top'
[Fri Nov  2 02:56:57 EDT 2018] The new-authz request is ok.
[Fri Nov  2 02:56:57 EDT 2018] Verifying:git.wangkx.top
[Fri Nov  2 02:56:57 EDT 2018] Nginx mode for domain:git.wangkx.top
[Fri Nov  2 02:56:58 EDT 2018] Found conf file: /etc/nginx/conf.d/git.wangkx.top.conf
[Fri Nov  2 02:56:58 EDT 2018] Backup /etc/nginx/conf.d/git.wangkx.top.conf to /root/.acme.sh/git.wangkx.top/backup/git.wangkx.top.nginx.conf
[Fri Nov  2 02:56:58 EDT 2018] Check the nginx conf before setting up.
[Fri Nov  2 02:56:58 EDT 2018] OK, Set up nginx config file
[Fri Nov  2 02:56:58 EDT 2018] nginx conf is done, let's check it again.
[Fri Nov  2 02:56:58 EDT 2018] Reload nginx
[Fri Nov  2 02:57:02 EDT 2018] Success
[Fri Nov  2 02:57:02 EDT 2018] Restoring from /root/.acme.sh/git.wangkx.top/backup/git.wangkx.top.nginx.conf to /etc/nginx/conf.d/git.wangkx.top.conf
[Fri Nov  2 02:57:02 EDT 2018] Reload nginx
[Fri Nov  2 02:57:02 EDT 2018] Verify finished, start to sign.
[Fri Nov  2 02:57:03 EDT 2018] Cert success.
-----BEGIN CERTIFICATE-----
MIIGBzCCBO+gAwIBAgISAz86RbZ+IGrr3ragVzKLwPvAMA0GCSqGSIb3DQEBCwUA
MEoxCzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MSMwIQYDVQQD
ExpMZXQncyBFbmNyeXB0IEF1dGhvcml0eSBYMzAeFw0xODExMDIwNTU3MDNaFw0x
OTAxMzEwNTU3MDNaMBkxFzAVBgNVBAMTDmdpdC53YW5na3gudG9wMIIBIjANBgkq
hkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAr557TlwyvjQHr2UmaAdsemr+2NnJzzz4
e5DaxTt1/p4Cx09MdoLnqr9W0JRV9RtbiMymJp4tf/BQvgFqsOUG0SZbOku2qcJA
lB4qDnrl1fi7qxc+zl2xMOvmWAHjKQT4nlSIN93njSyBeVvhtEuMSo36WySdUBQh
1WN/YrDQyrF3kN15Kw52IDRPHDVMbMm51HD476q8etv44SOP0vXw8tTkG4L6s7+8
RueOjlX9ni/5mJFrx2uMRb1zJ1VrxuyHPqisomV4UFAayONatqMH2/s94WVJ6L8b
+3qQyiU9j7MfaTuf9X9wPnizqUFqnpopNiShHdtFPKJ+6Q+K+Iq3TQIDAQABo4ID
FjCCAxIwDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEF
BQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBR0qq3e0/seomvuqOYKHfD/0+iG
8jAfBgNVHSMEGDAWgBSoSmpjBH3duubRObemRWXv86jsoTBvBggrBgEFBQcBAQRj
MGEwLgYIKwYBBQUHMAGGImh0dHA6Ly9vY3NwLmludC14My5sZXRzZW5jcnlwdC5v
cmcwLwYIKwYBBQUHMAKGI2h0dHA6Ly9jZXJ0LmludC14My5sZXRzZW5jcnlwdC5v
cmcvMBkGA1UdEQQSMBCCDmdpdC53YW5na3gudG9wMIH+BgNVHSAEgfYwgfMwCAYG
Z4EMAQIBMIHmBgsrBgEEAYLfEwEBATCB1jAmBggrBgEFBQcCARYaaHR0cDovL2Nw
cy5sZXRzZW5jcnlwdC5vcmcwgasGCCsGAQUFBwICMIGeDIGbVGhpcyBDZXJ0aWZp
Y2F0ZSBtYXkgb25seSBiZSByZWxpZWQgdXBvbiBieSBSZWx5aW5nIFBhcnRpZXMg
YW5kIG9ubHkgaW4gYWNjb3JkYW5jZSB3aXRoIHRoZSBDZXJ0aWZpY2F0ZSBQb2xp
Y3kgZm91bmQgYXQgaHR0cHM6Ly9sZXRzZW5jcnlwdC5vcmcvcmVwb3NpdG9yeS8w
ggEEBgorBgEEAdZ5AgQCBIH1BIHyAPAAdwDiaUuuJujpQAnohhu2O4PUPuf+dIj7
pI8okwGd3fHb/gAAAWbTNf7IAAAEAwBIMEYCIQC9QSs0GvI5OU2Cig1TPHrscmbz
0vlDr2A6mu7aaPUP9AIhAM4kuobcfKlSCoXNKLzYSogTGqeSYqk8MNHiuTWDAum9
AHUAKTxRllTIOWW6qlD8WAfUt2+/WHopctykwwz05UVH9HgAAAFm0zX+3QAABAMA
RjBEAiBMmI9ygmh3to5RdplcvnLSAIdOFolAwiTO7+8aLpAcoQIgaYjVISMS9xlJ
1eaw69Kwn1T+Vsl+/SkR0lUI0grbidcwDQYJKoZIhvcNAQELBQADggEBAAjSHKG9
Y3l9tJjF6kKQau0zJHHXdO7PDkLP4/AjMIyBLA/lWZQGBN+71ONFfFAcA6NwTtQi
6Ir6Tm5GG3OP8amKo0gVGBbL2b0nRliYm/tSaEVhKqNDOjAlVyVkch6Bd2H38eUo
e+Zfv2+97ZKXDFMMe4UFWkF5JBV8+ElSWGK3wwXDHrcE/g9PFgoHs0aod3R303Xp
RE6Cf9VCHUOYHX8U7poZ0R7GeWNIglu2VbYBmqZ+WxpemAT1Q4bTfit80FY9NWGh
uY2J1/lkiswPBI26g30HrZ+B4BfhP/jIu8vdDBtTzynx0KVI4pEy8zPm+CrRJwLu
Tdb/khyqH2s4Tog=
-----END CERTIFICATE-----
[Fri Nov  2 02:57:03 EDT 2018] Your cert is in  /root/.acme.sh/git.wangkx.top/git.wangkx.top.cer 
[Fri Nov  2 02:57:03 EDT 2018] Your cert key is in  /root/.acme.sh/git.wangkx.top/git.wangkx.top.key 
[Fri Nov  2 02:57:03 EDT 2018] The intermediate CA cert is in  /root/.acme.sh/git.wangkx.top/ca.cer 
[Fri Nov  2 02:57:03 EDT 2018] And the full chain certs is there:  /root/.acme.sh/git.wangkx.top/fullchain.cer 
[root@host ~]# 
```

没有报错，生成成功

```sh
# 找到如下内容，后面会用到：

Your cert is in  /root/.acme.sh/git.wangkx.top/git.wangkx.top.cer 

Your cert key is in  /root/.acme.sh/git.wangkx.top/git.wangkx.top.key 
```



#### Nginx跳转Gogs

修改之前添加的Nginx配置文件（/etc/nginx/conf.d/git.wangkx.top.conf），添加如下内容并删除原有内容:

```sh
server {
    listen 443 ssl http2;
    server_name git.wangkx.top;
    # 上面生成的cer、key信息
    ssl_certificate /root/.acme.sh/git.wangkx.top/git.wangkx.top.cer;
    ssl_certificate_key /root/.acme.sh/git.wangkx.top/git.wangkx.top.key;
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    ssl_protocols TLSv1.2;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
        proxy_set_header  X-Real-IP  $remote_addr;
        # 如果gogs容器映射的端口变了，记得一定要同步修改此处的端口
        proxy_pass http://127.0.0.1:10080$request_uri; 
    }
}

# 以下部分表示重定向 HTTP 请求到 HTTPS
server {
    listen 80;
    server_name git.wangkx.top; 
    return 301 https://$host$request_uri;
}

# 重新载入Nginx配置文件
nginx -t 
nginx -s reload

# 再次访问 
https://git.wangkx.top/ 可以看到默认的Nginx页面已经有了小绿锁头

```



## 设置Gogs

​	之前安装启动Gogs容器后并没有进行设置，现在配置好了域名，可以进行安装了

数据库我选择的SQLite3，默认设置

应用基本设置，运行系统用户git默认就好，docker封装的gogs里已经创建了git用户。需要注意的有`域名`，`SSH端口号`，`应用URL`

域名我填写的是`git.wangkx.top`，

SSH端口号填写的是`10022`，

应用URL填写的`https://git.wangkx.top/`

HTTP端口号，Gogs安装默认使用的3000，因为使用docker，所以外部访问使用的是10080端口，又因为使用Nginx代理，所以直接访问`https://git.wangkx.top/`就可以了



```sh
# 最终gogs/conf/app.in下的参数如下
[root@host conf]# cat app.ini 
APP_NAME = Gogs
RUN_USER = git
RUN_MODE = prod

[database]
DB_TYPE  = sqlite3
HOST     = 127.0.0.1:3306
NAME     = gogs
USER     = root
PASSWD   = 
SSL_MODE = disable
PATH     = data/gogs.db

[repository]
ROOT = /data/git/gogs-repositories

[server]
DOMAIN           = git.wangkx.top 
HTTP_PORT        = 3000 # gogs容器本身端口号
ROOT_URL         = https://git.wangkx.top/ # 域名
DISABLE_SSH      = false
SSH_PORT         = 10022  # ssh端口号
START_SSH_SERVER = false
OFFLINE_MODE     = false

[mailer]
ENABLED = false

[service]
REGISTER_EMAIL_CONFIRM = false
ENABLE_NOTIFY_MAIL     = false
DISABLE_REGISTRATION   = true
ENABLE_CAPTCHA         = false
REQUIRE_SIGNIN_VIEW    = false

[picture]
DISABLE_GRAVATAR        = false
ENABLE_FEDERATED_AVATAR = false

[session]
PROVIDER = file

[log]
MODE      = file
LEVEL     = Info
ROOT_PATH = /app/gogs/log

[security]
INSTALL_LOCK = true
SECRET_KEY   = wkcZBEqWYZmkTIc


#################################################################
MIIGBzCCBO+gAwIBAgISAz86RbZ+IGrr3ragVzKLwPvAMA0GCSqGSIb3DQEBCwUA
MEoxCzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MSMwIQYDVQQD
ExpMZXQncyBFbmNyeXB0IEF1dGhvcml0eSBYMzAeFw0xODExMDIwNTU3MDNaFw0x
OTAxMzEwNTU3MDNaMBkxFzAVBgNVBAMTDmdpdC53YW5na3gudG9wMIIBIjANBgkq
hkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAr557TlwyvjQHr2UmaAdsemr+2NnJzzz4
e5DaxTt1/p4Cx09MdoLnqr9W0JRV9RtbiMymJp4tf/BQvgFqsOUG0SZbOku2qcJA
lB4qDnrl1fi7qxc+zl2xMOvmWAHjKQT4nlSIN93njSyBeVvhtEuMSo36WySdUBQh
1WN/YrDQyrF3kN15Kw52IDRPHDVMbMm51HD476q8etv44SOP0vXw8tTkG4L6s7+8
RueOjlX9ni/5mJFrx2uMRb1zJ1VrxuyHPqisomV4UFAayONatqMH2/s94WVJ6L8b
+3qQyiU9j7MfaTuf9X9wPnizqUFqnpopNiShHdtFPKJ+6Q+K+Iq3TQIDAQABo4ID
FjCCAxIwDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEF
BQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBR0qq3e0/seomvuqOYKHfD/0+iG
8jAfBgNVHSMEGDAWgBSoSmpjBH3duubRObemRWXv86jsoTBvBggrBgEFBQcBAQRj
MGEwLgYIKwYBBQUHMAGGImh0dHA6Ly9vY3NwLmludC14My5sZXRzZW5jcnlwdC5v
cmcwLwYIKwYBBQUHMAKGI2h0dHA6Ly9jZXJ0LmludC14My5sZXRzZW5jcnlwdC5v
cmcvMBkGA1UdEQQSMBCCDmdpdC53YW5na3gudG9wMIH+BgNVHSAEgfYwgfMwCAYG
Z4EMAQIBMIHmBgsrBgEEAYLfEwEBATCB1jAmBggrBgEFBQcCARYaaHR0cDovL2Nw
cy5sZXRzZW5jcnlwdC5vcmcwgasGCCsGAQUFBwICMIGeDIGbVGhpcyBDZXJ0aWZp
Y2F0ZSBtYXkgb25seSBiZSByZWxpZWQgdXBvbiBieSBSZWx5aW5nIFBhcnRpZXMg
YW5kIG9ubHkgaW4gYWNjb3JkYW5jZSB3aXRoIHRoZSBDZXJ0aWZpY2F0ZSBQb2xp
Y3kgZm91bmQgYXQgaHR0cHM6Ly9sZXRzZW5jcnlwdC5vcmcvcmVwb3NpdG9yeS8w
ggEEBgorBgEEAdZ5AgQCBIH1BIHyAPAAdwDiaUuuJujpQAnohhu2O4PUPuf+dIj7
pI8okwGd3fHb/gAAAWbTNf7IAAAEAwBIMEYCIQC9QSs0GvI5OU2Cig1TPHrscmbz
0vlDr2A6mu7aaPUP9AIhAM4kuobcfKlSCoXNKLzYSogTGqeSYqk8MNHiuTWDAum9
AHUAKTxRllTIOWW6qlD8WAfUt2+/WHopctykwwz05UVH9HgAAAFm0zX+3QAABAMA
RjBEAiBMmI9ygmh3to5RdplcvnLSAIdOFolAwiTO7+8aLpAcoQIgaYjVISMS9xlJ
1eaw69Kwn1T+Vsl+/SkR0lUI0grbidcwDQYJKoZIhvcNAQELBQADggEBAAjSHKG9
Y3l9tJjF6kKQau0zJHHXdO7PDkLP4/AjMIyBLA/lWZQGBN+71ONFfFAcA6NwTtQi
6Ir6Tm5GG3OP8amKo0gVGBbL2b0nRliYm/tSaEVhKqNDOjAlVyVkch6Bd2H38eUo
e+Zfv2+97ZKXDFMMe4UFWkF5JBV8+ElSWGK3wwXDHrcE/g9PFgoHs0aod3R303Xp
RE6Cf9VCHUOYHX8U7poZ0R7GeWNIglu2VbYBmqZ+WxpemAT1Q4bTfit80FY9NWGh
uY2J1/lkiswPBI26g30HrZ+B4BfhP/jIu8vdDBtTzynx0KVI4pEy8zPm+CrRJwLu
Tdb/khyqH2s4Tog=

#################################################################
```



安装完成，新建一个`test-pub`工程

HTTP clone链接：https://git.wangkx.top/yieldone/test-pub.git

SSH clone链接： ssh://git@git.wangkx.top:10022/yieldone/test-pub.git

遗留问题：现在使用HTTPS方式Clone会报错:

```sh
[root@host ~]# git clone https://git.wangkx.top/yieldone/test-pub.git
Cloning into 'test-pub'...
fatal: unable to access 'https://git.wangkx.top/yieldone/test-pub.git/': Peer's Certificate issuer is not recognized.
```

这个问题，好像是git不认这个证书，clone之前运行命令可以解决这个问题

```sh
git config --global http.sslVerify false
# 生成证书
ssh-keygen -t rsa -b 4096 -C "185959236@qq.com"

git config --global user.name "network"
git config --global user.email "185959236@qq.com"
```

将公钥使用控制面板用户设置的SSH密钥导入

然后使用SSH进行clone测试

```
[root@host ~]# git clone ssh://git@git.wangkx.top:10022/yieldone/test-pub.git
Cloning into 'test-pub'...
remote: Counting objects: 8, done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 8 (delta 1), reused 0 (delta 0)
Receiving objects: 100% (8/8), done.
Resolving deltas: 100% (1/1), done.
```

说明：

1，文档中在Docker内不建议使用独立的SSH服务器

2，可以使用`docker exec -it gogs /bin/sh`命令进入容器，用户git在`/home/git`，git用户的authorized_keys文件不是标准的文件，查看`/home/git/.ssh/authorized_keys`文件，会发现是一串命令，在面板添加的ssh公钥就会生成一条这样的记录

3，主机上，docker内的gogs映射在`/var/gogs`目录下，gogs下的app.ini配置文件，可以进去进行修改，然后使用`docker restart gogs`重启

4，没事别闲着关闭iptables啥的，会导致端口映射失败，比如ssh clone 访问超时，端口显示不通

参考：

[[How to config SSH settings](https://discuss.gogs.io/t/how-to-config-ssh-settings/34)]

[使用 Let’s encrypt 证书，实现自动续签。](https://juejin.im/entry/5a0d58f8f265da431769b303)