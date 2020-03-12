---
title: AWS Lambda 와 Firebase Auth 를 이용하여 Kakao, Naver 로그인 구현하기 - 1 [AWS Lambda 배포 설정]]
date: 2020-03-12 15:30:14
categories: programming
tags:
  - firebase-auth
  - firebase
---

> AWS Lambda 와 Firebase Auth 를 이용하여 Kakao, Naver 로그인 구현하기
> - [AWS Lambda 와 Firebase Auth 를 이용하여 Kakao, Naver 로그인 구현하기 - 0 [전굽기]](/programming/kakao-and-naver-login-with-firebase-0)
> - [AWS Lambda 와 Firebase Auth 를 이용하여 Kakao, Naver 로그인 구현하기 - 1 [AWS Lambda 배포 설정]](/programming/kakao-and-naver-login-with-firebase-1)
> - [AWS Lambda 와 Firebase Auth 를 이용하여 Kakao, Naver 로그인 구현하기 - 2 [Kakao, Naver 로그인 적용]](/programming/kakao-and-naver-login-with-firebase-2)


---

해당 포스트에선
1. Serverless Framework 을 이용하여 쉽게 AWS Lambda 와 API Gateway 에 프로젝트를 배포한다.
2. GitLab Runner 를 이용하여 master 브랜치에 변경점이 생긴 경우 자동으로 (1.)을 실행한다.

---

해당 포스트는 [Serverless Framework](serverless.com)를 이용하여 레포지토리에 변경사항이 생길 경우 AWS Lambda 와 API Gateway 에 자동으로 배포하는 설정에 대한 포스트이다. NodeJS 에서 Firebase 에 Kakao, Naver 인증을 연동하는 내용은 [다음 포스트](/programming/kakao-and-naver-login-with-firebase-2)를 참고한다.

## 프로젝트 설정

각 요소(Serverless Framework, AWS Lambda, API Gateway)에 대해 깊은 이해는 필요없지만, 각각 어떤 기능을 제공하는지 간단하게라도 알아야 포스트를 읽기 수월할 것이라고 생각한다.

본인은 레포지토리로 GitLab을 자주 사용한다. 그래서 GitLab과 [GitLab Runner](https://docs.gitlab.com/runner/)를 사용하여 자동화를 할 예정이다. 이는 자신이 선호하는 조합(Github + action, Github + Travis 등)을 사용하면 된다.

GitLab 레포지토리를 생성하고 clone 받은 뒤 npm 프로젝트로 초기화(`npm init`) 한 뒤, eslint 같은 정적 분석 툴이나 의존 패키지들을 설치하면 된다.

Kakao, Naver API 에 Request 를 날릴 [request](https://www.npmjs.com/package/request) 와 [request-promise-native](https://www.npmjs.com/package/request-promise-native)도 install 하자. 20년 들어오면서 Depreacted 되었지만 별달리 괜찮은 대체제를 찾지 못했다. 꼭 `request`를 써야 할 필요는 없다, 자신이 편한걸 사용하면 된다.

마지막으로 Firebase auth 에 새 유저를 추가하고 CustomToken을 발급하는데 필요한 [firebase-admin](https://www.npmjs.com/package/firebase-admin)을 install 하자.

AWS Lambda 를 사용할 것이므로 기본적으로 위 `request`, `request-promise-native`, `firebase-admin` 세개만 있어도 충분하다. 이것저것 의존할거 없이 쉽게 프로젝트를 구성할 수 있는것도 AWS Lambda 의 장점이라고 생각한다.

## Handler 생성

`src/index.js` 파일을 생성하고 아래와 같이 기본 handler 를 선언한다.

```js
const handler = async (event) => {
  const body = JSON.parse(event.body)

  return {
    statusCode: 200,
    body: JSON.stringify(body)
  }
}

exports.handler = handler
```

AWS Lambda 만 사용할 때와 API Gateway 와 같이 사용할 때 handler 의 인자로 주입되는 event 값과 반환되어야 할 return 값이 다르다. event 에 어떤 구조의 값이 주입되는지는 [Using AWS Lambda with Other Services](https://docs.aws.amazon.com/lambda/latest/dg/lambda-services.html)를 참조한다.

handler 를 async 함수로 한 이유는 어차피 http request 를 보내는데 비동기 처리가 들어갈 것이기 때문에 미리 선언한다. Lambda 는 Promise 객체의 반환도 지원하므로 async 를 사용하거나 Promise 객체를 직접 반환하여도 문제없이 동작한다.

나중에 코드가 점점 추가되겠지만, 자동화 테스트에는 이정도의 코드만 있어도 충분하다.

## Serverless 설정 생성

프로젝트 루트 디렉토리에 `serverless.yml`을 생성하고 아래의 설정을 입력한다.
> Serverless framework의 AWS Lambda와 API Gateway 설정을 보다 깊게 알고 싶다면 [해당 문서](https://serverless.com/framework/docs/providers/aws/events/apigateway/)를 참고한다.

```yml
service: exampleProject #1
provider:
  name: aws
  runtime: nodejs10.x #2
  region: ap-northeast-2  #3

functions:
  custom_token:
    handler: src/index.handler #4
    events:
      - http: POST custom_token #5
    environment:
      SERVICE_ACCOUNT_KEY: ${env:SERVICE_ACCOUNT_KEY} #6

```

설명할 설정이 많아 주석마다 라벨을 지정하고 설명을 달도록 하겠다.

- \#1 에는 프로젝트의 이름을 지정해 주면 된다.
- \#2 에는 사용할 NodeJS 런타임을 지정하면 된다. [lambda-runtimes](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/lambda-runtimes.html) 참고한다.
- \#3 에는 AWS Lambda 가 배치 될 region 을 지정해주면 된다. [using-regions-availability-zones](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) 참고한다.
- \#4 에는 handler 를 export 하는 파일을 지정해주면 된다. `src/<fileName>.<handlerName>`의 포맷이며, 위 [Handler 생성](#Handler-생성) 기준으로는 `src/index.handler` 이다.
- \#5 사용할 Resource 를 지정한다, 이는 API Gateway 를 생성하는데 사용된다. 위 예제에선 POST Method 만 custom_token이라는 경로로 받을 것이기 떄문에 `POST custom_token`만 설정하였다.
- \#6 추후 Firebase Admin 에서 필요한 serviceAccountKey 의 값을 환경변수로 넣어준다. serviceAccountKey 발급에 대해서는 [해당 링크](https://firebase.google.com/docs/admin/setup?hl=ko#initialize_the_sdk)를 참고한다.

\#6 에 대해 부가적인 설명이 필요한데, serviceAccountKey 는 그냥 json 파일째로 프로젝트에 넣어 `require` 할 수 있지만, 그러면 레포지토리에 serviceAccountKey 가 노출되기때문에 보안을 위해 `.gitignore`에 등록하고 작업자들에게 직접 키를 배포한다거나 하는작업이 필요할 수 있다.
때문에 파일을 노출하지 않기위해 환경변수에서 받아 역직렬화하여 사용하는게 보다 관리하기 편하다. 이 환경변수는 local에서 테스트 할 경우에는 직접 추가하면 되고, 추후 배포시에는 GitLab Runner 의 설정으로 주입할 것이다. 해당 설정으로 주입한 환경변수는 AWS Lambda 의 환경변수로 전달되어 사용된다.(`process.env.SERVICE_ACCOUNT_KEY`로 사용할 예정)

해당 설정이 끝나면 `npx serverless deploy --stage production`로 배포할 수 있다. `serverless.yml`에 명시한 대로 AWS Lambda 와 API Gateway 에 배포, 설정될 것이다.

> `npx serverless deploy`를 하기 위해선 IAM에서 KEY ID 와 Secret 을 각각 환경변수 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`에 추가해주어야 한다. 이는 추후 `SERVICE_ACCOUNT_KEY`와 같이 GitLab Runner 의 설정에서 주입될 예정이다.
> Serverless 를 위한 IAM 키 할당, 발급은 [해당 문서](https://helloinyong.tistory.com/135)에 너무 잘 설명되어 있으니 참고한다.

## GitLab Runner 설정

우선 [Serverless 설정 생성](#Serverless-설정-생성)에 사용한 환경변수들은 GitLab Runner 의 환경변수에 넣어보자.

브라우저로 해당 프로젝트의 GitLab Repository로 이동한다음 Settings->CI/CD 페이지로 이동하여 `Variables`에 `SERVICE_ACCOUNT_KEY`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`를 환경변수로 등록한다.
> 환경변수 설정에 대해 더 알고싶으면 [GitLab CI/CD environment variables#via-the-ui](https://docs.gitlab.com/ee/ci/variables/README.html#via-the-ui)문서를 참고한다.

![Variables](/images/kakao-and-naver-login-with-firebase/variables.png)

위 사진과 같이 각 환경변수들을 설정하고 나면 `.gitlab-ci.yml`을 작성해야한다. 프로젝트 루트 디렉토리에 `.gitlab-ci.yml`파일을 만들고 아래와 같이 설정을 입력한다.
> 아래 설정은 [Deploying AWS Lambda function using GitLab CI/CD#crafting-the-gitlab-ciyml-file](https://docs.gitlab.com/ee/user/project/clusters/serverless/aws.html#crafting-the-gitlab-ciyml-file)에 잘 설명되어있다.

```yml
image: node:latest

stages:
  - deploy

production:
  only:
    - master
  stage: deploy
  before_script:
    - npm config set prefix /usr/local
    - npm install -g serverless
  script:
    - serverless deploy --stage production --verbose
  environment: production

```

위 설정에서 master 브랜치에 변경이 가해질 경우 `production` Job 이 동작하도록 설정하였으므로, 커밋&푸시하여 GitLab 레포지토리의 master 브랜치에 수정을 가해 자동으로 `serverless deploy --stage production --verbose`를 실행하게 할 수 있다.

## GitLab Runner 실행

GitLab 레포지토리에 커밋을 푸시해보자
> 웹 페이지에서 CI/CD -> Pipeline 페이지으로 이동한다음 Run Pipeline을 이용해 수동으로 실행할수도 있다.

푸시하고 브라우저에서 프로젝트 레포지토리로 이동한다음 좌측 탭 CI/CD의 Pipeline 페이지로 이동해보자.

![Pipeline](/images/kakao-and-naver-login-with-firebase/pipeline.png)

위 이미지대로 Pipeline 이 생성되어있을 것이고, 해당 파이프라인을 눌러 상세 정보와 로그를 볼 수 있다.

> 각 Job의 로그는 파이프라인을 눌러 파이프라인 페이지로 이동한 다음 해당 Job을 누르면 확인할 수 있다.

---

`GitLab Runner 실행` 까지 진행했다면, 앞으로 코드 수정사항에 있어 귀찮은 작업 없이 커밋&푸시 만으로 코드를 갱신할 수 있다.

이러한 작업들이 없다고 AWS Lambda 를 사용할 수 없는 것은 아니다. 직접 손으로 Lambda Function 을 만들고 프로젝트를 압축하여 업로드 할 수 있고, Function 설정에서 직접 API Gateway를 연결한 뒤 배포하여 사용할 수 있다.

그러나 코드 하나를 바꾸는 수정에도 직접 프로젝트를 압축하여(이것도 50MB 가 넘는 경우에는 S3를 통해 업로드 해야한다) 배포한다거나, serviceAccountKey 를 관리한다거나 하는 귀찮은 작업들을 자동화 시키면, 처음에는 귀찮고 짜증날지 몰라도 이후 진행되는 작업은 수월하게 진행할 수 있을 것이다.

## 참고문서

[TUTORIAL: Build a Hello World API with Lambda Proxy Integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-create-api-as-simple-proxy-for-lambda.html)
[서버에 Firebase Admin SDK 추가](https://firebase.google.com/docs/admin/setup?hl=ko#initialize_the_sdk)
[[2019.06.10] Serverless framework를 이용하여 lambda 배포하는 법](https://helloinyong.tistory.com/135)
[GitLab CI/CD environment variables#via-the-ui](https://docs.gitlab.com/ee/ci/variables/README.html#via-the-ui)
[Deploying AWS Lambda function using GitLab CI/CD](https://docs.gitlab.com/ee/user/project/clusters/serverless/aws.html)