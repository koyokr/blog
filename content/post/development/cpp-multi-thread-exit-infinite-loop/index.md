---
title: C++ 멀티스레드 무한루프 끝내기
slug: cpp-multi-thread-exit-infinite-loop
date: 2017-01-06 00:42:00 +0900 KST
categories: [development]
markup: mmark
---

까먹을 예정이라서 저장

만약 `bool`을 이용해서 종료를 해왔다면... 예를 들어 이렇게 써왔다면...

```cpp
#include <thread>

void func( bool& run ) {
    while ( run ) {
        // ...
    }
}

int main() {
    bool run{ true };
    std::thread th{ func, std::ref( run ) };
    // ...
    run = false;
    th.join();

    return 0;
}
```

이렇게 바꿔주자. c는 `volatile`을 사용해서... c11이라면 `_Atomic`이 있다.

```cpp
#include <thread>
#include <atomic>

void func( std::atomic< bool >& run ) {
    while ( run ) {
        // ...
    }
}

int main() {
    std::atomic< bool > run{ true };
    std::thread th{ func, std::ref( run ) };
    // ...
    run = false;
    th.join();

    return 0;
}
```

동일한 `bool` 변수에 여러 스레드가 같이 접근하면 데이터 경쟁이 일어난다.

즉, 정의되지 않은 동작.
