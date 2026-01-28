
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

