# <p align="center"> AWS-EKS-deploy

### AWS EKS 를 사용하여 Spring Boot 앱 배포하기

## 사용기술
- AWS EKS, ECR
- Docker
- Spring Boot
<br>

## 프로젝트 목적 🌷

AWS, Kubernetes 기반의 자동화된 애플리케이션 배포 
<br>

## 실습 개요 :star:

- step 01 : Docker로 AWS ECR에 SpringBoot 이미지 push 하기
- step 02 : AWS CLI로 EKS 설치하기
- step 03 : yaml 파일 생성 후 apply 하기
- step 04 : EKS 삭제하기

<br>

## 실습 과정 :mag_right:

### ☑️ step 01 : Docker로 AWS ECR에 이미지 push 하기
#### 1-1. Docker 설치하고 hub에서 이미지 가져오기
```
sudo apt install -y docker.io
sudo docker -v
docker login
docker pull suzyhw96/springapp:1.0
docker images
```

#### 1-2. ECR 
- AWS 콘솔 ECR 메뉴에서 프라이빗 리포지토리 생성
![image](https://github.com/user-attachments/assets/a8571e20-d956-4e09-b1bc-8ef56c4086e3)

- 생성 후 푸시 명령 보기 참고
![image](https://github.com/user-attachments/assets/e569f9df-0706-46b1-b3f4-c1e27fc22b8e)

```
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 646580111040.dkr.ecr.ap-northeast-2.amazonaws.com

docker tag suzyhw96/springapp:1.0 646580111040.dkr.ecr.ap-northeast-2.amazonaws.com/ce17-spring:latest

docker push 646580111040.dkr.ecr.ap-northeast-2.amazonaws.com/ce17-spring:latest

```


### ☑️ step 02 : AWS CLI로 EKS 설치하기
- EKS 설치
```
eksctl create cluster --name ce17-eks --version 1.30 --nodes=1 --node-type=t2.small --region ap-northeast-2
aws eks --region ap-northeast-2 update-kubeconfig --name ce17-eks
```

### ☑️ step 03 : yaml 파일 생성 후 apply 하기
- k8s.yaml 생성
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: springboot-app
  template:
    metadata:
      labels:
        app: springboot-app
    spec:
      containers:
        - name: springboot-app
          image: 646580111040.dkr.ecr.ap-northeast-2.amazonaws.com/ce17-spring:latest
          ports:
            - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: springboot-app-service
spec:
  selector:
    app: springboot-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8999
  type: LoadBalancer

```

- apply하기

```
kubectl apply -f k8s.yaml

kubectl get svc
kubectl get deployment
kubectl get pods
```
![image](https://github.com/user-attachments/assets/207f2718-6f7e-439f-9a07-b8644c0beb91)


- external ip 접속해 확인하기
![image](https://github.com/user-attachments/assets/5cd41be4-25e3-4c8c-bf89-f6584cff6989)


### ☑️ step 04 : EKS 삭제하기

```
eksctl delete cluster --name myeks --region ap-northeast-2
```


## 트러블슈팅 📝
### 01. ECR 로그인
- 기존 : chatgpt 이용했으나 로그인 오류 발생
- 변경 : ECR 콘솔 메뉴얼대로 진행

### 02. EKS 설치 설정 확인
- 기존 
```
-- 설정 없이 설치하여 EC2 리소스 및 비용 낭비
eksctl create cluster --name ce17-myeks --version 1.26 --region ap-northeast-2
```

- 변경 : node type 설정하여 설치
```
eksctl create cluster --name ce17-elk --version 1.30 --nodes=1 --node-type=t2.small --region ap-northeast-2
```

### 03.yaml 파일 targetport 지정 오류
- 기존 : ECR springboot 이미지 내부 포트 8999 이나, yaml 파일 targetport 8080으로 설정함
![image](https://github.com/user-attachments/assets/a0166fd0-b982-4f24-9800-cc0fbb612f5d)

- 변경 : yaml 파일 target port 변경 후 external ip 접속 정상 실행 
