


#### （一）Docker Swarm介绍




#### （二）集群演示
* ①主机信息


* ② manager节点初始化操作（192.168.66.100）


``` bash
docker swarm init --advertise-addr 192.168.66.100
```


* ③ 添加worker节点（192.168.66.101）



``` bash
docker swarm join --token SWMTKN-1-3gfv7tpeznhwsl7v3y0n9f5g7547lgzo7fjpv0pm5s6uzvdlgg-b0mlie5vhp2ms1xg1tyd7zwc2 192.168.66.100:2377

```


* ④ 添加worker节点（192.168.66.102）


``` bash
docker swarm join --token SWMTKN-1-3gfv7tpeznhwsl7v3y0n9f5g7547lgzo7fjpv0pm5s6uzvdlgg-b0mlie5vhp2ms1xg1tyd7zwc2 192.168.66.100:2377

```


* ⑤ manager查看节点

``` bash
docker node ls
```

* ⑥ 创建service服务



``` bash
docker service create --replicas 3 -p 80:80 --name nginx nginx 
docker service ls

docker service ps
```

* ⑥ 删除service服务

``` bash
docker service rm nginx
```


#### （二）docker swarm 运行docker-compose文件
* ①stack  


* ②测试docker-compose文件

``` bash
mkdir labs
cd labs
vi docker-compose.yml
```


``` yml
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```


* ③运行 docker-compose.yml

``` bash
docker stack deploy example --compose-file=docker-compose.yml
docker stack ls
docker stack services example
```





```
docker service scale example_vote=4

```


*   删除stack

```
docker stack rm example

```

