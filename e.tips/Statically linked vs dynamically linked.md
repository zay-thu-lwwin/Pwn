


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

