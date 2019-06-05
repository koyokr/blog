---
title: system 대신 fork - exec 쓰기
slug: fork-exec-instead-of-system
date: 2016-08-17 02:23:00 +0900 KST
categories: [development]
markup: mmark
---

> [fork - exec 대신 posix_spawn 쓰기](/post/posix_spawn-instead-of-fork-exec/)

c언어에서 기존 프로세스를 종료시키지 않고 외부 명령어를 실행하고 싶을 때가 있다.

`system` 함수는 보안상 문제가 많아서 안쓰는 게 좋다고 하고...
그러면 `exec` 계열 함수를 써야되는데 이 함수는 현재 프로세스를 실행한 외부 프로그램으로 교체해버린다.

예전에 이 문제 때문에 고민을 많이 했는데,
책을 보니까 `fork` 함수로 자식 프로세스를 생성해서 `exec` 계열 함수를 쓰면 된다고 한다.
난 왜 두개 같이 쓸 생각을 못햇대

아래는 내가 사용하고 싶었던 예시ㅋㅋ

```c
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>
#include <stdio.h>

void _execv(const char *path, char *const argv[]) {
    int ret;
    pid_t pid;

    pid = fork();
    if (pid == -1) { perror("fork"); exit(EXIT_FAILURE); }
    if (pid == 0) {
        ret = execv(path, argv);
        if (ret == -1) { perror("execv"); exit(EXIT_FAILURE); }
    }
    waitpid(-1, NULL, 0);
}

int main() {
    char path[] = "/sbin/iptables";
    char *argv_A[] = { "iptables", "-A", "OUTPUT", "-j", "NFQUEUE", "--queue-num", "0", NULL };
    char *argv_D[] = { "iptables", "-D", "OUTPUT", "-j", "NFQUEUE", "--queue-num", "0", NULL };
    char *argv_L[] = { "iptables", "-L", NULL };

    printf("\n[*] iptables -A OUTPUT -j NFQUEUE --queue-num 0\n");
    _execv(path, argv_A);
    _execv(path, argv_L);

    printf("\n[*] iptables -D OUTPUT -j NFQUEUE --queue-num 0\n");
    _execv(path, argv_D);
    _execv(path, argv_L);

    return 0;
}
```
