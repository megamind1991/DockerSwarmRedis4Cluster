# DockerSwarmRedis4Cluster
DockerSwarmRedis4Cluster

环境 3主机  docker swarm redis4 搭建redis 3主3从集群

先创建redis.conf 内容如下

port 6379
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes



创建redis容器服务 注意有个坑 文件以及文件的权限需要放开 不然配置文件在挂载中读取不到

docker service create   
--name "redis-1"   
--constraint 'node.labels.env == redis'   
--network hbasesoft-network   
--mount type=bind,source=/home/liujie/redis/mount/,target=/usr/local/etc/redis/   
redis redis-server /usr/local/etc/redis/redis.conf




