---
title: hexo-depoly-with-gitlab-ci
categories: blog
tags:
  - Hexo
  - NEXT
---

[Hexo와 NEXT 테마를 사용하는 블로그 시작.](/blog/blog) 포스트에 작성한대로, 나는 PC의 포맷이 잦고, 여러 기기에서 작업을 하는지라 (이곳 저곳 돌아다니면서 작업하는 것을 좋아하기 때문에) `Hexo` 프로젝트 자체를 깃헙 레포에 올려놓고 그때 그때 Clone || Pull 받아 포스트를 작성한다.

이렇게 작업하던 중 page의 레포지토리의 commit이 계속해서 증발하는 현상을 발견했고, 검색 후에 이러한 현상이 새 프로젝트로 clone받아 deploy할때 발생한다는 것을 알게되었다.

`Hexo`에서 고쳐줄 생각은 없어보이고, 나 또한 고쳐서 쓸 생각이 없었다. 그 중, CI/CD 로 `Hexo` 레포지토리에 푸시할때마다 자동으로 depoly 하게 하면 commit이 증발할 일이 없을거 같다는 생각이 들었다.

사실 중간부턴 커밋 로그가 증발한다는 문제보다, 그냥 CI/CD 붙이면 편할거 같다는 생각때문에 작업했다.

## GitlabRunner

`Hexo`레포지토리에는 Depoly를 위한 정보들이 들어갈 수 있으므로, private으로 하고싶었다.

처음엔 `Hexo`레포지토리를 Page와 같이 `Github`에 두고싶었으나, 대부분의 CI/CD 툴이 private repository에는 비용을 청구하였으므로 사용할 수 없었다. (직접 Runner 서버를 실행해야 하는 프로그램은 애초에 후보에도 올리지 않았다.) 때문에 `Gitlab`을 이용하기로 했고, 나는 `UDG`프로젝트 때문에 `Github`보다 `Gitlab`의 사용이 편했다.

해당 포스팅에선 `GitlabRunner`를 사용하는데 초점을 두지만, `.gitlab-ci.yml`에 대해선 그다지 설명하지 않을 예정이다. 필요하다면 포스팅을 하겠지만, 애초에 해당 포스팅에서 다루는 ci script가 그렇게 어렵지 않고 이미 [매우](https://lovemewithoutall.github.io/it/deploy-example-by-gitlab-ci/) [좋은 포스팅](https://namioto.github.io/2018/07/16/gitlab-ci%EB%A1%9C-%EC%9E%90%EB%8F%99%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0/)들이 여럿 있기 때문이다.

### .gitlab-ci.yml

`GitlabRunner`에게 요구할 기능은 간단하다.

1. `master` branch에 푸시할 경우, 자동으로 스크립트에 따라 빌드하는것.
2. 빌드된 결과를 github page 레포지토리에 푸시하는것.

위 내용에 대한 아래의 build script를 `.gitlab-ci.yml`에 추가한다음 커밋을 하면 끝이다.

```yml
추후 작성 예정
```

이후 `Gitlab`의 repository 대시보드에서, 좌측의 `CI/CD`메뉴를 클릭하여 빌드와 배포가 성공하였는지 확인한 다음, 페이지를 확인하면 원하는 대로 갱신된다.

**CI/CD 페이지 사진 추가**

---

(사족 - 삭제하고 초안 처리해서 publish 할것)

### Gitlab

잠시 주제에서 벗어나지만, 내가 `Gitlab`을 애용하게 된 계기는 다음과 같다.

`UDG` 프로젝트를 시작할때엔, `Github`은 private repository가 유료였기때문에 `Gitlab`을 사용했었다. 여러 단점(서버 불안정, 위키 증발-실제로 당했음-)에도 불구하고, `Gitlab`은 프로젝트를 진행하는데 충분한 기능(Issue, IssueBoard)을 제공했기때문에 굳이 `Github`이 private repository를 무료로 전환했어도 이주하지 않았다.

`Gitlab`을 사용하던 중, 가장 마음에 드는 부분들은, 프로젝트별로 `Docker`의 이미지를 업로드할 수 있는 `Registry`를 제공한다는 점과 `GitlabRunner(CI/CD)`가 무료이며-private repository에서도- 별다른 설정없이 깔끔하게 동작한다는 점이다.

`UDG`의 대부분의 백엔드 서비스들은 `GitlabRunner`로 자동으로 빌드되고 `Registry`에 배포되며 툴을 이용해서 서버의 인스턴스들을 갱신할 수 있게 구성되어있다.

---

## 참고문서

- [Gitlab-CI 구성 & .gitlab-ci.yml 예제](https://lovemewithoutall.github.io/it/deploy-example-by-gitlab-ci/)
- [Gitlab CI 시작하기](https://lovemewithoutall.github.io/it/deploy-example-by-gitlab-ci/)
