---
title: Windows에서 Python 3.4.4, Qt 4.8.7, PySide 1.2.4 설치하기
slug: install-python-qt-pyside-in-windows
date: 2016-10-08 01:17:00 +0900 KST
categories: [etc]
markup: mmark
---

PySide 설치하는 방법 자세하게 적는 글

현재 PySide는 Python 3.5와 Qt5를 지원하지 않는다.
PySide2는 Qt5를 지원하지만 아직 한창 개발 중인 것 같아 Qt4.8을 쓰기로 했다.

## 1. Python 설치

<https://www.python.org/downloads/windows/> 에서 3.4.4를 받는다. 나는 32비트를 받았다.

## 2. Qt 설치

VS 버전 설정이 어려울 것 같아서 그냥 MinGW를 위한 버전으로 설치했다.

Qt 4.8.7이 지원하는 MinGW-w64 버전은 4.8.2고, 다운로드는 아래 링크에서 받을 수 있다.
mingw32 폴더를 C:\에 풀자. 마음에 안들면 다른 데 풀덩가

<https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/4.8.2/threads-win32/dwarf/>

왜 옛날 MinGW 버전을 써야하냐면, 이 버전의 Qt가 그 옛날 버전으로 컴파일되서 그렇다고 알고 있다.

Qt 다운로드는 아래 링크에서 잘 골라서 받아보자.

<https://download.qt.io/official_releases/qt/4.8/4.8.7/>

## 3. PySide 설치

한 줄만 입력하면 된다. 넘나 편한것

pip3 install pyside

## 4. 확인

설치가 끝났으니 잘 됐는지 확인도 해본다.

```python
>>> import PySide
>>> PySide.__version__
'1.2.4'
>>> PySide.QtCore.__version__
'4.8.7'
>>> 
```

난 잘되는듯!
