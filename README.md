# DockerSwarmRedis4Cluster
DockerSwarmRedis4Cluster

环境 3主机  docker swarm redis4 搭建redis 3主3从集群

// 应为随机分配节点 如果想要固定节点的话 加上label指定就可以
docker node update --label-add env=nginx worker1
$ docker node update --label-add foo=xxx --label-add bar=xxx worker1

docker service create --name my_nginx --constraint 'node.labels.env == nginx' nginx


# (1)先创建redis.conf 内容如下

port 6379
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes



# (2)创建redis容器服务 
注意有个坑 文件以及文件的权限需要放开 不然配置文件在挂载中读取不到
           还有就是target一定也要是user/local下面才行 不然貌似也没有权限

docker service create   
--name "redis-1"   
--constraint 'node.labels.env == redis'   
--network hbasesoft-network   
--mount type=bind,source=/home/liujie/redis/mount/,target=/usr/local/etc/redis/   
redis redis-server /usr/local/etc/redis/redis.conf



# (3)创建单机版的reids-boot 里面有ruby用来进行redis集群的脚本执行
docker service create \
--name redis-boot \
--mount type=bind,source=/home/liujie/redis/mount/,target=/usr/local/etc/redis/ \
--constraint 'node.labels.env1 == redis-boot'  \
--network hbasesoft-network ruby sh -c '\
  gem install redis \
  && wget http://download.redis.io/redis-stable/src/redis-trib.rb \
  && sleep 3600'




