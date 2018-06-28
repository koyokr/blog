---
title: c 공부 정리 2
slug: c-study-2
date: 2016-08-10 02:32:00 +0900 KST
categories: [study]
markup: mmark
---

최근에 공부한 함수들 벌써 다 까먹어서 여기다 적기로 했당

## 1. open

### (1) O_ASYNC, O_SYNC

둘이 헷갈린다. O_ASYNC는 설명을 봐도 잘 모르겠어서 나중에 보기로 했음

O_SYNC 모드 설정해주면 디스크에 버퍼 바로바로 넘어간다.
fsync() 함수로 해줄거면 쓰지 말고

### (2) O_CLOEXEC

코드에서 외부 프로그램 실행할 때 파일 디스크립터 닫아준다.
안닫아주면 상속되나? 궁금

### (3) O_EXCL

파일이 있으면 안염. O_CREAT하고 같이 쓰는 경우가 많은듯

### (4) O_NONBLOCK

read 함수를 썼는데 들어오는게 없으면 평생 기다리게 될 수 잇음,
이 모드 쓰면 읽을 데이터 없으면 read는 -1 리턴함. write도

### (5) O_TRUNC

파일에 내용 있으면 삭제

### (6) O_NOFOLLOW

심볼릭 링크면 안염. 꼼수방지용??

### (7) O_NOATIME

읽으면 파일 액세스 시간이 바뀌는 식으로 시스템에 조금이라도 부하를 주게 되는데
이 모드면 그게 없다는듯

## 2. fsync, fdatasync

fsync는 버퍼를 비우고, fdatasync는 데이터만 비우고.
인자 줄 거 없고 에러나면 -1 반환. 라이브러리는 unistd.h

fflush 대신 써도 댐

## 3. lseek, pread, pwrite

lseek은 파일 위치 변경, 세번째 인자로 SEEK_SET, SEEK_CUR, SEEK_END가 있음.

pread랑 pwrite는 read랑 write과 거의 같은데
마지막 인자에 좌표값 줘서 읽거나 쓸 수 있음

## 4. truncate, ftruncate

해당 파일의 사이즈를 정하는 함수, truncate는 경로를 받고,
ftruncate는 파일 디스크립터를 받음

## 5. select, pselect

```c
int select(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
int pselect(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, const struct timespec *timeout, const sigset_t *sigmask);
```

이렇게 생겨먹었다. select 함수는 timeval 구조체를 받고,
pselect 함수는 simespec 구조체를 받는다.
pselect는 추가로 sigmask도 받는데, 이게 가장 큰 차이라고 한다.
둘 다 sys/select.h 헤더를 필요로 함

대충 이런 식으로 쓰면 되는 것 같은데...
아직 시그널 부분을 어떻게 써야되는지 모르겠다.

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/select.h> // 여기에서 time.h 인클루드함
#include <signal.h>

struct timespec ts;
fd_set readfds;
sigset_t sigs;

FD_ZERO(&readfds); // 초기화
FD_SET(STDIN_FILENO, &readfds); // stdin을 readfds에 설정?

/* 시간이랑 시그널 설정 생략 */

pselect(STDIN_FILENO + 1, &readfds, NULL, NULL, &ts, &sigs); // +1 해줘야댐
if(FD_ISSET(STDIN_FILENO, &readfds)) { /* stdin에 읽을 데이터가 있으면 내가 하고 싶은 거 */ }
```

반복문으로 돌릴 생각이라면 fd_set타입으로 readfds 상태를 백업해둘 공간이 필요하다. 안그러면 못쓰

## 6. poll, ppoll

select는 너무 기능이 적어~ 할 때 쓰는 것이 바로 poll 함수.
아래처럼 생김

```c
#include <poll.h>
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

쓰는 건 아래처럼.

```c
#include <stdio.h>
#include <unistd.h>
#include <poll.h>

struct pollfd fds[2];
int ret;

fds[0].fd = STDIN_FILENO;
fds[0].events = POLLIN;
fds[1].fd = STDOUT_FILENO;
fds[1].events = POLLOUT;

ret = poll(fds, 2, TIMEOUT * 1000);

if (!ret) { printf("%d seconds elapsed.\n", TIMEOUT); return 0; }
if (fds[0].revents & POLLIN) printf("stdin is readable\n");
if (fds[1].revents & POLLOUT) printf("stdout is writable\n");
```

select처럼 pselect도 있는데 리눅스에만 있음.
_GNU_SOURCE 정의해줘야 쓸 수 있음

```c
#define _GNU_SOURCE
#include <poll.h>
int ppoll(struct pollfd *fds, nfds_t nfds, const struct timespec *timeout, const sigset_t *sigmask);
```

timespec 구조체 넘조아

## 7. epoll

수백 명이 서버에 접속했는데 select나 poll 함수를 써서 처리하는 건 느리고 비효율적이라고 한다.

그래서 epoll 함수를 쓰는데 대충 이런 흐름으로 사용하는 것 같다.

```c
#include <sys/epoll.h> /* 다른 헤더는 생략 */

struct epoll_event ev;
struct epoll_event evlist[1];
/* 다른 선언도 생략 */

/* epoll 생성, epoll_create는 사이즈를 인자로 넣어주고(이제 안씀), epoll_create1은 플래그를 인자로 넣어준다.
   아 근데 무슨 플래그 넣어줘야하는지 아무리 검색해도 안나옴;; 그래서 그냥 0 넣음 */
epfd = epoll_create1(0);

/* 구조체에 무슨 이벤트인지랑 무슨 파일디스크립터를 넣을지 설정 */
ev.events = EPOLLIN; // 받을거임
ev.data.fd = STDIN_FILENO; // stdin을

/* 설정이 끝났으면 epoll_ctl 함수로 추가시켜준다. stdin을 추가해준단 뜻 */
epoll_ctl(epfd, EPOLL_CTL_ADD, STDIN_FILENO, &ev);

/* 이벤트 기다리기. poll 함수나 select 함수랑 비슷한 역할. 세번째 인자는 evlist 배열 갯수로 하면 되고, 마지막 인자는 시간인데 -1로 하면 안 기다림 */
epoll_wait(epfd, evlist, 1, -1);

if (evlist[1].events & EPOLLIN) { /* stdin이 들어올 때 내가 하고 싶은 거 */ }
```

아 간단하게 쓰려고 했는데 넘 길다.

이거 어떻게 쓰는지 감 잡으려면 내가 epoll 함수 쓴 소켓 서버 만들어봐야겠다.

더 써야 할게 많지만 잠자러
