---
title: 사지방 DNS 설정
slug: sjb-dns-config
date: 2017-12-10 18:02:00 +0900 KST
categories: [etc]
markup: mmark
aliases: [/96/, /post/configure-windows-dns/]
---

## DNS 서버 변경

Cloudflare Public DNS Server

```bat
netsh interface ip set dns "로컬 영역 연결" static 1.1.1.1 primary
netsh interface ip add dns "로컬 영역 연결" 1.0.0.1
ipconfig /flushdns
```

IBM Public DNS Server

```bat
netsh interface ip set dns "로컬 영역 연결" static 9.9.9.9 primary
netsh interface ip add dns "로컬 영역 연결" 149.112.112.112
ipconfig /flushdns
```

## hosts 파일 변경

172.0.0.0/8, 192.0.0.0/8 대역을 제외하여 GitHub, Google 서비스를 중점으로 찾았다.

```bat
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

echo       74.125.24.17      gmail.com
echo       74.125.24.18      gmail.com
echo       74.125.24.19      gmail.com
echo       74.125.24.83      gmail.com
echo       74.125.24.17      www.gmail.com
echo       74.125.24.18      www.gmail.com
echo       74.125.24.19      www.gmail.com
echo       74.125.24.83      www.gmail.com
echo       74.125.24.17      mail.google.com
echo       74.125.24.18      mail.google.com
echo       74.125.24.19      mail.google.com
echo       74.125.24.83      mail.google.com
echo       74.125.24.84      accounts.google.com
echo       74.125.24.91      dl.google.com
echo       74.125.24.93      dl.google.com
echo      74.125.24.136      dl.google.com
echo      74.125.24.190      dl.google.com
echo       74.125.24.91      safebrowsing.google.com
echo       74.125.24.93      safebrowsing.google.com
echo      74.125.24.136      safebrowsing.google.com
echo      74.125.24.190      safebrowsing.google.com
echo       74.125.24.91      sb-ssl.google.com
echo       74.125.24.93      sb-ssl.google.com
echo      74.125.24.136      sb-ssl.google.com
echo      74.125.24.190      sb-ssl.google.com
echo       74.125.24.91      youtube.com
echo       74.125.24.93      youtube.com
echo      74.125.24.136      youtube.com
echo      74.125.24.190      youtube.com
echo       74.125.24.91      www.youtube.com
echo       74.125.24.93      www.youtube.com
echo      74.125.24.136      www.youtube.com
echo      74.125.24.190      www.youtube.com
echo       74.125.24.91      artists.youtube.com
echo       74.125.24.93      artists.youtube.com
echo      74.125.24.136      artists.youtube.com
echo      74.125.24.190      artists.youtube.com
echo       74.125.24.91      tv.youtube.com
echo       74.125.24.93      tv.youtube.com
echo      74.125.24.136      tv.youtube.com
echo      74.125.24.190      tv.youtube.com
echo       74.125.24.91      youtu.be
echo       74.125.24.93      youtu.be
echo      74.125.24.136      youtu.be
echo      74.125.24.190      youtu.be
echo       74.125.24.94      id.google.com
echo       74.125.24.94      clientservices.googleapis.com
echo       74.125.24.94      update.googleapis.com
echo       74.125.24.94      gstatic.com
echo      74.125.24.120      gstatic.com
echo       74.125.24.94      www.gstatic.com
echo      74.125.24.120      www.gstatic.com
echo       74.125.24.94      csi.gstatic.com
echo      74.125.24.120      csi.gstatic.com
echo       74.125.24.94      fonts.gstatic.com
echo       74.125.24.94      maps.gstatic.com
echo      74.125.24.120      maps.gstatic.com
echo       74.125.24.94      ssl.gstatic.com
echo       74.125.24.94      recaptcha.net
echo       74.125.24.94      www.recaptcha.net
echo       74.125.24.94      google.co.kr
echo       74.125.24.94      www.google.co.kr
echo       74.125.24.94      id.google.co.kr
echo       74.125.24.94      maps.google.co.kr
echo       74.125.24.94      khms0.google.co.kr
echo       74.125.24.94      khms1.google.co.kr
echo       74.125.24.94      khms2.google.co.kr
echo       74.125.24.94      khms3.google.co.kr
echo       74.125.24.94      translate.google.co.kr
echo       74.125.24.94      google.co.jp
echo       74.125.24.94      www.google.co.jp
echo       74.125.24.94      accounts.google.co.jp
echo       74.125.24.94      translate.google.co.jp
echo       74.125.24.95      analyticsinsights-pa.clients6.google.com
echo       74.125.24.95      analyticssuitefrontend-pa.clients6.google.com
echo       74.125.24.95      appsitemsuggest-pa.clients6.google.com
echo       74.125.24.95      blobcomments-pa.clients6.google.com
echo       74.125.24.95      people-pa.clients6.google.com
echo       74.125.24.95      realtimesupport.clients6.google.com
echo       74.125.24.95      www.googleapis.com
echo       74.125.24.95      ajax.googleapis.com
echo       74.125.24.95      chart.googleapis.com
echo       74.125.24.95      content.googleapis.com
echo       74.125.24.95      fonts.googleapis.com
echo       74.125.24.95      incrementalwebfonts-pa.googleapis.com
echo       74.125.24.95      maps.googleapis.com
echo       74.125.24.95      safebrowsing.googleapis.com
echo       74.125.24.95      translate.googleapis.com
echo       74.125.24.95      youtube.googleapis.com
echo       74.125.24.97      googletagmanager.com
echo       74.125.24.97      www.googletagmanager.com
echo       74.125.24.97      ssl.google-analytics.com
echo       74.125.24.99      api.google.com
echo      74.125.24.103      api.google.com
echo      74.125.24.104      api.google.com
echo      74.125.24.105      api.google.com
echo      74.125.24.106      api.google.com
echo      74.125.24.147      api.google.com
echo       74.125.24.99      scholar.google.com
echo      74.125.24.103      scholar.google.com
echo      74.125.24.104      scholar.google.com
echo      74.125.24.105      scholar.google.com
echo      74.125.24.106      scholar.google.com
echo      74.125.24.147      scholar.google.com
echo       74.125.24.99      googleapis.com
echo      74.125.24.103      googleapis.com
echo      74.125.24.104      googleapis.com
echo      74.125.24.105      googleapis.com
echo      74.125.24.106      googleapis.com
echo      74.125.24.147      googleapis.com
echo       74.125.24.99      google-analytics.com
echo      74.125.24.103      google-analytics.com
echo      74.125.24.104      google-analytics.com
echo      74.125.24.105      google-analytics.com
echo      74.125.24.106      google-analytics.com
echo      74.125.24.147      google-analytics.com
echo       74.125.24.99      t0.gstatic.com
echo       74.125.24.99      t1.gstatic.com
echo       74.125.24.99      t2.gstatic.com
echo       74.125.24.99      t3.gstatic.com
echo      74.125.24.100      google.com
echo      74.125.24.101      google.com
echo      74.125.24.102      google.com
echo      74.125.24.113      google.com
echo      74.125.24.138      google.com
echo      74.125.24.139      google.com
echo      74.125.24.100      www.google.com
echo      74.125.24.101      www.google.com
echo      74.125.24.102      www.google.com
echo      74.125.24.113      www.google.com
echo      74.125.24.138      www.google.com
echo      74.125.24.139      www.google.com
echo      74.125.24.100      appspot.com
echo      74.125.24.100      www.appspot.com
echo      74.125.24.100      chromebook.com
echo      74.125.24.100      www.chromebook.com
echo      74.125.24.100      analytics.google.com
echo      74.125.24.100      admin.google.com
echo      74.125.24.100      ads.google.com
echo      74.125.24.100      apis.google.com
echo      74.125.24.100      apps.google.com
echo      74.125.24.100      books.google.com
echo      74.125.24.100      chrome.google.com
echo      74.125.24.100      chat.google.com
echo      74.125.24.100      clients1.google.com
echo      74.125.24.100      clients2.google.com
echo      74.125.24.100      clients3.google.com
echo      74.125.24.100      clients4.google.com
echo      74.125.24.100      clients5.google.com
echo      74.125.24.100      clients6.google.com
echo      74.125.24.100      cloud.google.com
echo      74.125.24.100      contacts.google.com
echo      74.125.24.100      developers.google.com
echo      74.125.24.100      docs.google.com
echo      74.125.24.100      drive.google.com
echo      74.125.24.100      earth.google.com
echo      74.125.24.100      edu.google.com
echo      74.125.24.100      fonts.google.com
echo      74.125.24.100      get.google.com
echo      74.125.24.100      gg.google.com
echo      74.125.24.100      groups.google.com
echo      74.125.24.100      gsuite.google.com
echo      74.125.24.100      hangouts.google.com
echo      74.125.24.100      keep.google.com
echo      74.125.24.100      khms.google.com
echo      74.125.24.100      lh2.google.com
echo      74.125.24.100      lh3.google.com
echo      74.125.24.100      lh4.google.com
echo      74.125.24.100      lh5.google.com
echo      74.125.24.100      lh6.google.com
echo      74.125.24.100      maps.google.com
echo      74.125.24.100      marketingplatform.google.com
echo      74.125.24.100      meet.google.com
echo      74.125.24.100      mt0.google.com
echo      74.125.24.100      myaccount.google.com
echo      74.125.24.100      news.google.com
echo      74.125.24.100      notifications.google.com
echo      74.125.24.100      ogs.google.com
echo      74.125.24.100      optimize.google.com
echo      74.125.24.100      photo.google.com
echo      74.125.24.100      photos.google.com
echo      74.125.24.100      play.google.com
echo      74.125.24.100      plus.google.com
echo      74.125.24.100      productforums.google.com
echo      74.125.24.100      redirector.gvt1.com
echo      74.125.24.100      safebrowsing-cache.google.com
echo      74.125.24.100      store.google.com
echo      74.125.24.100      support.google.com
echo      74.125.24.100      translate.google.com
echo      74.125.24.100      video.google.com
echo      74.125.24.100      www.google-analytics.com
echo      74.125.24.100      encrypted-tbn0.gstatic.com
echo      74.125.24.100      accounts.youtube.com
echo      74.125.24.100      img.youtube.com
echo      74.125.24.100      i1.ytimg.com
echo      74.125.24.100      i2.ytimg.com
echo      74.125.24.100      i3.ytimg.com
echo      74.125.24.100      i4.ytimg.com
echo      74.125.24.100      i9.ytimg.com
echo      74.125.24.100      s.ytimg.com
echo      74.125.24.100      ampproject.net
echo      74.125.24.100      www.ampproject.net
echo      74.125.24.100      doubleclick.net
echo      74.125.24.100      www.doubleclick.net
echo      74.125.24.100      goo.gl
echo      74.125.24.100      www.goo.gl
echo      74.125.24.100      books.google.co.kr
echo      74.125.24.100      gsuite.google.co.kr
echo      74.125.24.110      cse.google.com
echo      74.125.24.118      voice.google.com
echo      74.125.24.119      i.ytimg.com
echo      74.125.24.128      storage.googleapis.com
echo      74.125.24.132      cdn.ampproject.net
echo      74.125.24.132      1.bp.blogspot.com
echo      74.125.24.132      2.bp.blogspot.com
echo      74.125.24.132      3.bp.blogspot.com
echo      74.125.24.132      4.bp.blogspot.com
echo      74.125.24.132      geo0.ggpht.com
echo      74.125.24.132      geo1.ggpht.com
echo      74.125.24.132      geo2.ggpht.com
echo      74.125.24.132      geo3.ggpht.com
echo      74.125.24.132      lh3.ggpht.com
echo      74.125.24.132      lh4.ggpht.com
echo      74.125.24.132      lh5.ggpht.com
echo      74.125.24.132      yt3.ggpht.com
echo      74.125.24.132      yt4.ggpht.com
echo      74.125.24.132      googledrive.com
echo      74.125.24.132      www.googledrive.com
echo      74.125.24.132      clients2.googleusercontent.com
echo      74.125.24.132      feedback.googleusercontent.com
echo      74.125.24.132      webcache.googleusercontent.com
echo      74.125.24.132      ci3.googleusercontent.com
echo      74.125.24.132      ci4.googleusercontent.com
echo      74.125.24.132      ci5.googleusercontent.com
echo      74.125.24.132      ci6.googleusercontent.com
echo      74.125.24.132      doc-0k-5c-docs.googleusercontent.com
echo      74.125.24.132      drive-thirdparty.googleusercontent.com
echo      74.125.24.132      lh1.googleusercontent.com
echo      74.125.24.132      lh2.googleusercontent.com
echo      74.125.24.132      lh3.googleusercontent.com
echo      74.125.24.132      lh4.googleusercontent.com
echo      74.125.24.132      lh5.googleusercontent.com
echo      74.125.24.132      lh6.googleusercontent.com
echo      74.125.24.132      translate.googleusercontent.com
echo      74.125.24.132      tpc.googlesyndication.com
echo      74.125.24.141      golang.org
echo      74.125.24.141      www.golang.org
echo      74.125.24.141      blog.golang.org
echo      74.125.24.141      play.golang.org
echo      74.125.24.141      tour.golang.org
echo      74.125.24.141      survey.g.doubleclick.net
echo      74.125.24.141      cloud.withgoogle.com
echo      74.125.24.141      csp.withgoogle.com
echo      74.125.24.141      testmysite.withgoogle.com
echo      74.125.24.141      tourbuilder.withgoogle.com
echo      74.125.24.148      ad.doubleclick.net
echo      74.125.24.149      ad.doubleclick.net
echo      74.125.24.148      static.doubleclick.net
echo      74.125.24.149      ad.doubleclick.net
echo      74.125.24.148      2542116.fls.doubleclick.net
echo      74.125.24.149      ad.doubleclick.net
echo      74.125.24.153      chrome-devtools-frontend.appspot.com
echo      74.125.24.153      gweb-multiscreen-hub.appspot.com
echo      74.125.24.153      premiumyva.appspot.com
echo      74.125.24.153      ytkenobi.appspot.com
echo      74.125.24.154      cm.g.doubleclick.net
echo      74.125.24.155      cm.g.doubleclick.net
echo      74.125.24.156      cm.g.doubleclick.net
echo      74.125.24.157      cm.g.doubleclick.net
echo      74.125.24.154      googleads.g.doubleclick.net
echo      74.125.24.155      googleads.g.doubleclick.net
echo      74.125.24.156      googleads.g.doubleclick.net
echo      74.125.24.157      googleads.g.doubleclick.net
echo      74.125.24.154      pubads.g.doubleclick.net
echo      74.125.24.155      pubads.g.doubleclick.net
echo      74.125.24.156      pubads.g.doubleclick.net
echo      74.125.24.157      pubads.g.doubleclick.net
echo      74.125.24.154      securepubads.g.doubleclick.net
echo      74.125.24.155      securepubads.g.doubleclick.net
echo      74.125.24.156      securepubads.g.doubleclick.net
echo      74.125.24.157      securepubads.g.doubleclick.net
echo      74.125.24.154      stats.g.doubleclick.net
echo      74.125.24.155      stats.g.doubleclick.net
echo      74.125.24.156      stats.g.doubleclick.net
echo      74.125.24.157      stats.g.doubleclick.net
echo      74.125.24.154      adservice.google.com
echo      74.125.24.155      adservice.google.com
echo      74.125.24.156      adservice.google.com
echo      74.125.24.157      adservice.google.com
echo      74.125.24.154      googleadservices.com
echo      74.125.24.155      googleadservices.com
echo      74.125.24.156      googleadservices.com
echo      74.125.24.157      googleadservices.com
echo      74.125.24.154      www.googleadservices.com
echo      74.125.24.155      www.googleadservices.com
echo      74.125.24.156      www.googleadservices.com
echo      74.125.24.157      www.googleadservices.com
echo      74.125.24.154      pagead2.googlesyndication.com
echo      74.125.24.155      pagead2.googlesyndication.com
echo      74.125.24.156      pagead2.googlesyndication.com
echo      74.125.24.157      pagead2.googlesyndication.com
echo      74.125.24.154      www.googletagservices.com
echo      74.125.24.155      www.googletagservices.com
echo      74.125.24.156      www.googletagservices.com
echo      74.125.24.157      www.googletagservices.com
echo      74.125.24.154      adservice.google.co.kr
echo      74.125.24.155      adservice.google.co.kr
echo      74.125.24.156      adservice.google.co.kr
echo      74.125.24.157      adservice.google.co.kr
echo      74.125.24.189      0.client-channel.google.com
echo      74.125.24.189      cello.client-channel.google.com
echo      74.125.24.189      93.docs.google.com
echo      74.125.24.191      blogger.com
echo      74.125.24.191      www.blogger.com
echo      74.125.24.191      blogspot.com
echo      74.125.24.191      www.blogger.com
echo      74.125.24.191      blogblog.com
echo      74.125.24.191      www.blogblog.com
echo      74.125.24.191      resources.blogblog.com

echo     93.184.215.201      download.visualstudio.microsoft.com
)>> C:\Windows\System32\drivers\etc\hosts
```

명령 프롬프트 복붙 ㄱㄱ
