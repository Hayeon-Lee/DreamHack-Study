## pwntools API 사용법

### 1. process & remote

1. process 함수는 exploit 을 로컬 바이너리를 대상으로 할 때 사용하는 함수이다.(테스트, 디버깅 용도)

2. remote 함수는 원격 서버를 대상으로 할 때 사용하는 함수이다.(서버를 실제로 공격)

<hr />

### 2. send

데이터를 프로세스에 전송하기 위해 사용한다.

```python
from pwn import *
p = process('./test')
p.send('A')  # ./test에 'A'를 입력
p.sendline('A') # ./test에 'A'+'\n'을 입력
p.sendafter('hello', 'A')  # ./test가 'hello'를 출력하면, 'A'를 입력
p.sendlineafter('hello', 'A')  # ./test가 'hello'를 출력하면, 'A' + '\n'을 입력
```

<hr />

### 3. recv

프로세스에서 데이터를 받기 위해 사용한다.

- *recv(n): 최대 n바이트를 받음(그만큼 받지 못해도 에러를 발생시키지는 않음)*

- *recvn(n): 정확히 n바이트를 받을 때까지 계속 대기*

```python
from pwn import *
p = process('./test')
data = p.recv(1024)  # p가 출력하는 데이터를 최대 1024바이트까지 받아서 data에 저장
data = p.recvline()  # p가 출력하는 데이터를 개행문자를 만날 때까지 받아서 data에 저장
data = p.recvn(5)  # p가 출력하는 데이터를 5바이트만 받아서 data에 저장
data = p.recvuntil('hello')  # p가 출력하는 데이터를 'hello'가 출력될 때까지 받아서 data에 저장
data = p.recvall()  # p가 출력하는 데이터를 프로세스가 종료될 때까지 받아서 data에 저장
```

<hr />

### 4. packing & unpacking

- p32(): 32bit 리틀 엔디언으로 packing 함
- p64(): 64bit 리틀 엔디언으로 packing 함
- u32(): 32bit 리틀 엔디언을 unpacking 함
- u42(): 64bit 리틀 엔디언을 unpacking 함


```python
from pwn import *
s32 = 0x41424344
s64 = 0x4142434445464748

print(p32(s32)) #b'DCBA'
print(p64(s64)) #b'HGFEDCBA'

s32 = b"ABCD"
s64 = b"ABCDEFGH"

print(hex(u32(s32))) #0x44434241
print(hex(u64(s64))) #0x4847464544434241
```

<hr />

### 5. interactive

셀을 획득했거나, 익스플로잇의 특정 상황에 직접 입력을 주면서 출력을 확인하고 싶을 때 사용한다.

호출하고 나면 터미널로 프로세스에 데이터를 입력하고, 프로세스의 출력을 확인할 수 있습니다.

```python
from pwn import *
p = process('./test')
p.interactive()
```

<hr />

### 6. ELF

ELF: exploit 에 사용될 수 있는 여러 정보가 기록되어 있음

```python
from pwn import *
e = ELF('./test')
puts_plt = e.plt['puts'] # ./test에서 puts()의 PLT주소를 찾아서 puts_plt에 저장
read_got = e.got['read'] # ./test에서 read()의 GOT주소를 찾아서 read_got에 저장
```

<hr />

### 7. context.log

exploit 을 디버깅할 때 사용할 수 있는 로그 기능

```python
from pwn import *
context.log_level = 'error' # 에러만 출력
context.log_level = 'debug' # 대상 프로세스와 익스플로잇간에 오가는 모든 데이터를 화면에 출력
context.log_level = 'info'  # 비교적 중요한 정보들만 출력
```

<hr />

### 8. context.arch

프로그래머가 직접 아키텍쳐의 정보를 지정할 때 사용된다

```python
from pwn import *
context.arch = "amd64" # x86-64 아키텍처
context.arch = "i386"  # x86 아키텍처
context.arch = "arm"   # arm 아키텍처
```

<hr />

### 9. shellcraft

pwntools 에는 자주 사용되는 셸 코드들이 저장되어 있지만, 제약 조건이 존재할 가능성이 높기 때문에 직접 셸 코드를 작성하는 것도 좋은 방법이다.

x86-64를 대상으로 생성할 수 있는 여러 종류의 셸 코드를 하단의 링크에서 참고할 수 있다.
<https://docs.pwntools.com/en/stable/shellcraft/amd64.html>

<hr />

### 10. asm

아키텍쳐를 지정한 이후, 자신이 작성한 코드를 기계어로 assemble 할 때 사용한다.

```python
from pwn import *
context.arch = 'amd64' # 익스플로잇 대상 아키텍처 'x86-64'
code = shellcraft.sh() # 셸을 실행하는 셸 코드
code = asm(code)       # 셸 코드를 기계어로 어셈블
```
