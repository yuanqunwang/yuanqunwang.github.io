## Docker搭建Redis集群

1. 创建Docker共享网络。

   ```shell
   $ docker network create redis_cluster
   $ docker network ls
   ```

2. 集群模式下运行Redis的配置文件`redis.conf`

   ```shell
   port 6379
   cluster-enabled yes
   cluster-config-file nodes.conf
   cluster-node-timeout 5000
   appendonly yes
   ```

3. 使用集群模式启动多个Redis服务节点

   ```shell
   $ docker run -d -v $PWD/redis.conf:/usr/local/etc/redis/redis.conf \
     --name redis1 --net redis_cluster \
     redis redis-server /usr/local/etc/redis/redis.conf
   ```

4. 创建Redis集群

   * 手动将运行的节点加入到集群（`cluster meet`），再讲16384个槽（slot）分配给每个节点，当16384个槽都有节点处理时，集群处于ok状态。

   * 使用命令一次性创建集群(redis 5及以上版本)：

     ```shell
     $ redis-cli --cluster create 172.18.0.2:6379 172.18.0.3:6379 172.18.0.3:6379 172.18.0.4:6379 172.18.0.5:6379 172.18.0.6:6379 172.18.0.7:6379 --cluster-replicas 1
     ```

   * Redis 3及4版本，使用如下命令创建集群：

     ```shell
     $ redis-trib.rb create replicas 1 172.18.0.2:6379 172.18.0.3:6379 172.18.0.3:6379 172.18.0.4:6379 172.18.0.5:6379 172.18.0.6:6379 172.18.0.7:6379
     ```

   

## 参考资料：

1. [redis-cluster-tutorial](https://redis.io/topics/cluster-tutorial)
2. [Creating Redis Cluster using Docker](https://medium.com/commencis/creating-redis-cluster-using-docker-67f65545796d)
3. [Redis Cluster - How to create a cluster without redis-trib.rb file](https://pingredis.blogspot.com/2016/09/redis-cluster-how-to-create-cluster.html)



