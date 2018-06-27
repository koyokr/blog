---
title: re2c랑 bison으로 파서 만들기
slug: re2c-bison-parser
date: 2016-11-29 23:07:00 +0900 KST
categories: [development]
markup: mmark
---

파서를 만들 때,

lex로는 토큰을 만들고, yacc로는 토큰의 조합에 따른 행동을 만든다. 좀 이상하게 설명했는데 야매로 공부해서.. 하여튼 이 글에서 re2c는 lex에 대응하고, bison은 yacc에 대응한다.

flex와 bison을 같이 쓰는 예제는 많은데, re2c와 bison을 같이 쓰는 예제는 찾기가 어려웠다. 그래서 re2c에서 bison으로 토큰을 넘기는 예제를 만드는 데 많이 고생했다.

나중에 까먹을게 분명하니까 이렇게 글로 적어놓기로 했다.

그런데 글 다 쓰고 보니까 좀 배운 사람들은 나처럼 고생할 일이 애초에 없을 것 같음;

일단 bison은 보통 확장자로 y를 쓰고, 기본적으로 아래와 같은 형태를 가지고 있다.

```bison
%{
Prologue
%}

Bison declarations

%%
Grammar rules
%%

Epilogue
```

Prologue는 bison으로 생성될 파일의 처음에 복사될 코드를 적는다. %{와 %}로 감싸서 사용가능하고 c 언어의 전역변수나 함수 선언, 매크로, 헤더파일 포함에 사용한다.

Bison declarations는 토큰, 결합법칙, 타입 같은 심볼을 정의하는 구역이다. 어떤 정의가 있는지 자세히 알고 싶다면 아래 링크로.

<https://www.gnu.org/software/bison/manual/html_node/Declarations.html>

Grammar rules는 %%로 시작해서 %%로 끝나는 구역이다. 여기에는 Bison 문법 규칙이 들어간다. 규칙은 아래처럼 정의한다.

```bison
result: rule1-components { action1 }
      | rule2-components { action2 }
      ;
```

result는 심볼이다. :가 시작이고 ;가 끝이다.

rule-components는 심볼 또는 심볼의 조합. | 기호로 여러 규칙을 둘 수 있다.

action은 rule-components가 인식됐을 때 실행하는 코드이다. c 언어로 작성한다.

Epilogue는 Prologue의 반대로, 생성될 파일의 끝에 복사될 코드를 적는다. gnu 문서를 보면 yyparse 함수 앞에 올 필요가 없는 c 코드를 삽입하고 싶을 때 유용하다고 한다. 예를 들면 yylex나 yyerror 같은 함수들.

간단하게 bison 코드를 작성했다. 의도는 "abc = 123"과 같은 문자열을 파싱하는 것이다. 파일 이름은 example.y

```bison
%{
#include <stdio.h>

int yylex(void);
void yyerror(const char* s);
%}

%token TOKEN_VAR
%token TOKEN_NUM
%token TOKEN_EQ

%%
s: TOKEN_VAR TOKEN_EQ TOKEN_NUM
 ;
%%

void yyerror(const char* s)
{
    printf("yyerror: %s\n", s);
}
```

프롤로그에서 yylex 함수와 yyerror 함수를 선언했다. yyerror는 의도하지 않은 입력이 들어올 때 호출되는 함수다. yylex는 토큰을 반환하는 함수인데, re2c 파일에서 정의할 것이다. 참고로 printf를 사용했기 때문에 stdio.h를 인클루드했다.

그리고 토큰 세 개를 선언했다. TOKEN_VAR은 abc에, TOKEN_NUM은 123에, TOKEN_EQ는 =에 해당한다.

문법 규칙은 s라고 하여 "abc = 123"과 같은 형태를 만족했을 때 파싱되도록 작성했다.

마지막으로 에필로그에 yyerror 함수를 정의했다.

빌드는 이렇게 하기로 했다. d 옵션은 헤더파일을 만든단 뜻이고, l은 #line을 사용하지 않는단 뜻이다. 왜 헤더 파일을 만드냐면 이따가 설명함.

```sh
bison -dl -o example.tab.c example.y
```

이제 re2c 차례. re2c 파일은 주석을 제외하면 보통의 C/C++ 파일과 차이가 없다. 아래 코드를 봐보자. <http://re2c.org/>에 있는 예제 코드를 짧게 가져왔다.

```c
#include <stdio.h>

const char *lex(const char *YYCURSOR)
{
    const char *YYMARKER;
/*!re2c
    re2c:define:YYCTYPE = char;
    re2c:yyfill:enable = 0;

    end = "\x00";
    int = [1-9][0-9]*;
    hex = '0x' [0-9a-fA-F]+;

    *       { return "err"; }
    int end { return "int"; }
    hex end { return "hex"; }
*/
}

int main(int argc, char **argv)
{
    for (int i = 1; i < argc; ++i) {
        printf ("%s: %s\n", lex(argv[i]), argv[i]);
    }
    return 0;
}
```

/*!re2c와 */로 감싼 영역에 규칙을 작성하고 re2c를 실행하면 주석에 적힌 규칙을 c코드로 변환해준다.

YYMARKER는 역추적용 변수를 설정한 것이고, YYCTYPE은 YYCURSOR의 내용을 담는 변수인데 re2c:define:YYCTYPE로 자료형을 정해줬다. yyfill뭐시기는 입력버퍼가 가득찰 때 쓰는 뭐시기인데 아직 잘 모르겠으니 패스ㅎ

내가 작성한 lex 함수는 이렇게 작동한다.

YYCURSOR가 null값을 만나면 end 상태가 되고, 보통의 정수 형태를 만나면 int 상태가, 헥스값 형태를 만나면 hex 상태가 된다.

그리고 정수 상태에서 null값을 만나면 "int"를 반환하고, 헥스 상태에서 null값을 만나면 "hex"를 반환한다. 이도저도 아니면 "err"를 반환한다.

```sh
$ re2c -i -o example.c e.re
$ make example
cc     example.c   -o example
$ ./example 123 0123 0x123 0xzz
int: 123
err: 0123
hex: 0x123
err: 0xzz
```

바로 이렇게.

방금 re2c만 사용해서 프로그램을 하나 만들었는데, 이번에는 위에 작성한 bison 파일에 맞춰 토큰을 반환하는 코드를 작성했다. 이름은 example.re

```c
#include <stdio.h>
#include "example.tab.h"

char* yyin;
char* yyold;

int yylex(void)
{
start:
    yyold = yyin;

/*!re2c
    re2c:define:YYCTYPE  = char;
    re2c:define:YYCURSOR = yyin;
    re2c:yyfill:enable   = 0;

    end        = "\x00";
    whitespace = [\r\t ]+;
    digit      = [0-9]+;
    letter     = [a-zA-Z_];
    variable   = letter(letter|digit)*;

    end        { return 0; }
    variable   { return TOKEN_VAR; }
    "="        { return TOKEN_EQ;  }
    digit      { return TOKEN_NUM; }
    whitespace { goto start; }
*/

}

int main()
{
    char c;
    int i;

    do {
        char buf[128] = { 0 };
        i = 0;

        while ((c = getchar()) != EOF) {
            buf[i++] = c;
            if (c == '\n') {
                yyin = buf;
                yyparse();
                break;
            }
        }
    } while (c != EOF);

    return 0;
}
```

re2c와 bison을 같이 쓸 때 몇 가지 사항만 알면 쉽게 쓸 수 있다. 사실 이 주의사항이 쓰려던 글 내용 전부.

1. 0은 YYEOF다. 제 때에 이걸 반환해주지 않으면 제대로 파싱하지 못하고 오류가 난다.
2. 토큰을 반환하는 함수는 int yylex(void)로 한다. 값은 전역 변수로 전달한다.
3. 토큰 선언은 bison으로 생성한 example.tab.h에 있으니까 꼭 이걸 인클루드해서 토큰을 반환하자.

Makefile 작성은 이렇게 했다.

```makefile
TARGET := example
YACC   := bison
RE2C   := re2c
CC     := gcc

$(TARGET): $(TARGET).h $(TARGET).c $(TARGET).tab.c
	$(CC) -o $(TARGET) $(TARGET).c $(TARGET).tab.c

$(TARGET).c: $(TARGET).re
	$(RE2C) -i -o $(TARGET).c $(TARGET).re

$(TARGET).tab.c: $(TARGET).y
	$(YACC) -dl -o $(TARGET).tab.c $(TARGET).y

clean:
	rm -f $(TARGET) $(TARGET).c $(TARGET).tab.c $(TARGET).tab.h
```

재활용하려고 TARGET으로 도배했는데 생각보다 가독성이 굉장히 구린 듯ㅎㅎ...

이제 빌드하고 실행해보면

```sh
$ make
re2c -i -o example.c example.re
bison -dl -o example.tab.c example.y
gcc -o example example.c example.tab.c
$ ./example
b2 = 10
2b = 1
yyerror: syntax error
1 = b
yyerror: syntax error
sadasdsd s22  a = 1
yyerror: syntax error
ab = 111
$
```

의도한 대로 잘 실행된다.

bison을 어떻게 작성하는지, re2c는 어떻게 작성하는지, re2c와 bison을 같이 쓰는 건 어떻게 하는지 썼다.

아래 링크들은 이 글을 작성하면서 정말 많은 참고를 한 사이트들. 더 좋은 글 있으면 추천 좀요... 책도..

re2c 홈페이지.
<http://re2c.org/>

bison 홈페이지.

<https://www.gnu.org/software/bison/>

flex와 bison에 대해 설명한 글. 파서 개념 하나도 모를 때 봤는데 도움 많이 된듯.

Flex and Bison Howto
<http://www.joinc.co.kr/w/Site/Development/Env/Yacc>

re2c와 bison으로 만든 공부용 예제. 이거 많이 따라함

勉強のために作った電卓もどき
<https://github.com/youkidearitai/calc-modoki>

re2c와 bison에 대해 설명하고 적용하는 글. 이것도 많이 따라함.

Doc.19 字句解析ツールre2c
<http://akkera102.sakura.ne.jp/gbadev/index.php?Doc.19%20%BB%FA%B6%E7%B2%F2%C0%CF%A5%C4%A1%BC%A5%EBre2c>

Doc.20 構文解析ツールbison(1)
<http://akkera102.sakura.ne.jp/gbadev/index.php?Doc.20%20%B9%BD%CA%B8%B2%F2%C0%CF%A5%C4%A1%BC%A5%EBbison(1)>

Doc.20 構文解析ツールbison(2)
<http://akkera102.sakura.ne.jp/gbadev/index.php?Doc.21%20%B9%BD%CA%B8%B2%F2%C0%CF%A5%C4%A1%BC%A5%EBbison(2)>

Doc.20 構文解析ツールbison(3)
<http://akkera102.sakura.ne.jp/gbadev/index.php?Doc.22%20%B9%BD%CA%B8%B2%F2%C0%CF%A5%C4%A1%BC%A5%EBbison(3)>

