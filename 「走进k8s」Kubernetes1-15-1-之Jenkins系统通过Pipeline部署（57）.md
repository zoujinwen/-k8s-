
#### （一）Jenkins Pipeline 介绍


* ① 介绍


* ② 核心概念

>  填写脚本

``` pipline
node {
  stage('Clone') {
    echo "1.Clone Stage"
  }
  stage('Test') {
    echo "2.Test Stage"
  }
  stage('Build') {
    echo "3.Build Stage"
  }
  stage('Deploy') {
    echo "4. Deploy Stage"
  }
}
```

``` pipeline
node('jenkins-slave') {
    stage('Clone') {
      echo "1.Clone Stage"
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
      echo "3.Build Stage"
    }
    stage('Deploy') {
      echo "4. Deploy Stage"
    }
}
```
``` dockerfile
FROM java:openjdk-8
MAINTAINER liming 

COPY target/onlineshopping-1.0-SNAPSHOT.jar /onlineshopping.jar

ENTRYPOINT ["java","-jar","/onlineshopping.jar"]
```

* ⑦ 准备k8s 部署文件

``` yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jenkins-demo
spec:
  template:
    metadata:
      labels:
        app: jenkins-demo
    spec:
      containers:
      - image: zhugeaming/jenkins-demo:<BUILD_TAG>
        imagePullPolicy: IfNotPresent
        name: jenkins-demo
        env:
        - name: branch
          value: <BRANCH_NAME>
```


* ⑧ 编写pipeline

``` pipeline
node() {
def workspace = pwd()
def remote = [:]
remote.name = "master"
remote.host = "192.168.86.100"
remote.allowAnyHosts = true
  
 stage('Clone') {
    echo "1.Clone Stage"
    deleteDir()
    git url: "https://github.com/limingios/k8s-jenkins.git"
    script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        }
    }
  stage('Test') {
        echo "2.Test Stage"
        withMaven(maven: 'maven3.6.2'){
            sh 'mvn  -U clean install -Dmvn.test.skip=true -Ptest'
        }
    
  }
  stage('Build') {
        echo "3.Build Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'ssh', passwordVariable: 'sshPassword', usernameVariable: 'sshUserName')]) {
            remote.user = "${sshUserName}"
            remote.password = "${sshPassword}"
            sshCommand remote: remote, command: "cd /data/k8s/jenkins2/workspace/pipline-test/ && docker build -t zhugeaming/jenkins-demo:${build_tag} ."
    
        }
    }
  stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            sshCommand remote: remote, command: "docker login -u ${dockerHubUser} -p ${dockerHubPassword}"
            sshCommand remote: remote, command: "docker push zhugeaming/jenkins-demo:${build_tag}"
        }
    }
    
   stage('Deploy') {
        echo "5. Deploy Stage"
        def userInput = input(
            id: 'userInput',
            message: 'Choose a deploy environment',
            parameters: [
                [
                    $class: 'ChoiceParameterDefinition',
                    choices: "Dev\nQA\nProd",
                    name: 'Env'
                ]
            ]
        )
        echo "This is a deploy step to ${userInput}"
        sshCommand remote: remote, command: "cd /data/k8s/jenkins2/workspace/pipline-test/ && sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
        sshCommand remote: remote, command: "cd /data/k8s/jenkins2/workspace/pipline-test/ && sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
        if (userInput == "Dev") {
            // deploy DEV操作
        } else if (userInput == "QA"){
            // deploy QA操作
        } else {
            // deploy prod操作
        }
        sshCommand remote: remote, command: "cd /data/k8s/jenkins2/workspace/pipline-test/ && kubectl apply -f k8s.yaml"
    } 
}
```


* ⑨ 效果查看

``` bash
kubectl get deployment

kubectl get pod

kubectl logs -f jenkins-demo-86bcf9d7bc-zm6p7
```







