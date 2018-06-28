---
title: Windows DNS 설정
slug: configure-windows-dns
date: 2017-12-10 18:02:00 +0900 KST
categories: [how-to]
markup: mmark
aliases: [/96/]
---

```dos
netsh interface ip set dns "로컬 영역 연결" static 1.1.1.1 primary
netsh interface ip add dns "로컬 영역 연결" 1.0.0.1
```

Cloudflare Public DNS Server

빠르다

```dos
netsh interface ip set dns "로컬 영역 연결" static 9.9.9.9 primary
netsh interface ip add dns "로컬 영역 연결" 9.9.9.10
ipconfig /flushdns
```

IBM Public DNS Server

1.1.1.1이 잘 안되는 것 같을 때.

```dos
(
echo  #
echo      13.229.188.59      github.com
echo     13.250.177.223      github.com
echo      52.74.223.119      github.com
echo      13.229.188.59      www.github.com
echo     13.250.177.223      www.github.com
echo      52.74.223.119      www.github.com
echo      13.250.94.254      api.github.com
echo      13.250.168.23      api.github.com
echo     54.169.195.247      api.github.com
echo      13.229.188.59      gist.github.com
echo     13.250.177.223      gist.github.com
echo      52.74.223.119      gist.github.com
echo       13.229.189.0      codeload.github.com
echo     13.250.162.133      codeload.github.com
echo      54.251.140.56      codeload.github.com

echo       74.125.24.17      googlemail.l.google.com
echo       74.125.24.17      mail.google.com
echo       74.125.24.17      gmail.com
echo       74.125.24.17      www.gmail.com
echo       74.125.24.18      googlemail.l.google.com
echo       74.125.24.19      googlemail.l.google.com
echo       74.125.24.83      googlemail.l.google.com
echo       74.125.24.84      accounts.google.com
echo       74.125.24.91      dl.l.google.com
echo       74.125.24.91      sb.l.google.com
echo       74.125.24.91      sb-ssl.l.google.com
echo       74.125.24.91      youtube-ui.l.google.com
echo       74.125.24.91      dl.google.com
echo       74.125.24.91      safebrowsing.google.com
echo       74.125.24.91      sb-ssl.google.com
echo       74.125.24.91      youtube.com
echo       74.125.24.91      www.youtube.com
echo       74.125.24.91      tv.youtube.com
echo       74.125.24.93      dl.l.google.com
echo       74.125.24.93      youtube-ui.l.google.com
echo       74.125.24.94      www.google.co.kr
echo       74.125.24.94      google.co.kr
echo       74.125.24.94      accounts.google.co.kr
echo       74.125.24.94      www.google.co.jp
echo       74.125.24.94      google.co.jp
echo       74.125.24.94      accounts.google.co.jp
echo       74.125.24.94      gstatic.com
echo       74.125.24.94      www.gstatic.com
echo       74.125.24.94      fonts.gstatic.com
echo       74.125.24.94      ssl.gstatic.com
echo       74.125.24.95      googleadapis.l.google.com
echo       74.125.24.95      www.googleapis.com
echo       74.125.24.95      ajax.googleapis.com
echo       74.125.24.95      fonts.googleapis.com
echo       74.125.24.95      maps.googleapis.com
echo       74.125.24.99      api.l.google.com
echo       74.125.24.99      scholar.l.google.com
echo       74.125.24.99      api.google.com
echo       74.125.24.99      scholar.google.com
echo       74.125.24.99      googleapis.com
echo       74.125.24.99      google-analytics.com
echo      74.125.24.100      clients.l.google.com
echo      74.125.24.100      ytstatic.l.google.com
echo      74.125.24.100      ytimg.l.google.com
echo      74.125.24.100      www-google-analytics.l.google.com
echo      74.125.24.100      www3.l.google.com
echo      74.125.24.100      www.google.com
echo      74.125.24.100      google.com
echo      74.125.24.100      analytics.google.com
echo      74.125.24.100      apis.google.com
echo      74.125.24.100      books.google.com
echo      74.125.24.100      chrome.google.com
echo      74.125.24.100      clients1.google.com
echo      74.125.24.100      clients2.google.com
echo      74.125.24.100      clients3.google.com
echo      74.125.24.100      clients4.google.com
echo      74.125.24.100      clients5.google.com
echo      74.125.24.100      clients6.google.com
echo      74.125.24.100      developers.google.com
echo      74.125.24.100      drive.google.com
echo      74.125.24.100      earth.google.com
echo      74.125.24.100      fonts.google.com
echo      74.125.24.100      groups.google.com
echo      74.125.24.100      gsuite.google.com
echo      74.125.24.100      hangouts.google.com
echo      74.125.24.100      keep.google.com
echo      74.125.24.100      maps.google.com
echo      74.125.24.100      myaccount.google.com
echo      74.125.24.100      news.google.com
echo      74.125.24.100      ogs.google.com
echo      74.125.24.100      photos.google.com
echo      74.125.24.100      play.google.com
echo      74.125.24.100      plus.google.com
echo      74.125.24.100      redirector.gvt1.com
echo      74.125.24.100      safebrowsing-cache.google.com
echo      74.125.24.100      store.google.com
echo      74.125.24.100      support.google.com
echo      74.125.24.100      translate.google.com
echo      74.125.24.100      encrypted-tbn0.gstatic.com
echo      74.125.24.100      accounts.youtube.com
echo      74.125.24.100      s.ytimg.com
echo      74.125.24.100      i1.ytimg.com
echo      74.125.24.100      i2.ytimg.com
echo      74.125.24.100      i3.ytimg.com
echo      74.125.24.100      i4.ytimg.com
echo      74.125.24.100      i9.ytimg.com
echo      74.125.24.100      www.google-analytics.com
echo      74.125.24.101      www3.l.google.com
echo      74.125.24.102      www3.l.google.com
echo      74.125.24.103      api.l.google.com
echo      74.125.24.103      scholar.l.google.com
echo      74.125.24.104      api.l.google.com
echo      74.125.24.104      scholar.l.google.com
echo      74.125.24.105      api.l.google.com
echo      74.125.24.105      scholar.l.google.com
echo      74.125.24.106      api.l.google.com
echo      74.125.24.106      scholar.l.google.com
echo      74.125.24.113      www3.l.google.com
echo      74.125.24.119      ytimg-edge-static.l.google.com
echo      74.125.24.119      i.ytimg.com
echo      74.125.24.120      gstatic.com
echo      74.125.24.132      googlehosted.l.googleusercontent.com
echo      74.125.24.132      photos-ugc.l.googleusercontent.com
echo      74.125.24.132      webcache.googleusercontent.com
echo      74.125.24.132      lh1.googleusercontent.com
echo      74.125.24.132      lh2.googleusercontent.com
echo      74.125.24.132      lh3.googleusercontent.com
echo      74.125.24.132      lh4.googleusercontent.com
echo      74.125.24.132      lh5.googleusercontent.com
echo      74.125.24.132      lh6.googleusercontent.com
echo      74.125.24.132      yt3.ggpht.com
echo      74.125.24.132      yt4.ggpht.com
echo      74.125.24.136      dl.l.google.com
echo      74.125.24.136      youtube-ui.l.google.com
echo      74.125.24.138      www3.l.google.com
echo      74.125.24.139      www3.l.google.com
echo      74.125.24.141      golang.org
echo      74.125.24.141      www.golang.org
echo      74.125.24.147      api.l.google.com
echo      74.125.24.147      scholar.l.google.com
echo      74.125.24.190      dl.l.google.com
echo      74.125.24.190      youtube-ui.l.google.com
echo      74.125.24.191      blogger.com
echo      74.125.24.191      www.blogger.com
echo      74.125.24.191      blogspot.com
echo      74.125.24.191      www.blogger.com
echo       216.58.203.3      recaptcha.net
echo       216.58.203.3      www.recaptcha.net

echo     93.184.215.201      download.visualstudio.microsoft.com
)>> C:\Windows\System32\drivers\etc\hosts
```

명령 프롬프트 복붙 ㄱㄱ