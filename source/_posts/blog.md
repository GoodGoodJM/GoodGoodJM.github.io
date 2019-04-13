---
title: Hexo와 NEXT 테마를 사용하는 블로그 시작.
categories: blog
---

회사를 6월에 퇴사하기로 하고, 공부하면서 Notion에 끄적여 놨던 내용들을 한번 되새김질 하는 시간을 가져보려고 한다.

포트폴리오도 다시 정리할 겸, 이왕 정리하는 거 블로그로 정리해보면 재밌겠다는 생각이 들어 [Hexo](https://hexo.io/)로 블로그를 만드려고 한다.

`Hexo`를 선택한 이유는 단 하나 "JS로 만들어져서" 이다.

최근 다시 웹을 공부하면서 JS가 익숙하기도 하고, 직접 코드를 만지거나 테마를 만들면서 프로그램을 파악할 수 있다는게 큰 장점으로 다가왔다, 만일 `Jekyll`가 Ruby가 아니라 JS였다면 Jekyll를 사용했을 것이다.

블로그의 생성은 [hexo 블로그 시작하기](https://enesto.github.io/2018/11/18/181118_hexo%20%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0/)를 참고하였다. 당장 마음에 드는 테마가 없어 해당 포스트의 테마를 썼다.

## 기본 설정

나는 될 수 있으면 `npm install {name} -g`를 하지않고 `npx`를 쓰려고 한다. 전역으로 모듈을 설치해두고, 잊어버린다음 build script를 짯다가 나중에 원인을 찾는게 한참을 걸리는 바보같은 실수를 범한 적이 있어 될수있으면 전역 모듈은 설치하지 않으려 노력한다. (사실 익숙해지면 느린거 빼곤 편하다)

```shell
$ npx hexo init blog
$ cd blog
$ npm install
```

이후 원한다면 `_config.yml`의 내용을 수정해 주면된다 나같은 경우는 `title`, `author`, `language`, `timezone`, `url` 정도만 수정했다. 자세한 내용은 [공식 문서](https://hexo.io/docs/configuration)를 참고하면 된다.

`_config.yml`의 내용을 수정했다면 로컬서버를 실행하여 확인해 볼 수 있다.

```shell
$ npx hexo server
```

## NEXT 테마 설치

생각보다 `Hexo`의 기본 테마가 훌륭해서 그냥 쓸까 고민했지만, 살짝 허전해서 마저 테마를 설치했다.

해당 내용은 위에 적은 [hexo 블로그 시작하기](https://enesto.github.io/2018/11/18/181118_hexo%20%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0/)에 너무 잘 나와있어서 별달리 쓸 내용이 없다.

그냥 거슬려서 기본 테마인 landscape 테마 폴더를 지우는 것 정도만 하였다.

## 배포

`Hexo`는 몇가지 추가적인 설정으로 쉽게 github page에 build된 블로그를 배포할 수 있다.

`_config.yml`의 Depoly를 설정하고, git인 경우엔 `$ npm install --save hexo-depolyer-git`를 이용하여 플러그인 모듈을 설치해주면 준비가 끝난다.

```yml
# _config.yml
...
depoly:
  type: git
  repo: https:/github.com/<UserName>/<UserName>.github.io
  branch: master
```

이후 depoly 명령어 `$ npx hexo deploy -g`를 호출하면 `deploy.repo`에 해당하는 레포지토리에 depoly 된다.

## Categories, About 같은 추가 페이지 생성

Theme의 `_config.yml`를 살펴보다보면, Categories나 About기능을 지원해준다. 해당 설정의 주석을 해제해 줌으로서 기능을 활성화 할 수 있다.

```yml
...
menu:
  ...
  # 활성화 할 기능들의 주석을 해제
  about: /about/ || user
  categories: /categories/ || th
  ...
...
```

주석을 해제하면 블로그에 Catrgories와 About 메뉴가 나타난다. 하지만 해당하는 페이지가 존재하지 않아 다음과 같은 명령어로 페이지를 추가해주어야 한다.

```shell
$ npx hexo new page categories
$ npx hexo new page about
```


## 참고문서

- [공식 문서](https://hexo.io/docs/configuration)
- [hexo 블로그 시작하기](https://enesto.github.io/2018/11/18/181118_hexo%20%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0/)