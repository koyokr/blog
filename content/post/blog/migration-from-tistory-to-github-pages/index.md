---
title: Tistory에서 Github Pages로 이사하기
date: 2018-06-09 19:22:00 +0900 KST
categories: [blog]
tags: [tistory, github, hugo, travis-ci]
---

## Tistory에서

tistory 블로그를 써오면서 신경쓰이는 점이 두 개 정도 있었다.

1. 암호화 통신
2. 글 작성 방식

암호화 통신은 <http://koyo.kr>보다는 <https://koyo.kr>으로 접속하고 싶다는 이야기

더 신경 쓰이는 건 글 작성 방식이다.

tistory 제공 에디터로 글을 작성하고 나서 html 코드로 보면 굉장히 난잡해보인다.\
개인적으로 이런 건 정갈하게 관리하고 싶은 마음이 있어서 매번 글 발행 전에 html 코드를 정리한다.

html 편집 모드에서 글을 써야 할 때도 있다. 글에 코드블럭을 삽입할 때가 그런 때다.\
표를 삽입하고 싶을 때도 따로 저장해놓은 html 양식을 수정해서 붙여넣어 쓴다.

그런데 더 좋은 방법이 있을 거 아닌가.

github pages에 블로그를 호스트하는 사람들은 markdown으로 포스팅한다는 걸 알게 됐고.

따라했다.

## Github Pages로

github pages에 내 블로그를 올리기로 했다.

글을 어떻게 쓰나 찾아보니 아래와 같은 순서를 따르는 듯하다.

1. markdown 작성
2. static site generator로 페이지 생성
3. github repo에 업로드

github pages라서 markdown으로 글을 작성하는 게 아니라 static site generator가 markdown을 변환하는 거구나!\
라는 걸 알았다.

### Markdown

tistory에 작성한 글을 markdown으로 변환해야 했다.

이걸 다 수작업으로 옮길 수는 없으니까...\
블로그 본문을 다 긁어서 html 코드를 markdown으로 일괄 변환하는 코드를 작성하기로 했다.

python의 html2text 라이브러리를 사용는데 아니 이거 내 생각대로 안바꿔주는 거다.\
변환된 markdown을 내 입맛에 맞게 고쳐야 해서 여기에 시간을 많이 썼다.

기존 글은 <https://koyo.kr/96>과 같은 url로 접속해왔기 때문에 변환한 글도 그렇게 접속하게끔 url을 지정했다.\
나중에는 모든 글에 숫자가 아닌 slug를 지정해주고 기존 url로 접속하면 redirect되게끔 할 계획이다.

### Static Site Generator

정적 사이트 생성기로는 hugo를 선택했다.\
jekyll 말고 이걸 고른 이유를 굳이 말하자면 ruby보다 go가 더 좋아서?

다른 사람이 github에 호스트할 건데 추천해달라고 하면 jekyll를 추천해줄 것 같다.\
jekyll이 이쁘장한 테마도 많고 자료도 더 많은 것 같으니까.

#### Theme

테마를 계속 살펴봤는데 내가 생각하는, 확 다가오는 그럼 테마는 보이지 않았다.

불만 있으면 본인이 만들어야지...

그래서 그냥 괜찮은 테마 하나 골라서 썼다. 바로 [hugo-tranquilpeak-theme](https://github.com/kakawait/hugo-tranquilpeak-theme)다.

해당 repo를 fork 떠서 code highlighting, firefox에서 볼 때 깨지는 버튼, 한글 웹폰트 등을 수정하거나 적용했다.

웹폰트 부분에서 꽤나 삽질을 했는데,\
기존 폰트는 한글이 없어서 google fonts의 `Noto Sans KR`과 `Nanum Myeongjo`, 그리고 `Nanum Cothic Coding` 폰트들을 테마에 적용했다.

```css
@import url('https://fonts.googleapis.com/css?family=Noto+Sans+KR:400,700|Nanum+Myeongjo:400,700|Nanum+Gothic+Coding:700');
```

뭐 얼추 이런식으로 했는데 군대 사지방에서 블로그 접속에 15초 넘게 걸리는 거였다.

사지방에서는 google 접속이 안될 때가 잦은데, `import`한 google 서버의 css를 받으려고 하는 동안 사이트 렌더링에 블록이 걸리는 것이 원인이었다.

일단 해결을 위해 [webfontloader](https://github.com/typekit/webfontloader)를 사용해서 비동기로 폰트를 로드하도록 했다.
접속은 해야지ㅜㅜ

```js
WebFontConfig = {
  google: {
    families: ['Noto Sans KR:400,700', 'Nanum Myeongjo:400,700']
  },
  custom: {
    families: ['D2 coding'],
    urls: ['https://cdn.jsdelivr.net/gh/Joungkyun/font-d2coding@1.31.0/d2coding.css']
  }
};
(function(d) {
  var wf = d.createElement('script'), s = d.scripts[0];
  wf.src = 'https://cdn.jsdelivr.net/npm/webfontloader@1.6.28/webfontloader.min.js';
  wf.async = true;
  s.parentNode.insertBefore(wf, s);
})(document);
```

이 스크립트를 `head` 태그 내 맨 앞에 삽입했다.

### Github Repository

다른 사람은 어떻게 관리하는지 보니까 컨텐츠와 결과물을 분리해서 관리하고 있었다.

컨텐츠는 koyokr/koyo.kr에 저장하고,
결과물은 koyokr/koyokr.github.io에 저장하는 식으로.

#### Travis-CI

travis-ci 서비스를 이용해서 컨텐츠를 갱신할 때 자동으로 결과물도 갱신되게 하더라.

바로 `.travis.yml` 파일을 만들어서 연동까지 했다.

```yaml
language: go
go:
  - "1.10.1"
install:
  - go get github.com/gohugoio/hugo
script:
  - hugo
deploy:
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN
  keep_history: true
  local_dir: public
  repo: koyokr/koyokr.github.io
  target-branch: master
  committer-from-gh: true
  on:
    branch: master
```

뭐 얼추 이런 식으로...

`$GITHUB_TOKEN`은 github에서 토큰을 생성해서 travis-ci에 환경변수로 설정해줘야 한다.\
자세한 건 찾아보면 나오더라~

## 후기

성공적으로 옮긴 것 같아 뿌듯한데... 글 쓰는 속도는 거기서 거기다.\
애초에 게을러서 글도 안쓰고.

대신 글을 쓸 때 스트레스는 덜 받는 것 같다.
