## nginx学习

### 0、参考链接

1、[Nginx详细讲解 ](https://www.cnblogs.com/zhangjianchao/p/15328394.html)

2、[Ubuntu安装nginx](https://blog.csdn.net/jiayouwa_jiayou/article/details/125760333)

3、[Nginx 配置详解--菜鸟教程](https://www.runoob.com/w3cnote/nginx-setup-intro.html)

4、[Nginx 请求处理流程](https://blog.csdn.net/l619872862/article/details/109330911)

5、[Nginx 虚拟主机配置 ](https://www.cnblogs.com/wushuaishuai/p/9343044.html)

6、[解决 nginx: [error] invalid PID number "" in "/usr/local/nginx/logs/nginx.pid"](https://blog.csdn.net/cxs123678/article/details/80201412)







### 1、简介

#### 1、是什么

```bash
1、一个开源的，支持高性能，高并发的www服务和代理服务软件
2、具有高并发（特别是静态资源），占用系统资源少等特性
3、不但是一个优秀Web服务软件，还具有反向代理负载均衡功能和缓存服务功能，与lvs负载均衡及Haproxy等专业代理软件相比，Nginx部署起来更为简单，方便
4、在缓存功能方面，它又类似于Squid等专业的缓存服务软件
```



#### 2、重要特性

```bash
1、"支持高并发"：能支持几万并发连接（特别是静态小文件业务环境）
2、"资源消耗少"：在3万并发连接下，开启10哥Nginx线程消耗的内存不到200MB
3、可以做HTTP反向代理及加速缓存，即负载均衡功能，内置对RS节点服务器健康检查功能，这相当于专业的Haproxy软件或LVS的功能
4、具备Squid等专业缓存软件等的缓存功能
5、支持异步网络I／O事件模型epoll（linux2.6+）
```





### 2、安装配置

#### 1、CentOS系列

```bash
# 用本地yum仓库安装依赖包
yum install -y pcre-devel openssl-devel         
# 下载软件源码包
# wget -q http://nginx.org/download/nginx-1.10.2.tar.gz   

# 创建程序用户
useradd -s /sbin/nologin -M www 
# 解压缩
tar xf nginx-1.10.2.tar.gz -C /usr/src/     
cd /usr/src/nginx-1.10.2
# 预配置
./configure --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module 

# 编译和安装
make && make install 
# 给命令做软连接，以便PATH能找到
ln -s /usr/local/nginx/sbin/* /usr/local/sbin/  
# 启动nginx
/usr/local/nginx/sbin/nginx
```



#### 2、Ubuntu系列

##### 0、参考链接

1、[[已解决]Ubuntu安装libssl-dev失败](https://blog.csdn.net/guyongqiangx/article/details/73574851)





##### 1、实际步骤

```bash
# 升级依赖
apt-get update
apt-get install build-essential
apt-get install libtool
apt-get install libpcre3 libpcre3-dev
apt-get install  zlib1g-dev
apt-get install libssl-dev

# 使用wget下载nginx安装包
wget http://nginx.org/download/nginx-1.21.6.tar.gz

# 解压下载的压缩包
tar -xvf nginx-1.21.6.tar.gz

# 进入nginx包目录下
cd nginx-1.21.6

# 创建nginx用户（执行编译命令前先创建Nginx用户不然后面Nginx运行时会报错）
useradd nginx

# 执行命令编译安装(这一步安装的时候，出现了一个"E: Unable to correct problems, you have held broken packages."的错误)
# 解决方式参考此处链接1

./configure --prefix=/usr/local/nginx \
--user=nginx --group=nginx \
--with-http_gzip_static_module \
--with-http_flv_module \
--with-http_ssl_module \
--with-http_realip_module \
--with-http_v2_module \
--with-http_sub_module \
--with-http_mp4_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-pcre --with-stream \
--with-stream_ssl_module \
--with-stream_realip_module

make && make install


# 执行以上命令之后，nginx安装成功目录为/usr/local/nginx，进入该目录
cd /usr/local/nginx

# 配置nginx软连接
ln -s /usr/local/nginx/sbin/nginx /usr/bin/
# 软连接配置完成即以成功开启nginx服务即可，在浏览器上访问对应ip即可
```





##### e、所遇问题

**所遇问题1：**

```bash
#（解决上面遇到的问题）
$ sudo apt-get install aptitude # 如果已经安装了，则直接降低版本即可
$ sudo aptitude install libssl-dev

"..."

Keep the following packages at their current version:
1)     libssl-dev [Not Installed]                         


# 这里提示时，一定要选n，选Y跟apt-get install操作一样
Accept this solution? [Y/n/q/?] n

"..."

# 接受这里的降级处理，成功安装
Accept this solution? [Y/n/q/?] y
The following packages will be DOWNGRADED:
  libssl1.0.0 
```





### 3、nginx相关命令

```bash
# nginx平滑重启命令
nginx -s reload   # /usr/local/nginx/sbin/nginx -s reload
# nginx停止服务命令
nginx -s stop     # /usr/local/nginx/sbin/nginx -s stop
```





### 4、nginx主配置文件nginx.conf

**使用命令：**

```bash
egrep -v "#|^$" nginx.conf     #去掉包含#号和空行的内容
```



**conf文件具体内容：**

```bash
worker_processes  1;                           # worker进程的数量
# error_log  logs/error.log;                   # 错误日志（默认没开）
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg

pid        logs/nginx.pid;                     # 进程号（默认没开）
events {                                       # -----------事件区块开始
    accept_mutex on;                           # 设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;                           # 设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;                                # 事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;                  # 每个worker进程支持的最大连接数
}                                              # -----------事件区块结束
http {                                         # -----------http区块开始
    include       mime.types;                  # Nginx支持的媒体类型库文件包含
    default_type  application/octet-stream;    # 默认的媒体类型
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for';       #自定义格式
    # access_log off;                          # 取消服务日志    
    access_log log/access.log myFormat;        #combined为日志格式的默认值
    sendfile        on;                        # 开启高效传输模式
    sendfile_max_chunk 100k;                   # 每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout  65;                     # 连接超时
    
    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;       # 热备
    }
    error_page 404 https://www.baidu.com;      # 错误页
    
    
    server {        
        keepalive_requests 120;                # 单连接请求上限次数# 网站配置区域（第一个server第一个虚拟主机站点）
        listen       80;                       # 提供服务的端口，默认80
        server_name  www.chensiqi.org;         # 提供服务的域名主机名
        location / {                           # ------------第一个Location区块开始
            root   html;                       # 站点的根目录（相对于nginx安装路径）
            index  index.html index.htm;       # 默认的首页文件，多个用空格分开
        }
        error_page 500 502 503 504  /50x.html; # 出现对应的http状态码时，使用50x.html回应客户
       location  ~*^.+$ {                      # 请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;                         # 根目录
           #index vv.txt;                      # 设置默认页
           proxy_pass  http://mysvr;           # 请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;                     # 拒绝的ip
           allow 172.18.5.54;                  # 允许的ip           
        } 
    }
    server {                                   # 网站配置区域（第二个server第二个虚拟主机站点）
        listen       80;                       # 提供服务的端口，默认80
        server_name  bbs.chensiqi.org;         # 提供服务的域名主机名
        location / {                           # ------------服务区块
            root   html;                       # 相对路径（nginx安装路径）
            index  index.html index.htm;
        }
        location = /50x.html {                 # 发生错误访问的页面
            root   html;
        }
    }
}
```





### 5、Nginx虚拟主机配置实战

#### 1、基本概念

```bash
# 1、什么是虚拟主机？虚拟主机的作用是什么？有什么优缺点？类型有哪些？
1、概念：
   是"一种单一主机或主机群";
   其技术是互联网服务器采用的"节省服务器硬件成本的技术"，对外表现为多个服务器，从而"充分利用服务器硬件资源";
2、目的：
   方便管理: "彼此可以共享相同的配置设置"
   提高性能: 相同主机内的虚拟主机可以"共享彼此的程序集"（Process Pool），因此"可以缩短对客户端的回应时间"
   降低成本: 虚拟主机"使得单一服务器的资源可以被更有效的利用"，包括存储器、存储空间或处理器资源
3、类型：
   ①基于域名的虚拟主机
   ②基于端口的虚拟主机
   ③基于IP的虚拟主机

# 扩展
- nginx虚拟主机的特点为：
-- 使用一个"server{}标签"来标示一个虚拟主机
```





#### 2、基于域名

##### 0、单文件配置

```bash
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.abc.com;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
    }
   
    server {
        listen       80;
        server_name  bbs.abc.com;
        location / { 
            root   html/bbs;
            index  index.html index.htm;
        }   
    }   

    server {
        listen       80;
        server_name  blog.abc.com;
        location / { 
            root   html/blog;
            index  index.html index.htm;
        }   
    }   
}
```







##### 1、规范化nginx配置文件

```bash
# 将每个虚拟主机配置成单独的文件，放在统一目录中（如：vhosts）
# ===========================步骤=============================
# 1、创建vhosts目录
cd /usr/local/nginx/conf/
mkdir vhosts

# 2、编辑 nginx.conf 主配置文件
vim nginx.conf
# 配置多个虚拟主机配置文件
```

配置的`nginx.conf`的内容为：

```bash
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include vhosts/*.conf;
}
```



分别给每个虚拟主机配置文件：

```bash
vim vhosts/www.conf # 配置第一个名为www的虚拟主机的配置文件
```

```bash
server {
        listen       80;
        server_name  www.com;
        location / {
                root   html/www;
                index  index.html index.htm;
        }
}
```



```bash
vim vhosts/abs.conf # 配置第一个名为abs的虚拟主机的配置文件
```

```bash
server {
        listen       80;
        server_name  abs.com;
        location / {
                root   html/www;
                index  index.html index.htm;
        }
}
```



##### 2、创建虚拟主机站点对应的目录和文件

```bash
[root@localhost html]$ cd /usr/local/nginx/html/
[root@localhost html]$ for n in www abs blog
> do
> mkdir ${n}
> echo "http://${n}.com" > ${n}/index.html
> done
```



##### 3、编辑 /etc/hosts 文件，域名解析

```bash
echo "127.0.0.1 www.com bbs.com blog.com" >> /etc/hosts
```



##### 4、重新加载 Nginx 配置

```bash
# 检查配置文件语法是否正确
/usr/local/nginx/sbin/nginx -t

# 重启nginx服务
/usr/local/nginx/sbin/nginx -s reload
```



**这一步会出现的一个问题：**

**<img src="C:\Users\86185\AppData\Roaming\Typora\typora-user-images\image-20220925134725234.png" alt="image-20220925134725234" style="zoom:80%;" />**



问题描述：

```bash
root@ubuntu:/usr/local/nginx/sbin# ./nginx -s reload
nginx: [error] invalid PID number "" in "/usr/local/nginx/logs/nginx.pid"
```

**解决方式：**

```bash
# 这里实际上不用管有没有nginx.pid这个文件在logs文件夹下，都执行下方的命令

root@ubuntu:/usr/local/nginx/sbin# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
root@ubuntu:/usr/local/nginx/sbin# ./nginx -s reload
```





##### 5、访问测试

```bash
curl http://www.com
```

**<img src="C:\Users\86185\AppData\Roaming\Typora\typora-user-images\image-20220925134851142.png" alt="image-20220925134851142" style="zoom:80%;" />**



##### n、其他

```bash
# 使用grep过滤命令来生成基础的Nginx主配置文件nginx.conf
cd /usr/local/nginx/conf # 先到nginx.conf配置文件目录下

# 生成一个空白的配置文件
egrep -v "#|^$" nginx.conf.default >nginx.conf
```



```bash
# 检查语法
/usr/local/nginx/sbin/nginx -t
```

**<img src="https://raw.githubusercontent.com/Francis-cell/Picture/main/202209251210025.png" alt="image-20220925121056947" style="zoom:80%;" />**



由于**web存放的路径是相对路径**，所以需要**先创建一个目录**

```bash
root@ubuntu:/usr/local/nginx/conf# cd ../html/
root@ubuntu:/usr/local/nginx/html# pwd
/usr/local/nginx/html
root@ubuntu:/usr/local/nginx/html# ls
50x.html  index.html                                                    # 原本网页的目录
root@ubuntu:/usr/local/nginx/html# mkdir www                            # 创建一个目录叫做www
root@ubuntu:/usr/local/nginx/html# echo "I am www" > www/index.html     # 写入网页文件
root@ubuntu:/usr/local/nginx/html# cat www/index.html                   # 查看一下
I am www
```

**<img src="https://raw.githubusercontent.com/Francis-cell/Picture/main/202209251210164.png" alt="image-20220925121017086" style="zoom:80%;" />**





#### 3、基于端口

##### 1、编辑配置文件

```bash
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.abc.com;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
    }
   
    server {
        listen       81;
        server_name  www.abc.com;
        location / { 
            root   html/bbs;
            index  index.html index.htm;
        }   
    }   

    server {
        listen       82;
        server_name  www.abc.com;
        location / { 
            root   html/blog;
            index  index.html index.htm;
        }   
    }   
}
```



##### 2、创建虚拟主机站点对应的目录和文件

```bash
[root@localhost html]# cd /usr/local/nginx/html/
[root@localhost html]# for n in 80 81 82
> do
> mkdir ${n}
> echo "http://www.abc.com:${n}" > ${n}/index.html
> done
```



##### 3、编辑 /etc/hosts 文件，域名解析

```bash
echo "127.0.0.1 www.abc.com" >> /etc/hosts
```



##### 4、重新加载 Nginx 配置

```bash
# 检查配置文件语法是否正确
/usr/local/nginx/sbin/nginx -t

# 重启nginx服务
/usr/local/nginx/sbin/nginx -s reload
```



##### 5、访问测试

```bash
curl http://www.abc.com:80
```





#### 4、基于IP

不常使用。只需要将基于域名的虚拟主机中的域名修改为 IP 就可以了，前提是服务器有多个IP地址。如果需要不同的 IP 对应不同的服务，可在网站前端的负载均衡器上配置。







### Append

#### 1、客户端访问网站异常排查方式

```bash
# 1、在客户端上ping服务器端IP
ping XXX.XXX.XXX.XXX # 排除物理线路问题影响

# 2、在客户端上telnet服务器端IP，端口
telnet XXX.XXX.XXX.XXX XX   # 排除防火墙等影响

# 3、在客户端使用wget命令检测
wget 10.0.0.8(curl -I 10.0.0.8) # 模拟用户访问，排除http服务自身问题，根据输出在排错
```





#### 2、正向代理&反向代理

##### 1、正向代理

**<img src="C:\Users\86185\AppData\Roaming\Typora\typora-user-images\image-20220925141844230.png" alt="image-20220925141844230" style="zoom:80%;" />**



##### 2、反向代理

**<img src="C:\Users\86185\AppData\Roaming\Typora\typora-user-images\image-20220925142121017.png" alt="image-20220925142121017" style="zoom:80%;" />**





