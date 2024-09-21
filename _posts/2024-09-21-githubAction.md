---
layout: single
title:  "CI/CD - Github Action사용해보기"
categories: ci/cd
---

![action](/images/action.png)

## CI/CD

Continuous Integration, Continuous Deployment. 즉, 테스트, 통합, 배포과정을 자동화하는 것

**왜 사용하는가?** 

새로운 기능을 추가할 때, 즉 코드가 수정되었을 때 브랜치에 merge하고 배포를 하는데, 배포를 할때마다 서버에 접속하여 새로운 코드를 올려줘야한다. 그걸 자동화하기위해 사용한다.

**구축툴** 

깃허브액션, 젠킨스 등등..이 있지만 깃허브 액션을 사용해보려고 한다.


## 전체흐름

#### 개발(Develop)

- 개발 단계는 개발자가 로컬 환경에서 코드를 작성하고 변경하는 단계이다. 이 단계에서는 주로 IDE를 사용하여 기능을 구현하고, 버전 관리 시스템(Git)을 통해 변경 사항을 관리한다.

- 주요 활동
  - 새로운 기능 구현
  - 버그 수정
  - 코드 리팩토링

#### 커밋(Commit)

- 커밋 단계는 개발자가 로컬에서 작성한 코드를 Git 리포지토리에 커밋하고 푸시하는 단계이다. 이 단계에서 GitHub Actions이 이벤트를 감지하여 워크플로우를 실행한다.

- 주요 활동
  - git add, git commit, git push 명령어를 통해 변경 사항을 리포지토리에 반영

#### 빌드(Build)

- 빌드 단계는 소스 코드를 컴파일하고 실행 가능한 바이너리나 패키지로 만드는 과정이다.

- 주요 활동
  - Maven, Gradle 등을 사용하여 프로젝트 빌드
  - Docker 이미지를 빌드

#### 테스트(Test)

- 테스트 단계는 빌드된 코드가 올바르게 동작하는지 검증하는 과정입니다. 단위 테스트, 통합 테스트, E2E 테스트 등이 포함된다.

- 주요 활동
  - 자동화된 테스트 실행
  - 테스트 결과 리포팅

#### 배포(Deploy)

- 배포 단계는 테스트를 통과한 코드를 배포하는 과정이다.

- 주요 활동
  - 빌드된 애플리케이션을 서버에 배포
  - Docker 이미지를 레지스트리에 푸시
  - 클라우드 서비스에 배포

## Github Action 기본문법

깃허브 액션을 사용하기 위해서는 yml파일이 필요하다. 이 파일은 항상 `/.github/workflows/example.yml` 경로에 있어야하며, 프로젝트의 최상단에 있어야 한다.

`example.yml` 파일에는 다음과 같은 내용들이 들어가고, 주석을 확인하면 각각의 기능을 알 수 있다.

```
#Workflow이름
name: action123

#master branch에 push될 때 아래 Workflow를 실행
on:
  push:
    branches:
      - master

jobs:
  #Job을 실행하기 위한 id
  My-Deploy-Job:
    #Github Action을 실행시킬 서버 종류 선택
    runs-on: ubuntu-latest

    steps:
      - name: hello world
          run: echo "hello world"
```

## EC2 배포

테스트용으로 controller를 단순하게 만든다.

```java
@RestController
public class AppController {

    @GetMapping("/")
    public String index() {
        return "Good World!";
    }
}
```

그리고 이 파일을 깃허브에 올린 뒤 배포를 해보겠다.

배포에 관하여는 따로 설명하지 않고, 스프링을 배포할 때 주로 사용되는 명령어만 정리해놓겠다

```
Sudo apt update
Sudo apt install openjdk-17-jdk -y
./gradlew clean build
/build/lib에 있는 jar파일 실행
Nohup java -jar ~~ &
Sudo lsof -i:8080으로 확인
```
이렇게 파일이 배포되었다고 가정하자.

여기서 코드가 업데이트되면? Git에서 파일을 pull하고, 프로세스를 종료하고, 바뀐 코드로 다시 빌드해야한다.

즉, 아래와 같은 명령어들을 코드가 바뀔때마다 입력해야한다.

```
git pull origin master
./gradlew clean build
sudo fuser -k -n tcp 8080 || true
nohup java -jar build/libs/*SNAPSHOT.jar &
```

하지만 이 과정은 매우 귀찮기 때문에 이 과정을 자동화하기 위하여 Github Action을 사용한다.

## Github Action 적용

사실 적용하는 법은 굉장히 간단하다.

위에 적어놓은 기본문법을 살짝 변형해주면된다.

```
#Workflow이름
name: Deploy To EC2

#master branch에 push될 때 아래 Workflow를 실행
on:
  push:
    branches:
      - master

jobs:
  My-Deploy-Job:
    #Github Action을 실행시킬 서버 종류 선택
    runs-on: ubuntu-latest

    steps:
      - name: SSH로 EC2 접속
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{secrets.EC2_HOST}}
          username: ${{secrets.EC2_USERNAME}}
          key: ${{secrets.EC2_PRIVATE_KEY}}
          script_stop: true
          script: |
            cd /home/ubuntu/github_action_basic
            git pull origin master
            ./gradlew clean build
            sudo fuser -k -n tcp 8080 || true
            nohup java -jar build/libs/*SNAPSHOT.jar > ./output.log 2>&1 &
```

여기서 참고할 부분은 step부분이다. 

먼저, 라이브러리를 사용하게 되는데 이 라이브러리는 ssh로 ec2에 접속하여 쉽게 작업을 할 수 있도록 해주는 라이브러리이다.

또한, secret을 사용하게 되는데, 이 부분은 레포지토리에서 setting - secrets and variables - action에서 새로운 secret을 만들어 관리할 수 있다.

마지막으로, script에서 우리가 반복적으로 작성해야한다던 명령어들을 입력하면 완성이다.

#### 이렇게 설정을 하게 된다면, 코드를 수정하여 올렸을 때 자동으로 수정된 코드를 바탕으로 다시 배포를 하게 된다.
