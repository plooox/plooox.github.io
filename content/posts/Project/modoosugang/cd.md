---
title: "[k8s project] CI/CD Pipeline 구현 - ArgoCD를 사용한 CD 구축"
date: 2022-06-02T15:05:23+09:00
description: ""
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

# CD

- Continuous Deployment
- 변경 사항을 자동으로 배포/릴리스
  [참고자료](https://cwal.tistory.com/22)

> ArgoCD를 사용하여 k8s cluster 배포 자동화  
> ![image](https://user-images.githubusercontent.com/82520143/171565872-c2fd5372-0e82-425d-b825-30b6f5969811.png)

- kustomization 파일을 통해 쿠버네티스 오브젝트를 사용자가 원하는 대로 변경하는(customize) 독립형 도구

### 설치

```bash
curl -s "https://raw.githubusercontent.com/\
kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
sudo mv kustomize /usr/local/bin
kustomize version
```

### 사용법

1.  파일 구성  
    ![image](https://user-images.githubusercontent.com/82520143/171564048-22bf638d-b030-4a49-9d11-6dd1794ad3c9.png)
2.  `/base`

    1. `/base/back-test-deployment.yaml`

       ```bash
       apiVersion: apps/v1
       kind: Deployment
       metadata:
         name: api-server
       spec:
         replicas: 3
         selector:
           matchLabels:
             app: api-server
         template:
           metadata:
             name: api-server-pod
             labels:
               app: api-server
           spec:
             containers:
             - name: api-server
               image: api-server-image
               ports:
               - containerPort: 8080
       ```

    2. `/base/kustomization.yaml`

       ```bash
       apiVersion: kustomize.config.k8s.io/v1beta1
       kind: Kustomization
       resources:
       - deployment.yaml
       ```

3.  `/overlay`

    1. `/overlay/kustomization.yaml`

       ```bash
       apiVersion: apps/v1
       kind: Deployment
       metadata:
         name: api-server
       spec:
         containers:
         - name: api-server-image
       ```

    2. `/overlay/back-test-deployment-patch.yaml`

       ```bash
       apiVersion: kustomize.config.k8s.io/v1beta1
       kind: Kustomization
       patchesStrategicMerge:
       - back-test-deployment-patch.yaml
       resources:
       - ../../base
       images:
       - name: api-server-image
         newName: plox/modoosugang_server
         newTag: "7"
       ```

4.  kustomize시 base와 overlay/dev 내용이 merge되면서 manifest파일이 생성됨

    ````bash
    $ kustomize build overlay/dev

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: api-server
        spec:
          containers:
          - name: api-server-image
          minReadySeconds: 5
          replicas: 2
          selector:
            matchLabels:
              app: api-server
          template:
            metadata:
              labels:
                app: api-server
              name: api-server-pod
            spec:
              containers:
                image: **plox/modoosugang_server:7   # < - ./overlay/dev/kustomization.yaml 파일 내용 merge**
                name: api-server
                ports:
                - containerPort: 8080
        ```

    Kustomize에 대한 자세한 설명은 [이곳](https://velog.io/@pullee/Kustomize%EB%A1%9C-K8S-%EB%A6%AC%EC%86%8C%EC%8A%A4-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0)에 자세히 설명한 글이 있어 참고하였음.
    ````

## ARGO CD

1. Github repo에 Kustomize위한 계층 구조 생성  
   ![image](https://user-images.githubusercontent.com/82520143/171564634-9be9065a-c9b7-4b8f-9042-abb371c67f9b.png)
2. manifest 파일 자동 업데이트 설정
   - Jenkinsfile을 통해 구현되었음[(링크)](https://plooox.github.io/ci/)
3. Argo CD에서 Create App  
   ![image](https://user-images.githubusercontent.com/82520143/171566029-7d7a10df-3476-4754-ae26-027f0520f20b.png)
   ![image](https://user-images.githubusercontent.com/82520143/171566042-78b1f867-40e4-4911-8833-7111c83b5604.png)
   ![image](https://user-images.githubusercontent.com/82520143/171566062-6d3b8296-7a14-4749-98b6-ff9b71929e72.png)
   ![image](https://user-images.githubusercontent.com/82520143/171566073-8facdfe7-d86e-4f3f-adc6-6cfbe1019bc3.png)

## 배포 결과

![image](https://user-images.githubusercontent.com/82520143/171566201-47a5f76c-e219-4fa3-a6f2-247a425a5a40.png)
![image](https://user-images.githubusercontent.com/82520143/171566212-9f45eeca-cbe0-45d8-8999-81f6a9da2d57.png)
![image](https://user-images.githubusercontent.com/82520143/171566224-0119738b-535d-4225-9def-42a1eb68d7f0.png)
