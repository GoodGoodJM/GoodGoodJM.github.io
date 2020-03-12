---
title: 'AWS Lambda 와 Firebase Auth 를 이용하여 Kakao, Naver 로그인 구현하기 - 0 [전굽기]'
date: 2020-03-10 10:37:32
categories: programming
tags:
  - firebase-auth
  - firebase
---

> AWS Lambda 와 Firebase Auth 를 이용하여 Kakao, Naver 로그인 구현하기
> - [AWS Lambda 와 Firebase Auth 를 이용하여 Kakao, Naver 로그인 구현하기 - 0 [전굽기]](/programming/kakao-and-naver-login-with-firebase-0)
> - [AWS Lambda 와 Firebase Auth 를 이용하여 Kakao, Naver 로그인 구현하기 - 1 [AWS Lambda 배포 설정]](/programming/kakao-and-naver-login-with-firebase-1)
> - [AWS Lambda 와 Firebase Auth 를 이용하여 Kakao, Naver 로그인 구현하기 - 2 [Kakao, Naver 로그인 적용]](/programming/kakao-and-naver-login-with-firebase-2)


관리하기 쉬운 코드는 내가 관리할 코드가 적은 코드라고 생각한다, 때문에 나는 관리하는 코드를 줄이는 것을 좋아한다.

예전에는 어지간한 기능은 전부 만들었던 것 같다. 예를들면 gitlab-runner같은 솔루션들이 있음에도 불구하고, git의 hooks를 이용하여 직접 CI/CD를 만들거나 사용할 기능에 비해 라이브러리에 기능이 많으면 해당 기능만 직접 구현해서 쓰거나 했었다.

직접 작성한 코드는 작성자가 나이므로 문제의 발생 원인이나 수정이 매우 쉽다. 하지만 동시에 관리하는 코드가 많아지고, 결과적으로는 의도와 다르게 이런 저런 기능들이 들어가면서 기존에 봤던 라이브러리의 하위호환따위가 되고는 한다. 작성했던 코드에서 문제가 발생하면 더욱 답이 없다, 내가 작성한 모든 코드들은 나의 관리(책임)이므로 사소한 모든 에러사항이나 잘못된 동작은 나를 더욱 힘들게 한다.

외주를 간간히 받게되면서, 유지보수에 많은 시간을 할애하기 힘든 나는 필연적으로 내가 관리하는 코드를 줄이고 보다 관리하기 쉬운 코드를 만들려고 노력하게 되었다.

## Firebase Auth

[`Firebase Auth`](https://firebase.google.com/docs/auth/?gclid=Cj0KCQjw0pfzBRCOARIsANi0g0urbOcQ-rnyQ6_bUJgg76RpcxQ0cw-i2uMquzpJkEOPk2ygrYzRUGgaAmwCEALw_wcB)는 Firebase에서 제공하는 유저인증 솔루션이다. 모바일, 웹 플랫폼에 관계없이 누구나 쉽고 빠르게 로그인, 회원가입을 구현할 수 있고 여러 인증(SMS, Email)과 Social Login들을 지원한다. 유저의 구분이 필요한 거의 모든 프로그램은 인증 기능이 필요하므로 `Firebase Auth`에 익숙하면 프로그래머가 굳이 인증기능을 만드는데 시간을 할애할 필요없이 곧바로 메인 로직을 작업할 수 있다.

인증은 상당히 간단해 보이면서 귀찮은 절차가 많다. 간단하게 '유저의 아이디와 비밀번호를 조회하여 검증한다'의 뒤에 별개의 프로젝트로 구성하여 따로 서버를 구동시켜야 하는 경우도 있고, 유저 가입 처리, 로그인 처리, 수정, 탈퇴, SMS/Email 인증, 서드파티 인증, 토큰발급... 이미 구성해놓은 솔루션 없이 새로 작업하다보면 작업하다 지치는 경우도 많았다.

뿐만 아니라 Kakao, Naver같이 다른 제공업체를 이용해야 하는 경우는 로그인 프로세스가 1000ms 이상(네트워크 환경에 따라 다를 수 있음.)의 긴 실행시간을 가지는 경우가 많다. 때문에 여러 유저가 가입/접속/수정 할 때를 대비하여 확장 가능하고 안정성 있는 인프라를 구축하기 위해 Load Balancing이나 Health Check 등을 구현해서 작업해야할 필요가 있는데, 이는 너무나도 할게 많아 그냥 `에이 나중에 하지` 라는 생각으로 넘어갔다가 다시 작업하는 경우도 있었다.

당장 위에 적은 작업들만 봐도 인증을 구현하는데 필요한 작업은 너무나도 많다. 더군다나 개인 프로젝트로 하다보면 2~3주가 지났는데 아직까지 메인 기능을 작업하지 못하는 상황에 직면하다 지쳐 그만둬 버리는 경우가 많다.

때문에 나는 인증을 구현할때 `Firebase Auth`와 `AWS Lambda(혹은 이와 같은 서비스)`를 같이 사용하고 이런 방식으로 사용하는것을 추천한다.

## 왜 Firebase Auth를 선택했나.

Kakao나 Naver 로그인 같은 경우는 아직 솔루션 단에서 바로 제공하는 곳이 없다. 어느정도 추가 작업이 필요하다.
`Cognito`에서 작업한다고 하면 `Identity Pool 과 User Pool`과 Labmda를 같이 써서 인증을 해야하는데 이러한 부부에서 Firebase Auth는 CustomToken을 발급해버리면 끝이므로 비교적 쉽게 작업할 수 있다.
나는 Firebase Auth를 선호하여 사용하였지만 굳이 강제될 필요는 없다, 각자 자기가 편한 스택을 사용하면 된다고 생각한다.

## 왜 AWS Lambda인가?

관리하기 쉽다, 메모리 할당과 제한 시간을 설정할 수도 있고, 각종 AWS 스택을 가져다 붙일 수도 있다.
- `API Gateway`를 이용하여 Key별로 요청 가능한 할당량을 준다거나, 권한제어를 할 수 있다. `X-Ray`같은 기능을 붙일수도 있으며 Swgger로 API 명세를 뽑아줄 수도 있다.
- `CloudWatch`로 로깅을 하거나 Cronjob을 스케쥴 할 수 있다.
- `VPC`에 묶어 다른 AWS 스택(`EC2`, `RDS` 등등..)에 접근할 수 있다.

꼭 AWS에 묶일 필요는 없다. GCP 스택을 선호하면 `Cloud Functions`를 사용하면 되고, Azure 스택을 선호하면 `Azure Functions`를 사용하면 된다.

포스팅은 해당 포스트를 포함하여 3개로 진행될 예정이며 각 `AWS Lambda`나 `Firabase Auth` 자체에 대해 설명하거나 사용방법을 글에 쓰진 않을 예정이다.
내가 작성할 수 있는 것보다 훨씬 좋은 내용의 포스트들이 많으므로 해당 포스트들을 링크할 것이며, 진행중에 마주친 일반적이지 않은 에러에 대해선 간단하게 작성할 예정이다.
