---
title: Peach Fuzzer 2 WinDbg 경로 설정
slug: peach-fuzzer-2-configure-windbg-path
date: 2015-10-28 23:07:00 +0900 KST
categories: [how-to]
markup: mmark
---

퍼저가 자꾸 디버거 경로를 못잡았다.
검색해봐서 WinDbgPath로 경로 지정해주래서 따라해봤는데도 안됐었다.
3버전대 해결책은 2버전대에 안먹히나 보다.

2버전대에서는 디버거 폴더 안에 있는 것들을 C:\Program Files\Debugging Tools for Windows (x86) 폴더에 싸그리 복사하면 된다.

오늘 하루종일 이거 때문에 끙끙댔는데 좀 어처구니 없게 해결된 거 같다.
참고로 2.3.9 버전을 썼다.

그리고 아래 글을 작성하신 분은 너무 자상하신 거 같다. 사랑합니다.

<http://www.flinkd.org/2011/07/fuzzing-with-peach-part-1/>
