
#### （一）数据卷


* ①运行redis容器

``` bash
docker run --name some-redis -d -p 6379:6379 redis 
docker volume ls
```

* ②查看redis容器描述，找到对应的volume的ID

``` bash
docker inspect some-redis
```


``` bash
docker volume inspect ID
```

* ③再次运行redis容器

``` bash
docker run --name some-redis -d -p 6380:6379 redis 
```


* ④删除一个容器，看看volume会不会变化

* ⑤删除数据卷

``` bash
docker volume rm <volumeID>
```


* ⑥ 数据卷的名字是ID真的不太友好，换个方式


``` bash
docker volume create redis_volume
docker run --name some-redis3 -d -p 6380:6379 -v redis_volume:/usr/local/etc/redis/redis.conf redis 
docker volume inspect redis_volume
```

* ⑦ 数据卷的概念


#### （二）主机目录

* ①演示主机目录

``` bash
docker run --name some-redis4 -d -p 6381:6379 -v $(pwd):/usr/local/etc/redis/redis.conf redis 
docker inspect some-redis4
```

