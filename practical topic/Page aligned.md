


**Page** ဆိုတာ Computer ရဲ့ Memory (RAM) ကို စီမံခန့်ခွဲတဲ့အခါ အခြေခံအနေနဲ့ ပိုင်းခြားထားတဲ့ **အတုံးအခဲ (Block)** တစ်ခုပါ။ ဒီအတုံးတစ်ခုရဲ့ အရွယ်အစားကို **Page Size** လို့ခေါ်ပြီး၊ ပုံမှန်အားဖြင့် **4KB (4096 bytes)** ဖြစ်ပါတယ်။ (စနစ်ပေါ်မူတည်ပြီး 2MB, 1GB လည်းရှိနိုင်ပါတယ်။)

**Page-Aligned** ဆိုတာကတော့ **Memory Address (နေရာလိပ်စာ) တစ်ခုဟာ Page Size ရဲ့ ဆထက်ထ (multiple) အတိအကျ ဖြစ်နေခြင်း**ကို ဆိုလိုပါတယ်။

ဥပမာ Page Size = 4096 (0x1000) ဆိုရင်၊ အောက်ပါ Address တွေဟာ Page-Aligned ဖြစ်ပါတယ်။

- `0x00000000`
- `0x00001000` (4096)
- `0x00002000` (8192)
- `0x7fff1000` စသဖြင့်။

ဒီ Address တွေရဲ့ နောက်ဆုံး 12 bits (Hexadecimal နဲ့ဆိုရင် နောက်ဆုံး 3 digits) က `000` ဖြစ်နေမှာ ဖြစ်ပါတယ်။ (ဘာဖြစ်လို့လဲဆိုတော့ 4096 = 2^12 ဖြစ်လို့ပါ။)

---

 Binary Exploitation မှာ ဘာကြောင့် အရေးပါတာလဲ

Hacker တွေအနေနဲ့ Memory Corruption (Buffer Overflow, Use-After-Free, Heap Spraying စသဖြင့်) ကို အသုံးချတဲ့အခါ Page-Aligned က အောက်ပါ အချက်တွေအတွက် အရမ်းအရေးကြီးပါတယ်။

####  Memory Mapping (mmap) နဲ့ ဆက်စပ်မှု
Operating System (OS) က Memory ကို ခွဲဝေပေးတဲ့အခါ (malloc ကြီးကြီးတွေ၊ mmap နဲ့ Shared Libraries တွေ) **အမြဲတမ်း Page-Aligned ဖြစ်အောင် ခွဲဝေပေးပါတယ်။**
ဒါကြောင့် သင်က Heap Spraying လုပ်ပြီး Shellcode တွေ ဖြည့်တဲ့အခါ၊ သင်ထည့်လိုက်တဲ့ Data ရဲ့ အစပိုင်း (starting address) က Page-Aligned ဖြစ်နေတတ်ပါတယ်။

####  NOP Sled နဲ့ တွဲသုံးခြင်း
Exploit ရေးတဲ့အခါ ခုန်ပျံဖို့ (Jump) အတွက် NOP (No-Operation) စာကြောင်းတွေ အများကြီး ထည့်လေ့ရှိပါတယ်။
သင်က Return Address ကို ပြောင်းပြီး ခုန်ချင်တဲ့ နေရာက **Page-Aligned ဖြစ်နေရင်**၊ CPU ရဲ့ Cache Line နဲ့ TLB (Translation Lookaside Buffer) တွေကို ပိုပြီး ကောင်းကောင်း အသုံးချနိုင်ပြီး၊ Exploit အောင်မြင်နှုန်း (Reliability) ပိုမြင့်မားစေပါတယ်။

####  Heap Exploitation (ptmalloc) နဲ့ ဆိုင်ခြင်း
Heap ပေါ်မှာ ကြီးမားတဲ့ Memory (ဥပမာ 512KB အထက်) တောင်းတဲ့အခါ `mmap` က သွားခွဲပေးပြီး အဲဒီ Address က Page-Aligned ဖြစ်နေပါတယ်။
ဒီအခါ သင်က **House of Force** လိုမျိုး Attack တွေလုပ်တဲ့အခါ Page-Aligned ကို တွက်ချက်ပြီး Top Chunk ရဲ့ Size ကို လှည့်စားရာမှာ အသုံးပြုပါတယ်။

####  Kernel Exploitation (Linux Kernel)
Kernel ထဲမှာ Page-Aligned က ပိုပြီး အရေးပါပါတယ်။
ဥပမာ - **Double-Free** လိုမျိုး Vulnerability တွေမှာ Kernel က Page-Aligned မဟုတ်တဲ့ Address ကို Free ခိုင်းရင် BUG() လို့ခေါ်တဲ့ Error တက်ပြီး Kernel Panic ဖြစ်သွားတတ်ပါတယ်။
ဒါ့အပြင် **Userfaultfd** လိုမျိုး Technique တွေမှာ Page-Aligned ဖြစ်မှသာ အောင်မြင်စွာ အလုပ်လုပ်ပါတယ်။

---

 လက်တွေ့ ဘယ်လို စစ်ဆေးမလဲ။

Address တစ်ခုက Page-Aligned ဟုတ်မဟုတ် စစ်ချင်ရင် Bitwise Operation သုံးပါတယ်။

```
(address & (PAGE_SIZE - 1)) == 0
```

ဥပမာ Page Size = 4096 (0x1000) ဆိုရင် `(address & 0xFFF) == 0` ဖြစ်ရပါမယ်။
GDB လိုမျိုး Debugger မှာ `info proc mappings` ရိုက်ကြည့်ရင် Memory Region အကုန်လုံးရဲ့ အစပိုင်းက Page-Aligned ဖြစ်နေတာ တွေ့ရပါမယ်။

---

##### Alignment vs Page-Aligned)

- **Alignment** (ဥပမာ 4-byte alignment, 8-byte alignment) က Data Type (int, long) နဲ့ ဆိုင်ပြီး၊
- **Page-Aligned** က Memory Management (OS နဲ့ MMU) နဲ့ ဆိုင်ပါတယ်။

Exploit ရေးတဲ့အခါ သင်က Return Address ကို Overwrite လုပ်တဲ့အခါ သူ့ကို Page-Aligned ဖြစ်အောင် သေချာရွေးချယ်ရင်၊ **Segmentation Fault** ဖြစ်နိုင်ခြေ နည်းပြီး **Stable Exploit** ရဖို့ ပိုလွယ်ကူစေပါတယ်

----

