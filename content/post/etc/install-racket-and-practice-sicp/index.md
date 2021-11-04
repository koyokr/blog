---
title: Racket 설치하고 SICP 실습하기
date: 2017-12-17 21:12:00 +0900 KST
categories: [etc]
---

우분투 터미널 환경에서 racket을 설치하고 vim으로 코드를 작성하여
sicp 예제를 실행하기 위한 환경을 설정하는 것

vim 버전이 쫌 높아야 되는 걸루 기억

racket 설치

```sh
sudo add-apt-repository ppa:plt/racket
sudo apt update
sudo apt install racket
```

sicp 모듈 설치

```sh
raco pkg install sicp
```

vim-plug 설치

```sh
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

아래 내용으로 ~/.vimrc 파일 저장

```vim
call plug#begin('~/.vim/plugged')
Plug 'wlangstroth/vim-racket'
Plug 'scrooloose/syntastic'
call plug#end()

set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*
let g:syntastic_always_populate_loc_list = 1
let g:syntastic_auto_loc_list = 1
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0
let g:syntastic_enable_racket_racket_checker = 1
```

vim에서 아래 명령어를 입력하여 vim-racket 설치

```vim
:PlugInstall
```

아래 코드를 실행해보자

```racket
#lang sicp

(inc
  (inc 0))
```

코드를 실행해보고

```sh
racket inc.rkt
```

실행파일도 만들어보고

```sh
raco exe inc.rkt
```

'-^

참고한 글

<https://racket-lang.org/>

<https://docs.racket-lang.org/sicp-manual/>

<https://medium.com/usevim/configuring-vim-for-sicp-119504226319>

<http://www.tuestudy.org/htdp/2012/03/11/ed-84-b0-eb-af-b8-eb-84-90-ea-b3-bc-vim-ec-9d-84-ec-9d-b4-ec-9a-a9-ed-95-9c-racket-ec-bd-94-eb-94-a9.html>
