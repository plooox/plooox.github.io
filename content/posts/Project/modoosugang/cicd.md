---
title: "[k8s project] CI/CD Pipeline 구현 - Jenkins를 사용한 CI 구축"
date: 2022-05-25T20:55:02+09:00
description: "k8s 전문가 양성 과정 중 진행하게된 프로젝트"
draft: true
tags: ["k8s", "AWS", "jenkins", "argocd"]
categories: ["project"]

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: false
lightgallery: true
fraction: true

toc:
  enable: true
  auto: true
code:
  copy: true
  # ...
math:
  enable: true
  # ...
comment:
  enable: true
---

<!--more-->

![image](https://user-images.githubusercontent.com/82520143/170256707-d1f3fa65-5880-4ecb-95f5-accfe8fee26c.png)

# CI

- Continuous Integration
- 빌드 / 테스트 자동화
- 소스&버전관리 시스템(예시: git)에 대한 변경사항을 정기적으로 커밋 → 모든 사람에게 동일한 작업 기반 제공

> Jenkins에서 CI 파이프라인 구축
> ![image](https://user-images.githubusercontent.com/82520143/170256783-a0421797-65d2-408a-a578-5f462250553d.png)

- EC2 인스턴스 안에서 Docker를 통해 Jenkins 컨테이너 생성 (java나 이런저런 설정의 간편화)
- Jenkins 인스턴스에 Elastic IP 할당 → 나중에 webhook등 설정할 때 값 고정하기 위해

## Step By Step

### 0. EC2 Instance에 Docker 설치

```bash
# SSH로 EC2 Instance 접속
# .pem파일 있는 경로에서
ssh -i ./[FILE_NAME].pem ubuntu@[IP Address]

# docker 설치
sudo apt-get update
sudo apt-get install docker -y
# docker 서비스 시작
sudo service docker start
# docker 서비스 상태 확인
systemctl status docker.service
```

### 1. Jenkins Container 생성 및 실행

Jenkins Container에도 Docker를 설치

```bash
# Dockerfile 생성
sudo vim Dockerfile
---------------------------------
# Dockerfile
# Dockerfile 작성 시작
FROM jenkins/jenkins:jdk11

#도커를 실행하기 위한 root 계정으로 전환
USER root

#도커 설치
COPY docker_install.sh /docker_install.sh
RUN chmod +x /docker_install.sh
RUN /docker_install.sh

# 도커 그룹에 사용자 추가
RUN usermod -aG docker jenkins
USER jenkins
```

```bash
-------------------------------------
# docker_install.sh 파일 생성
sudo vim docker_install.sh
--------------------------------------
# docker_install.sh 파일 작성 시작
#!/bin/sh
apt-get update && \
apt-get -y install apt-transport-https \
  ca-certificates \
  curl \
  gnupg2 \
  zip \
  unzip \
  software-properties-common && \
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
$(lsb_release -cs) \
stable" && \
apt-get update && \
apt-get -y install docker-ce
-----------------------------------------
```

```bash
# 권한 설정
sudo chmod 666 /var/run/docker.sock

# 이미지 생성
docker build -t jenkins .

# jenkins 폴더 만들기
mkdir jenkins

# 해당 폴더에 대해 권한 부여하기
sudo chown -R 1000 ./jenkins

# 컨테이너 생성
sudo docker run -d --name jenkins \
-v /home/ec2-user/jenkins:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
-p 8080:8080 \
-e TZ=Asia/Seoul \
jenkins
```

[참고자료 1](https://velog.io/@haeny01/AWS-Jenkins%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Docker-x-SpringBoot-CICD-%EA%B5%AC%EC%B6%95)

### 2. 프로젝트에 Dockerfile 만들기

- Springboot Project

```
FROM openjdk:11-jdk
EXPOSE 8081
ARG JAR_FILE=build/libs/modoosugang_be-0.0.1-SNAPSHOT.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

- React Project

```
FROM nginx:latest
RUN mkdir /app
WORKDIR /app
RUN mkdir ./build
ADD ./build ./build
RUN rm /etc/nginx/conf.d/default.conf
COPY ./nginx.conf /etc/nginx/conf.d
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 3. Jenkins 접속 후 세팅

1. 암호입력
   <img width="507" alt="image" src="https://user-images.githubusercontent.com/82520143/170260153-cf651542-acbb-408c-9414-62fd33c807bc.png">

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

2. Install suggested plugins
3. Jenkins페이지 → jenkins 관리 → System Configuration → 플러그인 관리
   1. gradle
   2. github integration
   3. post build task
   4. publish over ssh
   5. Docker pipeline
   6. NodeJS
4. Jenkins페이지 → jenkins 관리 → Security → Manage Credentials
   - Github 계정 등록
   - DockerHuB 계정 등록
5. Credential 추가
   ❗이부분에서 잘 설정해줘야 에러가 적게남: pipeline script 에러 1순위(경험상)
   ![image](https://user-images.githubusercontent.com/82520143/170260705-2eabe301-218a-46b3-a5be-c06cd75e1bad.png)
   ![image](https://user-images.githubusercontent.com/82520143/170260849-fb3647ee-aad9-4605-b886-0fe478331f1b.png)

   1. Github Credential 추가
      ![image](https://user-images.githubusercontent.com/82520143/170261036-f2b8cd60-73f9-45d1-be5f-4a0498723fc0.png)

6. 필요에 따라 환경변수 선언
   Jenkins 관리 -> 시스템 설정 -> Global properties
