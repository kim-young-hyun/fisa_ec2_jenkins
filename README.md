<<<<<<< HEAD
## README
=======
# EC2에서 Jenkins로 build하기

**jenkins를 이용해 AWS EC2에서 자동으로 build 되도록 구현한다.**

1. 프로젝트는 spring boot, gradle을 이용해 생성
2. github에 변경사항이 있으면 자동으로 build

## 1. EC2생성

### 환경 구현

build를 위한 JDK 설치(openjdk-17), docker 설치, jenkins 구동

```
$ sudo apt-get update
$ sudo apt install openjdk-17-jdk-headless
$ sudo apt install -y docker.io
$ docker run --name myjenkins --privileged -p 8080:8080 jenkins/jenkins:lts-jdk17
```

### 보안 그룹 설정

TCP 8080에 대해서 외부 접근을 가능하도록 설정

TCP 8080, 0.0.0.0/0을 인바운드 규칙에 추가

## 2. Jenkins 설치, 설정

### 로그인

jenkins 비밀번호 확인

```
$ docker exec myjenkins sh -c 'cat /var/jenkins_home/secrets/initialAdminPassword' 
```

http://< EC2 퍼블릭 IPv4 주소 > : 8080으로 접속

## 파이프라인 설정

GitHub project 체크: Project url에 github repo주소 입력

GitHub hook trigger for GITScm polling 체크: github에서 변경이 생기면 webhook으로 알려줌

파이프라인 스크립트

```shell
pipeline {
    agent any
    stages {
        stage('git pull') {
            steps {
                echo 'pull start'
                git branch: '-----', credentialsId: '---------', url: '---------'
                echo 'pull end'
            }
        }
        stage('build') {
            steps {
                echo 'build start'
                sh './gradlew build'
                echo 'build end'
            }
        }
    }
}
```

## 3. github 설정

### webhook 생성

Settings -> Webhooks -> add webhook

Payload URL: http://< EC2 퍼블릭 IPv4 주소 >:8080/github-webhook/

Content type: application/json

### gradlew 권한 변경

윈도우에서는 644로 생성되니 실행이 가능하도록 실행권한을 추가

```
git update-index --chmod=+x gradlew
```

## 4. swap file 생성

메모리가 부족해 실행중 무한 로딩이 걸린다면 swap file을 생성해 해결한다. (https://repost.aws/ko/knowledge-center/ec2-memory-swap-file)

```
$ sudo dd if=/dev/zero of=/swapfile bs=128M count=32
$ sudo chmod 600 /swapfile
$ sudo mkswap /swapfile
$ sudo swapon /swapfile
$ sudo swapon -s
```

/etc/fstab 파일 끝에 "/swapfile swap swap defaults 0 0"을 추가

```
$ sudo vi /etc/fstab

/swapfile swap swap defaults 0 0
```

>>>>>>> 664bd9e (Update README.md)
