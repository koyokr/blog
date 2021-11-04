---
title: Hyper-V 활성화/비활성화
date: 2017-04-12 21:25:00 +0900 KST
categories: [virtual-machine]
---

관리자 권한으로 실행한다.

끄기.

```powershell
bcdedit /set hypervisorlaunchtype off
```

켜기.

```powershell
bcdedit /set hypervisorlaunchtype auto
```

명령어 실행 후 재부팅이 필요하다.
