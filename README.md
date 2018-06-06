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
  
  
# (4)进入ruby中执行

  首先创建setup.sh
  
           REDIS_NUM=6
           list=""
           for i in $( seq 1 $REDIS_NUM )
           do
             addr=$(getent hosts "redis-$i" | awk '{ print $1 }')
             list="$list $addr:6379 "
           done

           ruby /redis-trib.rb create --replicas 1 $list
  
  
  
  然后redis-boot的具体名字
  
  进入ruby
  
  docker exec  -i -t redis-boot.1.s65ofkrfxdwoai50huhkb3xmu /bin/bash
  
  然后执行 ./setup.sh
  
  
           >>> Creating cluster
           >>> Performing hash slots allocation on 6 nodes...
           Using 3 masters:
           10.9.9.27:6379
           10.9.9.29:6379
           10.9.9.34:6379
           Adding replica 10.9.9.38:6379 to 10.9.9.27:6379
           Adding replica 10.9.9.41:6379 to 10.9.9.29:6379
           Adding replica 10.9.9.36:6379 to 10.9.9.34:6379
           M: edfd4791132cba2ee575b74cf310b13b58f36fb8 10.9.9.27:6379
              slots:0-5460 (5461 slots) master
           M: be8f84f67df2f5504461eb5e35a243158db09815 10.9.9.29:6379
              slots:5461-10922 (5462 slots) master
           M: 475cbf8478e542d6ac7fed197775c34f09ab5377 10.9.9.34:6379
              slots:10923-16383 (5461 slots) master
           S: 18a8aa4ab914cdcb42edd01eb7ca793381a86aa9 10.9.9.36:6379
              replicates 475cbf8478e542d6ac7fed197775c34f09ab5377
           S: 3a94b368a59cdfc47cbfb5906b618b2c759e490b 10.9.9.38:6379
              replicates edfd4791132cba2ee575b74cf310b13b58f36fb8
           S: 45a7bf45ea3880db87324520cb9c78053ae6d4a3 10.9.9.41:6379
              replicates be8f84f67df2f5504461eb5e35a243158db09815
           Can I set the above configuration? (type 'yes' to accept): 
  


参考 https://blog.newnius.com/setup-redis-cluster-based-on-docker-swarm.html

