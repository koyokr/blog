---
title: PowerShell에서 Bash 같은 alias 쓰기
date: 2017-02-26 22:14:00 +0900 KST
categories: [development]
---

bash에는 `alias`가 있다...

이렇게 쓸 수 있다...

```sh
alias ub='docker run --rm ubuntu $@'
```

파워쉘에도 Set-Alias가 있지만... 대신 function 기능을 쓰는게 유익한 것 같다...

```powershell
function ub { docker run --rm ubuntu $args }
```

이렇게...
