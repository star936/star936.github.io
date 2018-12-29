---
title: 使用Jenkins MultiBranch实现Docker build镜像并push到DockerHub
date: 2018-12-30 00:05:28
tags: [Jenkins]
categories: Spring
toc: true
---


> 使用Docker部署Jenkins，并通过MultiBranch功能自动对项目进行打包并docker build和push。



#### 1. Docker部署Jenkins

**拉取镜像**

```bash
docker pull jenkinsci/jenkins:2.150.1
```

**启动**

```bash
docker run --name jenkins -d -p 8080:8080 -p 50000:50000 -v ~/workspace/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v /usr/local/bin/docker:/bin/docker -t jenkinsci/jenkins:2.150.1
```

*其中：`-v /var/run/docker.sock:/var/run/docker.sock -v /usr/local/bin/docker:/bin/docker`作用是使用主机的docker命令。否则可能Jenkins创建docker镜像时会报`docker not found`错误。*

**修改容器中docker权限**

```bash
# 进入Jenkins容器内
docker exec -it -u root 5c8 bash
# 修改docker权限
chmod 777 /var/run/docker.sock
```

*如果不修改容器中docker权限，则Jenkins进行到docker build时会报`dial unix /var/run/docker.sock: connect: permission denied`错误。*

**Jenkins插件安装和创建multibranch job请自行完成。**

#### 2. 编写Dockerfile

```dockerfile
FROM openjdk:8-jdk-alpine3.7
VOLUME /tmp

ARG JAR_FILE
ADD target/${JAR_FILE} target/app.jar

EXPOSE 8080

RUN touch target/app.jar
ENTRYPOINT ["java","-jar","target/app.jar"]
```

#### 3. 编写Jenkinsfile

```groovy
node {
    def app
    def mvnHome = tool 'maven'
    env.PATH = "${mvnHome}/bin:${env.PATH}"
    def IMAGE_NAME = "star936/rest-api-mybatis"

    stage('Package') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
        sh 'mvn -v'
        sh 'mvn clean package -Dmaven.test.skip=true -Ddockerfile.skip=true -P dev'
    }

    stage('Build and Push image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        echo env.BRANCH_NAME
        echo env.WORKSPACE
        sh 'docker version'
        def JAR_FILE = IMAGE_NAME + '.jar'
        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
            app = docker.build("${env.IMAGE_NAME}:${BRANCH_NAME}", "--build-arg JAR_FILE=${JAR_FILE} .")
            app.push()
        }
    }
}
```

**`tool 'maven'`中的`maven`是在`Jenkins->系统管理->全局工具配置->Maven->Add Maven`处填写的名字.**

**说明：`Dockerfile`和`Jenkinsfile`中出现的`JAR_FILE`参数，与`pom.xml`的`JAR_FILE`相互应。**

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>${dockerfile-maven-version}</version>
    <configuration>
        <useMavenSettingsForAuth>true</useMavenSettingsForAuth>
        <repository>${docker.image.prefix}/${docker.repository.name}</repository>
        <tag>${project.version}</tag>
        <buildArgs>
            <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

#### 4. 配置GitHub Webhooks

**想要实现git push后Jenkins自动构建变化的分支,则还需要配置`GitHub Webhooks`: 在项目的`settings->Webhooks->Add webhook->Payload URL`处填写`http://{jenkins服务器IP}/git/notifyCommit?url={MultiBranch添加repository源时填写的URL}&delay=0sec`**

*对于在本地搭建Jenkins的,推荐使用[ngrok](https://ngrok.com/).*

