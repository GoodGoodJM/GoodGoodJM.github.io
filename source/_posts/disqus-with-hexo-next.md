---
title: Hexo NEXT 템플릿 블로그에 Disqus 댓글 기능 추가하기
tags:
  - Hexo
  - NEXT
  - Disqus
  - Comment
categories: blog
date: 2019-04-14 15:35:58
---

[Hexo와 NEXT 테마를 사용하는 블로그 시작.](/blog/blog) 포스트에서 블로그를 생성하였지만, 댓글 기능이 있으면 좋을 것 같아서 추가하려고 한다.

NEXT테마는 gitment와 disqus같은 댓글 추가 기능들을 설정만으로 추가, 제거할 수 있게 할수 있도록 기능을 미리 제공한다. 때문에 별다른 추가작업(템플릿을 고친다거나) 이 없이 붙일 수 있다는 점에서 다시한번 놀랐다.

원래는 [gitment](https://github.com/imsun/gitment)가 좀 더 Gick 해보여서 사용하려 했었지만, AutoBuild때문에 추후 Gitlab으로 넘어갈 생각이라 플랫폼을 덜타는 Disqus를 사용하기로 했다.

## Disqus 가입

Disqus를 가입, 로그인 한 뒤 우 상단의 메뉴들 중 `Add Disqus To Site`를 눌러 설정 페이지로 들어간다.

19/04/14 현재는 중간에 간단한 설명 페이지가 나타나며, 맨 아래의 `GET STARTED`를 눌러 다음 페이지로 이동한 다음 `I want to install Disqus on my site`를 눌러 설정을 해야한다.

설정을 어떻게 해야할 지 잘 모르겠다면 [Github Blog에 댓글(disqus) 기능 추가하기](https://devmjun.github.io/archive/addComments)문서에 잘 나와 있어 참고하면 좋을 듯 하다.

## NEXT의 `_config.yml`에 Disqus설정 추가 

자신 페이지의 `shortname`이 뭔지 확인되었으면, NEXT 테마의 `_config.yml`의 `disqus.enable`. `disqus.shortname`의 값을 바꿔준다.

```yml
# _config.yml
...

# Disqus
disqus:
  enable: true
  shortname: <short_name>
```

## 확인

위 설정을 마친 다음 다시 페이지를 확인하면 하단에 disqus댓글 기능이 추가되어 있는 걸 확인할 수 있다.

## 참고문서
- [Add Disqus Comments to Your Hexo Site](https://equalsequals.io/2017/12/13/how-to-add-disqus-comments-to-hexo-blog/)
- [Github Blog에 댓글(disqus) 기능 추가하기](https://devmjun.github.io/archive/addComments)
- [Hexo 集成 Disqus 评论](http://www.cylong.com/blog/2017/03/26/hexo-next-disqus/)