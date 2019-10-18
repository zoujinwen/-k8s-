

#### （一）Docker Compose 



* ②由来




* ③安装


``` bash
sudo yum -y install python-pip
sudo pip install docker-compose
```


* ④查看版本

``` bash
docker-compose -version
```

* ⑤卸载

``` bash
sudo pip uninstall docker-compose
```

* ⑥测试docker-compose

> docker-compose.yml 编写

``` bash
version: '3'
services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: root
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     volumes:
        - wp_site:/var/www/html
     ports:
       - "80:80"
       - "443:443"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
    db_data:
    wp_site:
```


* ⑦以守护进程模式运行加-d选项

``` bash
docker-compose up -d
```



* ⑧查看某个service的compose日志

``` bash
#docker-compose logs <service名称>
docker-compose logs db
```


* ⑨停止compose服务

``` bash
#docker-compose.yml 目录下执行
docker-compose stop
```

* ⑩启动compose服务

``` bash
#docker-compose.yml 目录下执行
docker-compose start
```

* ⑪重启compose服务

``` bash
docker-compose restart
```


* ⑫ kill compose服务

> kill的服务状态码是Exit137
``` bash
docker-compose kill
```



* ⑬删除compose服务

``` bash
docker-compose rm
```



``` bash
docker-compose build [options] [SERVICE...]
```

 
* ②config
> 验证 Compose 文件格式是否正确，若正确则显示配置，若格式错误显示错误原因。

``` bash
#校验当前文件夹下的docker-compose.yml
docker-compose config
```


* ③down

``` bash
#校验当前文件夹下的docker-compose.yml
docker-compose down
```

 

* ④exec


``` bash
docker-compose exec <service> /bin/sh
```


* ⑤help



``` bash
docker-compose 命令 help
```

* ⑥images


``` bash
docker-compose images
```

* ⑦pause


``` bash
docker-compose pause [SERVICE...]
```



* ⑧unpause


``` bash
docker-compose unpause [SERVICE...]
```



* ⑨port

``` bash
docker-compose port [options] SERVICE PRIVATE_PORT
```


* ⑩pull


``` bash
docker-compose pull [options] [SERVICE...]
```

 

* ⑪push

``` bash
docker-compose push 
```



* ⑫run

``` bash
docker-compose run wordpress echo "2222222"
```



*  ⑬scale


``` bash
# 将启动 3 个容器运行 db 服务，2 个容器运行 db 服务。因为端口占用启动不了那么多，但是这样是可行的。
 docker-compose scale db=3 wordpress=2
```
