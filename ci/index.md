# [k8s project] CI/CD Pipeline 구현 - Jenkins를 사용한 CI 구축


<!--more-->

![image](https://user-images.githubusercontent.com/82520143/170256707-d1f3fa65-5880-4ecb-95f5-accfe8fee26c.png)

# CI

- Continuous Integration
- 빌드 / 테스트 자동화
- 소스&버전관리 시스템(예시: git)에 대한 변경사항을 정기적으로 커밋 → 모든 사람에게 동일한 작업 기반 제공

> Jenkins에서 CI 파이프라인 구축  
> ![image](https://user-images.githubusercontent.com/82520143/171565286-06a9b55c-2b4e-48f1-a0d7-efc808a0b123.png)

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
7. Gradle & NodeJS 세팅
   - Jenkins페이지 → jenkins 관리 → \***\*Global Tool Configuration\*\***  
     ![image](https://user-images.githubusercontent.com/82520143/171557709-c4486eda-000c-4bed-b1ff-b135823bf732.png)  
     ![image](https://user-images.githubusercontent.com/82520143/171557752-5118f1bb-0d75-401f-aaf3-0cb0b567d7bb.png)

### 4. Servier project 생성

jenkinsfile을 생성하여 pipdline 프로젝트 생성

- 새로운 Item → pipeline 선택
- Springboot에서 사용한 환경변수들은 k8s secret을 통해 Deployment 부분에서 선언
- **jenkinsfile**

```groovy
pipeline {
    agent any
    environment {
				// 환경변수 설정
        dockerHubRegistry = 'plox/modoosugang-server'
        dockerHubRegistryCredential = '8181e4ed-46b2-4fd1-bfbf-1fa5ab5fc938'
      }
    stages {
				// 단계 1. github repo 가져오기
        stage('Checkout Application Git Branch') {
            steps {
								// 가져올 github repo와 branch를 설정하고, 여기에 권한을 가진 credential ID 입력
                git credentialsId: '4b1dcb95-da80-40e8-8a26-312fb71aed5a',
                    url: 'https://github.com/modooSugangOrg/modooSugang_BE.git',
                    branch: 'main'
            }
            post {
                    failure {
                      echo 'Repository clone failure !'
                    }
                    success {
                      echo 'Repository clone success !'
                    }
            }
        }
				// 단계 2. java build
        stage('Jar Build') {
            steps {
								// gradlew 실행권한 부여
                sh 'chmod +x gradlew'
								// clean 후 build
                sh './gradlew clean build'
            }
            post {
                    failure {
                      echo 'jar build failure !'
                    }
                    success {
                      echo 'jar build success !'
                    }
            }
        }
				//  단계 3. Docker Image Build
        stage('Docker Image Build') {
            steps {
                sh "docker build . -t plox/modoosugang-server:${BUILD_NUMBER}"
                sh "docker build . -t plox/modoosugang-server:latest"
            }
            post {
                    failure {
                      echo 'Docker image build failure !'
                    }
                    success {
                      echo 'Docker image build success !'
                    }
            }
        }
				// 단계 4. Docker Image Push
        stage('Docker Image Push') {
            steps {
								// DockerHub 관련 Credential ID
                withDockerRegistry([ credentialsId: '8181e4ed-46b2-4fd1-bfbf-1fa5ab5fc938', url: "" ]) {
                                    sh "docker push ${dockerHubRegistry}:${BUILD_NUMBER}"
                                    sh "docker push ${dockerHubRegistry}:latest"
                                }

            }
            post {
                    failure {
                      echo 'Docker Image Push failure !'
                      sh "docker rmi ${dockerHubRegistry}:${BUILD_NUMBER}"
                      sh "docker rmi ${dockerHubRegistry}:latest"
                    }
                    success {
                      echo 'Docker image push success !'
                      sh "docker rmi ${dockerHubRegistry}:${BUILD_NUMBER}"
                      sh "docker rmi ${dockerHubRegistry}:latest"
                    }
            }
        }
        // 단계 5. k8s manifest 파일 업데이트
				// docker image tag를 빌드 번호로 업데이트
        stage('K8S Manifest Update') {
            steps {
								// Manifest repo 가져오기
                git credentialsId: '4b1dcb95-da80-40e8-8a26-312fb71aed5a',
                    url: 'https://github.com/modooSugangOrg/modoosugang_BE_manifest.git',
                    branch: 'main'
                // 수정할 파일(kustomization.yaml) 경로에 들어가서 && 파일의 내용을 수정
								// sed 명령어 해석(뇌피셜)
								//      sed -i 's/{newTag로 시작하는 부분을}\$/{요걸로 바꾸겠습니다.}/g' {바꾸고 싶은 파일}
                sh "cd ./overlay/dev && sed -i 's/newTag:.*\$/newTag: \"${BUILD_NUMBER}\"/g' kustomization.yaml"
                sh "cat ./overlay/dev/kustomization.yaml"

								//  커밋 & 푸시 전 사용자 정보 입력
                sh 'git config --global user.name plooox'
                sh 'git config --global user.email insukim97@gmail.com'

								// credentialId의 Id와 Password 데이터를 'usernameVariable'과 'passVariable'로 할당
                withCredentials([usernamePassword(credentialsId: '4b1dcb95-da80-40e8-8a26-312fb71aed5a', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh "git pull origin main"   // 혹시 모를 충돌 대비 pull
                        sh "git add ./overlay/dev/kustomization.yaml"   // build파일도 같이 커밋되는 걸 막기 위해 수정한 파일만 추가
                        sh "git commit -m '[:alien: UPDATE] modoosugang_server ${BUILD_NUMBER} image versioning'"  // 커밋
                        sh "git push http://$user:$pass@github.com/modooSugangOrg/modoosugang_BE_manifest.git main"  // id & pw 를 통해 push
                    }
            }
            post {
                    failure {
                      echo 'K8S Manifest Update failure !'
                    }
                    success {
                      echo 'K8S Manifest Update success !'
                    }
            }
        }
    }
}
```

<details>
<summary> ⚠️  Credential ID 확인하는 법 </summary>
<div markdown="1">

![image](https://user-images.githubusercontent.com/82520143/171558639-333f74e0-fab5-452c-9225-efd71d262a3c.png)

</div>
</details>
  
### 5. Client project 생성
- react의 경우, 빌드된 파일만 Dockerize하기 때문에 필요한 환경변수 파일을 미리 넣어서 빌드해야함
    - react에서 사용되는 환경변수 파일들은 전부 `REACT_APP_`으로 시작 (REACT_APP_TEST_MESSAGE, REACT_APP_SERVER_URL등)
    - jenkinsfile에서 필요한 `.env`파일을 생성하여 환경변수를 처리할 수 있도록 하였음
- 사전 작업
    - Global Tool Configuration
        
        ![image](https://user-images.githubusercontent.com/82520143/171558992-3fbd9b95-0ac6-48d0-897a-96a4a77616a7.png)

    - 시스템 설정

        ![image](https://user-images.githubusercontent.com/82520143/171559004-18c27887-2566-41e0-adc2-7a99a9314252.png)

    - Jenkins에서 발생하는 CI 에러 방지

        ![image](https://user-images.githubusercontent.com/82520143/171559030-21c22d9f-4a74-4a26-a2ab-ef46b5064316.png)

- **jenkinsfile**

```groovy
pipeline {
    agent any
		// 빌드에 사용할 툴 ( Global Tool Configuration에서 선언한 Name을 토대로 {사용 Tool }:{Name})
    tools {
        nodejs "nodejs" // Global Tool Configuration에서 NodeJS 16.14.2를 "nodejs"로 선언했음
    }
    environment {
        dockerHubRegistry = 'plox/modoosugang-client'
        dockerHubRegistryCredential = '8181e4ed-46b2-4fd1-bfbf-1fa5ab5fc938'
      }
    stages {
        stage('Checkout Application Git Branch') {
            steps {
                git credentialsId: '4b1dcb95-da80-40e8-8a26-312fb71aed5a',
                    url: 'https://github.com/modooSugangOrg/modooSugang_FE.git',
                    branch: 'main'
            }
            post {
                    failure {
                      echo 'Repository clone failure !'
                    }
                    success {
                      echo 'Repository clone success !'
                    }
            }
        }
        stage('npm Build') {
            steps {
                // 필요한 환경변수들 -> .env 파일에 저장
                sh 'echo REACT_APP_SERVER_URL=${REACT_APP_SERVER_URL} > .env'
                sh 'echo REACT_APP_TEST_MESSAGE=${REACT_APP_TEST_MESSAGE} >> .env'

                sh 'npm install'
                sh 'npm run build'
            }
            post {
                    failure {
                      echo 'npm build failure !'
                    }
                    success {
                      echo 'npm build success !'
                    }
            }
        }
        stage('Docker Image Build') {
            steps {
                sh "docker build . -t ${dockerHubRegistry}:${BUILD_NUMBER}"
                sh "docker build . -t ${dockerHubRegistry}:latest"
            }
            post {
                    failure {
                      echo 'Docker image build failure !'
                    }
                    success {
                      echo 'Docker image build success !'
                    }
            }
        }
        stage('Docker Image Push') {
            steps {
                withDockerRegistry([ credentialsId: dockerHubRegistryCredential, url: "" ]) {
                                    sh "docker push ${dockerHubRegistry}:${BUILD_NUMBER}"
                                    sh "docker push ${dockerHubRegistry}:latest"
                                    sleep 10 /* Wait uploading */
                                }

            }
            post {
                    failure {
                      echo 'Docker Image Push failure !'
                      sh "docker rmi ${dockerHubRegistry}:${BUILD_NUMBER}"
                      sh "docker rmi ${dockerHubRegistry}:latest"
                    }
                    success {
                      echo 'Docker image push success !'
                      sh "docker rmi ${dockerHubRegistry}:${BUILD_NUMBER}"
                      sh "docker rmi ${dockerHubRegistry}:latest"
                    }
            }
        }

        stage('K8S Manifest Update') {
            steps {
                git credentialsId: '4b1dcb95-da80-40e8-8a26-312fb71aed5a',
                    url: 'https://github.com/modooSugangOrg/modoosugang_FE_manifest.git',
                    branch: 'main'

                sh "cd ./overlay/dev && sed -i 's/newTag:.*\$/newTag: \"${BUILD_NUMBER}\"/g' kustomization.yaml"

                sh 'git config --global user.name plooox'
                sh 'git config --global user.email insukim97@gmail.com'

                withCredentials([usernamePassword(credentialsId: '4b1dcb95-da80-40e8-8a26-312fb71aed5a', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh "git pull origin main"
                        sh "git add ./overlay/dev"
                        sh "git commit -m '[:alien: UPDATE] modoosugang_client ${BUILD_NUMBER} image versioning'"
                        sh "git push http://$user:$pass@github.com/modooSugangOrg/modoosugang_FE_manifest.git main"
                    }
            }
            post {
                    failure {
                      echo 'K8S Manifest Update failure !'
                    }
                    success {
                      echo 'K8S Manifest Update success !'
                    }
            }
        }
    }
}
```

### 6. github webhook 설정

- github에서 commit과 같은 이벤트가 발생시, HTTP POST Payload를 지정된 URL로 전송
- git repo가 업데이트 되면 자동으로 Jenkins 프로젝트가 실행

![image](https://user-images.githubusercontent.com/82520143/171559316-db70ed55-7069-407a-bfb3-c471c7bff736.png)  
![image](https://user-images.githubusercontent.com/82520143/171559353-ab65dc30-8813-4f48-a6fb-5cc8e0f09758.png)

## 결과

![image](https://user-images.githubusercontent.com/82520143/171559487-a986a175-b726-4d56-b90a-47364231f38d.png)
![image](https://user-images.githubusercontent.com/82520143/171559507-31e325b2-028f-4d1a-a85b-512073e5c41c.png)

