---
title: Pwnable KR - blackjack
slug: pwnable-kr-blackjack
date: 2016-04-21 19:08:00 +0900 KST
categories: [write-up]
markup: mmark
---

## 문제

![Pwnable KR blackjack](pwnable-kr-blackjack-page.png)

```c
int betting() //Asks user amount to bet
{
 printf("\n\nEnter Bet: $");
 scanf("%d", &bet);

 if (bet > cash) //If player tries to bet more money than player has
 {
        printf("\nYou cannot bet more money than you have.");
        printf("\nEnter Bet: ");
        scanf("%d", &bet);
        return bet;
 }
 else return bet;
} // End Function
``

## 풀이

배팅할 금액을 입력하는 부분에서 취약점이 있다.

배팅 금액을 보유한 금액보다 크게 적으면 다시 적으라고 하는데, 반복문이 아니기 때문에 한번 더 크게 적고 이기면 된다.

아래 코드도 보자

```c
if(p>21) //If player total is over 21, loss
{
    printf("\nWoah Buddy, You Went WAY over.\n");
    loss = loss+1;
    cash = cash - bet;
    printf("\nYou have %d Wins and %d Losses. Awesome!\n", won, loss);
    dealer_total=0;
    askover();
}
```

음수값을 크게 적고 져버리는 것도 한 방법이다.

```text
YaY_I_AM_A_MILLIONARE_LOL


Cash: $100001500
-------
|H    |
|  9  |
|    H|
-------

Your Total is 9

The Delaer Has a Total of 1

Enter Bat: $
```

난 처음에 이 문제를 소켓으로 통신하는 프로그램을 작성해서 목표 금액을 달성할 때까지 게임을 무한 반복시켜서 깨려고 했다. 멍청...
