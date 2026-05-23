
### Registers (အရေးကြီးဆုံး)

Registers တွေက သင်္ချန်တဲ့နေရာ။ x86-64 CPU မှာ အဓိက ၁၆ ခုရှိတယ်။

**8-bit registers (သေးငယ်ဆုံး)**

|Register|အမည်အပြည့်|အသုံး|
|---|---|---|
|AL|Accumulator Low|တွက်ချက်ရန်|
|BL|Base Low|memory address အတွက်|
|CL|Counter Low|loop ရေတွက်ရန်|
|DL|Data Low|data သိမ်းရန်|

**16-bit registers** (AH + AL ပေါင်းထားတာ)

- AX = AH (high byte) + AL (low byte)
    
- BX, CX, DX အလားတူ
    

**32-bit registers**

- EAX, EBX, ECX, EDX
    

**64-bit registers**

- RAX, RBX, RCX, RDX
    

> 📌 **မှတ်ရန်**: မင်း AL ကိုထိရင် 8 bits ပဲထိတယ်။ EAX ထိရင် 32 bits လုံးကိုထိတယ်။

#### လေ့ကျင့်ခန်း (Chapter 2 - စာမျက်နှာ ၁)

- AL က ဘယ်နှစ် bit လဲ။ (အဖြေ: 8 bits)
    
- AX က AL နဲ့ ဘယ်လိုဆက်စပ်သလဲ။ (အဖြေ: AX ရဲ့ အနိမ့်ဆုံး 8 bits က AL)