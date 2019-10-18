

#### gitlab-ci如何使用

* ① 根目录添加.gitlab-ci.yml

``` bash
stages:
  - test
  - build
  - deploy
  
job1:
  stage: test
  tags:
    - test1
  script:
    - echo "--------job1----------"
job2:
  stage: build
  tags:
    - test1
  script:
    - echo "--------job2----------"
job3:
  stage: deploy
  tags:
    - test1
  script:

    - echo "--------job3----------"
```


``` bash
#原来是域名的问题，开始修改。
Running with gitlab-ci-multi-runner 9.5.1 (96b34cc)  on gitlab-ci (4d12d67f)
Using Shell executor...Running on gitlab-ci...
Cloning repository...Cloning into '/home/gitlab-runner/builds/4d12d67f/0/root/test1'...
fatal: unable to access 'http://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@gitlab.example.com/root/test1.git/': 
Could not resolve host: gitlab.example.com; Unknown errorERROR: Job failed: exit status 1
```

* 进入gitlab-ci主机
>修改hosts文件里面添加gitlab.example.com对应的ip。
```
sudo vi /etc/hosts 
# 添加 172.28.128.3 gitlab.example.com
```



