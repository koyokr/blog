---
title: pyenv로 파이썬을 설치하기 전에 해야할 것
date: 2016-10-08
taxonomies:
  categories: [etc]
---

이걸 안해주면 나중에 모듈 설치할 때 골치아프다.

```sh
export CONFIGURE_OPTS="--enable-shared"
export PYTHON_CONFIGURE_OPTS="--enable-shared"
```
