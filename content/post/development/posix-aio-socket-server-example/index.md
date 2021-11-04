---
title: posix aio socket server 예제
date: 2016-08-15 00:08:00 +0900 KST
categories: [development]
---

열심히 코드를 작성하면서 검색하는데 aio는 소켓을 지원하지 않는다는 글이 있어서 깜짝 놀랐다 ㄷㄷ

<http://www.clien.net/cs2/bbs/board.php?bo_table=kin&wr_id=2298319>

댓글 보니까 된다고... 실제로도 되서 다행이다.

비동기/블록 방식에는 주로 `select`, `poll`, `epoll` 함수가 쓰이고, 비동기/논블록 방식에는 aio 함수가 쓰인다.

aio 라이브러리를 쓰면 입출력을 기다리고 있을 필요 없이 편하게 딴짓할 수 있다는 점이 좋은 것 같다.

코드 작성하면서 엄청엄청 많이 본 글들. 1번째 글에서 개념 이해한 다음에, 2번째 글에 있는 예제 따라했다.

<http://djkeh.github.io/articles/Boost-application-performance-using-asynchronous-IO-kor/>

<http://jeremyko.blogspot.kr/2012/10/linux-asynchronous-io.html>

<http://youmin3.egloos.com/2188597>

아래는 작성한 에코 소켓 서버 예제

최근본은 <https://github.com/koyokr/aio_echo_server> 에 있당.

```c
/* gcc -o aio aio.c -lrt */
#include <aio.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

void aio_completion_handler(sigval_t sigval);

void set_aiocb(struct aiocb *cbp, int fd) {
    cbp->aio_fildes     = fd;
    cbp->aio_lio_opcode = LIO_READ; /* for lio_listio */
    cbp->aio_buf        = malloc(BUFSIZ);
    cbp->aio_nbytes     = BUFSIZ;
    cbp->aio_offset     = 0;
    cbp->aio_sigevent.sigev_notify            = SIGEV_THREAD;
    cbp->aio_sigevent.sigev_notify_function   = aio_completion_handler;
    cbp->aio_sigevent.sigev_notify_attributes = NULL;
    cbp->aio_sigevent.sigev_value.sival_ptr   = cbp;
}

void perror_exit(const char *s) {
    perror(s);
    exit(EXIT_FAILURE);
}

int g_fd;

int main(int argc, char *argv[]) {
    if (argc != 2) {
        printf("usage: %s <port>\n", argv[0]);
        return 1;
    }

    struct sockaddr_in ssin = {
        .sin_family      = AF_INET,
        .sin_port        = htons(atoi(argv[1])),
        .sin_addr.s_addr = htonl(INADDR_ANY)
    };

    /* socket, bind, listen */
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd == -1)
        perror_exit("socket");
    if (bind(fd, (struct sockaddr *)&ssin, sizeof(struct sockaddr)) == -1)
        perror_exit("bind");
    if (listen(fd, 5) == -1)
        perror_exit("listen");

    /* set */
    struct aiocb cb[2] = {};
    struct aiocb *cblist[2] = { &cb[0], &cb[1] };

    set_aiocb(cblist[0], STDIN_FILENO);
    set_aiocb(cblist[1], fd);

    g_fd = fd;

    /* lio_listio */
    if (lio_listio(LIO_NOWAIT, cblist, 2, NULL) == -1)
        perror_exit("lio_listio");

    while (1)
        sleep(86400);

    return 0;
}

void aio_completion_handler(sigval_t sigval) {
    struct aiocb *cbp = sigval.sival_ptr;
    if (aio_error(cbp) == -1)
        perror_exit("aio_handler");

    char *buf = (char *)cbp->aio_buf;
    int buflen = aio_return(cbp);

    if (cbp->aio_fildes == STDIN_FILENO) {
        /* exit */
        buf[buflen] = '\0';
        if (!strcmp(buf, "exit\n"))
            exit(EXIT_SUCCESS);

        /* aio_read */
        if (aio_read(cbp) == -1)
            perror_exit("aio_read");
    }
    else if (cbp->aio_fildes == g_fd) {
        /* accept */
        struct sockaddr_in csin;
        socklen_t socklen = sizeof(struct sockaddr);

        int cfd = accept(g_fd, (struct sockaddr *)&csin, &socklen);
        if (cfd == -1)
            perror_exit("accept");
        printf("\x1b[0;32m[*] aceept\x1b[0m\n");

        /* new client */
        struct aiocb *ccbp = malloc(sizeof(struct aiocb));
        set_aiocb(ccbp, cfd);

        /* aio_read */
        if (aio_read(cbp)  == -1 ||
            aio_read(ccbp) == -1)
            perror_exit("aio_read");
    }
    else {
        int cfd = cbp->aio_fildes;
        if (buflen <= 0) {
            /* close */
            close(cfd);
            printf("\x1b[0;31m[*] close\x1b[0m\n");

            /* delete client */
            free(buf);
            free(cbp);
        }
        else {
            /* write */
            buf[buflen] = '\0';
            if (write(cfd, buf, buflen) != buflen)
                perror_exit("write");
            printf("%s", buf);

            /* aio_read */
            if (aio_read(cbp) == -1)
                perror_exit("aio_read");
        }
    }
}
```

더 공부할 것

1. `aio_error` 함수 공부하기: `aio_error` 리턴값을 제대로 알지 못해서 코드 작성할 때 이 부분은 좀 슬쩍 넘어갔다
2. `volatile void *`로 선언된 `aio_buf` 변수를 캐스팅하지 않고 gcc에서 컴파일하니까 경고가 뜬다. 일단 캐스팅은 했는데 다른 방법이 있는지 알아봐야겠다.
3. posix aio 말고 libaio도 써보기
4. callback 함수는 어떻게 구현됐는지 알아보기
