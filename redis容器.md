### Redis高可用-docker部署

需要启动6个容器

![image-20210122035732355](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122035732355.png)

```shell
for port in $(seq 1 6);
do
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF > /mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done
```

```shell
[root@localhost ~]# docker run -p 6371:6379 --name redis-1 -v /mydata/redis/node-1/data:/data -v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf -d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

[root@localhost ~]# docker run -p 6372:6379 --name redis-2 -v /mydata/redis/node-2/data:/data -v /mydata/redis/node-2/conf/redis.conf:/etc/redis/redis.conf -d --net redis --ip 172.38.0.12 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

。。。。。
等6台redis容器
[root@localhost ~]# docker exec -it redis-1 /bin/sh
/data # redis-cli  --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.1
3:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1 
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.38.0.15:6379 to 172.38.0.11:6379
Adding replica 172.38.0.16:6379 to 172.38.0.12:6379
Adding replica 172.38.0.14:6379 to 172.38.0.13:6379
M: ac80f36fa89fc4ac9801958c0c596086f75c44bc 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
M: 7002ed4fad18d04a38d1f2027fa55334e4be07df 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
M: 4b151c5e8ce0a063b626825ba7b51346bc9f0b9a 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
S: 3bbf8e369a9122aa54ed10097b56461ff0870db8 172.38.0.14:6379
   replicates 4b151c5e8ce0a063b626825ba7b51346bc9f0b9a
S: df3b3ad811ecc9e9b2f8b3b866b2e8f49148df15 172.38.0.15:6379
   replicates ac80f36fa89fc4ac9801958c0c596086f75c44bc
S: c724f7c01794e11ec72d99f29cd87d52aade0d91 172.38.0.16:6379
   replicates 7002ed4fad18d04a38d1f2027fa55334e4be07df
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 172.38.0.11:6379)
M: ac80f36fa89fc4ac9801958c0c596086f75c44bc 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 7002ed4fad18d04a38d1f2027fa55334e4be07df 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: c724f7c01794e11ec72d99f29cd87d52aade0d91 172.38.0.16:6379
   slots: (0 slots) slave
   replicates 7002ed4fad18d04a38d1f2027fa55334e4be07df
M: 4b151c5e8ce0a063b626825ba7b51346bc9f0b9a 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 3bbf8e369a9122aa54ed10097b56461ff0870db8 172.38.0.14:6379
   slots: (0 slots) slave
   replicates 4b151c5e8ce0a063b626825ba7b51346bc9f0b9a
S: df3b3ad811ecc9e9b2f8b3b866b2e8f49148df15 172.38.0.15:6379
   slots: (0 slots) slave
   replicates ac80f36fa89fc4ac9801958c0c596086f75c44bc
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

![image-20210122042716677](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122042716677.png)