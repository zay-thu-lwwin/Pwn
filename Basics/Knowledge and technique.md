

### Strings and null byte (\x00)

string မှာ null byte ပါတယ် string ဆိုတာ character တွေလေးတွေစုစည်းထားတဲ့ကောင်
string length ကိုသတ်မှတ်ရင် null byte အတွက်ပါထည့်ပေးရ
`strlen`, `strcpy`, `strcat`, `printf("%s")` အားလုံးဟာ null byte ကိုရှာပြီးမှ အလုပ်လုပ်

```
char greeting[6] = {'H', 'e', 'l', 'l', 'o', '\0'};
```



### New line (\n) (\x0a)


##### `n` (0x0A) နဲ့ `\0` (0x00) ကို function တစ်ခုချင်းစီ ဘယ်လိုသဘောထားလဲ

ဒီမှာ **Logics 3 မျိုး** ရှိတယ်။

1. **As a normal byte** → သာမန် data တစ်ခုလို သိမ်း ၊ ဖတ် ၊ copy လုပ်
    
2. **As a terminator / delimiter** → loop ရပ်၊ နောက်ထပ် မဆက်တော့
    
3. **As an injector / transformer** → `\n` ကို `\0` ပြောင်း (gets) ၊ `\0` ကို output မှာ ထူးခြားဆက်ဆံ

| Function           | Category                  | `\n` Handling                                                | `\0` Handling                                 | Exploit Use                                                   |
| ------------------ | ------------------------- | ------------------------------------------------------------ | --------------------------------------------- | ------------------------------------------------------------- |
| `gets()`           | Input                     | ကို `\0` နဲ့ replace                                         | string terminator                             | Stack buffer overflow (classic)                               |
| `fgets()`          | Input                     | ကို buffer ထဲထည့် (keep)                                     | `\n` ရဲ့နောက်မှာ `\0` ထည့်                    | Off-by-one (rare)                                             |
| `scanf("%s")`      | Input                     | delimiter အဖြစ်ရပ် (မသိမ်း)                                  | terminator အလိုလိုထည့်                        | Heap overflow (if large input)                                |
| `scanf("%[^\n]")`  | Input                     | `\n` ကိုတွေ့မှရပ် (keep no)                                  | `\0` ထည့်                                     | Bypass `\n` filter                                            |
| `getchar()`        | Input                     | character `0x0A` အနေနဲ့ return                               | (မသက်ဆိုင်)                                   | Read one byte (leak)                                          |
| `read()` (syscall) | Input (raw)               | raw byte `0x0A` အနေနဲ့ သိမ်း                                 | `\0` မထည့်ပေး                                 | Heap/stack overflow (no null terminator)                      |
| `strcpy()`         | Copy                      | ပုံမှန် byte `0x0A` အဖြစ် copy                               | src ရဲ့ `\0` အထိ copy                         | Off-by-one, null byte overflow                                |
| `strncpy()`        | Copy (limited)            | ပုံမှန် copy                                                 | n bytes ပြည့်ရင် `\0` **မထည့်**               | Missing null terminator → info leak                           |
| `strcat()`         | Concatenate               | ပုံမှန် byte အဖြစ်                                           | dest ရဲ့ `\0` နေရာကစပြီး src ကို `\0` အထိ     | Buffer overflow (chaining)                                    |
| `strncat()`        | Concatenate (limited)     | ပုံမှန်                                                      | အနည်းဆုံး `\0` တစ်လုံးထည့်                    | Safe? still overflow possible                                 |
| `sprintf()`        | Output (string)           | `\n`ကိုစာလုံးအဖြစ် output                                    | format string ထဲက `%s` က `\0` အထိ             | Format string (write)                                         |
| `snprintf()`       | Output (bounded)          | `\n` ကိုစာလုံးအဖြစ်                                          | n-1 bytes ပြီးရင် `\0` ထည့်                   | Safe but can truncate                                         |
| `printf()`         | Output (formatted)        | `\n` တွေ့ → buffer flush                                     | `%s` အတွက် `\0` အထိပဲ print                   | Format string vulnerability (leak)                            |
| `fprintf()`        | Output (file)             | `\n` → line buffering                                        | `%s` → `\0` terminator                        | File stream exploitation                                      |
| `puts()`           | Output (string)           | string ရဲ့အဆုံးမှာ `\n` အလိုအလျောက်ထည့်                      | `\0` အထိပဲ output                             | No direct vuln (but use with `\0` overwrite)                  |
| `fputs()`          | Output (file)             | `\n` **မထည့်ပေး** (လိုရင်ကိုယ်တိုင်ထည့်)                     | `\0` အထိပဲ output                             | Similar to puts                                               |
| `strchr()`         | Search                    | `\n` ကို normal char အနေနဲ့ရှာ                               | `\0` ကိုရှာရင် NULL return                    | Find newline in buffer                                        |
| `strstr()`         | Search substring          | substring ထဲက `\n` ကိုပုံမှန်                                | `\0` termination                              | Pattern matching for ROP                                      |
| `memcpy()`         | Memory copy               | raw byte `0x0A`                                              | `\0` ကိုလည်း raw byte အနေနဲ့ copy             | No null termination concept                                   |
| `memmove()`        | Memory copy (overlap)     | Same as memcpy                                               | Same as memcpy                                | Same as memcpy                                                |
| `memset()`         | Memory set                | `\n` ကို byte value `0x0A` အနေနဲ့                            | `\0` ကို byte value `0x00` အနေနဲ့             | Fill buffer with nulls or newlines                            |
| `strtok()`         | Tokenize                  | delimiter (ဥပမာ `"\n"`) အနေနဲ့သုံးလို့ရ                      | internal static pointer ကို `\0` ထိ update    | Bypass delimiter checks                                       |
| `open()`           | File descriptor (syscall) | `\n` ကို pathname ထဲမှာ normal byte (0x0A)                   | `\0` → path string terminator                 | Path traversal (`/proc/self/fd`), race condition (TOCTOU)     |
| `write()`          | Output (syscall)          | raw byte `0x0A` ကို **အလိုအလျောက်မပြောင်းဘဲ** တိုက်ရိုက်ထုတ် | `\0` ကိုလည်း raw byte `0x00` အနေနဲ့ ထုတ်နိုင် | Arbitrary write (if fd可控), info leak (write to stdout/stderr) |
| `read()`           | Input (syscall)           | raw byte `0x0A` ကိုသိမ်း                                     | `\0` **မထည့်ပေး** (manual terminate လုပ်ရ)    | Heap/stack overflow, off-by-one, no null terminator           |
| `close()`          | Resource mgmt             | (မသက်ဆိုင်)                                                  | (မသက်ဆိုင်)                                   | Use-after-free (UAF) with fd reuse                            |
| `lseek()`          | File offset               | (မသက်ဆိုင်)                                                  | (မသက်ဆိုင်)                                   | Arbitrary read/write (seek to controlled offset)              |

----

## Wrap around 


	wrap around  က ဘာလဲဆို နာရီလိုပါဘဲ 12 ကျော်သွားရင် 1 က ပြန်စတယ်
	max ကျော်သွားရင် min က min အောက်လျော့သွားရင် max ကပြန်စပြီးလျော့
	wrap arround က byte တွေဘယ်လောက်ယူတာလဲဆိုအပေါ် range ကွဲပြားပါယတယ်
	အဲဒါအပြင် singned (+ or -) ကြရင် - ဘက်သို့wrap arround ဖြစ်ပြီး unsigned (+)ကြရင် 0ကနေစပြီး wrap arround ဖြစ်ပါတယ်
### C Language မှာ:

| Data Type | Size (bits) | Range                           | Calculation   |
| --------- | ----------- | ------------------------------- | ------------- |
| char      | 8           | -128 to 127                     | -2⁷ to 2⁷-1   |
| short     | 16          | -32,768 to 32,767               | -2¹⁵ to 2¹⁵-1 |
| int       | 32          | -2,147,483,648 to 2,147,483,647 | -2³¹ to 2³¹-1 |
| long      | 32 or 64    | System dependent                |               |
| long long | 64          | -9.22×10¹⁸ to 9.22×10¹⁸         | -2⁶³ to 2⁶³-1 |

### Unsigned Integers (အနှုတ်မပါ):

| Data Type      | Size | Range              | Calculation |
| -------------- | ---- | ------------------ | ----------- |
| unsigned char  | 8    | 0 to 255           | 0 to 2⁸-1   |
| unsigned short | 16   | 0 to 65,535        | 0 to 2¹⁶-1  |
| unsigned int   | 32   | 0 to 4,294,967,295 | 0 to 2³²-1  |

---

# Statically linked and dynamically linked

### `Statically linked`

	program compile လုပ်တဲ့အချိန်မှာ libc ထဲက လိုအပ်တဲ့ function တွေကို binary နဲ့အတူတစ်ပါးတည်းထည့်ပေးတာ ဆိုတော့ libc ကို porgaram run တဲ့အချိန်မှာမသုံးဘူး ထည့်ထားတဲ့ function address တွေကိုဘဲယူလို့ရမယ် 
	-PIE disable ဆိုရင် memory address တိုက်ရိုက်ယူလို့ရတယ် enable ဆိုရင် binary base address ကနေ offset ပေး
	-အခြား binary ထဲက ပါတဲ့ function တွေသုံးလို့မရဘူး


```
from pwn import *
elf = ELF('./static_prog')

# ဒီလိုရတယ်
print(hex(elf.symbols['main']))     # 0x400c8c - Real address
print(hex(elf.symbols['read']))     # 0x41b9a0 - Real address in binary
print(hex(elf.symbols['printf']))   # 0x46c340 - Real address in binary

# ဒီလိုမရဘူး
print(elf.symbols['execve'])        # KeyError! (မသုံးထားလို့မပါ)
```



### `Dynamically linked`

	compile လုပ်တဲ့ အချိန်မှာ လိုအပ်တဲ့  built in function မှန်သမျှ libc ဆီမှာသွားတောင်းတယ်

```
elf = ELF('./dynamic_prog')

# ကိုယ်ပိုင်တွေရတယ်
print(hex(elf.symbols['main']))     # 0x401136 - Real address
print(hex(elf.symbols['my_func']))  # 0x40114b - Real address

# Libc functions တွေက symbols table ထဲမှာမရှိ!
# print(elf.symbols['printf'])      # KeyError!
# print(elf.symbols['read'])        # KeyError!

# ဒါပေမယ့် PLT မှာရှိ
print(hex(elf.plt['printf']))       # 0x401030 - PLT stub address
print(hex(elf.plt['read']))         # 0x401040 - PLT stub address
```

