---
title: printf와 8바이트 정수
slug: printf-8byte-integer
date: 2016-08-10 18:43:00 +0900 KST
categories: [development]
markup: mmark
---

`int64_t`는 보통 아래처럼 정의되어 있다.
`uint64_t`는 여기에 `unsigned`만 붙었다.

```c
#if __WORDSIZE == 64
typedef long int       int64_t
#else
__extension__
typedef long long int  int64_t
#endif
```

`long int`를 `printf`로 출력하기 위한 서식문자는 `%ld`이고,
`long long int`를 `printf`로 출력하기 위한 서식문자는 `%lld`이다.
그래서 환경이 32비트일 때랑 64비트일 때량 써야 하는 서식문자가 다르다.

사람들은 보통 두 가지 방법을 쓰는 것 같다.

## 1. j 서식문자, intmax_t 타입

j 서식문자는 intmax_t와 대응한다. 그리고 intmax_t 타입은 아래처럼 정의됐다. int64_t랑 똑같긴 한데 정의상 언제 달라질지 모르니까...

```c
#if __WORDSIZE == 64
typedef long int                intmax_t;
typedef unsigned long int       uintmax_t;
#else
__extension__
typedef long long int           intmax_t;
__extension__
typedef unsigned long long int  uintmax_t;
#endif
```

사용은 이렇게 하면 된다.

```c
int64_t i = 1;
uint64_t ui = 2;
printf("%jd %ju\n", (intmax_t)i, (uintmax_t)ui);
```

## 2. 매크로

`inttypes.h` 중에 이렇게 정의된 부분이 있다.

```c
#if __WORDSIZE == 64
# define __PRI64_PREFIX  "l"
#else
# define __PRI64_PREFIX  "ll"
#endif

#define PRId64  __PRI64_PREFIX "d"

#define PRIu64  __PRI64_PREFIX "u"
```

사용은 이렇게 한다.

```c
int64_t i = 1;
uint64_t ui = 2;
printf("%"PRId64" %"PRIu64"\n", i, ui);
```

위 `printf`는 다음과 동일하다.

```c
/* 32bit */
printf("%lld %llu\n", i, ui);

/* 64bit */
printf("%ld %lu\n", i, ui);
```

## 참고

<http://en.cppreference.com/w/cpp/types/integer>

이 페이지 한번 읽는 게 좋을듯
