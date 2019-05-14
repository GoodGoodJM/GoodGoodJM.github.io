---
title: Travis-CI로 Hexo블로그 Deploy 자동화 하기
tags:
  - Hexo
  - NEXT
categories: blog
date: 2019-05-12 13:48:39
catecories:
---

## 참고문서

[Hexo와 NEXT 테마를 사용하는 블로그 시작.](/blog/blog) 포스트에 언급한대로, 나는 PC의 포맷이 잦고, 여러 기기에서 작업을 하는지라 (이곳 저곳 돌아다니면서 작업하는 것을 좋아하기 때문에) `Hexo` 프로젝트 자체를 깃헙 레포에 올려놓고 그때 그때 Clone || Pull 받아 포스트를 작성한다.

이렇게 작업하던 중 page의 레포지토리의 commit이 계속해서 증발하는 현상을 발견했고, 검색 후에 이러한 현상이 새 프로젝트로 clone받아 deploy할때 발생한다는 것을 알게되었다.

`Hexo`에서 고쳐줄 생각은 없어보이고, 나 또한 고쳐서 쓸 생각이 없었다. 그 중, CI/CD 로 `Hexo` 레포지토리에 푸시할때마다 자동으로 deploy 하게 하면 commit이 증발할 일이 없을거 같다는 생각이 들었다.

사실 중간부턴 커밋 로그가 증발한다는 문제보다, 그냥 CI/CD 붙이면 편할거 같다는 생각때문에 작업했다.

## CI/CD

`Hexo`레포지토리에는 Deploy를 위한 정보들이 들어갈 수 있으므로, private으로 하고싶었다. 때문에 `UDG`개발에 주로 사용하던 `Gitlab`에서 `GitlabRunner`로 CI/CD를 하려 했었다.

하지만 문서 작성 중, 막상 설정을 뒤져보니 새 환경에서 `git push`시 `Github`에 로그인 해야하는 부분 말고는 그다지 private한 정보가 없었고 환경변수로 `GithubToken`따위를 넘겨서 인증을 하면 될거같다는 점에서 그냥 `Travis-CI`를 이용하면 될거 같다는 생각이 들어 `Github + Travis-CI`의 조합을 밀어 사용하기로 마음먹었다.

때문에 `Travis-CI`를 사용하여 비슷한 처리를 하는것을 구글링하였고, 아니나 다를까 [더이상 좋게 쓰지 못할정도로 잘 설명된 문서](https://e.printstacktrace.blog/hexo-git-deployer-removes-commits-history-lets-do-something-about-that/)를 발견했다.

위 문서때문에 더이상 문서를 쓸 필요는 없겠지만, 지워진 `hexo-deploy-with-gitlab-ci`문서를 기리기 위해서라도 문서를 마무리 지어야겠다.

### Travis-CI

내가 `Travis-CI`에게 요구할 기능은 간단하다.

1. `edit` branch에 푸시할 경우, 자동으로 스크립트에 따라 빌드하는것.
2. 빌드된 결과를 `hexo deploy`하여 `master` branch에 푸시하는 것.

위 내용에 대한 아래의 build script와 `.travis.yml`를 추가한다음 커밋을 하면 끝이다.

`deploy.sh`

```bash
#!/bin/bash

# Initialize target with currently deployed files
git clone --depth 1 --branch=master https://github.com/GoodGoodJM/GoodGoodJM.github.io.git .deploy_git

cd .deploy_git

# Remove all files before they get copied from ../public/
# so git can track files that were removed in the last commit
find . -path ./.git -prune -o -exec rm -rf {} \; 2> /dev/null

cd ../

# Run deployment
hexo clean
hexo deploy
```

`.travis.yml`

```yml
language: node_js

node_js:
  - "6"

cache:
  directories:
    - node_modules

branches:
  only:
    - edit

before_install:
  - npm install hexo-cli -g
  - npm install hexo --save
  - npm install

before_script:
  - git config --global user.name 'Travis CI'
  - git config --global user.email 'bot@travis-ci.org'
  - sed -i'' "s~https://github.com/GoodGoodJM/GoodGoodJM.github.io.git~https://${GH_TOKEN}:x-oauth-basic@github.com/goodgoodjm/GoodGoodJM.github.io.git~" _config.yml # 주의!

script:
  - hexo generate

deploy:
  skip_cleanup: true
  provider: script
  script: sh deploy.sh
  on:
    branch: edit
```

`npm install -g`는 쓰지않는 성격이지만 어차피 나의 머신이 아니라 관심 없고, 원작자를 존중하기 위해 그냥 사용했다.

`before_script`의 맨 아래 커맨드(`# 주의!` 주석이 달린 부분)은, `sed`라는 명령어를 이용하여 문자열을 치환해주는 명령이다. 본인의 `_config.yml`의 `deploy.repo` 값이 `https://github.com/GoodGoodJM/GoodGoodJM.github.io.git`으로 되어있기 때문에 위 같이 작성하였는데, `~{origin_url}~{changed_url}~`의 `origin_url` 부분에 자신의 `deploy.repo`에 지정된 url을 추가해주면 된다. `changed_url`의 `${GH_TOKEN}`은 `Travis-CI`의 환경변수에 `GH_TOKEN`을 추가해 주어야 한다.

환경변수 설정은 `Travis-CI`에서 레포지토리를 지정한 다음, setting에서 변경할 수 있으며, Token은 [해당 페이지](https://github.com/settings/tokens)에서 발급 받을 수 있다.

이후 `Travis-CI`를 레포지토리에 붙인 뒤, 이후 edit 브랜치에 푸시를 할 때 마다 제대로 갱신되는지 확인해 주면 된다.

![success](https://i.imgur.com/C6Icwvv.png)

---

## 참고문서

- [Hexo git deployer removes commits history? Let's do something about that!](https://e.printstacktrace.blog/hexo-git-deployer-removes-commits-history-lets-do-something-about-that/)
- [리눅스 find 명령어 사용법 정리 (파일, 디렉토리 검색, 찾기)](https://withcoding.com/97)
