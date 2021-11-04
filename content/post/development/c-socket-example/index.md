---
title: c socket 예제
date: 2016-08-12 00:52:00 +0900 KST
categories: [development]
---

여러 예제를 보고 따라하면서 많은 이해가 됐다.

`sockaddr` 구조체와 `sockaddr_in` 구조체는 크기가 같다는 것이 인상깊었다.

아래는 작성한 예제

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <string.h>

int main () {
    int ret;

    pid_t pid = fork();
    /* client */
    if (pid == 0) {
        sleep(1);
        /* create client socket */
        int client_sockfd;
        client_sockfd = socket(AF_INET, SOCK_STREAM, 0);
        if (client_sockfd == -1) { perror("[C] socket"); return -1; }
        printf("[C] socket\n");

        /* set client_sockaddr */
        struct sockaddr_in client_sockaddr;
        memset(&client_sockaddr, 0, sizeof(struct sockaddr_in));
        client_sockaddr.sin_family      = AF_INET;
        client_sockaddr.sin_port        = htons(4000);
        inet_pton(AF_INET, "127.0.0.1", &client_sockaddr.sin_addr.s_addr);

        /* connect */
        printf("[C] connect\n");
        ret = connect(client_sockfd, (struct sockaddr *)&client_sockaddr, sizeof(struct sockaddr_in));
        if (ret == -1) { perror("[C] connect"); return -1; }

        /* send */
        printf("[C] send\n");
        int writelen;
        writelen = send(client_sockfd, "hello, world!", strlen("hello, world!"), 0);

        /* recv */
        char buf[512];
        int readlen;
        readlen = recv(client_sockfd, buf, 512, 0);
        if (readlen < 1) { perror("[C] recv"); return -1; }
        printf("[C] recv\n");
        printf("[C] buf \"%s\"\n", buf);

        close(client_sockfd);
    }
    /* server */
    else {
        /* create socket socket */
        int server_sockfd;
        server_sockfd = socket(AF_INET, SOCK_STREAM, 0);
        if (server_sockfd == -1) { perror("[S] socket"); return -1; }
        printf("[S] socket\n");

        /* set server_sockaddr */
        struct sockaddr_in server_sockaddr;
        memset(&server_sockaddr, 0, sizeof(struct sockaddr_in));
        server_sockaddr.sin_family      = AF_INET;
        server_sockaddr.sin_port        = htons(4000);
        server_sockaddr.sin_addr.s_addr = htonl(INADDR_ANY);

        /* bind */
        ret = bind(server_sockfd, (struct sockaddr *)&server_sockaddr, sizeof(struct sockaddr_in));
        if (ret == -1) { perror("[S] bind"); return -1; }
        printf("[S] bind\n");

        /* listen */
        ret = listen(server_sockfd, 5);
        if (ret == -1) { perror("[S] listen"); return -1; }
        printf("[S] listen\n");

        /* loop */
        int client_sockfd;
        struct sockaddr_in client_sockaddr;
        int socklen, readlen, writelen;
        char buf[512];
        /* for (;;) { */
            /* accept */
            memset(&client_sockaddr, 0, sizeof(struct sockaddr_in));
            socklen = sizeof(struct sockaddr_in);
            client_sockfd = accept(server_sockfd, (struct sockaddr *)&client_sockaddr, &socklen);
            if (client_sockfd == -1) { perror("[S] accept"); return -1; }
            printf("[S] accept\n");

            /* recv */
            readlen = recv(client_sockfd, buf, 512, 0);
            if (readlen == -1) { perror("[S] recv"); return -1; }
            printf("[S] recv\n");
            printf("[S] buf \"%s\"\n", buf);

            /* send */
            printf("[S] send\n");
            writelen = send(client_sockfd, buf, readlen, 0);

            /* close socket */
            close(client_sockfd);
        /* } */

        /* close server */
        close(server_sockfd);

        wait(&ret);
    }

    return 0;
}
```

실행 하면 이렇게 나온다.

```console
$ ./socket
[S] socket
[S] bind
[S] listen
[C] socket
[C] connect
[C] send
[S] accept
[S] recv
[S] buf "hello, world!"
[S] send
[C] recv
[C] buf "hello, world!"
$
```

[*] 더 공부해야할 것.

`bind`와 `listen`은 정확히 무슨 역할을 할까?

`read`와 `write`로 통신하는 것과 `send`와 `recv`로 통신하는 것은 무슨 장단점이 있을까?

aio로 동시접속을 제어하는 서버 구현하기

온르은 일찍 자야지ㅎ
