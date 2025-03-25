# 一、Nginx简介与安装
#### 1、Nginx简介
> 高性能的WEB服务器

#### 2、进程
- worker process工作进程是单线程的，可以多配置几个提高并发量
- master process主进程，用来协调

> 问题：为什么worker process一个进程能够处理成千上万个连接？因为Nginx使用了NIO非阻塞型，在进行数据操作的时候，Nginx不会去等待，拷贝数据的时候会存放在一个select列表，Nginx会去遍历已经准备好的请求，然后返回。

#### 3、Nginx配置语法
```
分类：
    语句块：使用 { }
    属性： 属性名 values（多个）
    参数： 属性 = /参数 { 代码行 }
    
events { } 事件语句块
http {
    
    server {
        #配置一个具体的站点，可以有多个
        listen 监听端口
        server_name 域名（多个，空格分开）比如 www.baidu.com *.baidu.com
        location / {
            root /html;
            index index.html index.html
        }
        location = /basic_status {
            alias 配置别名，站点下的路径可能有区别
        }
        location = /baidu {
            proxy_pass http://www.baidu.com;   将请求转发到百度
        }
    }
    
} http语句块 只能有一个。
root和alias的区别：
    root是站点的根目录，绝对匹配。可以server，location模块下匹配
    alias是站点别名，匹配的时候会去除location中对应的前缀，只能location下匹配
location：（不同的url导流到不同的站点中）
    = 优先级最高
    / 用来匹配url
```

#### 2、反向代理
> 正向代理和反向代理实现没有任何区别，如果用来屏蔽后端服务以及实现负债均衡就是反向代理，如果用来屏蔽客户端IP以及解决客户端不能直连服务端的问题就是正向代理

```
负载均衡：
    #配置块,代理服务器集群
    upstream backend{
        server 127.0.0.1:8001 weight=3; 权重
        server 127.0.0.1:8002 weight=1;
    }
    server {
        listen 80 default;
        location / {
            proxy_pass http://backend/;
        }
    }

```