---
title: "CS:APP - 예외적인 제어흐름, 시그널 2"
date: 2017-05-25 15:34:00 +0900 KST
categories: [csapp]
---

이 글에서는 동시성을 다룬다.

## 1. 시그널 블록, 블록 해제

1. 묵시적 블록 방법
    * 커널은 핸들러에 의해 처리되고 있는 모든 대기 시그널의 처리를 막는다.
2. 명시적 블록 방법
    * `sigprocmask`와 같은 함수를 사용해 명시적으로 블록하거나 블록 해제할 수 있다.

```c
#include <signal.h>

int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signum);
int sigdelset(sigset_t *set, int signum);
// Returns: 0 if OK, -1 on error

int sigismember(const sigset_t *set, int signum);
// Returns: 1 if member, 0 if not, -1 on error
```

* `sigprocmask` 함수는 현재 블록된 시그널의 집합(blocked 비트 벡터)을 변경한다. 함수의 동작은 how 인자의 값에 따라 다르다.
* `SIG_BLOCK`: blocked |= set. set에 있는 시그널들을 blocked에 추가한다.
* `SIG_UNBLOCK`: blocked  &= ~set. set에 있는 시그널들을 blocked에서 제거한다.
* `SIG_SETMASK`: blocked = set. `oldset`이 NULL이 아니면 blocked 비트 벡터의 이전 값이 `oldset`에 저장된다.
* `sigemptyset` 함수는 set를 빈 집합으로 초기화하고, `sigfillset` 함수는 모든 시그널을 set에 추가한다.
* `sigaddset` 함수는 `signum`을 set에 추가하고, `sigdelset` 함수는 `signum`을 set에서 지운다.
* `sigismember` 함수는 `signum`이 set의 멤버인지 확인한다.

## 2. 시그널 핸들러 작성

핸들러를 다루는 데 어려운 측면이 여러가지 있다.

1. 핸들러는 메인 프로그램과 동시에 돌아가고, 같은 전역변수를 공유한다.
2. 시그널의 수신이 언제될 수 있는지 종종 직관적이지 않다.
3. 다른 시스템은 다른 시그널 처리 방식을 가진다.

책에서 제시하는 '안전하고, 정확하고, 이식성이 높은 시그널 핸들러 작성을 위한 방법'을 정리한다.

### (1) 안전한 시그널의 처리

만일 핸들러가 메인 프로그램과 동시에 같은 메모리에 접근하게 되면 우리는 그 결과를 예측할 수 없다.

다음 6가지 수칙을 잘 새겨듣자.

#### G0. " _핸들러는 가능한 한 간단하게 유지하라._ "

핸들러를 작고 단순하게 유지하면 문제를 쉽게 피할 수 있다.

#### G1. " _핸들러에서 비동기성-시그널-안전한 함수만 호출하라._ "

비동기-시그널-안전한 함수(Async-signal-safe function)나 안전한 함수(safe function)는
재진입 가능하거나, 다른 시그널 핸들러에 의해 중단될 수 없다.

즉, 시그널 핸들러에게 안전하게 호출될 수 있다.
async-signal-safe function의 목록은 'man 7 signal'에서 확인할 수 있다.

예를 들어, 시그널 핸들러에서 출력을 하기 위해 `printf` 함수를 호출하는 것은 안전하지 않다. 유일하게 안전한 방법은 `write` 함수를 사용하는 것이다.

다음과 같이 안전한 함수(<http://csapp.cs.cmu.edu/3e/ics3/code/src/csapp.c>)를 작성해서 사용하는 것도 좋다.

```c
ssize_t sio_puts(char s[])
{
    return write(STDOUT_FILENO, s, sio_strlen(s));
}

ssize_t sio_putl(long v)
{
    char s[128];

    sio_ltoa(v, s, 10);
    return sio_puts(s);
}

void sio_error(char s[])
{
    sio_puts(s);
    _exit(1);
}
```

아래 코드는 `SIGINT`를 잡으면 `"Caught SIGINT!\n"`를 출력하는 핸들러의 예제다. 이전 글 예제와 비교해보자.

```c
#include "caspp.h"

void sigint_handler(int sig)
{
    Sio_puts("Caught SIGINT!\n");
    _exit(0);
}
```

#### G2. " _errno를 저장하고 복원하라._ "

많은 async-signal-safe function은 에러를 가지고 리턴할 때 전역변수 `errno`를 설정한다.
때문에 errno에 접근하는 프로그램의 다른 부분에서 문제가 생길 수 있다.

핸들러는 진입할 때 `errno`를 지역변수에 저장하고,
핸들러가 리턴하기 전에 `errno`을 복원해야 한다.

핸들러가 `_exit` 함수를 호출해서 종료한다면 당연히 신경쓰지 않아도 된다.

#### G3. " _모든 시그널을 블록시켜서 공유된 전역 자료구조들로의 접근을 보호하라._ "

상상해보자. 메인 프로그램이 특정한 자료구조에 접근하고 있는데,
해당 자료구조에 접근하는 핸들러가 호출되는 상황을.

우리는 이러한 상황을 막기 위해 메인 프로그램이 특정 자료구조에 접근하는 동안
시그널을 일시적으로 블록할 수 있다.

#### G4. " _전역변수들을 volatile로 선언하라._ "

전역변수 `g`를 갱신하는 핸들러와 전역변수 `g`를 읽는 메인 프로그램이 있다고 가정하자.

최적화 컴파일러는 메인 프로그램에서는 `g` 값을 레지스터에 담아서 사용해도
안전하다고 판단할 수 있다.

```c
volatile int g;
```

우리는 위와 같이 `volatile`형 지시자를 사용해서 매번 `g`를 읽을 때마다
메모리에서 읽어오도록 지시할 수 있다.

#### G5. " _sig_atomic_t로 플래그들을 선언하라._ "

c 언어는 `sig_atomic_t` 자료형에 대해 원자형(atomic) 읽기, 쓰기를 보장한다.

```c
volatile sig_atomic_t flag;
```

위와 같이 선언하면 시그널의 일시적인 블록 없이 읽기, 쓰기를 안전하게 수행할 수 있다.

단, `flag++`나 `flag += 10`과 같이 다수의 인스트럭션을 필요로 하는 동작에서는
안전성을 보장하지 않으니 주의하자.

개인적인 생각인데 `volatile` 대신에 C11에서 `_Atomic`을, C++11에선 `std::atomic`을 사용하는 것도 좋을 것 같다.

### (2) 정확한 시그널 처리

이전 글에서 특정 타입의 pending 시그널은 최대 한 개만 존재할 수 있다고 했다.

특정 타입의 시그널에 대한 핸들러를 실행하는 동안,
2개 이상 동일한 타입의 시그널을 해당 프로세스에 보내면
첫번째 시그널만 큐에 남는다는 이야기다.

새로운 클라이언트가 확인될 때마다 fork로 자식 프로세스를 생성하는 웹서버를 상상해보자.

이 웹서버는 자식 프로세스가 종료하거나 정지할 때
커널이 보내주는 `SIGCHLD` 시그널에 대응하는 핸들러로 자식들을 청소한다.

핸들러 부분을 다음과 같이 작성해보자.

```c
#include "csapp.h"

void handler1(int sig)
{
    int olderrno = errno;

    if (waitpid(-1, NULL, 0) < 0)
        sio_error("waitpid error");
    Sio_puts("Handler reaped child\n");
    Sleep(1);
    errno = olderrno;
}

int main()
{
    int i, n;
    char buf[MAXBUF];

    if (signal(SIGCHLD, handler1) == SIG_ERR)
        unix_error("signal error");

    for (i = 0; i < 3; i++) {
        if (Fork() == 0) {
            printf("Hello from child %d\n", getpid());
            exit(0);
        }
    }

    if ((n = read(STDIN_FILENO, buf, sizeof(buf))) < 0)
        unix_error("read");

    printf("Parent processing input\n");
    while(1);

    exit(0);
}
```

프로그램을 실행하니 다음과 같은 결과가 나왔다.

```console
$ ./signal1
Hello from child 34737
Handler reaped child
Hello from child 34736
Hello from child 36735
Handler reaped child

Parent processing input
^Z
[1]+  Stopped                 ./signal1
$ ps t
   PID TTY     STAT   TIME COMMAND
 34448 pts/1   Ss     0:00 -bash
 34734 pts/1   T      0:01 ./signal1
 34736 pts/1   Z      0:00 [signal1] <defunct>
 34738 pts/1   R+     0:00 ps t
$
```

세번째 `SIGCHLD` 시그널이 수신되지 못하고 좀비 프로세스가 생긴 걸 확인할 수 있다.

핸들러를 다음과 같이 수정하면 종료된 자식 프로세스를 소거하지 못하는 문제를 해결할 수 있다.

```c
void handler2(int sig)
{
        int olderrno = errno;

        while (waitpid(-1, NULL, 0) > 0)
                Sio_puts("Handler reaped child\n");
        if (errno != ECHILD)
                Sio_error("waitpid error");
        Sleep(1);
        errno = olderrno;
}
```

프로그램 실행 결과는 다음과 같다.

```console
$ ./signal2
Hello from child 34855
Hello from child 34854
Handler reaped child
Hello from child 36853
Handler reaped child

Parent processing input
^Z
[1]+  Stopped                 ./signal1
$ ps t
   PID TTY     STAT   TIME COMMAND
 34448 pts/1   Ss     0:00 -bash
 34852 pts/1   T      0:01 ./signal1
 34856 pts/1   R+     0:00 ps t
$
```

### (3) 호환성 있는 시그널 핸들링

유닉스 시스템간 서로 시그널 처리 방식이 다른 경우가 있다.

일부 오래된 유닉스 시스템에서는 `signal` 함수로 설치한 핸들러가 시그널을 잡으면
해당 시그널에 대한 동작이 기본 동작으로 복원된다.

그리고 일부 오래된 유닉스 시스템에서 `read`, `wait`, `accept`와 같은
느린 시스템 콜(slow system call)은 시그널 핸들러가 리턴할 때
`errno`를 `EINTR`로 설정해서 사용자에게 즉시 리턴한다.

위와 같은 호환성 문제를 해결하기 위해 POSIX 표준은 `sigaction` 함수를 정의하고 있다.

```c
#include <signal.h>

int sigaction(int signum, struct sigaction *act, struct sigaction *oldact);
// Returns: 0 if OK, -1 on error
```

`sigaction`을 매번 설정해주는 건 불편하기 때문에 책에서는 아래 같은 래퍼 함수 사용을 추천한다.

```c
handler_t *Signal(int signum, handler_t *handler)
{
    struct sigaction action, old_action;

    action.sa_handler = handler;
    sigemptyset(&action.sa_mask);
    action.sa_flags = SA_RESTART;

    if (sigaction(signum, &action, &old_action) < 0)
        unix_error("Signal error");
    return old_action.sa_handler;
}
```

## 3. 치명적인 동시성 버그를 피하기 위한 흐름 동기화

어설픈 동기화 코드는 race condition를 가진다. 주의하자.

예제가 궁금하면 아래 링크에서 `procmask1.c`의 문제가 뭔지 생각해보고,
`procmask2.c`에서 어떻게 문제를 해결하는지 보도록 하자.

<http://csapp.cs.cmu.edu/3e/ics3/code/ecf/procmask1.c>

<http://csapp.cs.cmu.edu/3e/ics3/code/ecf/procmask2.c>

## 4. 명시적인 시그널 대기

다음 예제를 보자.

```c
#include "csapp.h"

volatile sig_atomic_t pid;

void sigchld_handler(int s)
{
    int olderrno = errno;
    pid = Waitpid(-1, NULL, 0);
    errno = olderrno;
}

void sigint_handler(int s)
{
}

int main(int argc, char *argv[])
{
    sigset_t mask, prev;

    Signal(SIGCHLD, sigchld_handler);
    Signal(SIGINT, sigint_handler);
    Sigemptyset(&mask);
    Sigaddset(&mask, SIGCHLD);

    while (1) {
        Sigprocmask(SIG_BLOCK, &mask, &prev); // Block SIGCHLD
        if (Fork() == 0)
            exit(0);

        pid = 0;
        Sigprocmask(SIG_SETMASK, &prev, NULL); // Unblock SIGCHLD

        while (!pid)
            ;

        printf(".");
    }
    exit(0);
}
```

매 루프마다 `Fork` 함수를 실행하고 다음 루프를 돌기 전에 자식 프로세스의 종료를 기다리는 코드다.

`SIGCHLD`를 수신할 때까지 다음 두 줄의 코드로 명시적으로 대기하는 것을 확인할 수 있다.

```c
while (!pid)
    ;
```

이 코드는 확실하게 동작하지만 시스템 자원을 너무 많이 먹는다는 문제가 있다.

그렇다면 아래와 같은 코드는?

```c
while (!pid)
    pause();
```

이 코드는 race condition을 갖는다. `while` 테스트와 `pause` 호출 사이에 `SIGCHLD`가 호출되면 프로그램은 영영 멈출 것이다.

그렇다면 아래와 같은 코드는?

```c
while (!pd)
    sleep(1);
```

race condtion도 없고, 시스템 자원도 많이 먹지 않는다. 그러나 프로그램이 느려진다.

이 문제를 해결하기 위해 `sigsuspend` 함수를 사용할 수 있다.

```c
#include <signal.h>

int sigsuspend(const sigset_t *mask);
// Returns: -1
```

위 함수는 아래 세 줄 코드의 원자형(atomic) 버전이다. 즉, race condition 문제에서 자유롭다.

```c
sigprocmask(SIG_SETMASK, &mask, &prev);
pause();
sigprocmask(SIG_SETMASK, &prev, NULL);
```

그럼 이제 `sigsuspend` 함수를 사용해서 예제를 수정해보자.

```c
#include "csapp.h"

volatile sig_atomic_t pid;

void sigchld_handler(int s)
{
        int olderrno = errno;
        pid = Waitpid(-1, NULL, 0);
        errno = olderrno;
}

void sigint_handler(int s)
{
}

int main(int argc, char **argv)
{
        sigset_t mask, prev;

        Signal(SIGCHLD, sigchld_handler);
        Signal(SIGINT, sigint_handler);
        Sigemptyset(&mask);
        Sigaddset(&mask, SIGCHLD);

        while (1) {
                Sigprocmask(SIG_BLOCK, &mask, &prev); // Block SIGCHLD
                if (Fork() == 0)
                        exit(0);

                pid = 0;
                while (!pid)
                        sigsuspend(&prev);

                Sigprocmask(SIG_SETMASK, &prev, NULL); // Optionally unblock SIGCHLD

                printf(".");
        }
        exit(0);
}
```

위 코드는 시스템 자원을 과도하게 먹지 않으며, 잠재적인 race condtion도 없고,
시간을 쓸데없이 소모하지도 않는다.

## 5. 마무리

책의 마지막 장에 가면 동시성 프로그래밍을 따로 다룬다.

빨리 거기까지 읽어야지.

## 출처

'Computer Systems A Programmer's Perspective (3rd Edition)'
