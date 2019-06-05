---
title: vcpkg integrate install에서 dynamic 대신 static 적용하기
slug: vcpkg-static-instead-of-dynamic
date: 2017-03-19 22:08:00 +0900 KST
categories: [development]
markup: mmark
---

우연히 vcpkg를 알게 됐는데 그동안 내가 찾았던 기능이 여기 있었다.

`vcpkg integrate install`를 실행하면
비주얼 스튜디오에서 일일히 프로젝트 설정으로 include할 위치... lib 위치...
이런 걸 지정해주지 않아도 된다.

자동으로 해준다.

ㅎㅎ

ㅎㅎ

그런데 `x86-windows-static`나 `x64-windows-static`으로 설치한 라이브러리는 이게 안됐다.

`x86-windows`나 `x64-windows`로 설치한 라이브러리는 잘만 되는데...

찾아보니 프로젝트 파일을 직접 수정하는 방법이 있었다.

내가 이해한 게 맞다면 새로 생성한 프로젝트마다 같은 작업을 해줘야 한다.
그건 너무 귀찮은 것 아닌가?

더 찾아보니 이런 방법이 있었다.

아래 파일에서 `x86-windows`, `x64-windows`로 된 부분을 각각 `x86-windows-static`, `x64-windows-static`으로 바꿔준다.
<https://github.com/Microsoft/vcpkg/blob/master/scripts/buildsystems/msbuild/vcpkg.targets>

이렇게 하면 비주얼 스튜디오에서 `x86-windows-static`, `x64-windows-static`으로 설치한 라이브러리가 자동으로 잡힌다.

참고로 프로젝트 설정에서 C++ > 코드 생성 > 런타임 라이브러리를 /MT로 해야 된다.
