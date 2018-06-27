---
title: Ubuntu GCC 보안 옵션
slug: ubuntu-gcc-security-flags
date: 2017-05-12 14:58:00 +0900 KST
categories: [development]
markup: mmark
---

## 기준

ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-77-generic x86_64)

gcc 5.4.0 20160609

## gcc랑 관련없는 건데 이 운영체제에서 계속 ASLR을 끄고 싶어

echo "kernel.randomize_va_space=0" > /etc/sysctl.d/01-disable-aslr.conf

## 32비트로 컴파일하고 싶어

-m32

## -m32 옵션을 줬는데 이상해

sudo apt install gcc-multilib

## 32비트로 컴파일하니까 main 함수에 스택을 정렬하는 이상한 인스트럭션이 있어서 짜증나

-mpreferred-stack-boundary=2

## 최적화 옵션을 주니까 printf@plt가 __printf_chk@plt로 바뀌는 게 싫어

-U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=0

## CANARY를 없애고 싶어

-fno-stack-protector

## 스택에 실행 권한을 주고 싶어

-z execstack

## No RELRO가 기본값인데 Partial RELRO가 필요해

-z relro

## No RELRO가 기본값인데 Full RELRO가 필요해

-z relro -z now

## Partial RELRO가 기본값인데 No RELRO가 필요해

-z norelro

## Partial RELRO가 기본값인데 Full RELRO가 필요해

-z now

## .text가 랜덤이면 좋겠어

-fpie

## PIE가 쓰고 싶어

-fpie -pie

## 2018-03-26 추가사항

레드햇에서 정리한 플래그

<https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/>
