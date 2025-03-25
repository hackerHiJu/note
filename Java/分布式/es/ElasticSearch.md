# ElasticSearch

## 1、单机ES搭建

- 解压jar包：tar -zxvf elasticsearch-7.12.0-linux-x86_64.tar.gz
- 修改conf/elasticsearch.yml文件

```
#集群名称
cluster.name: my-es
#节点名称
node.name: node-1
# 存放数据位置，自定义
path.data: /usr/local/software/elasticsearch-7.12.0/data
# 日志存放位置
path.logs: /usr/local/software/elasticsearch-7.12.0/logs
# 对外提供服务地址
network.host: 192.168.60.46
http.port: 9200
# 单机节点就只有node-1
cluster.initial_master_nodes: ["node-1"]
```

- 添加es用户进行运行，安全问题，es不能使用root服务运行
  - groupadd es
  - useradd es
  - 给当前用户赋予es安装目录下的所有权限 chown -R es:es /es
- 错误异常信息：
  - **错误信息：**max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
    - 修改vim /etc/sysctl.conf
    - 在文件末尾追加：vm.max_map_count=655360
    - 配置生效sysctl -p
  - **错误信息：** [1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
    - 修改 vim /etc/security/limits.conf   (如果没有 就追加)
    - \* soft nofile 65536
    - \* hard nofile 65536
    - \* soft nproc 4096
    - \* hard nproc 4096
  - **错误信息：**system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk
    - 修改 vim elasticsearch.yml
    - bootstrap.memory_lock: false  # 释放出来
    - bootstrap.system_call_filter: false  # 追加
- 启动：nohup ./bin/elasticsearch > /logs &   以后台方式启动

