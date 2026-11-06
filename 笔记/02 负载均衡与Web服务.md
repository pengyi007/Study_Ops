 #LVS #Keepalived  #Nginx #Tomcat 

# 第一章 Nginx

## 一、概述


## 二、部署

### 1 yum部署方式
```powershell
# 配置好yum源后，使用yum命令下载nginx
[root@localhost ~]# yum install -y nginx
```
### 2 编译安装
官网地址：https://nginx.org/en/
下载地址：https://nginx.org/en/download.html
![nginx官网](https://img2024.cnblogs.com/blog/2716860/202402/2716860-20240222162318597-609688536.png)
![nginx官网](https://img2024.cnblogs.com/blog/2716860/202402/2716860-20240222162328741-1261699445.png)
```powershell
# 1、上传nginx压缩包至linux服务器
[root@localhost ~]# mkdir /mnt/images
[root@localhost ~]# cd /mnt/images/
[root@localhost images]# ll
total 1088
-rw-r--r-- 1 root root 1112471 Feb 22 15:37 nginx-1.24.0.tar.gz
# 2、解压压缩包
[root@localhost images]# tar xf nginx-1.24.0.tar.gz  
[root@localhost images]# ll
total 1088
drwxr-xr-x 8 1001 1001     158 Apr 11  2023 nginx-1.24.0
-rw-r--r-- 1 root root 1112471 Feb 22 15:37 nginx-1.24.0.tar.gz
# 3、下载相关依赖
[root@localhost images]# yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
# 4、生成Makefile编译
[root@localhost images]# cd nginx-1.24.0/
## 执行configure脚本，设置安装nginx的初始化配置（--with-http_ssl_module：启动 SSL 的支持），生成 Makefile 可编译文件
[root@localhost nginx-1.24.0]# ./configure --with-http_ssl_module
[root@localhost nginx-1.24.0]# ll
total 816
drwxr-xr-x 6 1001 1001    326 Feb 22 15:45 auto
-rw-r--r-- 1 1001 1001 323312 Apr 11  2023 CHANGES
-rw-r--r-- 1 1001 1001 494234 Apr 11  2023 CHANGES.ru
drwxr-xr-x 2 1001 1001    168 Feb 22 15:45 conf
-rwxr-xr-x 1 1001 1001   2611 Apr 11  2023 configure
drwxr-xr-x 4 1001 1001     72 Feb 22 15:45 contrib
drwxr-xr-x 2 1001 1001     40 Feb 22 15:45 html
-rw-r--r-- 1 1001 1001   1397 Apr 11  2023 LICENSE
-rw-r--r-- 1 root root    438 Feb 22 15:57 Makefile
drwxr-xr-x 2 1001 1001     21 Feb 22 15:45 man
drwxr-xr-x 3 root root    125 Feb 22 15:57 objs
-rw-r--r-- 1 1001 1001     49 Apr 11  2023 README
drwxr-xr-x 9 1001 1001     91 Feb 22 15:45 src
# 5、预编译及编译安装
[root@localhost nginx-1.24.0]# make
[root@localhost nginx-1.24.0]# make install
# 6、启动nginx
[root@localhost nginx-1.24.0]# cd  /usr/local/nginx/sbin
[root@localhost sbin]# ./nginx 
# 7、验证确认
## 若有ss命令
[root@localhost sbin]# ss -ntl
State       Recv-Q Send-Q                                              Local Address:Port                                                             Peer Address:Port              
LISTEN      0      128                                                             *:80                                                                          *:*      
## 无ss命令
[root@localhost sbin]# ps -ef |grep nginx
root       4864      1  0 16:05 ?        00:00:00 nginx: master process ./nginx
nobody     4865   4864  0 16:05 ?        00:00:00 nginx: worker process
root       5157   1995  0 16:10 pts/1    00:00:00 grep --color=auto nginx
[root@localhost sbin]# netstat -nlpt|grep 4864
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      4864/nginx: master  
# 8、访问验证
修改hosts文件：C:\Windows\System32\drivers\etc
(第一次无权限操作的话，请选择hosts文件右键单击属性->安全，对角色添加写入权限)
## linux系统查询本机ip，将本机ip加入到windows的hosts中去。
[root@localhost sbin]# hostname -I
10.0.0.200 
# 使用浏览器访问linux系统ip就可以访问了
```

![nginx初次访问](https://img2024.cnblogs.com/blog/2716860/202402/2716860-20240222162219882-247986113.png)

```
# 执行configure脚本扩展
--prefix=PATH：指定 nginx 的安装目录（默认/usr/local/nginx）
--conf-path=PATH：指定 nginx.conf 配置文件路径
--user=NAME：nginx 工作进程的用户
--with-pcre：开启 PCRE 正则表达式的支持
--with-http_ssl_module：启动 SSL 的支持
--with-http_stub_status_module：用于监控 Nginx 的状态
--with-http-realip_module：允许改变客户端请求头中客户端 IP 地址
--with-file-aio：启用 File AIO
--add-module=PATH：添加第三方外部模块
该内容参考：https://blog.csdn.net/weixin_42585386/article/details/122215776
```

## 三、模块
### 1、优先访问该网站

+ **default_server

```powershell
[root@web01 /etc/nginx/conf.d]# cat default.conf 
#default.conf
server {
 listen 80 default_server;
 server_name _;
 charset utf8;
 default_type text/plain;
 return 200 "别乱访问网站";
}
```
### 2、日志模块

+  access.log访问日志 
+  error.log 错误日志 

```powershell
#定义访问日志格式
 log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                 '$status $body_bytes_sent "$http_referer" '
                 '"$http_user_agent" "$http_x_forwarded_for"';
```

| nginx内置变量 | (nginx访问日志的格式) 说明 |
| --- | --- |
| $remote_addr | 客户端ip地址 |
| $remote_user | 用户名(开启ngx认证模式的用户名) 大部分为空 |
| $time_local | 时间 |
| $request | 请求报文的起始行 GET / HTTP/1.1 请求方法 请求的uri http版本 |
| $status | 状态码 |
| $body_bytes_sent | 服务端给用户响应的大小(文件,页面,资源大小) |
| $http_referer | 用户从哪里跳转访问网站的 (如果直接访问一般空) |
| $http_user_agent | 用户的客户端(浏览器) |
| $http_x_forwarded_for | 使用代理(或负载均衡)用于记录用户真实的ip地址 |
#### （1）访问日志
| access_log选项 | |
| --- | --- |
| buffer | 缓冲用户访问日志. |
| gzip | 日志是否要压缩, 对于旧的日志 |
| flush | 多久把日志写入到磁盘中. |
```powershell
## access_log off; #关闭访问日志.
[root@web01 /etc/nginx/conf.d]# head *.conf 
==> default.conf <==
#default.conf
server {
 listen 80 default_server;
 server_name _;
 charset utf8;
 default_type text/plain;
 return 200 "别乱访问网站";
 access_log /var/log/nginx/default.log main;
}
```
#### （2）错误日志

```powershell
#nginx 错误日志级别 
error #服务报错才会记录. 
debug #超级详细的日志
#error_log /var/log/nginx/error.log error;
[root@web01 /etc/nginx/conf.d]# head *.conf
==> default.conf <==
#default.conf
server {
 listen 80 default_server;
 server_name _;
 charset utf8;
 default_type text/plain;
 return 200 "别乱访问网站";
 access_log /var/log/nginx/default-access.log main;
 error_log /var/log/nginx/default-error.log error;
 }
```
#### （3）日志切割

+  防止单个日志过大. 
+  方案01:系统定时任务(每天) + logroate 
+  方法02:自己书写脚本,mv 改名,重启服务, (sersync) 

```shell
#日志切割 /var/log/messages /var/log/secure /var/log/cron 
[root@web01 /var/log/nginx]# cat /etc/logrotate.d/syslog 
/var/log/cron
/var/log/maillog
/var/log/messages
/var/log/secure
/var/log/spooler
{
 missingok
 sharedscripts
 postrotate
 /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true 
#重启系统日志服务 rsyslog 
 endscript
}
#日志切割 /var/log/nginx/xxxx.log 
[root@web01 /var/log/nginx]# cat /etc/logrotate.d/nginx 
/var/log/nginx/*.log {
 create 0640 nginx root
 daily
 rotate 10
 missingok
 notifempty
 compress
 delaycompress
 sharedscripts
 postrotate
 /bin/kill -USR1 `cat /run/nginx.pid 2>/dev/null` 2>/dev/null || true
 endscript
}
#logroate 命令 对应的配置文件 (日志切割的配置文件)
[root@web01 /var/log/nginx]# cat /etc/logrotate.d/nginx 
/var/log/nginx/*.log {  #指定要切割哪些文件
 create 0640 nginx root #切割后的文件所有者权限信息 
 daily 									#每天切割 
 rotate 10 							#保留10次 切割的日志 
 missingok 							#如果日志不存在 不切割(跳过)
 notifempty 						#not if empty 如果日志是空的跳过不切割
 compress 							#切割后的日志要进行压缩
 delaycompress 					#延迟压缩 延迟1次 
 sharedscripts 					#开启切割后或切割前 执行脚本/命令功能功能
 postrotate 						#post切割之后执行什么命令.
 /bin/kill -USR1 `cat /run/nginx.pid 2>/dev/null` 2>/dev/null || true
 endscript
}
#根据对应的切割配置 执行切割
logroate -f /etc/logrotate.d/nginx
```
### 3、网站列表功能

+  autoindex（下载功能） 
+  应用场景：实现用户访问的时候显示网站的目录及内容 

| autoindex |                            |
| --------- | -------------------------- |
| yntax:    | 格式 autoindex on            |
| Default:  | 默认 autoindex off;          |
| Context:  | 放在哪 http, server, location |
### 4、限制模块

+ 访问增加些限制： 只让某些用户访问（白名单），禁止一些用户访问（黑名单）

| ngx-access模块 |          |                                      |      |
| ------------ | -------- | ------------------------------------ | ---- |
| allow        | Syntax:  | allow ip                             | all; |
|              | Default: | —                                    |      |
|              | Context: | http, server, location, limit_except |      |
| deny         |          |                                      |      |
|              | Syntax:  | deny ipaddr                          | all; |
|              | Default: | —                                    |      |
|              | Context: | http, server, location, limit_except |      |

```powershell
#白名单-指定的ip或网段可以访问,默认所有人拒绝访问
 allow 172.16.1.0/24;
 allow 10.0.0.1;
 deny all;
#黑名单 默认所有人都可以访问
限制或封掉某些ip使用.
deny 10.0.0.7;
```
### 5、认证模块

+ 用户访问网站或指定页面/路径的需要输入用户名和密码

| 模块 | 格式 | |
| --- | --- | --- |
| auth_basic | "closed site"; #指定 提示信息 | |
| | Syntax: | auth_basic string |
| | Default: | auth_basic off; |
| | Context: | http, server, location, limit_except |
| auth_basic_user_file | conf/htpasswd; #指定用户名和密码文件 密码要加密 通过htpasswd | |
| | Syntax: | auth_basic_user_file file; 指定密码文件 |
| | Default: | — |
| | Context: | http, server, location, limit_except |
### 6、状态模块

+ 应用场景:用于我们内部统计,收集,监控ngx状态使用

|          |                         |
| -------- | ----------------------- |
| Syntax:  | 开启状态模块 stub_status;     |
| Default: | —                       |
| Context: | 可以放在哪里 server, location |
```powershell
#sngx7个状态指标
Active connections(活跃连接) #当前与nginx已经建立的连接 (nginx 并发) 实时
server 											#accepts nginx 已经接受了多少个用户的连接(总数)
handled 										#nginx已经处理了多少个连接(总数)
requests 										#用户向ngx发出的请求数量(总数)
Reading(读取请求) 		 			 #ngx正在读取多少个用户的请求头信息(实时)
Writing(给予响应) 		 			 #ngx正在给多少个用户发送响应信息(实时)
Waiting 									  #正在等待ngx处理的请求(实时)
```
### 7、location格式
| | |
| --- | --- |
| Syntax: | location [ = | ~ | ~* | ^~ ]uri { ... } location @name  { ... } |
| Default: | — |
| Context: | server, location |
```powershell
#第1种格式 使用率频率最高格式 ※※※※※
location 符号 uri {xxxxxxx} #用于匹配uri 
#第2个种格式
location @name {} #一般用于nginx内部跳转 定义了一个标记
```
#### 7.1、匹配规则（符号）
| location规则 | |
| --- | --- |
| location / {xxxx} | 默认规则，如果其他location规则匹配失败，则使用location / 里面的设置 |
| location /svip/ {xxxx} | 匹配指定的uri , 如果用户的uri是 /svip/ /svip/oldboy.html 符号这个规则 |
| location ~ .php$ | 支持正则，来匹配uri,区分大小写 |
| location ~* .avi$ {} | 支持正则，来匹配uri,不区分大小写 |
| location = /404.html {} | 精确匹配，优先级最高 |
| location ^~ /image/lidao.png {} | 不支持正则，也有较高的优先级 |
#### 7.2、规则优先级
| location优先级 | |
| --- | --- |
| 1 最高 | location = uri {} |
| 2 | location ^~ uri {} |
| 3 | locatiopn ~ uri {} 或 location ~* uri {} |
| 4 | location /uri {} #匹配对应路径 |
| 5 | location / {} |
#### 7.3、应用场景
| | |
| --- | --- |
| location / {} | location默认规则 |
| location /uri {} | 匹配指定的uri路径 location /admin {a} |
| location ~ uri {} | 正则，区分大小写， .php$ .jsp$ 匹配语言 |
| location ~* uri {} | 正则，不区分大小写 ， 通用，通过正则匹配，图片，静态资源。。。 |
| locastion = uri {} | 精确匹配 location = /404.html {} 一般不单独使用 还要有配合。 |
### 8、模块总结
| 模块 | |
| --- | --- |
| 核心模块core模块 | root index server_name listen location http server ...... error_log |
| log模块 日志模块 | access_log log_format ... |
| autoindex模块 列表功能模块 | autoindex |
| access模块 访问控制模块 | allow deny |
| auth_basic模块 认证模块 | auth_basic auth_basic_user_file |
| stub_status模块 状态模块 | stub_statu |
## 四、详解

### 1、虚拟主机部分

#### 1.1、基于域名的虚拟主机
```nginx
[root@web01 /etc/nginx/conf.d]# cat game.oldboylinux.cn.conf 
server {
 listen 80;
 server_name game.oldboylinux.cn;  #每个域名不同
 root /app/code/game;
 location / {
 index index.html; 
 } 
}
[root@web01 /etc/nginx/conf.d]# cat live.oldboylinux.cn.conf 
server {
 listen 80;
 server_name live.oldboylinux.cn;
 root /app/code/live;
 location / {
 index index.html; 
 } 
}
```
#### 1.2、基于端口的虚拟主机
```nginx
[root@web01 /etc/nginx/conf.d]# head *.conf
==> game.oldboylinux.cn.conf <==
server {
 listen 82;   #端口不同
 server_name game.oldboylinux.cn;
 root /app/code/game;
 location / {
 index index.html; 
 } 
}
==> live.oldboylinux.cn.conf <==
server {
 listen 81;
 server_name live.oldboylinux.cn;
 root /app/code/live;
 location / {
 index index.html; 
 } 
}
```
#### 1.3、基于IP的虚拟主机
```nginx
[root@web01 /etc/nginx/conf.d]# head *.conf 
server {
 listen 172.16.1.7:8888;  #端口不同
 server_name game.oldboylinux.cn;
 root /app/code/game;
 location / {
 index index.html; 
 } 
}

server {
 listen 172.16.1.8:8888;
 server_name live.oldboylinux.cn;
 root /app/code/live;
 location / {
 index index.html; 
 } 
}
```
### 2、书写配置文件
```nginx
[root@web01 /etc/nginx/conf.d]# cat liferestart.linux.conf 
server {
    listen 80;
    server_name liferestart.linux.cn;
    root /app/scripts/liferestart;
    access_log /app/logs/liferestart.linux.cn-access.log main;
    error_log /app/logs/liferestart.linux.cn-error.log error;
  location / {
 index index.html; 
 }
}

[root@web01 /etc/nginx/conf.d]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
### 3、日志模块范本
```nginx
service {
 access_log /var/log/nginx/game.oldboylinux.cn-access.log main;
 error_log /var/log/nginx/game.oldboylinux.cn-error.log error;
}
```
### 4、网站列表模块范本
```nginx
service {
 access_log /var/log/nginx/dl.oldboylinux.cn-access.log main ;
 error_log /var/log/nginx/dl.oldboylinux.cn-error.log error;
 autoindex on;
}
```
### 5、限制模块范本
```nginx
 server {
 access_log /var/log/nginx/dl.oldboylinux.cn-access.log main ;
 error_log /var/log/nginx/dl.oldboylinux.cn-error.log error;
 charset utf8;   #中文不乱码
 autoindex on;
 autoindex_localtime on;
 allow 172.16.1.0/24;
 allow 10.0.0.1;
 deny all;
}
```
### 6、认证模块范本
```nginx
location / {
 index index.html; 
 }
 location /zz-svip { #用户访问 域名/zz-svip才会 提示输入
用户名密码
 auth_basic "pls input password";
 auth_basic_user_file /etc/nginx/key/pass;
 } 
}

[root@web01 ~]# htpasswd -bc /etc/nginx/key/pass oldboy 2
Adding password for user oldboy
[root@web01 ~]# cat /etc/nginx/key/pass 
oldboy:$apr1$MeHCXY6N$ElynJ/FNQGYW5QkdHBmTX/
chmod 600 /etc/nginx/key/pass
```
### 7、状态模块范本
```nginx
 location /status {
 stub_status; #开启nginx状态显示功能
 allow 10.0.0.1;
 allow 172.16.1.0/24;
 deny all;
 }
}
```
### 8、location规则范文
```nginx
#敏感路径，限制用户访问 /admin ※※※※※※
location /admin {
 allow 10.0.0.0/24;
 allow 172.16.1.0/24; 
 deny all;
}
#用户的请求是动态请求 则交给php或java ※※※※※※
location ~ \.(php|jsp)$ {
 proxy_pass ......;
}
#用户请求的图片，css,js,html静态资源 进行缓存（用户浏览器） ※※※※※
location ~* .(png|jpg|jpep|bmp|gif|css|js|html)$ {
 expires max; #10年 不经常变化资源就可以缓存
}
#指定错误及错误页面 熟悉
###5xx错误 
 error_page 500 502 503 504 /50x.html;
 location = /50x.html {
 root html;
 }
```
### 9、密码鉴权
```nginx
auth_basic "Please input password";      ##输入用户名密码提示框
auth_basic_user_file /etc/nginx/conf.d/auth_pwd;  ##配置用户名密码验证文件路径

在server /http /location 都可以
```

# 第二章 LVS


# 第三章 keeplived

## 一、概述

+ HA (High Available) 高可用 (一直都可用)、
+ Keepalived (keep alived 保持活着) - 开源高可用软件
+ HeartBeat (heart beat 心脏跳动)
### 1、keepalived 原理

+ 2台机器做主备高可用环境
+ 主 (MASTER) 备(BACKUP)
+ keepalived 基于 VRRP协议(虚拟路由冗余协议:最开始给企业级路由器使用,)
+ 原理01: 主节点不断向备发送消息(我还活呢),备节点接收数据. 能够收到数据认为主活着.
+ 原理02: 备节点一但收不到数据包,则认为主挂了,成为主.,接管对应的资源
### 2、脑裂

+ 原因: 互相认为对方挂了 现象主备节点都有VIP(虚拟ip)
+ 检查: 
    - 书写脚本在被节点,检查是否有vip
+ 解决: 
    - 防火墙问题
    - 配置不当
    - 物理服务器-网线问题
    - 只要发生脑裂,就让主真的挂了,(智能断电...)
## 二、部署

### 1、环境准备

```powershell
#web01 web02 
[root@web02 /etc/nginx/conf.d]# cat game.oldboylinux.cn.conf 
server {
 listen 80;
 server_name game.oldboylinux.cn;
 root /app/code/game/type/;
 access_log /var/log/nginx/game.oldboylinux.cn-access.log main;
 error_log /var/log/nginx/game.oldboylinux.cn-error.log error;
 location / {
 index index.html; 
 } 
}
curl -H Host:game.oldboylinux.cn 10.0.0.8

#lb01 lb02 
[root@lb01 ~]# cat /etc/nginx/conf.d/game.oldboylinux.cn.conf
upstream default_pools {
 server 10.0.0.8:80;
}
upstream static_pools {
 server 10.0.0.7:80;
}
server {
 listen 80;
 server_name game.oldboylinux.cn;
 location / {
 proxy_pass http://default_pools;
 include proxy_params;
 }
 location ~* \.(jpg|jpeg|bmp|gif|png|ogg)$ {
 proxy_pass http://static_pools;
 include proxy_params;
 }
}
```

### 2、keepalived

```powershell
#lb01 lb02
yum install -y keepalived
```

+ keepalived 三大部分 
    - global_defs全局部分
    - vrrp_instance 配置vip
    - keepalived管理lvs部分(暂时不用)

```shell
! Configuration File for keepalived
# global 全局定义部分
global_defs {
 router_id LB01 #这个id是每个keepalived软件的标记 这个id与其他的keepalived不用相同
 }
 #vrrp 实例部分 
#vrrp实例名字 要主备一致. 
vrrp_instance LB {
 state MASTER #主 MASTER 备 BACKUP
 interface eth0 #指定VIP绑定的网卡
 virtual_router_id 51 #虚拟路由id号 同一对主备之间 id要一致。
 priority 100 #优先级 主比备多50
 advert_int 1 #心跳间隔 主多久向备发送1次消息
 authentication { #设置主备之间验证 
 auth_type PASS #使用密码验证
 auth_pass 1111 #指定密码 主备见密钥一致 不推荐修改太复杂
 }
 virtual_ipaddress { #vip 
 10.0.0.3/24 dev eth0 label eth0:0 #指定vip 10.0.0.3
 }
}
```
#### 2.1、keepalived与lb

+ keepalived与负载均衡结合
+ 域名解析到 vip 上面即可
## 三、应用

### 1、 Keepalived监控ngx

+ nginx负载挂了.那么keepalived也随之关闭.
+ 如何衡量(检查)ngx是否挂了
+ 如何让ngx挂了,keepalived也随之关闭
#### 1.1、如何衡量(检查)ngx是否挂了

```powershell
进程/端口
curl/wget 获取ngx页面状态 (负载均衡)
##检查端口
ss -lntup |grep nginx
[root@lb01 ~]# ss -lntup |grep nginx |wc -l 
2
[root@lb01 ~]# pkill nginx 
[root@lb01 ~]# ss -lntup |grep nginx |wc -l 
0
##检查进程
[root@lb01 ~]# ps -ef |grep nginx |grep -v grep |wc -l 
0
[root@lb01 ~]# ps -ef |grep nginx |grep -v grep |wc -l 
0
[root@lb01 ~]# 
[root@lb01 ~]# 
[root@lb01 ~]# ps -ef |grep '[n]ginx'
[root@lb01 ~]# ps -ef |grep '[n]ginx' |wc -l 
0
[root@lb01 ~]# # 进程名字是 grep '[n]ginx'
[root@lb01 ~]# # 进程运行的时候 grep 'nginx'
[root@lb01 ~]# # 进程名字不同,grep 没有把自己过滤出来
#curl /wget
```
#### 1.2、如何让ngx挂了,keepalived也随之关闭

```shell
#ngx状态检查脚本
#如果ngx挂了(端口或进程数为0) 则关闭keepalived 
#!/bin/b ash
# author: lidao996
# desc: chk ngx && stop keepalived 
#if [ ngx端口数 等于 0 ];then 
# 关闭keepalived 
#fi
# -eq equal 
port_count=`ss -lntup |grep nginx |wc -l`
if [ $port_count -eq 0 ];then
 systemctl stop keepalived
fi
```
#### 1.3、keepalived调用ngx监控脚本

```powershell
vrrp_script #指定一个脚本,keepalived 会定时运行脚本 注意:脚本加上执行权限 +x 
#vrrp 实例上面 
 vrrp_script chk_lb { #给脚本起个命令(keepalived使用)
    script /server/scripts/chk_lb.sh # 指定脚本路径
    interval 1 # 执行脚本的间隔 默认1秒
    timeout 30 # 脚本执行的超时时间 一般用于curl/wget等操作
    weight 1 # 权重(优先级) 一般用于keepalived中有多个监控脚本
 }

#vrrp实例里面
 track_script { #写在vrrp 实例部分 表示调用之前定义的监控脚本
    chk_lb #名字 是上面vrrp_script 的名字
        #<SCRIPT_NAME> weight <-254..254> #权重
 }
 
 
 vrrp_script chk_lb { 
    script /server/scripts/chk_lb.sh 
    interval 1 
    timeout 30 
    weight 1 
}
track_script { 
   chk_lb 
}
```
### 2、双主模式

+ 双主模式 : keepalived2个实例 互为主备(这里实例类似于rsync多个模块)
+ 应用场景: 不同的域名访问不同的负载,减轻负载压力.(默认都解析域访问到同一台负载上面)
### 3、非抢占式

+  抢占式: keepalived主备模式中,主节点故障,备节点接管. 主节点恢复主解决接管资源. 
+  非抢占式: 2个节点平等(BACKUP), 一个节点故障则发展切换,切换后一直接管资源(及时故障节点恢复) 
+  使用流程 
    - 主备---> 备备 (BACKUP)
    - 加上非抢占式标记 nopreempt

```powershell
[root@lb01 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
 router_id LB01
}
vrrp_instance LB {
  state BACKUP
   nopreempt
 interface eth0
 virtual_router_id 51
 priority 100
 advert_int 1
 authentication {
  auth_type PASS
  auth_pass 1111
 }
 virtual_ipaddress {
 10.0.0.3/24 dev eth0 label eth0:0
 }
}
```

# 第四章 Tomcat

## 一、介绍
### 1、Tomcat目录结构
| 目录      |                                  |
| ------- | -------------------------------- |
| bin     | 存放tomcat管理脚本                     |
| conf    | 配置文件目录                           |
| logs    | 日志目录                             |
| webapps | 站点目录                             |
| lib     | tomcat依赖,jar结尾                   |
| temp    | 临时目录                             |
| work    | tomcat运行代码的编译临时目录 (tomcat运行生成产物) |
#### (1)、bin目录
| bin/        |                                               |
| ----------- | --------------------------------------------- |
| startup.sh  | 启动tomcat                                      |
| shutdown.sh | 关闭tomcat                                      |
| catalina.sh | tomcat核心脚本(开,关tomcat都会调用) 进行优化jvm开启tomcat监控功能 |
#### (2)、conf目录
| conf 配置文件目录        |                        |
| ------------------ | ---------------------- |
| server.xml         | 类似于nginx.conf 主配置文件    |
| web.xml            | 配置tomcat内置功能           |
| logging.properties | 日志格式 ( properties 属性)  |
| tomcat-users.xml   | tomcat管理端配置文件(线上环境中关闭) |
#### (3)、logs目录
| logs | |
| --- | --- |
| catalina.out | tomcat主要日志,运行状态,启动,关闭,(每天被切割后,大小不变.,该文件会持续增大.) |
| catalina.2021-11-26.log | 与上面类似,切割日志.,tomcat每天对catalina.out进行切割 |
| localhost_access_log.2021-11-26.txt | tomcat访问日志 |
| host-manager.2021-11-26.log | |
| localhost.2021-11-26.log | |
| manager.2021-11-26.log | |
### 2、server.xml
#### (1)、主配置文件

```powershell
# tomcat shutdown 端口 连接这个端口 输入暗号 tomcat关闭.
<Server port="8005" shutdown="SHUTDOWN">


# 用户请求 进入到 connector 中 连机器中. 
<Connector port="8080" protocol="HTTP/1.1"
 			   connectionTimeout="20000"
 			   redirectPort="8443" />

# 交给engine部分处理 指定默认的 host(默认的虚拟主机)
<Engine name="Catalina" defaultHost="localhost"> #defaultHost未来是 虚拟主机的域名 (server_name) 

# host tomcat虚拟主机部分的配置
 Host 部分 name === nginx server_name 域名
 
 <Host name="localhost" appBase="webapps"
 	   unpackWARs="true" autoDeploy="true">
 <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
	    prefix="localhost_access" suffix=".log"
 	  	pattern="%h %l %u %t &quot;%r&quot; %s %b" />
 		#directory 访问日志存放目录
 		#prefix 日志前缀
		#suffix 日志后缀
 
 localhost_access_log.2021-11-26.txt
 前缀 后缀
 #pattern 访问日志格式
 
 </Host>
```
#### (2)、tomcat和nginx配置对比
| Tomcat虚拟主机配置 | tomcat                   | ngx         |
| ------------ | ------------------------ | ----------- |
| 域名           | Host部分 name=""           | server_name |
| 站点目录         | Host部分 appBase           | root        |
|              | unpackWARs 是否自动解压war包.   |             |
|              | autoDeploy 是否自动部署(加载jvm中 |             |
#### (3)、日志格式
| Tomcat访问日志格式      | tomcat              | nginx                 |
| ----------------- | ------------------- | --------------------- |
| 客户端ip地址           | %h                  | $remote_addr          |
| 用户名               | %l                  |                       |
| 用户名               | %u                  |                       |
| 日期和时间             | %t                  | $local_time           |
| 用户请求的起始行(请求方法和uri |                     | %r                    |
| 状态码               | %s                  | $status               |
| 响应的大小(服务端--->客户端) | %b                  | $body_bytes_sent      |
| 用户从哪里跳转来的         | %{Referer}i         | $http_referer         |
| 用户的客户端            | %{User-Agent}i      | $http_user_agent      |
| 用户的真实ip地址         | %{X-Forwarded-For}i | $http_x_forwarded_for |
```powershell
%{User-Agent}i #incoming 进入 获取用户请求头中的内容 直接写请求头的名字即可. 
%h %l %u %t &quot;%r&quot; %s %b
```

```powershell
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8005" shutdown="SHUTDOWN">
 <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
 <Listener className="org.apache.catalina.core.AprLifecycleListener"
SSLEngine="on" />
 <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener"
/>
 <Listener 
className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
 <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener"
/>
 <GlobalNamingResources>
  <Resource name="UserDatabase" auth="Container"
 		   type="org.apache.catalina.UserDatabase"
  		   description="User database that can be updated and saved"
 		   factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
 		   pathname="conf/tomcat-users.xml" />
</GlobalNamingResources>
<Service name="Catalina">
  <Connector port="8080" protocol="HTTP/1.1"
             connectionTimeout="20000"
             redirectPort="8443" />
  <Engine name="Catalina" defaultHost="localhost">
    <Realm className="org.apache.catalina.realm.LockOutRealm">
     <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
            resourceName="UserDatabase"/>
     </Realm>
     <Host name="localhost" appBase="webapps"
           unpackWARs="true" autoDeploy="true">
       <Valve className="org.apache.catalina.valves.AccessLogValve"
directory="logs"
              prefix="localhost_access_log" suffix=".txt"
              pattern="%h %l %u %t &quot;%r&quot; %s %b &quot;%{Referer}i&quot; &quot;%{User-Agent}i&quot; %{X-Forwarded-For}i " />
       </Host>
    </Engine>
  </Service>
</Server>
```
#### (4)、虚拟主机
| 虚拟主机   域名 | 站点目录 | 访问日志 |
| --- | --- | --- |
| locathost | webapps | access.时间.log |
| live.penglinux.cn | live | access.时间.log |
| www.penglinux.cn | www | access.时间.log |

```powershell
server.xml 中engine 部分配置

 	<Engine name="Catalina" defaultHost="localhost">
 	  <Realm className="org.apache.catalina.realm.LockOutRealm">
	    <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
			   resourceName="UserDatabase"/>
	  </Realm>
	  <Host name="localhost" appBase="webapps"
 	 	 	unpackWARs="true" autoDeploy="true">
	    <Valve className="org.apache.catalina.valves.AccessLogValve"
directory="logs"
 			    prefix="access" suffix=".log"
 			    pattern="%h %l %u %t &quot;%r&quot; %s %b &quot;%{Referer}i&quot; &quot;%{User-Agent}i&quot; %{X-Forwarded-For}i " />
	 </Host>
 	 <Host name="live.penglinux.cn" appBase="live"
		   unpackWARs="true" autoDeploy="true">
 		<Valve className="org.apache.catalina.valves.AccessLogValve"
directory="logs"
 			   prefix="access" suffix=".log"
			   pattern="%h %l %u %t &quot;%r&quot; %s %b &quot;%{Referer}i&quot; &quot;%{User-Agent}i&quot; %{X-Forwarded-For}i " />
	 </Host>
	 <Host name="www.penglinux.cn" appBase="www"
 		   unpackWARs="true" autoDeploy="true">
 		<Valve className="org.apache.catalina.valves.AccessLogValve"
directory="logs"
 			   prefix="access" suffix=".log"
 			   pattern="%h %l %u %t &quot;%r&quot; %s %b &quot;%{Referer}i&quot; &quot;%{User-Agent}i&quot; %{X-Forwarded-For}i " />
	 </Host>
 </Engine>

[root@web03 ~]#  mkdir -p /app/tools/tomcat/{live,www}/ROOT
[root@web03 ~]#  echo live > /app/tools/tomcat/live/ROOT/index.jsp
[root@web03 ~]#  cho www > /app/tools/tomcat/www/img/index.jsp

#用户访问 tomcat的时候 url
www.penglinux.cn:8080/index.jsp ---> 站点目录下面的/ROOT查找/index.jsp
www.penglinux.cn:8080/img/index.jsp ---> 站点目录下面的/img/index.jsp
```
#### (5)、动态网站

1. 配置文件及虚拟主机 (域名与站点目录)
2. 准备环境
3. 部署与调试应用

```powershell
http://www.penglinux.cn:8080/zrlog/install
#tomcat 环境准备
 	<Host name="www.penglinux.cn" appBase="zrlog"
		  unpackWARs="true" autoDeploy="true">
	  <Valve className="org.apache.catalina.valves.AccessLogValve"
directory="logs"
			 prefix="access" suffix=".log"
			 pattern="%h %l %u %t &quot;%r&quot; %s %b &quot;%{Referer}i&quot; &quot;%{User-Agent}i&quot; %{X-Forwarded-For}i " />
 </Host>

[root@web03 ~]# mkdir /app/tools/tomcat/zrlog
[root@web03 ~]# mv /server/tools/zrlog-2.2.1-efbe9f9-release.war /server/tools/zrlog.war

#tomcat部署代码
cp /server/tools/zrlog.war /app/tools/tomcat/zrlog/
```
### 3、Tomcat 处理用户请求流程

1. 用户发出请求
2. 请求到达tomcat connector 部分处理http请求
3. 交给engine部分(配置默认的虚拟主机) 的host部分处理 (虚拟主机)
4. 处理完成返回给用户
## 二、部署

### 1、jdk

```powershell
#jdk 二进制包 yum则 yum install java(1.8) java-11 (jdk)
[root@web03 ~]# tar xf jdk-8u301-linux-x64.tar.gz -C /app/tools/
[root@web03 ~]# ln -s /app/tools/jdk1.8.0_301/ /app/tools/jd
##配置java,tomcat环境变量 (yum安装不用配置)
cat >>/etc/profile<<'EOF'
export JAVA_HOME=/app/tools/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export
CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
export TOMCAT_HOME=/app/tools/tomcat
EOF
[root@web03 ~]# source /etc/profile
[root@web03 ~]# java -version 
java version "1.8.0_301"
Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)


#说明
cat >>/etc/profile<<'EOF'
export JAVA_HOME=/app/jdk 
 #jdk的安装路径 
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH 
 #配置jdk环境变量
export
CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar 
#java 类文件路径,类似于依赖
export TOMCAT_HOME=/app/tomcat 
 #tomcat家目录 可以不配置
EOF
```
### 2、tomcat

```powershell
#解压到/app/tools/tomcat 目录
[root@web03 ~]# ln -s /app/tools/apache-tomcat-9.0.52/ /app/tools/tomcat
[root@web03 ~]# /app/tools/tomcat/bin/version.sh 
Using CATALINA_BASE: /app/tools/tomcat
Using CATALINA_HOME: /app/tools/tomcat
Using CATALINA_TMPDIR: /app/tools/tomcat/temp
Using JRE_HOME: /app/tools/jdk
Using CLASSPATH: 
/app/tools/tomcat/bin/bootstrap.jar:/app/tools/tomcat/bin/tomcat-juli.jar
Using CATALINA_OPTS: 
Server version: Apache Tomcat/9.0.52
Server built: Jul 31 2021 04:12:17 UTC
Server number: 9.0.52.0
OS Name: Linux
OS Version: 3.10.0-1160.el7.x86_64
Architecture: amd64
JVM Version: 1.8.0_301-b09
JVM Vendor: Oracle Corporation
```
### 3、使用

```powershell
#启动 tomcat 
 /app/tools/tomcat/bin/startup.sh
#关闭 tomcat 
 /app/tools/tomcat/bin/shutdown.sh
###检查端口(注意是java) 
 [root@web03 ~]# ss -lntup |grep java
tcp LISTEN 0 100 [::]:8080 [::]:* 
users:(("java",pid=2960,fd=57))
tcp LISTEN 0 1 [::ffff:127.0.0.1]:8005 [::]:* 
 users:(("java",pid=2960,fd=65))
###检查进程(注意java)
[root@web03 ~]# ps -ef |grep java 
root 2960 1 14 12:08 pts/0 00:00:09 /app/tools/jdk/bin/java -
Djava.util.logging.config.file=/app/tools/tomcat/c
onf/logging.properties -
Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -
Djdk.tls.ephemeralDHKeySize=2048 -
Djava.protocol.handler.pkgs=org.apache.catalina.webresources -
Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs=
-classpath /app/tools/tomcat/bin/bootstrap.jar:/app/tools/tomcat/bin/tomcatjuli.jar -Dcatalina.base=/app/tools/tomcat -Dcatalina.home=/app/tools/tomcat -
Djava.io.tmpdir=/app/tools/tomcat/temp org.apache.catalina.startup.Bootstrap start
root 2993 2604 0 12:09 pts/0 00:00:00 grep --color=auto java
###浏览器访问
```
## 三、基础使用

+ tomcat部署(运行)app
+ tomcat日志查看
+ tomcat systemctl 管理脚本
### 1、部署

+ 方式01：war包(压缩包)放到站点目录下面
+ 方法02：jar包 java -jar xxx.jar 运行 java app
### 2、查看日志

+ info表示通知或提示 正常提示
+ error failed exception (异常) 在这些词汇上下几行翻译与理解.
+ startup ms毫秒 应用已经启动 加载到jvm内存中.
+ finished 应用部署完成
## 四、其他

### 1、tomcat管理端

```powershell
manager-gui
admin-gui

[root@web03 /app/tools/tomcat]# cat conf/tomcat-users.xml 
<?xml version="1.0" encoding="UTF-8"?>
<tomcat-users xmlns="http://tomcat.apache.org/xml"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
 version="1.0">
 <role rolename="manager-gui"/>
 <role rolename="admin-gui"/>
 <user username="tomcat" password="tomcat" roles="manager-gui,admin-gui"/>
</tomcat-users>
[root@web03 /app/tools/tomcat/webapps]# sed -i 's#127#\\d+#g' ./host-manager/META-INF/context.xml ./manager/META-INF/context.xml
[root@web03 /app/tools/tomcat/webapps]# pwd
/app/tools/tomcat/webapps
[root@web03 /app/tools/tomcat]# curl -utomcat:tomcat 
http://127.0.0.1:8080/manager/statu
```
### 2、多实例

```powershell
#复制代码
cd /app/tools/
cp -r tomcat/ tomcat_8081
cp -r tomcat/ tomcat_8082
#修改端口
sed -i 's#8080#8081#g' tomcat_8081/conf/server.xml
sed -i 's#8005#8006#g' tomcat_8081/conf/server.xml
sed -i 's#8080#8082#g' tomcat_8082/conf/server.xml
sed -i 's#8005#8007#g' tomcat_8082/conf/server.xml
#进行调试
[root@web03 /app/tools]# grep -P 'port="\d+"' tomcat_808*/conf/server.xml 
tomcat_8081/conf/server.xml:<Server port="8006" shutdown="SHUTDOWN">
tomcat_8081/conf/server.xml: <Connector port="8081" protocol="HTTP/1.1"
tomcat_8082/conf/server.xml:<Server port="8007" shutdown="SHUTDOWN">
tomcat_8082/conf/server.xml: <Connector port="8082" protocol="HTTP/1.1"
```
### 3、监控

+ 检查tomcat状态(jvm状态)
+ 命令行命令:jps jmap jstack
+ 命令行脚本:
+ 配置远程监控功能(zabbix)
#### (1)、java环境监控命令

```powershell
# java ps 命令 jps 
jps -lvm # ps -ef |grep java 

java 进程信息 ,java jvm信息 
命令+ java进程pid

# jmap java jvm信息,内存信息
# jstack java 进程信息
# jstat java 进程信息
```
#### (2)、Java环境监控脚本

```powershell
show-busy-java-threads.sh
process 进程 
thread 线程
```
#### (3)、 Tomcat远程监控功能(ZBX)

+ 通过监控工具获取tomcat里面jvm信息 
    - 方法01 通过命令/脚本
    - 方法02 开启tomcat远程监控功能 jmx (windows访问)

```powershell
 catalina.sh 修改与配置tomcat启动选项 java -xx -xxxx
 
 CATALINA_OPTS="$CATALINA_OPTS
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=12345
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
-Djava.rmi.server.hostname=10.0.0.9"

CATALINA_OPTS 修改或增加 java(tomcat) 启动选项
CATALINA_OPTS="$CATALINA_OPTS 
-Dcom.sun.management.jmxremote # jmxremote 开启功能
-Dcom.sun.management.jmxremote.port=12345 # 指定端口号 12345固定端口 +2个随机端口
-Dcom.sun.management.jmxremote.authenticate=false #是否开启认证功能
-Dcom.sun.management.jmxremote.ssl=false #ssl 关闭 
-Djava.rmi.server.hostname=10.0.0.9" #本地网卡ip 监听哪个网卡

#坑 坑 坑: 
CATALINA_OPTS="$CATALINA_OPTS\
-Dcom.sun.management.jmxremote\
-Dcom.sun.management.jmxremote.port=12345\
-Dcom.sun.management.jmxremote.authenticate=false\
-Dcom.sun.management.jmxremote.ssl=false\
-Djava.rmi.server.hostname=10.0.0.9"

[root@web03 /app/tools/tomcat]# grep -A5 ^CATALINA_OPTS= bin/catalina.sh
CATALINA_OPTS="$CATALINA_OPTS \
-Dcom.sun.management.jmxremote \
-Dcom.sun.management.jmxremote.port=12345 \
-Dcom.sun.management.jmxremote.authenticate=false \
-Dcom.sun.management.jmxremote.ssl=false \
-Djava.rmi.server.hostname=10.0.0.9"

[root@web03 /app/tools/tomcat]# ss -lntup |grep 12345 
tcp LISTEN 0 50 [::]:12345 [::]:* 
users:(("java",pid=8178,fd=21))
[root@web03 /app/tools/tomcat]#
```
### 4、tomcat和ngx

#### (1)、不涉及负载均衡

+ nginx+tomcat组合 
    - ngx处理静态资源(用户浏览器缓存,开启缓存功能)
    - tomcat处理动态资源

```powershell
[root@web03 /app/tools/tomcat/zrlog]# cat 
/etc/nginx/conf.d/zrlog.oldboylinux.cn.conf 
server {
 listen 80;
 server_name zrlog.oldboylinux.cn; 
 root /app/tools/tomcat/zrlog/ROOT;
 location / {
 proxy_pass http://10.0.0.9:8080;
 proxy_set_header Host $http_host; 
 }
 #location 图片静态资源 缓存
 #expires max;
# location ~* \.(js|css|html|png|jpg)$ {
# expires max;
# }
}
```
#### (2)、加入负载

ngx 负载均衡  
nginx (如果做了 代码动静分离可以使用这个)  
tomcat

| 角色    |                           |
| ----- | ------------------------- |
| lb01  |                           |
| web03 | 动态 tomcat 8080 zrlog      |
| web04 | ngx /app/code/zrlog/ROOT/ |
+ web04

```powershell
[root@web04 ~]# cat /etc/nginx/conf.d/zrlog.oldboylinux.cn.conf
server {
 listen 80;
 server_name zrlog.oldboylinux.cn; 
 root /app/code/zrlog/ROOT;
 location / {
 index index.html;
 }
 ##location 图片缓存
}
[root@web04 ~]# nginx -t 
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@web04 ~]# systemctl enable nginx 
Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to 
/usr/lib/systemd/system/nginx.service.
[root@web04 ~]# systemctl start nginx
```

+ lb01

```powershell
[root@lb01 ~]# cat /etc/nginx/conf.d/zrlog.oldboylinux.cn.conf
upstream zrlog_default {
 server 10.0.0.9:8080;
 server 10.0.0.9:8081;
 server 10.0.0.9:8082;
}
upstream zrlog_static {
 server 10.0.0.10:80;
 }
server {
 listen 80;
 server_name zrlog.oldboylinux.cn; 
 location / {
 proxy_pass http://zrlog_default; 
 include proxy_params;
 }
 location ~* \.(js|css|html|png|jpg)$ {
 proxy_pass http://zrlog_static;
 include proxy_params;
 }
}
```
## 五、优化

### 1、安全优化
#### (1)、修改Tomcat shutdown端口及暗号

```powershell
<Server port="8005" shutdown="SHUTDOWN">
修改
<Server port="8555" shutdown="dangrous">
```
#### (2)、ajp 连接端口保护 (已经注释)

```powershell
已经从tomcat 8.5 后 关闭 注释
```
#### (3)、禁用管理端

```powershell
#server.xml 中关于管理端的注释掉
<!--
 <GlobalNamingResources>
  <Resource name="UserDatabase" auth="Container"
		    type="org.apache.catalina.UserDatabase"
 			description="User database that can be updated and saved"
 			factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
 			pathname="conf/tomcat-users.xml" />
 </GlobalNamingResources>
-->

#管理端相关文件 删除掉
webapps/*

[root@web03 /app/tools/tomcat]# ll webapps/
total 8
drwxr-x--- 15 root root 4096 Nov 26 11:53 docs
drwxr-x--- 7 root root 99 Nov 26 11:53 examples
drwxr-x--- 6 root root 79 Nov 26 11:53 host-manager #管理端
drwxr-x--- 6 root root 114 Nov 26 11:53 manager #管理端
drwxr-x--- 3 root root 41 Nov 26 15:47 memtest
-rw-r--r-- 1 root root 643 Nov 26 15:14 memtest.war
drwxr-x--- 3 root root 223 Nov 26 11:53 ROOT
```
#### (4)、降权启动（强制）(监牢模式）

keep in jail 监牢模式 (让服务通过普通用户运行)  
tomcat目录所有者 改为普通用户即可.  
linux 下面 1-1024是特权端口 只能root管理与运行

```powershell
#root 关闭 tomcat 
#01 修改 目录所有者
#02 授权用户可以重启tomcat服务 (sudo) 
oldboy ALL=(ALL) /bin/systemctl restart tomcat 

#03 修改systemctl配置
[root@web03 /app/tools/tomcat]# cat /usr/lib/systemd/system/tomcat.service
[Unit]
Description= The tomcat web server by lidao996 v1.0
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target
[Service]
Type=forking
EnvironmentFile=/etc/sysconfig/tomcat
User=oldboy
ExecStart=/app/tools/tomcat/bin/startup.sh
ExecStop=/app/tools/tomcat/bin/shutdown.sh
ExecReload=/app/tools/tomcat/bin/shutdown.sh && sleep 2
&&/app/tools/tomcat/bin/startup.sh
[Install]
WantedBy=multi-user.target
```
#### (5)、文件列表访问控制(已经关闭)、

```shell
#web.xml 
 <init-param>
	 <param-name>listings</param-name>
	 <param-value>false</param-value>
 </init-param>
```
#### (6)、版本信息隐藏（强制）

```powershell
#web.xml 
<error-page>
<error-code>403</error-code>
 	<location>/forbidden.jsp</location>
</error-page>
<error-page>
<error-code>404</error-code>
	 <location>/notfound.jsp</location>
</error-page>
<error-page>
<error-code>500</error-code>
 	<location>/systembusy.jsp</location>
</error-page>
#
error_page
```
#### (7)、修改响应头的信息 Server Header

```powershell
修改 server.xml 中 8080部分
 <Connector port="8080" protocol="HTTP/1.1"
 			Server="Nginx/1.20.2" #增加这个即可
 			connectionTimeout="20000"
 			redirectPort="8443" />
```
#### (8)、访问限制(选作)

```powershell
# server.xml 中 Host部分内容
#path 匹配uri
<Context path="/admin/" docBase="/home/work/tomcat"
 		 debug="0" reloadable="false" crossContext="true">
 	<Valve className="org.apache.catalina.valves.RemoteAddrValve"
 			allow="61.148.18.138,61.135.165.*" deny="*.*.*.*"/>
</Context>
```
#### (9)、停脚本权限回收

```powershell
.sh 脚本 权限700 或750
```
#### (10)、修改访问日志格式

```powershell
pattern="%h %l %u %t &quot;%r&quot; %s %b &quot;%{Referer}i&quot; &quot;%{User-Agent}i&quot; %{X-Forwarded-For}i"
```
### 2、性能优化
#### (1)、tomcat工作模式(io模型)
| 模式 | 英文 | 含义 |
| --- | --- | --- |
| bio | blocking io | tomcat 7及之前, 同步模型 阻塞 一个线程处理一个请求，缺点：并发量高时，线程数较多，浪费资源。 |
| nio | new io | tomcat 8及以后的工作模式 异步 非阻塞 nio1(默认的) nio2 可以通过少量的线程处理大量的请求 |
| apr | Apache Portable Runtime | 应对高并发场景 Tomcat对静态文件的处理性能。Tomcat apr也是在Tomcat上运行高并发应用的首选模式 |
```powershell
30-Nov-2021 11:17:14.578 INFO [main] org.apache.coyote.AbstractProtocol.start 
Starting ProtocolHandler ["http-nio-8080"]
```

```powershell
<Connector port="8080" protocol="HTTP/1.1"
			 connectionTimeout="20000"
			 redirectPort="8443" /> 

#修改后 
 <Connector port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol" 
#修改protocal部分即可
 			Server="Nginx/1.20.2"
 			connectionTimeout="20000"
 			redirectPort="8443" />
```
#### (2)、禁用dns逆向解析功能

```powershell
enableLookups="false" #dns-- 域名--->ip ip----->域名

<Connector port="8080" protocol="HTTP/1.1"
 		   Server="Nginx/1.20.2"
		   enableLookups="false"
		   connectionTimeout="20000"
		   redirectPort="8443" />
```
#### (3)、开启压缩功能 (gzip响应)

静态资源: html js css

```powershell
#nginx gzip on;
#tomcat compression; 
compression="on" #开启 tomcat压缩功能 静态文本资源 html js css
compressionMinSize="2048" #压缩文件 最小2048字节
compressableMimeType="text/html,text/plain,text/css,application/javascript,applica
tion/json,application/x•font-ttf,application/x-font-otf" #压缩哪些类型的文件 文本类型 js,css,html

compression="on"
compressionMinSize="2048"
compressableMimeType="text/html,text/plain,text/css,application/javascript,applica
tion/json,application/x-font-ttf,application/x-font-otf"
```
#### (4)、动态选项-线程数量

设置好数字,然后通过压力测试软件调试.配合监控.

+ 压力测试工具 
    - ab(http)
    - stress(系统)
    - jmeter(http/接口...) Java
    - LoadRunner 专业做压力测试

```powershell
maxThreads="500" #最大的线程数量 200-400之间
acceptCount="500" #当达到最大线程数量的时候 队列长度
#一般与maxThreads 一致
acceptorThreadCount="2" #分为几对 与cpu核心总数一致或2倍 默认是1 
minSpareThreads="10" #空闲时候最小的线程数量 不忙的时候 最少留几个服务员 
值班的线程数量
```
#### (5)、jvm内存调整

+ jvm初始内存
+ jvm最大内存

```powershell
#设置 jvm初始内存大小 (物理内存1/64) jvm最大内存大小 (物理内存的1/4 )
##修改 catalina.sh文件
JAVA_OPTS='-Xms1024m -Xmx1024m -Xloggc:/var/log/tomcat_gc.log'
#-Xms jvm 初始内存 s start 
#-Xmx max 最大 x max jvm最大内存
#一般 -Xmx 是 -Xms 2倍
#gc garbage collect 垃圾回收 定期清理 jvm内存
#-Xloggc 执行gc的时候 日志 出现故障tomcat崩溃 挂了 catalaina.out gc日志
java -jar xxx.jar -Xms1024m -Xmx1024m
```
## 六、故障案例
### 1、tomcat 开机自启动故障（定时任务重启 tomcat失败）

```powershell
#tomcat开机自启动
##方法01 
使用 systemctl 配置 然后通过systemctl enable tomcat 形式启动。

##方法02 tomcat启动命令 /etc/rc.d/rc.local /etc/rc.local 
/app/tools/tomcat/bin/startup.sh 

发现重启后tomcat启动失败

原因：
开机的时候或定时任务运行脚本的时候，无法识别到 我们自定义的PATH环境变量，没有java
相关路径，导致开机tomcat启动失败

解决：
脚本开头重新定义PATH变量或脚本开头重新加载变量
. /etc/profile
/app/tools/tomcat/bin/startup.sh
```
### 2、 tomcat(java) 服务器 运行占用大量swap 内存 占用较少

```powershell
原因： 代码问题。
临时解决： 

1. 临时增大swap 
2. 调整linux内核参数 让系统更优先使用 内存
echo 'vm.swappiness = 0' >>/etc/sysctl.conf #vm.swappiness swap亲和性 越
大越优先使用swap 越小越优先使用内存
 sysctl -p #生效
 
3. 看程序代码问题。
```
### 3、 tomcat负载高

+  整体思路与流程 
    - 先定位tomcat的线程
    - 各种导出统计线程，进程信息
    - 与开发一起看
+  排错流程 
    - 通过ps aux/top命令排查出tomcat有问题
    - 通过top -Hp tomcat 进程pid
    - 通过jstack命令 进程id 导出与显示进程的信息 并过滤线程信息
    - 辅助手段tomcat日志

```powershell
#详细流程
##01 过ps aux/top命令排查出tomcat有问题
top
ps aux 

##02 通过top -Hp tomcat 进程pid 查看哪个线程有问题

[root@web03 ~]# top -Hp 1389 #java进程id 
top - 09:18:12 up 19 min, 1 user, load average: 0.02, 0.06, 0.07
Threads: 30 total, 0 running, 30 sleeping, 0 stopped, 0 zombie
%Cpu(s): 0.3 us, 0.3 sy, 0.0 ni, 99.3 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
KiB Mem : 2027868 total, 1496784 free, 347396 used, 183688 buff/cache
KiB Swap: 2097148 total, 2097148 free, 0 used. 1529540 avail Mem 
 PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND 
 
 1415 root 20 0 2548984 130108 13836 S 0.3 6.4 0:00.78 java 
 1642 root 20 0 2548984 130108 13836 S 0.3 6.4 0:01.70 java 
 1668 root 20 0 2548984 130108 13836 S 0.3 6.4 0:00.10 java  
 1389 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.03 java  
 1399 root 20 0 2548984 130108 13836 S 99 6.4 0:08.00 java #这个有问题 
 1418 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.01 java  
 1419 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.01 java  
 1424 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java  
 1426 root 20 0 2548984 130108 13836 S 0.0 6.4 0:08.62 java  
 1427 root 20 0 2548984 130108 13836 S 0.0 6.4 0:02.99 java  
 1428 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java  
 1520 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.19 java  
 1555 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.19 java  
 1639 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java  
 1640 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java  
 1641 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java  
 1643 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java  
 1671 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java  
 1674 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.16 java  
 1678 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java  
 1681 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java  
 1683 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java
 1684 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java  
 1691 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.00 java  
 1692 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.02 java  
 1698 root 20 0 2548984 130108 13836 S 0.0 6.4 0:00.20 java

##03 :three: 通过jstack命令 导出与显示进程的信息 并过滤线程信息
jstack 1389 #tomcat 进程pid 过滤 线程的16进制id 
[root@web03 ~]# echo 'obase=16;1399' |bc
577
jstack 1389 |grep 577 #注意： 577是 线程id的16进制形式
查看里面state的状态 一般状态
RUNNABLE
TIMED_WAITING
WAITING
[root@web03 ~]# jstack 1389|grep -A10 577
"main" #1 prio=5 os_prio=0 tid=0x00007f1d4800a800 nid=0x577 runnable [0x00007f1d4f68e000]
 java.lang.Thread.State: RUNNABLE
 at java.net.PlainSocketImpl.socketAccept(Native Method)
 at java.net.AbstractPlainSocketImpl.accept(AbstractPlainSocketImpl.java:535)
 at java.net.ServerSocket.implAccept(ServerSocket.java:545)
 at java.net.ServerSocket.accept(ServerSocket.java:513)
 at org.apache.catalina.core.StandardServer.await(StandardServer.java:602)
 at org.apache.catalina.startup.Catalina.await(Catalina.java:864)
 at org.apache.catalina.startup.Catalina.start(Catalina.java:810)
 at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
 at 
sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)

#把前3步骤合并 请使用 show_busy_java_threads.sh 
[root@web03 ~]# sh /server/scripts/show-busy-java-threads.sh 
[1] Busy(0.5%) thread(1694/0x69e) stack of java process(1677) under user(root):
"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007f0fc40d8800 nid=0x69e 
waiting on condition [0x0000000000000000]
 java.lang.Thread.State: RUNNABLE
 
[2] Busy(0.4%) thread(1713/0x6b1) stack of java process(1677) under user(root):
"Thread-8" #18 prio=5 os_prio=0 tid=0x00007f0fc433b800 nid=0x6b1 waiting on 
condition [0x00007f0fc8f76000]
 java.lang.Thread.State: TIMED_WAITING (sleeping)
 at java.lang.Thread.sleep(Native Method)
 at 
com.zrlog.plugincore.server.config.PluginConfig$1.run(PluginConfig.java:78)

[3] Busy(0.4%) thread(1426/0x592) stack of java process(1389) under user(root):
"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007f1d480b2000 nid=0x592 
waiting on condition [0x0000000000000000]
 java.lang.Thread.State: RUNNABLE
```

+ Tomcat负载高-进阶排查流程 
    1. 整体排查: vmstat(r cpu b io) /top/ps aux 找出哪个进程的问题
    2. 找出进程对应线程id 
        1. 通过top -Hp java进程id 找出是哪个java线程的问题
        2. 或者 show_busy_java_threads.sh
    3. 转换:线程id--->16进制:问题线程的id 转换为16进制 线程id
    4. 找出线程的详细信息: jstack (显示java 进程信息) jstack java进程id 过滤 java线程的16进制id 与开发沟通
    5. 找出线程的详细信息: jstack (显示java 进程信息) jstack java进程id 过滤 java线程的16进制id 与开发沟通
    6. 找出线程的详细信息: jstack (显示java 进程信息) jstack java进程id 过滤 java线程的16进制id 与开发沟通
    7. 给开发分析jvm导出文件:通过 mat(Eclipse Memory Analyzer Tool )分析 windows

```powershell
# 05 1. :five: 显示jvm信息: jmap (显示java jvm信息) jmap -heap java进程id 显示jvm
的内存使用情况
2. :six:
[root@web03 ~]# jmap -heap 1389
Attaching to process ID 1389, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.301-b09

using thread-local object allocation.
Mark Sweep Compact GC

Heap Configuration:
 MinHeapFreeRatio = 40
 MaxHeapFreeRatio = 70
 MaxHeapSize = 520093696 (496.0MB)
 NewSize = 11141120 (10.625MB)
 MaxNewSize = 173342720 (165.3125MB)
 OldSize = 22413312 (21.375MB)
 NewRatio = 2
 SurvivorRatio = 8
 MetaspaceSize = 21807104 (20.796875MB)
 CompressedClassSpaceSize = 1073741824 (1024.0MB)
 MaxMetaspaceSize = 17592186044415 MB
 G1HeapRegionSize = 0 (0.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
 capacity = 10158080 (9.6875MB)
 used = 5658696 (5.396553039550781MB)
 free = 4499384 (4.290946960449219MB)
 55.706353956653224% used
Eden Space:
 capacity = 9043968 (8.625MB)
 used = 5184400 (4.9442291259765625MB)
 free = 3859568 (3.6807708740234375MB)
 57.324395663496375% used
From Space:
 capacity = 1114112 (1.0625MB)
 used = 474296 (0.45232391357421875MB)
 free = 639816 (0.6101760864257812MB)
 42.571662454044116% used
To Space:
 capacity = 1114112 (1.0625MB)
 used = 0 (0.0MB)
 free = 1114112 (1.0625MB)
 0.0% used
tenured generation:
 capacity = 22413312 (21.375MB)
 used = 17675368 (16.856544494628906MB)
 free = 4737944 (4.518455505371094MB)
 78.86102687545687% used

18785 interned Strings occupying 2006464 bytes.

## 06 :six: jvm内存内容导出(内存信息取样) :jmap (导出 jvm内存的内容 ) `jmap -
dump:format=b,file=/root/tomcat.bin 进程pid`
[root@web03 ~]# jmap -dump:format=b,file=/root/jvm.bin 1389
Dumping heap to /root/jvm.bin ...
Heap dump file created
[root@web03 ~]# ll -h /root/jvm.bin 
-rw------- 1 root root 34M Nov 30 10:08 /root/jvm.bin
[root@web03 ~]# file /root/jvm.bin
/root/jvm.bin: data

## 07 给开发分析jvm导出文件:通过 mat(Eclipse Memory Analyzer Tool )分析 windows
MemoryAnalyzer.exe
```

