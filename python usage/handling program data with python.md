

Target program (C နဲ့ရေးထားတာ) က bytes တွေကို သူ့နည်းနဲ့သူ အဓိပ္ပာယ်ဖွင့်တယ်
Python script ကနေ ဒီ program ကို data ပို့တဲ့အခါ program က မျှော်လင့်တဲ့ format အတိုင်း ပို့ပေးရတယ်
Program က integer မျှော်နေရင် → Python က ပို့တဲ့ data က 4 bytes ဖြစ်ရမယ်
Program က string မျှော်နေရင် → Python က ASCII bytes တွေပို့ရမယ်

#### Python Data Types

> Program ကို data ပို့တဲ့အခါ **bytes** အနေနဲ့ပဲ ပို့လို့ရတယ်။ Integer/string တိုက်ရိုက်မပို့ရ

##### integer (int)
```python
x = 3735928559
y = 0xdeadbeef    # integer ပဲ၊ hex format နဲ့ရေးထားတာ
```

 notation မတူပေမဲ့ code အတွက်ကတူတူဘဲ

```python
# 0xdeadbeef ဆိုတာ hex notation နဲ့ရေးထားတဲ့ integer ပါ
# Python က ဒါကို integer အဖြစ် understand လုပ်တယ်

>>> type(0xdeadbeef)
<class 'int'>

>>> 0xdeadbeef == 3735928559  # အတူတူပါပဲ
True
```

```python 
# p32() က integer ကိုပဲ လက်ခံတယ်
# 0xdeadbeef က integer ဖြစ်လို့ အလုပ်လုပ်တယ်

p32(0xdeadbeef)   # OK - 0xdeadbeef က integer
p32(3735928559)   # OK - ဒါလည်းအတူတူပဲ
p32("deadbeef")   # Error! - string ကိုတော့မလက်ခံဘူး
```

##### String (str) - Unicode text
```python
text = "Hello"
```

##### Bytes - raw binary data
```python
data = b"Hello"   # b'Hello'
data = b'\xde\xad\xbe\xef'


```

---

#### Scenario
##### Scenario 1 - Program  accept integer 4 bytes 

```python
# Program code (C)
# int value;
# read(0, &value, 4);

# Python exploit
value = 0xdeadbeef
payload = p32(value)   # b'\xef\xbe\xad\xde'
p.send(payload)
```

##### Scenario 2 - Program  accept string (null-terminated) 

```python
# Program code (C)
# char name[100];
# gets(name);

# Python exploit
name = "admin"
payload = name.encode() + b'\x00'   # b'admin\x00'
p.send(payload)
```

##### Scenario 3 - Program send back address as hex string in output (printf)

```python
# Program code (C)
# printf("%p", ptr);  // output: "0xf7e45670"

# Python exploit
output = p.recvline().decode().strip()   # "0xf7e45670"76
address = int(output, 16)                 # 4157879920 (integer)
print(hex(address))                       # 0xf7e45670
```

##### Scenario 4 - Program send back raw bytes in output (write, puts)

```python
# Program code (C)
# write(1, &ptr, 4);  // sends raw 4 bytes

# Python exploit
raw_bytes = p.recv(4)      # b'\x70\x56\xe4\xf7'
address = u32(raw_bytes)    # 0xf7e45670
print(hex(address))
```


##### Function that give Raw Memory (Bytes)  

ဒီ function တွေက memory ထဲမှာရှိတဲ့ data ကို ဘာမှ ပြုပြင်မွမ်းမံခြင်း မရှိဘဲ အရှိအတိုင်း ထုတ်ပေးတာပါ။ ဒါကြောင့် **`u32()` / `u64()`** နဲ့ ပြန် unpack လုပ်ရပါတယ်။

- **`puts(addr)`**:
    - **Fact**: Memory ထဲက byte တွေကို Null byte (`\x00`) မတွေ့မချင်း ထုတ်ပေးတယ်။
    - **Data Type**: Raw Bytes.

- **`write(fd, addr, len)`**:
    - **Fact**: အသေချာဆုံး leak function ပါ။ Null byte ပါပါမပါပါ မင်းသတ်မှတ်တဲ့ length အတိုင်း ကွက်တိထုတ်ပေးတယ်။
    - **Data Type**: Raw Bytes.
        
- **`printf(addr)`** (Format string မပါဘဲ သုံးလျှင်):
    - **Fact**: `puts` လိုပဲ Null byte အထိ raw memory ကို ထုတ်ပေးတယ်။
        

##### Function that give Text (String)

ဒီ function တွေက memory ထဲက binary data ကို ငါတို့လူတွေ ဖတ်လို့ရအောင် ASCII စာသား (0x...hex...) အဖြစ် ပြောင်းပြီးမှ ထုတ်ပေးတာပါ။ ဒါကြောင့် **`int(..., 16)`** နဲ့ ပြန်ပြောင်းရပါတယ်။

- **`printf("%p")` သို့မဟုတ် `printf("%x")`**:
    - **Fact**: နံပါတ်ကို hex စာသားအဖြစ် ပြောင်းထုတ်ပေးတယ်။
    - **Data Type**: ASCII String (e.g., `b"0xf7e10450"`).
        
- **`printf("%d")`**:
    - **Fact**: နံပါတ်ကို Decimal (ဆယ်လီစိတ်) စာသားအဖြစ် ပြောင်းထုတ်ပေးတယ်။
    - **Data Type**: ASCII String (e.g., `b"4158728448"`).
        

##### Function that accepts input (Sending Data)

payload ပို့တဲ့အခါမှာလည်း function ပေါ်မူတည်ပြီး ပို့ရတာ ကွာခြားပါတယ်

- **`gets(buf)` / `read(fd, buf, len)`**:
    - သူတို့က **Raw Bytes** ကို လက်ခံတာပါ ဒါကြောင့် `p32(addr)` နဲ့ တိုက်ရိုက်ပို့လို့ရပါတယ်။Null byte တွေကိုလည်း လက်ခံပါတယ်
        
- **`scanf("%s", buf)`**:
    - သူကလည်း **Raw Bytes** ကို လက်ခံပေမဲ့ **Space** (နေရာလွတ်) ဒါမှမဟုတ် **Newline** တွေ့ရင် ရပ်သွားပါတယ်။
        
- **`scanf("%d", &num)`**:
    - သူက **ASCII String** ကိုပဲ လက်ခံတာပါ။ ဒါကြောင့် address တွေ ပို့လို့မရပါဘူး။ "123" လို စာသားမျိုးပဲ ပို့လို့ရပါတယ်။ (မင်းရဲ့ `bruteforce` ထဲမှာ `str(i).encode()` သုံးရတာ ဒီကြောင့်ပါ။


| process            | Code                             | Input Type | Output Type |
| ------------------ | -------------------------------- | ---------- | ----------- |
| 32-bit int → bytes | `p32(0x1234)`                    | int        | bytes (4)   |
| 64-bit int → bytes | `p64(0x1234)`                    | int        | bytes (8)   |
| bytes → 32-bit int | `u32(b'\x34\x12\x00\x00')`       | bytes (4)  | int         |
| Hex string → int   | `int("0x1234", 16)`              | str        | int         |
| Int → hex string   | `hex(0x1234)`                    | int        | str         |
| String → bytes     | `b"hello"` or `"hello".encode()` | str        | bytes       |
| Bytes → string     | `b"hello".decode()`              | bytes      | str         |


|#|From|To|Method (pwn)|Method (vanilla)|Example|Result|
|---|---|---|---|---|---|---|
|**1**|Bytes|String|`.decode()`|`str(b, 'utf-8')`|`b'hello'.decode()`|`'hello'`|
|**2**|String|Bytes|`.encode()`|`str.encode()`|`'hello'.encode()`|`b'hello'`|
|**3**|Bytes|Hex String|`.hex()`|`binascii.hexlify(b).decode()`|`b'\xde\xad'.hex()`|`'dead'`|
|**4**|Hex String|Bytes|-|`bytes.fromhex()`|`bytes.fromhex('dead')`|`b'\xde\xad'`|
|**5**|Integer|Hex String|`hex(x)`|`hex(x)`|`hex(0xdead)`|`'0xdead'`|
|**6**|Hex String|Integer|`int(s,16)`|`int(s,16)`|`int('dead',16)`|`57005`|
|**7**|Integer|Bytes (32-bit LE)|`p32(x)`|`x.to_bytes(4,'little')`|`p32(0xdeadbeef)`|`b'\xef\xbe\xad\xde'`|
|**8**|Integer|Bytes (32-bit BE)|`p32(x, endian='big')`|`x.to_bytes(4,'big')`|`p32(0xdeadbeef, endian='big')`|`b'\xde\xad\xbe\xef'`|
|**9**|Integer|Bytes (64-bit LE)|`p64(x)`|`x.to_bytes(8,'little')`|`p64(0xdeadbeef)`|`b'\xef\xbe\xad\xde\x00\x00\x00\x00'`|
|**10**|Integer|Bytes (64-bit BE)|`p64(x, endian='big')`|`x.to_bytes(8,'big')`|`p64(0xdeadbeef, endian='big')`|`b'\x00\x00\x00\x00\xde\xad\xbe\xef'`|
|**11**|Bytes (LE)|Integer (32-bit)|`u32(b)`|`int.from_bytes(b,'little')`|`u32(b'\xef\xbe\xad\xde')`|`0xdeadbeef`|
|**12**|Bytes (LE)|Integer (64-bit)|`u64(b)`|`int.from_bytes(b,'little')`|`u64(b'\xef\xbe\xad\xde\x00\x00\x00\x00')`|`0xdeadbeef`|
|**13**|Bytes (BE)|Integer|-|`int.from_bytes(b,'big')`|`int.from_bytes(b'\xde\xad','big')`|`57005`|
|**14**|Integer|Char (ASCII)|`chr(x)`|`chr(x)`|`chr(0x41)`|`'A'`|
|**15**|Char|Integer|`ord(c)`|`ord(c)`|`ord('A')`|`65`|
|**16**|Integer|Byte|`p8(x)`|`bytes([x])`|`p8(0x41)`|`b'A'`|
|**17**|Byte|Integer|`u8(b)`|`b[0]`|`u8(b'A')`|`65`|
|**18**|Bytes|Hex Bytes|-|`binascii.hexlify(b)`|`binascii.hexlify(b'\xde\xad')`|`b'dead'`|
|**19**|Hex Bytes|Bytes|-|`binascii.unhexlify(b)`|`binascii.unhexlify(b'dead')`|`b'\xde\xad'`|


|From (Type)|To (Type)|Function / Code|Example|Output|
|---|---|---|---|---|
|**String**|**Bytes**|`"text".encode()`|`"Hello".encode()`|`b'Hello'`|
|**String**|**Hex (String)**|`"text".encode().hex()`|`"A".encode().hex()`|`'41'`|
|**String**|**Int**|`int("123")`|`int("FF", 16)`|`255`|
|**String**|**List**|`list("abc")`|`list("Hello")`|`['H','e','l','l','o']`|
|**String**|**ASCII Code List**|`[ord(c) for c in "text"]`|`[ord(c) for c in "ABC"]`|`[65,66,67]`|
|**Bytes**|**String**|`b'text'.decode()`|`b'Hello'.decode()`|`'Hello'`|
|**Bytes**|**Hex (String)**|`b'text'.hex()`|`b'A'.hex()`|`'41'`|
|**Bytes**|**Int**|`int.from_bytes(b'\x01\x00', 'little')`|`int.from_bytes(b'\x01\x00', 'little')`|`1`|
|**Bytes**|**List (Ints)**|`list(b'text')`|`list(b'ABC')`|`[65,66,67]`|
|**Hex (String)**|**Bytes**|`bytes.fromhex("41")`|`bytes.fromhex("48656c6c6f")`|`b'Hello'`|
|**Hex (String)**|**String**|`bytes.fromhex(hex_str).decode()`|`bytes.fromhex("41").decode()`|`'A'`|
|**Hex (String)**|**Int**|`int(hex_str, 16)`|`int("FF", 16)`|`255`|
|**Int**|**Bytes**|`int.to_bytes(length, byteorder)`|`(255).to_bytes(1, 'little')`|`b'\xff'`|
|**Int**|**Hex (String)**|`hex(number)`|`hex(255)`|`'0xff'`|
|**Int**|**Hex (without 0x)**|`format(number, 'x')`|`format(255, 'x')`|`'ff'`|
|**Int**|**Binary (String)**|`bin(number)`|`bin(255)`|`'0b11111111'`|
|**Int**|**Character**|`chr(number)`|`chr(65)`|`'A'`|
|**Int**|**String**|`str(number)`|`str(255)`|`'255'`|
|**List (Ints)**|**Bytes**|`bytes(list)`|`bytes([65,66,67])`|`b'ABC'`|
|**List (Ints)**|**String**|`bytes(list).decode()`|`bytes([65,66,67]).decode()`|`'ABC'`|
|**List (Strings)**|**String**|`"".join(list)`|`"".join(["H","i"])`|`'Hi'`|

---
