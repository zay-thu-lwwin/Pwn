
#####  Stack Alignment and Return-Oriented Programming (ROP)

Heap မှာ `16-byte alignment` ရှိသလို၊ 64-bit Ubuntu/Linux ရဲ့ Stack ပေါ်မှာလည်း **Stack Alignment** စည်းမျဉ်း ရှိပါတယ်

- **The Rule:** x86_64 ABI (Application Binary Interface) စည်းမျဉ်းအရ `call` instruction တစ်ခု မပွင့်ခင် (Function တစ်ခုထဲ မဝင်ခင်) မှာ Stack Pointer (`$rsp`) ရဲ့ လိပ်စာဟာ `16` နဲ့ စားပြတ်ရပါမယ် (Must be 16-byte aligned)
- **The Component:** `movaps` ဆိုတဲ့ CPU instruction ကြီး ဖြစ်ပါတယ် သူက Stack ပေါ်က Data တွေကို လျင်မြန်စွာ ရွှေ့ပြောင်းပေးတဲ့ကောင် ဖြစ်ပြီး `$rsp` သာ 16 နဲ့ စားမပြတ်ရင် (Misaligned ဖြစ်ရင်) ပရိုဂရမ်ကို ချက်ချင်း **Crash (Segmentation Fault)** ဖြစ်စေပါတယ်


#####  Binary Exploitation (ROP) affect

Buffer Overflow ကနေတစ်ဆင့် ROP Chain ရေးပြီး `system("/bin/sh")` ကို လှမ်းခေါ်တဲ့အခါ ပရိုဂရမ်က `system()` ထဲရောက်ခါနီး `movaps` instruction နဲ့ တိုးပြီး ခဏခဏ Crash တတ်ပါတယ်။

- **The Fix (The ROP Skip):** ဟက်ကာတွေက ဒါကို ကျော်ဖို့အတွက် ROP Chain ထဲမှာ ဘာမှမလုပ်တဲ့ `ret` gadget (Stack ကို 8 bytes တိုးပေးမည့်ကောင်)** တစ်ခုကို ကြားထဲ ညှပ်ထည့်လိုက်ရပါတယ်။ အဲဒီအခါ Misaligned ဖြစ်နေတဲ့ Stack က `+8 bytes` တိုးသွားပြီး 16-byte ကွက်တိ ပြန်ဖြစ်သွားတဲ့အတွက် `system()` ကို ချောချောမွေ့မွေ့ ခေါ်နိုင်သွားတာ ဖြစ်ပါတယ်(ဒါကို CTF လောကမှာ "The `ret` gadget trick for stack alignment" လို့ လူသိများပါတယ်)
    

##### Structure Padding in C and Type Confusion

C language နဲ့ ရေးထားတဲ့ Binary တွေထဲက Struct (Structure) တွေထဲမှာလည်း Alignment စည်းမျဉ်း ရှိပါတယ် CPU ဖတ်ရလွယ်အောင် Struct ထဲက Member variable တွေကြားထဲမှာ Heap manager လိုပဲ **Padding (နေရာအလွတ်)** တွေ ထည့်သွင်းတတ်ပါတယ်။


```c
struct Vulnerable {
    char type;     // 1 byte
    // 7 bytes padding space (Alignment အရ ကြားညှပ်လာခြင်း)
    int *pointer;  // 8 bytes (64-bit alignment စည်းမျဉ်းကြောင့် 8 ဆတ္တံကိန်းနေရာမှ စရမည်)
};
```

##### effect on Binary Exploitation 

ဟက်ကာတစ်ယောက်က Structure layout ရဲ့ Alignment ကွာဟချက်ကို သေချာတွက်ချက်ပြီး **Type Confusion** သို့မဟုတ် **Information Leak** တိုက်ကွက်တွေ သုံးနိုင်ပါတယ်

- ဥပမာ- ကြားထဲက ကွန်ပျူတာ အလိုအလျောက် ဖြည့်လိုက်တဲ့ Padding `7 bytes` နေရာအလွတ်ထဲမှာ အရင်က သုံးဖူးခဲ့တဲ့ Memory Address အဟောင်းတွေ (ဥပမာ `libc` သို့မဟုတ် `stack` address တွေ) ကျန်နေတတ်ပါတယ်
- ပရိုဂရမ်က အဲဒီ struct ကြီးတစ်ခုလုံးကို initialization သေချာမလုပ်ဘဲ `write()` သို့မဟုတ် `printf()` နဲ့ ထုတ်ပြလိုက်ရင် အဲဒီ ကြားညှပ် padding တွေထဲက address တွေ လျှံထွက်လာပြီး **ASLR Bypass (Address Leak)** ရသွားတတ်ပါတယ်
    

##### Page Alignment and Shellcode Execution

Operating System (Kernel) က Memory ကို စီမံတဲ့အခါ Chunk အသေးလေးတွေအပြင် **Page (ပေ့ချ်)** လို့ခေါ်တဲ့ $4096\text{ bytes}$ ($4\text{ KB}$) အကျယ်ရှိတဲ့ Memory အတုံးကြီးတွေနဲ့ သိမ်းပါတယ်။ ဒါကို **Page Alignment** စည်းမျဉ်းလို့ ခေါ်ပါတယ်

- **The Rule:** Memory ရဲ့ ခွင့်ပြုချက် (Read, Write, Execute - `rwx` flags) တွေကို Kernel က ပေးတဲ့အခါ Byte တစ်ခုချင်းစီ ပေးလို့မရပါဘူး။ Page alignment အလိုက် $4\text{ KB}$ တစ်တုံးစီကိုပဲ Flag သတ်မှတ်ပေးလို့ ရပါတယ်။
    

##### Binary Exploitation ပေါ် သက်ရောက်မှု:

မင်းက `mprotect()` စနစ်ကို သုံးပြီး Stack သို့မဟုတ် Heap ပေါ်မှာ ကိုယ်ပိုင် Shellcode (စက်ကုဒ်) ကို Run ချင်တယ် ဆိုပါစို့။

- `mprotect(target_address, size, PROT_READ | PROT_WRITE | PROT_EXEC)` ကို လှမ်းခေါ်တဲ့အခါ မင်းပေးလိုက်တဲ့ `target_address` ဟာ **အမြဲတမ်း $4096$ နဲ့ စားပြတ်တဲ့ Page Boundary Address ဖြစ်ရပါမယ်** (ဥပမာ- `0x...000` နဲ့ ဆုံးရမည်)။
    
- အကယ်၍ `0x555555559123` လိုမျိုး အလယ်ကောင် address ကြီး သွားပေးရင် `mprotect` က `EINVAL` (Invalid Argument) error ပြပြီး Shellcode execution အလုပ်လုပ်မှာ မဟုတ်ပါဘူး။ ဒါကြောင့် Shellcode မပြေးခင် address ကို `0xfffff000` နဲ့ `AND` တွက်ပြီး Page Align အရင်လုပ်ပေးရပါတယ်။
    

##### Shellcode Vector Alignment (SIMD/AVX Exploitation)

Modern Exploitation တွေမှာ (ဥပမာ- Browser Exploit တွေ၊ Remote Kernel Exploit တွေ) တိုက်ခိုက်သူတွေဟာ CPU ရဲ့ Multi-media instruction တွေဖြစ်တဲ့ **AVX (Advanced Vector Extensions)** သို့မဟုတ် **SSE** တွေကို အသုံးချပြီး Data တွေကို တစ်ပြိုင်နက် အများကြီး ဖတ်/ရေး လုပ်တတ်ကြပါတယ်။

- **The Rule:** AVX register တွေဟာ $256\text{-bit}$ หรือ $512\text{-bit}$ အထိ ကျယ်တာကြောင့် Memory Address ဟာ **`32-byte alignment` သို့မဟုတ် `64-byte alignment`** တိတိကျကျ ရှိနေဖို့ လိုအပ်ပါတယ်။
    
- **The Exploitation Context:** မင်းရဲ့ Shellcode သို့မဟုတ် ROP payload က ညွှန်ပြလိုက်တဲ့ shellcode storage address သာ ဗွီဒီယို ဖတ်တဲ့ ကုဒ်တွေနဲ့ သွားတိုးလို့ Misaligned ဖြစ်သွားရင် CPU က အလုပ်မလုပ်တော့ဘဲ တစ်ခါတည်း Crash သွားမှာ ဖြစ်ပါတယ်။
    

