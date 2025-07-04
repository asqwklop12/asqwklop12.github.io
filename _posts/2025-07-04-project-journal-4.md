---
layout: post
title: "[프로젝트 일지 #3-2] CI/CD"
date: 2025-07-04 11:57:55 +0900
categories: 
  - Project
  - CI/CD
tags:
  - 프로젝트
  - CI/CD
---


## 🎯 배경
저번에 GIT을 사용하면 어떤 이점이 있는지 생각해보았다. 이제 CI/CD툴차례인거 같다. 그전에도 말헀던거 같은데 CI/CD툴은 배포지식이 없는 사람들도 배포가 가능하게 도와준다. 하지만 모든 인원들이 배포지식이 없지는 않을거 같다. 최상위 결정자 혹은 미들 계층인 사람들은 이에 대한 지식이 필요하다. 그래야 STEP BY STEP으로 생각할 수 있기 때문이다. 나는 올해로 4년차 개발자다. 그렇기 때문에 이 부분도 고민을 해봐야한다.이번 장에서는 CI가 뭔지 CD가 뭔지에 대해 자세하게 
학습하지는 않을 예정이다. 그 부분은 추후에 진행할 수 있을거 같기때문이다. 그래서 이번 장에서는 CI/CD툴에 대한 종류를 학습해보고 어떤것이 있는지 확인해보자.
(왜 외부 GIT 종류는 안했냐하면 걔네는 비슷비슷하다고 생각하기 때문이다.)

## 종류
내가 아는 CI/CD툴부터 학습을 하자. 
괴장히 많은 툴들이 있긴하지만 걔네를 전부 알필요는 없을 거 같구 나는 백엔드지 인프라가 아니기떄문에 기술 따위는 상관이 없다 생각한다.
그래서 여기에는 백엔드 관점으로 작성될 예정이며, 이게 더 좋고 저게 좋고 기능적으로 휘륭하고 이런건 잘 모르겠다.
내가 아는 CI/CD툴은 다음과 같다.
Github action, jenkins한개더 있긴한데 그건 뭔지 까먹었다. 그래사 2가지로만 비교해서 작성해볼예정이다.

### Github action

github action은 github에서 제공하는 CI/CD툴이다.
yml를 통해 작성하는거 같다.
간단하게 gpt를 통해 코어 지식을 알아보자.

총 3가지의 개념이 있다고 한다.
1. Event
    - 가장 먼저 알아야 할 건 "무슨 일이 생겼을 때 실행할 것인가?"이다.
    - GitHub에서는 다음과 같은 이벤트가 발생할 수 있다:

| Event | Description |
| --- | --- |
| push | 브랜치에 커밋이 푸시될 때 |
| pull_request | PR 생성/업데이트될 때 |
| workflow_dispatch | 수동 실행 (버튼 클릭 등) |
| schedule | cron 형식으로 정기 실행 |
| release | 릴리즈 생성/수정 등 |

그러면 이것을 어떻게 사용할수 있을까?

대표적으로 다음 처럼 사용이 되어진다고 한다.

``` yml
on: <event>
```
하위는 추가적으로 추가될 예정이다.
위를 통해 다시 작성해보면

``` yml
on: 
  push:
```
요렇게 쓰인다는걸 알 수 있다. 세부적인건 전부 작성을 한다음 생각해보자.

2. Workflow
얘 같은 경우는 그냥 yml자체인데 어떤식으로 동작을 시킬지 정의하는 부분이라고 한다.

- .github/workflows/ 디렉토리에 있는 YAML 파일 하나가 워크플로 단위다.
- 워크플로에는 여러 개의 **잡(job)**이 있고, 각 잡은 독립된 환경에서 병렬로 실행됨.

하나의 단위이며 다음과 같이 작성이 된다고 한다.

``` yml
name: <workflow_name>
```
하위에는 evnet를 넣구 job을 넣는다.

3. Jop & Step
이제 마디막이다.

- Job: 독립적인 환경에서 실행되는 하나의 작업 단위
    - VM or 컨테이너에서 실행됨
    - 예: 테스트, 빌드, 배포 등

- Step: 잡 내에서 순차적으로 실행되는 명령
    - uses: 외부 액션 사용
    - run: 쉘 명령어 실행


하나의 큰 덩치로 생각했을때
workflow -> event -> job -> step 순으로 생각할 수 있다. 근데 우리가 궁금한건 이게 어디로 배포가 되는지 그게더 궁금할거 같다.
ec2를 이용하게되면 주소가 나오는데 그것을 어떻게 지정할 수 있는걸까?

예시는 다음과 같다.

``` yml
name: Deploy to EC2

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      - name: EC2 서버에 배포
        run: |
          ssh -o StrictHostKeyChecking=no ec2-user@${{ secrets.EC2_HOST }} << 'EOF'
            cd /home/ec2-user/my-app
            git pull origin main
            ./deploy.sh
          EOF
        env:
          SSH_PRIVATE_KEY: ${{ secrets.EC2_SSH_KEY }}
        shell: bash

      - name: SSH 키 등록
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.EC2_SSH_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
```

이것을 해석하면 main에서 push이벤트가 발생했을때 deploy job을 실행한다.
자세한건 설명하지는 않았지만
github action같은 경우는 github와 굉장히 밀접한 관련이 있다고 생각한다.
만약에 github을 사용한다면 젠킨스보다 얘를 선택하는것이 더 효율적인거 같다.

### jenkins
이제 젠킨스에 대해 설명하겠다. 얘는 github action과 달리 웹에서 실행되는것이 아닌 프로그램을 통해 실행이 되어진다.
또, 젠킨스는 github action과 달리 다양한 방법으로 배포가 가능하다.
내용은 github action에 비해 길지는 않을거 같긴하지만 아무튼 저 방법은 젠킨스에서는 파이프라인이라고 불려진다.

파일로 관리되어지는 githubaction과 달리 
젠킨스는 파일이 아닌 폴더로 관리가 되어져
직접 파일을 작성할 수 있다.

개인적으로 무엇이 더 가독성이 좋은지를 생각해보면 젠킨스라고 생각이 든다.

[젠킨스 파이프라인](https://b-programmer.tistory.com/433)
[젠킨스 프리스타일 vs 파이프라인](https://b-programmer.tistory.com/433)

## 📝 요약 (One-liner)
GitHub Actions는 GitHub과의 원활한 통합이 장점인 
반면, Jenkins는 더 다양한 배포 옵션과 유연한 파이프라인 구성을 제공하는 CI/CD 도구

## 결론
어떤걸 선택하든 백엔드 관점에서 생각했을때 어떤 기술을 사용하는지 상관없을거 같다.
다만 이런점은 있는거 같다.

모든 프로젝트가 gihub으로만 관리가 되어진다면 github action을
일부 프로젝트가 gitlab, github등등 여러가지로 관리가 되어진다면 젠킨스가 더 좋을거 같다.

다만 젠킨스는 서버를 따로 사용해야 하기때문에 비용측면을 생각하면 github action을 사용하는 것이 훨씬 좋을거 같다.

즉 정리하면
1. github을 사용하면 github을
2. 서버비에 부담을 느끼지 않는다면 젠킨스를

개인적으로 이 기술을 사용하지 않아서 안하다는건 그건 개발자로서 아닌거 같다.

## 다음에 진행할일
아직 백이랑프론트를 구분을 짓지 않았다.
만약 하게 된다면 2차 배포 준비를 해야할거 같다.
그리고 코드도 AI에서 작성된걸 내 코드로 바꾸는 작업이 진행을 해야 할거 같다.

이거 끝나면 다이어그램을 한번 작성해보자.

