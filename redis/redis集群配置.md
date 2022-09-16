> redis集群需要至少三个master节点，我们这里搭建master节点，并且给每个master再搭建一个slave节点，总共6个redis节点，这里用三台机器部署6个redis实例，每台机器一主一从。

---

准备三台服务器：192.168.0.1 192.168.0.2 192.168.3

## 第一步

- 在第一台机器的/usr/local下创建文件夹redis-cluster,然后在其下面分别创建2个文件夹如下
  
```
    mkdir -p /usr/local/redis-cluster
    mkdir 8001 8004
  ```

## 第二步

- 把之前的redis.conf配置文件copy到8001，修改如下内容
  
```
  1) daemonize yes
  2) port 8001 (分别对每个机器的端口号进行设置)
  3) pidfile /var/run/redis_8001.pid [](把pid进程写入pidfile配置的文件)
  4) dir /usr/local/redis-cluster/8001/ (指定数据文件存在位置，必须要指定不同的目录位置，不然会丢失数据)
  5) cluster-enabled yes (启动集群模式)
  6) cluster-config-file nodes-8001.conf (集群节点信息文件，这里800x最好和port对应上)
  7) cluster-node-timeout 15000 (根据业务需求配置对应的超时时间)
  8) # bind 127.0.0.1 （bind绑定的是自己机器网卡的ip，如果有多块网卡可以配多个ip，代表允许客户端通过机器的哪些网卡ip去访问，内网一般可以不配置bind，注释掉即可）
  9) protected-mode no (关闭保护模式)
  10) appendonly yes
  如果要设置密码需要增加如下配置
  11) requirepass xxxxxxx (设置redis访问密码)
  12) masterauth xxxxxxx (设置集群节点间访问密码，跟上面一致)
  ```

## 第三步

- 把修改后的配置文件，copy到8004，修改第2、3、4、6项里的端口号，可以用批量替换
  
```
  :%s/源字符串/目的字符串/g
  ```
  
  ## 第四步
  
  - 另外两台机器也需要做上几步操作，第二台机器用8002和8005，第三台机器用8003和8006

## 第五步

- 分别启动6个实例，然后检查是否启动成功
  
```
  redis-server redis.conf
  ```

## 第六步

   #下面命令里的1代表为每个创建的主服务节点创建一个服务器节点
   #执行这条命令需要确认三台机器之间的redis实例要能互相访问，可以先简单把所有机器防火墙关掉，如果不关闭防火墙则需要打开redis服务端口和集群节点

```
redis‐cli ‐a passwd ‐‐cluster create ‐‐cluster‐replicas 1 192.168.0.1:8001 192.168.0.2:8002 192.168.0.3:8003 192.168.0.1:8004 192.168.0.2:8005 192.168.0.3:8006
```

Ps:

    1. 再次启动集群不需要使用上述命令，只需要启动对应的redis实例即可，因为创建成功后，会将配置持久化

    2. 不需要手动指定主从关系，redis会自动进行匹配，尽量让主库和从库不在同一台机器上，可以登录客户端使用cluster node命令查看信息

## 第七步

- 验证集群，可以通过设置key，去到不同的客户端查看key或者通过cluster info、cluster nodes查看，闭集群则需要逐个进行关闭，使用命令
  
```
  redis‐cli ‐a passwd 192.168.0.x ‐p 800* shutdown
  ```
