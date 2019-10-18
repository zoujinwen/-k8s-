
#### （一）实例springboot


* idea配置springboot


> 修改SpringbootApplication

``` java
package com.idig8.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@SpringBootApplication
@Configuration
public class SpringbootApplication {
    @RequestMapping("hello")
    @ResponseBody
    public String hello(){
        return "hello world！";
    }


    public static void main(String[] args) {
        SpringApplication.run(SpringbootApplication.class, args);
    }

}

```

![](https://upload-images.jianshu.io/upload_images/11223715-71c2df4895d49a5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* pom.xml文件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.idig8</groupId>
    <artifactId>springboot</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <!-- 指定启动类 -->
        <start-class>com.idig8.springboot.SpringbootApplication</start-class>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

>

#### （二）多阶段构建


* ②上传项目

``` bash
yum -y install lrzsz
yum -y install zip unzip
cd ~
mkdir test2
rz
ll
```



* ③解压项目

``` bash
unzip springboot.zip
ls
rm -rf springboot.zip
```


* ④编写多构建Dockerfile

``` bash
cd springboot
vi Dockerfile
```


``` dockerfile
FROM maven:3.6.1-jdk-8 AS build-env
ADD . /java/src/app
WORKDIR /java/src/app
RUN mvn clean package -Ptest -Dmaven.test.skip=true
RUN cp /java/src/app/target/*.jar /java/src/app/app.jar


FROM openjdk:8-jre
ENV SPRING_OUTPUT_ANSI_ENABLED=ALWAYS \
    JAVA_OPTS=""
COPY --from=build-env /java/src/app/app.jar  /usr/local/bin/app-server/app.jar
CMD echo "The application will start " && \
    java ${JAVA_OPTS} -Djava.security.egd=file:/dev/./urandom -jar /usr/local/bin/app-server/app.jar
EXPOSE 8080
```


* ⑤构建Dockerfile

``` bash
docker build -t zhugeaming/docker-multi-java-demo:latest .
```


* ⑥查看效果 

```  bash
mvn  -s "settingsXXX.xml" clean package -Ptest -Dmaven.test.skip=true
```
