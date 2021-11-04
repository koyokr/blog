---
title: "CS:APP - 예외적인 제어흐름, 프로세스 제어"
date: 2017-05-20 17:20:00 +0900 KST
categories: [csapp]
---

이번 글에선 머리 굴려서 정리할 게 없다.

예제 코드에서 include한 csapp.h는 <http://csapp.cs.cmu.edu/3e/ics3/code/include/csapp.h>를 사용했다.

책이나 인강 보셈!

## 1. 프로세스 ID 가져오기

```c
#include <sys/types.h>
#include <unistd.h>

pid_t getpid(void);
pid_t getppid(void);
// Returns: PID of either the caller or the parent
```

각 프로세스는 0이 아닌 고유의 양수 프로세스 ID를 가진다.

getpid 함수는 호출하는 프로세스의 ID를 반환하며,
getppid 함수는 부모 프로세스의 ID를 반환한다.

## 2. 프로세스 생성과 종료

프로세스는 다음 세 가지 상태 중 하나로 생각할 수 있다.

### (1) 실행중 (Running)

프로세스는 CPU에서 실행하고 있거나 실행을 기다리고 있다. 커널에 의해 스케줄된다.

### (2) 정지 (Stopped)

프로세스는 정지된 상태다. 커널에 의해 스케줄되지 않는다.
SIGSTOP, SIGTSTP, SIGTTIN, SIGTTOU 시그널을 받게 되면
SIGCONT 시그널을 받을 때까지 정지 상태로 남는다. (나중에 자세히 설명)

### (3) 종료 (Terminated)

프로세스는 영구적으로 정지된다. 이유는 세 가지다.
프로세스를 종료하는 시그널, 메인 루틴에서 리턴, 그리고 exit 함수 호출이다.

```c
#include <stdlib.h>

void exit(int status);
// This function does not return
```

exit는 종료 상태 status로 프로세스를 종료한다.

```c
#include <sys/types.h>
#include <unistd.h>

pid_t fork(void);
// Returns: 0 to child, PID of child to parent, -1 on error
```

자식 프로세스는 부모 프로세스의 가상 주소 공간의 복사본을 갖는다.
또한 부모 프로세스가 연 파일 식별자의 복사본을 갖는다.
즉, 자식 프로세스는 부모 프로세스가 연 파일 식별자를 사용할 수 있다.

fork 함수 예제를 보고 더 이야기해보자.

```c
#include "csapp.h"

int main()
{
    pid_t pid;
    int x = 1;

    pid = Fork();
    if (pid == 0) {
        printf("child : x=%d\n", ++x);
        exit(0);
    }

    printf("parent: x=%d\n", --x);
    exit(0);
}
```

내 경우엔 다음과 같은 출력이 나왔다.

```console
$ ./fork
parent: x=0
$ child : x=2

```

여기서 다음과 같은 사실을 생각해보자.

1. fork 함수는 한 번 호출하고 두 번 리턴한다.
2. 부모 프로세스와 자식 프로세스는 동시에 돌아간다.
    * 위 출력에서는 부모 프로세스가 먼저 출력했다. 그러나 항상 부모가 먼저 출력한다고 가정할 수 없다.
3. 데이터는 중복되지만 별도의 주소 공간에 있다.
    * fork 함수가 리턴한 시점에서 부모자식 모두 동일한 값의 지역변수 x를 가지고 있다. 그러나 다른 공간에 있기 때문에 x 값의 변경이 다른 프로세스에 반영되지 않는다.
4. 파일을 공유한다.

자식은 부모에서 열린 stdout 파일을 상속했다.

## 3. 자식 프로세스의 청소

종료된 프로세스는 즉각 시스템에서 제거되지 않는다.

부모 프로세스는 종료된 자식 프로세스를 청소해야 한다.
종료됐으나 청소되지 않은 자식 프로세스는 좀비(zombie)라고 한다.

커널은 자식 프로세스가 부모를 잃으면 init 프로세스(pid: 1)가 부모가 되도록 하며,
init 프로세스는 알아서 좀비 프로세스를 정리한다.

자식 프로세스의 종료를 기다리는 함수가 있다.

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int *statusp, int options);
// Returns: PID of child if OK, 0 (if WNOHANG), or -1 on error
```

waitpid 함수는 인자 pid와 동일한 자식 프로세스를 대기 집합에 추가하며
-1을 주면 모든 자식 프로세스를 대기 집합에 추가한다.

인자 options에 줄 수 있는 옵션을 소개하면

1. WNOHANG: 아직 종료된 자식 프로세스가 없으면 0을 리턴한다.
2. WUNTRACED: 대기 집합의 프로세스가 종료되거나 정지될 때까지 호출한 프로세스의 실행을 정지한다.
3. WCONTINUED: 대기 집합에 속한 프로세스가 종료되거나 대기 집합에 속한 정지된 프로세스가 다시 실행될 때까지 호출한 프로세스의 실행을 정지한다.

statusp 인자가 NULL이 아닐 때, waitpid 함수는 리턴하게 만든 자식 프로세스의 상태 정보를
status로 인코딩한다.

wait.h 헤더에는 인코딩된 status 인자를 해석하기 위한 매크로를 제공한다.

1. WIFEXITED(status): exit 또는 리턴을 통해 정상적으로 종료됐으면 true 반환.
2. WEXITSTATUS(status): 정상 종료된 자식 프로세스의 exit status를 반환.
3. WIFSIGNALED(status): 시그널을 통해 종료됐으면 true 반환.
4. WTERMSIG(status): 자식 프로세스를 종료시킨 시그널을 반환.
5. WIFSTOPPED(status): 자식 프로세스가 정지된 상태면 true 반환.
6. WSTOPSIG(status): 자식 프로세스를 정지시킨 시그널을 반환.
7. WIFCONTINUED(status): 자식 프로세스가 SIGCONT 시그널을 받아 재시작됐으면 true 반환.

waitpid 함수의 단순화된 버전, wait 함수가 있다.

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t wait(int *statusp);
// Returns: PID of child if OK or -1 on error
```

wait( &status)은 waitpid(-1, &status, 0)과 동일하다.

waitpid 함수의 사용 예제를 보자.

```c
#include "csapp.h"
#define N 2

int main()
{
    int status, i;
    pid_t pid;

    for (i = 0; i < N; i++)
        if ((pid = Fork()) == 0)
            exit(100+i);

    while ((pid = waitpid(-1, &status, 0)) > 0) {
        if (WIFEXITED(status))
            printf("child %d terminated normally with exit status=%d\n",
                    pid, WEXITSTATUS(status));
        else
            printf("child %d terminated abnormally\n", pid);
    }

    if (errno != ECHILD)
        unix_error("waitpid error");

    exit(0);
}
```

```console
$ ./waitpid1
child 13197 terminated normally with exit status=101
child 13196 terminated normally with exit status=100
```

책에서는 exit status=100인 프로세스가 먼저 청소됐는데,
내가 했을 땐 exit status=101인 프로세스가 먼저 청소됐다.

중요한 사실은 위 코드에서 생성된 두 자식프로세스 중 어느 프로세스가
먼저 청소될 것인지 예측할 수 없다는 점이다.

## 4. 프로세스 재우기

```c
#include <unistd.h>

unsigned int sleep(unsigned int secs);
// Returns: seconds left to sleep
int pause(void);
// Always returns -1
```

sleep 함수는 일정 기간 동안 프로세스를 정지시킨다.

pause 함수는 시그널이 수신될 때까지 프로세스를 정지시킨다.

## 5. 프로그램의 로딩과 실행

현재 프로그램의 context에서 새로운 프로그램을 로드하고 실행하는 함수로 execve가 있다.

```c
#include <unistd.h>

int execve(const char *filename, const char *argv[], const char *envp[]);
// Does not return if OK; returns -1 on error
```

execve 함수는 절대로 리턴하지 않는다. 에러만 안나면.

execve는 filename을 로드하고 시작 코드를 호출한다.
시작 코드는 스택을 설정하고 제어를 새 프로그램의 메인 루틴으로 전달한다.

이 루틴의 프로토타입은 우리가 잘 아는 그것이다.

int main(int argc, char *argv[], char *envp[]);

1. argc: argv 배열의 NULL이 아닌 포인터의 수
2. argv: 인자 배열
3. envp: 환경 배열

리눅스는 환경 배열을 조작하기 위한 함수를 제공한다.

```c
#include <stdlib.h>

char *getenv(const char *name);
// Returns: pointer to name if it exists, NULL if no match
```

getenv 함수는 환경 배열에서 "name=value" 문자열을 검색하며,
찾으면 value에 대한 포인터를 반환한다.

```c
#include <stdlib.h>

int setenv(const char *name, const char *newvalue, int overwrite);
// Returns: 0 on success, -1 on error
void unsetenv(const char *name);
// Returns: nothing
```

setenv 함수는 overwrite가 0이 아니라면 value를 newvalue로 교체한다.
만약 name을 찾지 못하면 "name=newvalue"를 배열에 추가한다.

unsetenv 함수는 검색한 "name=value" 문자열을 삭제한다.

## 6. 프로그램을 실행하기 위해 fork와 execve 사용하기

생략

## 7. 마무리

다음엔 시그널에 대해서 공부

## 출처

'Computer Systems A Programmer's Perspective (3rd Edition)'
