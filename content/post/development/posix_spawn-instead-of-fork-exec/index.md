---
title: fork - exec 대신 posix_spawn 쓰기
date: 2017-02-19 19:01:00 +0900 KST
categories: [development]
---

`fork` - `exec` 조합으로 외부 프로그램을 실행하는 방법은 별로 권장하지 않는다고 한다.

대신 `posix_spawn` 함수를 추천하는데...

예전에 `system` 함수 대신 `fork` - `exec`를 쓴다고 한 글
[system 대신 fork - exec 쓰기](/post/fork-exec-instead-of-system/)에서 사용한 예시를 가져와보면

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

이걸 고쳐보면

```c
#include <stdio.h>
#include <spawn.h>
#include <sys/wait.h>

int execute(const char *path, char *const argv[]) {
    pid_t pid;
    int ret = posix_spawn(&pid, path, NULL, NULL, argv, NULL);

    int status;
    waitpid(pid, &status, 0);

    return ret;
}

int main(int argc, char *argv[]) {
    const char *path = "/sbin/iptables";
    char *const argv_A[] = { "iptables", "-A", "OUTPUT", "-j", "NFQUEUE", "--queue-num", "0", NULL };
    char *const argv_D[] = { "iptables", "-D", "OUTPUT", "-j", "NFQUEUE", "--queue-num", "0", NULL };
    char *const argv_L[] = { "iptables", "-L", NULL };

    printf("\n[*] iptables -A OUTPUT -j NFQUEUE --queue-num 0\n");
    execute(path, argv_A);
    execute(path, argv_L);

    printf("\n[*] iptables -D OUTPUT -j NFQUEUE --queue-num 0\n");
    execute(path, argv_D);
    execute(path, argv_L);

    return 0;
}
```

짜잔~

`spawn.h`를 include해서 사용하자.
