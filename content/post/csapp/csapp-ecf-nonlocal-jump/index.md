---
title: "CS:APP - 예외적인 제어흐름, 비지역성 점프"
slug: csapp-ecf-nonlocal-jump
date: 2017-05-27 15:20:00 +0900 KST
categories: [csapp]
markup: mmark
---

## 1. 비지역성 점프(Nonlocal jumps)

C는 setjmp와 longjmp 함수를 제공한다.

```c
#include <setjmp.h>

int setjmp(jmp_buf env);
int sigsetjmp(sigjmp_buf env, int savesigs);
// Returns: 0 from setjmp, nonzero from longjmps
```

setjmp 함수는 현재 '호출하는 환경'을 env 버퍼에 저장하며 0을 리턴한다.

'호출하는 환경'은 프로그램 카운터, 스택 포인터, 범용 레지스터 등을 포함한다.

setjmp가 리턴하는 값은 switch문이나 if문의 조건으로 사용될 수 있다.
그리고 리턴값이 변수에 할당되면 안된다.

```c
#include <setjmp.h>

void longjmp(jmp_buf env, int retval);
void siglongjmp(sigjmp_buf env, int savesigs);
// Never returns
```

longjmp 함수는 호출하는 환경을 env 버퍼에서 복원하고,
가장 최근의 setjmp 호출에서 리턴한다.

이 때, setjmp는 longjmp의 인자 retval를 리턴한다.

예제 보면 어떻게 쓰는지 감이 온다.

```c
#include "csapp.h"

jmp_buf buf;

int error1 = 0;
int error2 = 0;

void foo(void), bar(void);

int main()
{
    switch(setjmp(buf)) {
    case 0:
        foo();
        break;
    case 1:
        printf("Detected an error1 condition in foo\n");
        break;
    case 2:
        printf("Detected an error2 condition in foo\n");
        break;
    default:
        printf("Unknown error condition in foo\n");
    }
    exit(0);
}

void foo(void)
{
    if (error1)
        longjmp(buf, 1);
    bar();
}

void bar(void)
{
    if (error2)
        longjmp(buf, 2);
}
```

이렇게 소프트웨어 예외 처리용으로 쓸 수 있고.

```c
#include "csapp.h"

sigjmp_buf buf;

void handler(int sig)
{
    siglongjmp(buf, 1);
}

int main()
{
    if (!sigsetjmp(buf, 1)) {
        Signal(SIGINT, handler);
        Sio_puts("starting\n");
    }
    else
    Sio_puts("restarting\n");

    while(1) {
        Sleep(1);
        Sio_puts("processing...\n");
    }
    exit(0);
}
```

이렇게 시그널을 활용할 수도 있다.

시그널을 활용할 때는 다음을 주의하자.

1. siglongjmp를 호출하는 핸들러를 설치하기 전에 sigsetjmp를 먼저 설정하라.
    * sigsetjmp 환경 설정 전에 시그널을 수신받는 상황을 방지하기 위해서다.
2. sigsetjmp와 siglongjmp는 async-signal-safe function이 아니다.
    * siglongjmp에서 도달가능한 모든 코드는 안전한 함수만을 호출해야 한다.

## 2. 마무리

예외적 제어흐름 ECF에 대해서 모두 알아봤다.

## 출처

'Computer Systems A Programmer's Perspective (3rd Edition)'
