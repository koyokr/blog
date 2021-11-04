---
title: c 공부 정리 1
date: 2016-08-09 02:49:00 +0900 KST
categories: [development]
---

## 1. 자료형 크기

long은 4바이트 고정이고 int가 변동한다고 생각해왔는데 거꾸로였다.
long long int는 32비트 환경에서 8바이트고,
long double은 64비트 환경에서는 16바이트인 것도 오늘 알았다.
아니면 배워놓고 까먹었거나.
그런데 이것도 컴파일러마다 다를 수 있다고 하니까 난 stdint.h만 굳게 믿기로 했다.

그런데 방금 printf로 int64_t 값 출력하려니까 32비트 환경에서는 %lld 써야 하고,
64비트 환경에서는 %ld 써야 한다.

```console
$ ./type32
### 1 Byte = 8 bit ###
long : 4 byte
int : 4 byte
unsigned int : 4 byte
long int : 4 byte
unsigned long int : 4 byte
long long int : 8 byte
float : 4 byte
double : 8 byte
long double : 12 byte
(void *) : 4 byte

INT_MAX = 2147483647
UINT_MAX = 4294967295d
LONG_MAX = 2147483647
ULONG_MAX = 4294967295d
koyo@vbox:~/bob/sort$ ./type64
### 1 Byte = 8 bit ###
long : 8 byte
int : 4 byte
unsigned int : 4 byte
long int : 8 byte
unsigned long int : 8 byte
long long int : 8 byte
float : 4 byte
double : 8 byte
long double : 16 byte
(void *) : 8 byte

INT_MAX = 2147483647
UINT_MAX = 4294967295d
LONG_MAX = 9223372036854775807
ULONG_MAX = 18446744073709551615d
```

## 2. 코드 수행 시간 측정

timespec 구조체가 어떻게 생겼는지 궁금해서 time.h를 보니까 요렇게 생겼다.

```c
struct timespec
  {
    __time_t tv_sec;            /* Seconds.  */
    __syscall_slong_t tv_nsec;  /* Nanoseconds.  */
  };
```

__time_t는 time_t로 정의됐는데 long int형으로 보고 사용하면 될 것 같다.

그리고 __syscall_slong_t은 time_t와 달리 __extension__인가 뭔가 붙어있고 복잡하던데,
이것도 long이나 long int로 선언된다고 보고 쓰면 될듯.
그런데 나노초는 4바이트 정수로 해도 범위가 넘을 일이 없을 것 같은데...

타입이 8바이트일 때는 printf로 출력할 때 %ld를 써야한다.
빌드할때 경고 뜨는 거 신경쓰임

왜 unsigned가 아닌지 생각해봤는데
지금 경우처럼 시간끼리 빼고 더할 때 음수가 나오는 경우가 있어서 그런 것 같다.

내가 시간 잴 때 방법. 저장용임

```c
#include <time.h>

struct timespec start, end;
time_t sec;
long int nsec;

clock_gettime(CLOCK_MONOTONIC, &start);
/* code */
clock_gettime(CLOCK_MONOTONIC, &end);

sec  = end.tv_sec  - start.tv_sec;
nsec = end.tv_nsec - start.tv_nsec;
if (nsec < 0) nsec += 1000000000;
printf("time: %3ld.%09lds\n", sec, nsec);
```

CLOCK_REALTIME을 안쓰고 CLOCK_MONOTONIC을 쓰는 이유는
CLOCK_REALTIME은 프로그램 실행 중에 바뀔 위험(내가 직접 바꾸거나, 동기화가 되거나 등등)이
있기 때문에 정확성을 위해서 CLOCK_MONOTONIC을 쓰는 거라고 배웠다.

sys/time.h를 불러와서 gettimeofday 함수로도 할 수 있다.
timevar 구조체라서 마이크로초를 쓰는데 비표준이니까 쓰지 말랫음

## 3. 스택 메모리 크기

1억개의 4바이트 정수를 정렬하려고 했다.
char buf[100000000]; 선언하려고 했는데 세그먼트 오류가 났다.
스택 공간은 무제한이 아니었다...

실험은 안해봤는데 수 메가 바이트 정도가 한계인 것 같다.
제한을 푸려면 무슨 명령어를 치라는데 쳐보지는 않음.

## 4. readline 라이브러리

fgets 같은 함수가 저수준 파일 입출력 함수 중에 있으면 좋겠다고 생각해서
검색하다가 알게 된 라이브러리.

위키백과 예제 복사해서 딱 한번 써봤는데 진짜 편하다.
우분투에서는 libreadline-dev 설치하고 컴파일할 땐 -lreadline 넣어주면 된다.

라이센스는 GPL다.

```sh
sudo apt-get install libreadline-dev
gcc -o readline readline.c -lreadline
```

더 쓰고 싶은데 자야겠다.
