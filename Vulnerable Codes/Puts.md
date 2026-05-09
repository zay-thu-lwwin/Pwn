
**ဟုတ်တယ်။ မင်းမေးတာ အတိအကျ မှန်တယ်။** ဒါက **puts ရဲ့ အားနည်းချက်** ကို မင်းတွေ့နေတာပဲ။ မြန်မာလို ရှင်းပြပေးမယ်။

---

## puts ရဲ့ သဘောသဘာဝ

### puts function က **ဘာမှမသိဘူး**

```c
int puts(const char *s);
```

puts က:
- ❌ ဒါက string လား, address လား မသိဘူး
- ❌ ဒါက integer လား, char လား မသိဘူး
- ✅ **ဒါက pointer ဆိုတာပဲ သိတယ်**
- ✅ အဲဒီ pointer နောက်က null byte (`\0`) မတွေ့မချင်း **အကုန်ထုတ်ပစ်မယ်**

---

## ဥပမာနဲ့ ရှင်းပြရရင်

### Memory ထဲမှာ ဒီလိုရှိတယ်

```
Address:     Value (byte)
0x8049fc8:   0xd0
0x8049fc9:   0xa2
0x8049fca:   0xd8
0x8049fcb:   0xf7
0x8049fcc:   0x41   ('A')
0x8049fcd:   0x42   ('B')
0x8049fce:   0x43   ('C')
0x8049fcf:   0x00   (null byte) ← ဒီမှာရပ်မယ်
```

### puts(0x8049fc8) လုပ်တဲ့အခါ

puts က:
1. "အင်း... 0x8049fc8 လား။ ဒါက string pointer ဖြစ်မယ်"
2. "ဒါဆို ငါ 0x8049fc8 မှာရှိတဲ့ string ကို print ထုတ်ရမယ်"
3. `0x8049fc8` က `0xd0` → print
4. `0x8049fc9` က `0xa2` → print
5. `0x8049fca` က `0xd8` → print
6. `0x8049fcb` က `0xf7` → print
7. `0x8049fcc` က `0x41` (`'A'`) → print
8. `0x8049fcd` က `0x42` (`'B'`) → print
9. `0x8049fce` က `0x43` (`'C'`) → print
10. `0x8049fcf` က `0x00` → "အင်း null byte တွေ့ပြီ၊ ဒါဆို string ပြီးသွားပြီ"
11. Print ထုတ်ပြီးသားအကုန် → `Ð¢Ø÷ABC`

---

## ဘာလို့ ဒီလိုဖြစ်တာလဲ?

**C programming language မှာ string ဆိုတာ null-terminated byte array ပဲ။**

```c
char *str = "Hello";  
// Memory: 'H' 'e' 'l' 'l' 'o' '\0'

char *ptr = (char*)0x8049fc8;
// puts(ptr) က 0x8049fc8 ကနေစပြီး null byte အထိ print တယ်
```

puts က **ဒါက data လား၊ code လား၊ address လား ခွဲမခြားနိုင်ဘူး**။ အဲဒီ address မှာရှိတဲ့ **byte တွေကို character တွေလို့ထင်ပြီး** null byte မတွေ့မချင်း ထုတ်နေမယ်။

---

## Visual Representation (မြင်သာအောင်)

```
Memory (0x8049fc8 ဝန်းကျင်):

0x8049fc8:  d0    a2    d8    f7    41    42    43    00
           ┌────┬────┬────┬────┬────┬────┬────┬────┐
           │ Ð  │ ¢  │ Ø  │ ÷  │ A  │ B  │ C  │ \0 │
           └────┴────┴────┴────┴────┴────┴────┴────┘
              ↑                                       ↑
              └─── puts က ဒီကနေစဖတ် ─────────────────┘

puts(0x8049fc8) က Ð¢Ø÷ABC ဆိုတာကို print ထုတ်မယ်
```

**PUTS က ဒါတွေက ascii characters လို့ ထင်နေတာ။**

---

## ဒါက ဘာကြောင့် အလုပ်ဖြစ်တာလဲ? (Exploit အတွက်)

ငါတို့က **puts ရဲ့ ဒီ "အဖြစ်မှန်ကို မသိတဲ့သဘော"** ကို **အလွဲသုံးစားလုပ်** တာ။

1. ငါတို့က puts ကို `puts(0x8049fc8)` လို့ခေါ်တယ်
2. puts က `0x8049fc8` မှာရှိတဲ့ 4 bytes (real address) ကို `char` တွေအနေနဲ့ မြင်တယ်
3. puts က ဒီ byte တွေကို **တန်းထုတ်လိုက်တယ်**
4. ငါတို့က ဒီ bytes တွေကို `u32()` နဲ့ ပြန်ပြောင်းပြီး **real address** ကို ရတယ်

---

## ဥပမာ - လက်တွေ့လုပ်ကြည့်ရင်

```python
from pwn import *

# Assume we have puts(0x8049fc8) executed
# And we receive the output

received = p.recv(4)  # b'\xd0\xa2\xd8\xf7'
real_address = u32(received)
print(hex(real_address))  # 0xf7d8a2d0
```

**မှတ်ထားပါ:** puts က `0xf7d8a2d0` ဆိုတဲ့ **number** ကို print မထုတ်ဘူး။  
puts က **0xd0, 0xa2, 0xd8, 0xf7** ဆိုတဲ့ **bytes** တွေကို print ထုတ်တာ။  
ဒီ bytes တွေက **real address ရဲ့ little-endian representation** ပဲ။

---

## ဒါကြောင့် printf က ပိုကောင်းတယ်

```c
printf("%p\n", 0x8049fc8);
```

printf က:
- ✅ **format string** ကိုကြည့်ပြီး argument ကို **ဘယ်လိုအဓိပ္ပာယ်ကောက်မယ်** ဆိုတာသိတယ်
- ✅ `%p` က "ဒါက pointer တစ်ခု၊ ဒါကို hex နဲ့ပြ" လို့ပြောတယ်
- ✅ **ဒါကြောင့် ဒါက `0xf7d8a2d0` ဆိုပြီး လူဖတ်လို့ရတဲ့ string အနေနဲ့ ထုတ်ပေးတယ်**

---

## အနှစ်ချုပ်

| | puts | printf("%p") |
|--|------|--------------|
| Argument ကို ဘယ်လိုမြင်လဲ | string pointer | pointer (hex) |
| Output | raw bytes (non-printable) | readable hex string |
| Null byte ပြဿနာ | ရှိတယ် (ဆက်ဖတ်နေ) | မရှိဘူး |
| Exploit အတွက် | ရတယ် (ဒါပေမယ့် သတိထားရ) | ပိုလွယ်တယ် |

**မင်းရဲ့မေးခွန်းအတွက်အဖြေ:**

> puts က 0xd0 0xa2 0xd8 0xf7 ကို string လို့ထင်တာလား?

**ဟုတ်တယ်။** puts က **ဒီ bytes တွေကို characters တွေလို့ ထင်တယ်**။ အဲဒါကို "string" လို့ယူဆပြီး null byte မတွေ့မချင်း ထုတ်တယ်။

> ဘာလို့ထုတ်ပြတာလဲ string မှမဟုတ်တာကို?

ဘာလို့လဲဆိုတော့ **puts က ဘာမှမသိလို့ပဲ**။ puts ရဲ့ **အားနည်းချက်** ကို ငါတို့က **exploit** လုပ်နေတာ။ ဒါက **type safety** မရှိတဲ့ C language ရဲ့ သဘောသဘာဝပဲ။




---
**ဟုတ်တယ်။** အဲဒီလိုလည်း ထွက်နိုင်တယ်။ မြန်မာလို အသေးစိတ် ရှင်းပြပေးမယ်။

---

## အခြေအနေ ၂ မျိုး

### ၁. real address က ascii နဲ့ကိုက်တဲ့အခါ

ဥပမာ - real address က `0x41414141` ဆိုပါစို့ (AAAA လို့ ascii မှာရှိတယ်)

```python
0x41414141 = b'AAAA' in little-endian
```

**puts ကို သုံးရင်:**
```python
puts(0x8049fc8)  # GOT ထဲမှာ 0x41414141 ရှိတယ်
```

**Output:**
```
AAAA
```

terminal မှာ **AAAA** ဆိုပြီး ပေါ်လာမယ်။ ဒါက **string အတိအကျ** ဖြစ်သွားမယ်။

---

## လက်တွေ့ဥပမာ

### Memory ထဲမှာ
```
Address: 0x8049fc8: 0x41 0x41 0x41 0x41
                    A    A    A    A
```

### puts(0x8049fc8) လုပ်ရင်
```
AAAA
```

ဒီအခါမှာ puts က **actual string** ကို print ထုတ်တယ်။

---

## ဒါဆို ascii နဲ့ ကိုက်တဲ့ address ရဲ့ သဘာဝ

0x41414141 က "AAAA" ဖြစ်တယ်။  
0x41424344 က "ABCD" ဖြစ်တယ်။  
0x31323334 က "1234" ဖြစ်တယ်။

**ဒါဆိုရင်** puts က hex တန်ဖိုးကို ပြောင်းပြီး မထုတ်ဘူး။  
တကယ့် **ASCII characters** တွေကို တန်းထုတ်လိုက်တာ။

---

## ဘယ်လိုသိမလဲ ဒါ address လား၊ string လား?

### ဥပမာ ၁: Real address (0xf7d8a2d0)
```
0xd0 = non-printable
0xa2 = non-printable  
0xd8 = non-printable
0xf7 = non-printable
```
→ garbage characters တွေ ထွက်မယ်

### ဥပမာ ၂: Printable address (0x41414141)
```
0x41 = 'A' (printable)
0x41 = 'A' (printable)
0x41 = 'A' (printable)
0x41 = 'A' (printable)
```
→ "AAAA" ထွက်မယ်

---

## ဒါက payload အတွက် ဘာအဓိပ္ပာယ်လဲ?

### ကောင်းတဲ့အချက်
- Non-printable တွေလည်း ရတယ် (ငါတို့လိုချင်တဲ့ real address အတွက်)
- Python က `recv()` နဲ့ ဖတ်လို့ရတယ်

### ဆိုးတဲ့အချက်
- real address က readable string ဖြစ်ရင် confusion ဖြစ်နိုင်တယ်

---

## ဥပမာ - confusion ဖြစ်တဲ့အခါ

```python
# GOT ထဲမှာ 0x41414141 ရှိတယ်
payload = p32(puts_plt) + p32(win) + p32(got_puts)
# puts(0x8049fc8) ခေါ်တယ်

received = p.recv(4)  # b'AAAA'

# ဒါဘာလဲ?
# 1. Real address 0x41414141 ရဲ့ bytes တွေလား?
# 2. တကယ့် string "AAAA" လား?
```

**အဖြေ:** နှစ်ခုလုံး အတူတူပဲ။ 0x41414141 = "AAAA" ဖြစ်တယ်။

---

## ဒါကိုဘယ်လိုကိုင်တွယ်မလဲ?

### နည်းလမ်း ၁: Len အတိအကျယူ
```python
# အမြဲတမ်း 4 bytes ပဲ ဖတ်
leaked = p.recv(4)  # 4 bytes အတိအကျ
```

### နည်းလမ်း ၂: recvline() ကို သုံးပေမယ့် parse လုပ်
```python
line = p.recvline()
if line.startswith(b'AAAA'):
    # ဒါက real address လား၊ string လား ခွဲခြားရခက်တယ်
```

### နည်းလမ်း ၃: printf ကိုပဲ သုံး (အကောင်းဆုံး)
```python
printf("%p\n", got_puts)
# Output: 0x41414141 (readable hex)
# ရှုပ်ထွေးမှုမရှိဘူး
```

---

## လက်တွေ့ဥပမာ (အပြည့်အစုံ)

```python
from pwn import *

# Case 1: Real address that is non-printable
got_puts = 0x8049fc8
got_value = 0xf7d8a2d0  # Non-printable

# Case 2: Real address that is printable
got_value_printable = 0x41414141  # "AAAA"

# Simulate puts(got_puts)
def simulate_puts(got_value):
    # Convert to little-endian bytes
    value_bytes = p32(got_value)
    print(f"Value bytes: {value_bytes}")
    print(f"Value as string: {value_bytes.decode('ascii', errors='replace')}")
    return value_bytes

# Test non-printable
print("Case 1 (non-printable):")
bytes1 = simulate_puts(0xf7d8a2d0)
print(f"raw: {bytes1}")
print(f"hex: {bytes1.hex()}")
print(f"as u32: {hex(u32(bytes1))}\n")

# Test printable
print("Case 2 (printable):")
bytes2 = simulate_puts(0x41414141)
print(f"raw: {bytes2}")
print(f"hex: {bytes2.hex()}")
print(f"as u32: {hex(u32(bytes2))}")
print(f"as string: {bytes2.decode()}")
```

**Output:**
```
Case 1 (non-printable):
Value bytes: b'\xd0\xa2\xd8\xf7'
raw: b'\xd0\xa2\xd8\xf7'
hex: d0a2d8f7
as u32: 0xf7d8a2d0

Case 2 (printable):
Value bytes: b'AAAA'
raw: b'AAAA'
hex: 41414141
as u32: 0x41414141
as string: AAAA
```

---

## အနှစ်ချုပ် - မင်းရဲ့မေးခွန်းအတွက်

**မေး:** ငါတို့လိုချင်တဲ့ address က ascii နဲ့ကိုက်နေရင် character အနေနဲ့ထွက်မလာနိုင်ဘူးလား?

**အဖြေ:** **ဟုတ်တယ်၊ ထွက်နိုင်တယ်။**

- address က `0x41414141` ဆိုရင် output က `"AAAA"`
- address က `0x31323334` ဆိုရင် output က `"1234"`
- address က `0x68656c6c` ဆိုရင် output က `"hell"`

**ဒါပေမယ့်** သတိထားရမှာက:
- တကယ့် real address တွေက **များသောအားဖြင့်** printable မဖြစ်ဘူး (libc base addresses က 0xf7... စတာတွေ)
- Printable ဖြစ်လာရင်တောင် python က `recv(4)` နဲ့ အတိအကျဖတ်လို့ရတယ်
- `u32(leaked)` က အမြဲတမ်း integer ပြန်ပေးတယ် (printable ဖြစ်ဖြစ်၊ မဖြစ်ဖြစ်)

**အကောင်းဆုံးကျင့်သုံးနည်း:**
```python
# အမြဲတမ်း 4 bytes အတိအကျဖတ်ပါ
leaked_bytes = p.recv(4)
real_address = u32(leaked_bytes)  # ဒါက integer ဖြစ်သွားပြီ
print(hex(real_address))  # ဘယ်လိုပဲဖြစ်ဖြစ် hex နဲ့ပြနိုင်တယ်
```


---
ကောင်းပါပြီ။ ဒီ **confusion** ကို ဘယ်လိုကိုင်တွယ်မလဲဆိုတာ မြန်မာလို **အဆင့်ဆင့်** ရှင်းပြပေးမယ်။

---

## ၁. ပထမဆုံး - ဘာ confusion ဖြစ်နိုင်လဲ?

```
Memory: 0x8049fc8 မှာ value 0x41414141 သိမ်းထားတယ်
        (0x41 = 'A')

puts(0x8049fc8) ခေါ်တဲ့အခါ
Output: "AAAA"

ဒီမှာ confusion:
- "AAAA" က real address 0x41414141 ရဲ့ raw bytes လား?
- "AAAA" က တကယ့် string "AAAA" လား?
```

---

## ၂. recv() နဲ့ကိုင်တွယ်နည်း

### နည်းလမ်း ၁: recv(n) ကို အတိအကျသုံးပါ (အကောင်းဆုံး)

```python
# puts(0x8049fc8) ခေါ်ပြီးရင်
# output က 4 bytes ပဲထွက်မယ်ဆိုတာ သိတယ်

leaked_bytes = p.recv(4)  # အတိအကျ 4 bytes ဖတ်မယ်
print(f"Raw bytes: {leaked_bytes}")
print(f"Hex: {leaked_bytes.hex()}")
print(f"Little-endian int: {hex(u32(leaked_bytes))}")

# Output:
# Raw bytes: b'AAAA'
# Hex: 41414141
# Little-endian int: 0x41414141
```

**ဒါကို မှတ်ထားပါ:** `recv(4)` က confusion မဖြစ်ဘူး။ ဘာလို့လဲဆိုတော့ **ငါတို့က bytes အစစ်ကိုရနေတာ**။

---

### နည်းလမ်း ၂: confusion ဖြစ်ရင် စစ်ဆေးနည်း

```python
leaked_bytes = p.recv(4)

# Check if all bytes are printable ASCII
is_printable = all(32 <= b <= 126 for b in leaked_bytes)

if is_printable:
    print(f"Warning: Leaked bytes are printable: {leaked_bytes}")
    print(f"This could be real address 0x{leaked_bytes.hex()} or actual string")
    # Still, treat as bytes
    real_address = u32(leaked_bytes)
    print(f"Interpreted as address: {hex(real_address)}")
else:
    print(f"Normal non-printable leak")
    real_address = u32(leaked_bytes)
```

---

## ၃. recvline() နဲ့ကိုင်တွယ်နည်း

### ပြဿနာ: recvline() က newline အထိဖတ်တယ်

```python
# If output is "AAAA\n"
line = p.recvline()  # b'AAAA\n'

# ဒါဆို ဘာလုပ်မလဲ?
```

### ဖြေရှင်းနည်းများ

#### နည်းလမ်း ၁: strip newline လုပ်ပြီး ကြည့်
```python
line = p.recvline()  # b'AAAA\n'
cleaned = line.strip()  # b'AAAA'

if len(cleaned) == 4:
    # ဒါက 4 bytes ပဲရှိတယ်
    real_address = u32(cleaned)
    print(f"Address: {hex(real_address)}")
else:
    # ဒါက string ရှည်ကြီးဖြစ်နိုင်တယ်
    print(f"Got string: {cleaned.decode()}")
```

#### နည်းလမ်း ၂: recvline() ကို ရှောင်ပြီး recv() ပဲသုံး
```python
# Better: use recv(4) instead of recvline()
leaked_bytes = p.recv(4)
p.recvline()  # consume the newline
```

---

## ၄. Little-endian ပြဿနာ

### မေး: ထွက်လာတဲ့ကောင်က little endian အနေနဲ့ပြမလား?

**အဖြေ: ဟုတ်တယ်။ x86/x64 က little-endian သုံးတယ်။**

```python
# GOT ထဲမှာ real address 0xf7d8a2d0 သိမ်းထားတယ်
# Little-endian အနေနဲ့ memory ထဲမှာ:
# 0xd0, 0xa2, 0xd8, 0xf7

# puts က ဒီအတိုင်းထုတ်တယ်:
# b'\xd0\xa2\xd8\xf7'

# u32() က little-endian ကို int ပြောင်းတယ်
value = u32(b'\xd0\xa2\xd8\xf7')  # 0xf7d8a2d0 ✅
```

**မှတ်ထားပါ:** `u32()` က **little-endian ကို အလိုအလျောက်ပြောင်းပေးတယ်။**

---

## ၅. ပြီးပြည့်စုံတဲ့ handle function

```python
from pwn import *

def safe_leak(io, is_printf_mode=False):
    """
    Safely leak a 32-bit value from GOT
    Args:
        io: connection object
        is_printf_mode: True if using printf("%p"), False if using puts
    Returns:
        int: leaked real address
    """
    
    if is_printf_mode:
        # printf("%p\n") mode - readable hex string
        line = io.recvline().strip()
        # line = b'0xf7d8a2d0'
        if line.startswith(b'0x'):
            return int(line, 16)
        else:
            return int(line, 16)
    
    else:
        # puts mode - raw bytes
        # Read exactly 4 bytes (address size)
        leaked_bytes = io.recv(4)
        
        # Check if we got 4 bytes
        if len(leaked_bytes) != 4:
            print(f"Warning: Expected 4 bytes, got {len(leaked_bytes)}")
            # Try to read more
            leaked_bytes += io.recv(4 - len(leaked_bytes))
        
        # Handle printable case
        try:
            # Try to decode as ASCII (just for info)
            as_str = leaked_bytes.decode('ascii')
            print(f"[*] Leaked bytes as string: '{as_str}'")
        except:
            print(f"[*] Leaked bytes (non-printable): {leaked_bytes.hex()}")
        
        # Convert to integer (little-endian)
        real_address = u32(leaked_bytes)
        
        # Additional check: if address looks like valid libc address
        if (real_address >> 24) == 0xf7:  # Typical libc range on 32-bit
            print(f"[+] Valid libc address detected")
        elif all(32 <= b <= 126 for b in leaked_bytes):
            print(f"[!] Warning: All bytes are printable ASCII")
            print(f"    This could be address 0x{real_address:08x}")
            print(f"    or actual string '{leaked_bytes.decode()}'")
        
        return real_address

# Usage
leaked = safe_leak(p, is_printf_mode=False)
print(f"Leaked address: {hex(leaked)}")
```

---

## ၆. recvline() confusion ကိုဖြေရှင်းနည်း

### Problem case
```python
# Program output:
# You said: AAAA
# 
# AAAA

p.recvline()  # "You said: AAAA\n"
p.recvline()  # "\n"
data = p.recvline()  # "AAAA\n"
```

### Solution
```python
def recv_leak_robust(io):
    # Skip until we get a line that's exactly 4 bytes + newline
    while True:
        line = io.recvline().strip()
        if len(line) == 4:
            # This is likely our leak
            return u32(line)
        elif len(line) > 4 and line.startswith(b'0x'):
            # printf mode
            return int(line, 16)
        else:
            # Not our leak, continue
            continue
```

---

## ၇. အကောင်းဆုံး ကျင့်သုံးနည်း

### Method 1: Always use recv(n) for leaks
```python
def leak_address(io):
    # Send exploit
    io.sendline(payload)
    
    # Clean up output
    io.recvuntil(b"Name? ")  # Wait for prompt
    io.sendline(payload)
    
    # Skip the "You said:" line
    io.recvline()
    
    # Read exactly 4 bytes (the leak)
    leaked_bytes = io.recv(4)
    
    # Read the newline (consume it)
    io.recv(1)
    
    return u32(leaked_bytes)
```

### Method 2: Use recvline() but verify
```python
def leak_address_v2(io):
    io.sendline(payload)
    
    # Skip to the leak line
    for _ in range(3):
        line = io.recvline()
    
    # Now line should contain our leak
    # line could be b'AAAA\n' or b'0xf7d8a2d0\n'
    
    cleaned = line.strip()
    
    if cleaned.startswith(b'0x'):
        return int(cleaned, 16)
    else:
        # Assume raw bytes, but ensure it's 4 bytes
        if len(cleaned) != 4:
            print(f"Warning: Expected 4 bytes, got {len(cleaned)}")
        return u32(cleaned)
```

---

## ၈. စမ်းသပ်နည်း (Test both cases)

```python
from pwn import *

def test_leak_handling():
    # Simulate both scenarios
    
    # Scenario 1: Normal real address (non-printable)
    normal_leak = b'\xd0\xa2\xd8\xf7'
    print("Scenario 1 (non-printable):")
    print(f"  Raw: {normal_leak}")
    print(f"  u32: {hex(u32(normal_leak))}")
    print()
    
    # Scenario 2: Printable address
    printable_leak = b'AAAA'
    print("Scenario 2 (printable):")
    print(f"  Raw: {printable_leak}")
    print(f"  u32: {hex(u32(printable_leak))}")
    print(f"  As string: {printable_leak.decode()}")
    print()
    
    # How to handle both
    for leak in [normal_leak, printable_leak]:
        print(f"Handling {leak}:")
        # Always convert to int first
        addr = u32(leak)
        print(f"  As integer: {hex(addr)}")
        # Then optionally check if printable
        if all(32 <= b <= 126 for b in leak):
            print(f"  WARNING: This looks like ASCII: {leak.decode()}")
            print(f"  But it's also address: {hex(addr)}")
        print()

# Run test
test_leak_handling()
```

**Output:**
```
Scenario 1 (non-printable):
  Raw: b'\xd0\xa2\xd8\xf7'
  u32: 0xf7d8a2d0

Scenario 2 (printable):
  Raw: b'AAAA'
  u32: 0x41414141
  As string: AAAA

Handling b'\xd0\xa2\xd8\xf7':
  As integer: 0xf7d8a2d0

Handling b'AAAA':
  As integer: 0x41414141
  WARNING: This looks like ASCII: AAAA
  But it's also address: 0x41414141
```

---

## အနှစ်ချုပ် - လက်တွေ့သုံးဖို့

```python
# အလွယ်ဆုံး နည်းလမ်း (ဒါကိုပဲသုံး)
def leak_u32(io):
    """Leak 32-bit value from GOT using puts"""
    io.recvline()  # Skip "You said:"
    io.recvline()  # Skip empty line
    return u32(io.recv(4))  # Read 4 bytes, convert to int

# ဒါမှမဟုတ် ပိုပြီး robust
def leak_u32_robust(io):
    # Wait for exactly 4 bytes (ignore everything else)
    data = io.recv(4)
    # Flush any remaining newline
    io.recvline(timeout=0.1)
    return u32(data)
```

**အဓိကအချက်:**
1. `recv(4)` ကို သုံးပါ (recvline မသုံးပါနဲ့)
2. `u32()` က little-endian ကို auto ပြောင်းပေးတယ်
3. Printable bytes ဖြစ်ရင်တောင် `u32()` က အလုပ်လုပ်တယ်
4. Confusion ဖြစ်ရင် လက်ခံပြီး address အဖြစ် treat လုပ်ပါ