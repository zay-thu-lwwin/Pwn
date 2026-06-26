

### CPU ဆိုတာဘာလဲ (Central Processing Unit)

CPU က ကွန်ပျူတာရဲ့ **ဦးနှောက်**။ ၎င်းက **machine code (binary)** ကိုဖတ်ပြီး အလုပ်လုပ်တယ်။

CPU ရဲ့ အဓိက အစိတ်အပိုင်း (၃) ခု

1. **ALU (Arithmetic Logic Unit)** - တွက်ချက်တယ် (ပေါင်း၊ နှုတ်၊ AND, OR)
    
2. **CU (Control Unit)** - instruction တွေကို ဖြန့်ဝေတယ်
    
3. **Registers** - သေးငယ်တဲ့ မြန်ဆန်တဲ့ memory (CPU ထဲမှာ)

## CPU Architecture 

Von-Neumann ဗိသုကာကို Hungarian သင်္ချာပညာရှင် John von Neumann က တီထွင်ခဲ့တာပါ။ ၎င်းမှာ အဓိက အပိုင်း ၄ ပိုင်း ပါဝင်တယ်-

- **Memory** (မှတ်ဉာဏ်)
- **Control Unit** (ထိန်းချုပ်ယူနစ်)
- **Arithmetical Logical Unit (ALU)** (ဂဏန်းသင်္ချာနဲ့ ယုတ္တိယူနစ်)
- **Input/Output Unit** (ထည့်သွင်း/ထုတ်ယူ ယူနစ်)

Von-Neumann ဗိသုကာမှာ အရေးကြီးဆုံး ယူနစ်နှစ်ခုဖြစ်တဲ့ ALU နဲ့ Control Unit (CU) ကို Central Processing Unit (CPU) ထဲမှာ ပေါင်းစပ်ထားတယ်။ CPU က instruction တွေကို execute လုပ်ဖို့နဲ့ flow control အတွက် တာဝန်ယူတယ်။ Instruction တွေကို တစ်ခုပြီးတစ်ခု အဆင့်လိုက် လုပ်ဆောင်သွားတယ်။ CU က memory ကနေ command နဲ့ data တွေကို ယူတယ်။

Processor, memory, နဲ့ input/output unit ကြား ဆက်သွယ်မှုကို bus system လို့ခေါ်တယ်။ ဒါက Von-Neumann မူရင်းမှာ မပါပေမယ့် လက်တွေ့မှာ အရေးပါတယ်။ Von-Neumann ဗိသုကာမှာ instruction နဲ့ data အားလုံးကို bus system ကနေ လွှဲပြောင်းတယ်။

### Memory (မှတ်ဉာဏ်)

Memory ကို နှစ်မျိုးခွဲလို့ရတယ်-

- **Primary Memory** (ပင်မမှတ်ဉာဏ်)
- **Secondary Memory** (ဒုတိယမှတ်ဉာဏ်)

#### Primary Memory

Primary Memory ဆိုတာ Cache နဲ့ Random Access Memory (RAM) ပါ။ ယုတ္တိအရ စဉ်းစားရင် memory ဆိုတာ information တွေ သိမ်းဖို့ နေရာတစ်ခုပါ။ ကိုယ့်သူငယ်ချင်းဆီမှာ ပစ္စည်းတစ်ခုခု သိမ်းထားပြီး နောက်မှ ပြန်ယူသလိုမျိုးပေါ့။ ဒါပေမယ့် အဲဒီအတွက် ပြန်ယူဖို့ သူငယ်ချင်းရဲ့ လိပ်စာကို သိဖို့လိုတယ်။ RAM လည်း အဲဒီလိုပါပဲ။ RAM က memory address တွေနဲ့ တိုက်ရိုက် random ဝင်ရောက်လို့ရတဲ့ memory အမျိုးအစားပါ။

Cache က processor ထဲမှာ တပ်ဆင်ထားပြီး buffer အနေနဲ့ အလုပ်လုပ်တယ်။ အကောင်းဆုံးအခြေအနေမှာ processor ကို data နဲ့ program code တွေ အမြဲကျွေးမွေးနိုင်အောင် လုပ်ပေးတယ်။ Program code နဲ့ data တွေ processor ထဲ မဝင်ခင် RAM က data သိုလှောင်ရာနေရာအဖြစ် ဆောင်ရွက်တယ်။ RAM ရဲ့ size က processor အတွက် သိမ်းဆည်းနိုင်တဲ့ data ပမာဏကို ဆုံးဖြတ်ပေးတယ်။ ဒါပေမယ့် primary memory က ပါဝါပြတ်သွားရင် သိမ်းထားသမျှ အကုန်ဆုံးရှုံးသွားတယ်။

#### Secondary Memory

Secondary Memory ဆိုတာ HDD/SSD, Flash Drives, CD/DVD-ROMs လိုမျိုး ပြင်ပ data သိုလှောင်ရာတွေပါ။ ဒါတွေကို CPU က တိုက်ရိုက် access မလုပ်ဘဲ I/O interfaces ကနေ သွားတယ်။ ဒါက mass storage device တစ်ခုပါ။ လက်ရှိမလုပ်ဆောင်ရသေးတဲ့ data တွေကို အမြဲတမ်းသိမ်းဖို့ သုံးတယ်။ Primary memory နဲ့ ယှဉ်ရင် storage capacity ပိုများတယ်၊ ပါဝါမပါဘဲ အမြဲတမ်းသိမ်းထားနိုင်တယ်၊ ဒါပေမယ့် အများကြီး နှေးတယ်။

### Control Unit (ထိန်းချုပ်ယူနစ်)

Control Unit (CU) က processor ရဲ့ အစိတ်အပိုင်းတွေ မှန်မှန်ကန်ကန် အတူတူအလုပ်လုပ်ဖို့ တာဝန်ယူတယ်။ CU ရဲ့ အလုပ်တွေကို အောက်ပါအတိုင်း ခြုံပြောလို့ရတယ်-

- RAM ကနေ data ဖတ်ခြင်း
- RAM ထဲကို data သိမ်းခြင်း
- Instruction တစ်ခုကို ပေးအပ်၊ decode၊ နဲ့ execute လုပ်ခြင်း
- peripheral device တွေဆီက input တွေ လုပ်ဆောင်ခြင်း
- peripheral device တွေဆီကို output တွေ လုပ်ဆောင်ခြင်း
- Interrupt ထိန်းချုပ်ခြင်း
- စနစ်တစ်ခုလုံး စောင့်ကြည့်ခြင်း

CU မှာ Instruction Register (IR) ပါတယ်။ ဒီထဲမှာ processor က decode လုပ်ပြီး သင့်တော်သလို execute လုပ်မယ့် instruction တွေ အားလုံးပါတယ်။ Instruction decoder က instruction တွေကို ဘာသာပြန်ပြီး execution unit ကို ပေးတယ်။ ပြီးရင် execution unit က instruction ကို execute လုပ်တယ်။ Execution unit က data တွေကို ALU ဆီ တွက်ချက်ဖို့ လွှဲပြောင်းပေးပြီး အဖြေကို ပြန်လက်ခံတယ်။ Execute လုပ်စဉ်မှာ သုံးတဲ့ data တွေကို registers ထဲမှာ ယာယီသိမ်းတယ်။

### Central Processing Unit (CPU)

Central Processing Unit (CPU) က ကွန်ပျူတာထဲက တကယ့် processing power ကို ပေးတဲ့ functional unit ပါ။ Information တွေ လုပ်ဆောင်ဖို့နဲ့ processing operations တွေ ထိန်းချုပ်ဖို့ တာဝန်ယူတယ်။ ဒီလိုလုပ်ဖို့ CPU က memory ကနေ command တွေကို တစ်ခုပြီးတစ်ခု ယူပြီး data processing ကို စတင်တယ်။

Processor ကို single electronic circuit ထဲမှာ ထည့်ထားရင် (ဥပမာ ကိုယ့် PC တွေထဲက) Microprocessor လို့လည်း မကြာခဏ ခေါ်ကြတယ်။

CPU တိုင်းမှာ သူတည်ဆောက်ထားတဲ့ architecture တစ်ခုစီရှိတယ်။ လူသိအများဆုံး CPU architecture တွေက-

- **x86/i386** - (AMD & Intel)
- **x86-64/amd64** - (Microsoft & Sun)
- **ARM** - (Acorn)

ဒီ CPU architecture တွေထဲက တစ်ခုချင်းစီကို သီးခြားနည်းလမ်းနဲ့ တည်ဆောက်ထားတယ်။ အဲဒါကို Instruction Set Architecture (ISA) လို့ခေါ်တယ်။ CPU က သူ့ရဲ့ processes တွေ execute လုပ်ဖို့ ISA ကို သုံးတယ်။ ISA က CPU ရဲ့ အပြုအမူကို သုံးတဲ့ instruction set နဲ့ ပတ်သက်ပြီး ဖော်ပြတယ်။ Instruction set တွေကို သီးခြား implementation တစ်ခုနဲ့ မမှီခိုဘဲ သတ်မှတ်ထားတယ်။ အထူးသဖြင့် ISA က machine code ရဲ့ unified behavior ကို assembly language ထဲက registers, data types စတာတွေနဲ့ နားလည်နိုင်အောင် ပေးတယ်။

ISA အမျိုးအစား ၄ မျိုးရှိတယ်-

- **CISC** - Complex Instruction Set Computing
- **RISC** - Reduced Instruction Set Computing
- **VLIW** - Very Long Instruction Word
- **EPIC** - Explicitly Parallel Instruction Computing

### RISC

RISC က Reduced Instruction Set Computer ရဲ့ အတိုကောက်ပါ။ ဒါက microprocessor architecture ရဲ့ ဒီဇိုင်းတစ်မျိုးဖြစ်ပြီး assembly programming အတွက် instruction set ရဲ့ ရှုပ်ထွေးမှုကို clock cycle တစ်ခုအထိ ရိုးရှင်းအောင် ရည်ရွယ်တယ်။ ဒါက CPU ရဲ့ clock frequency ကို ပိုမြင့်စေပြီး instruction set သေးသေးတွေသုံးလို့ ပိုမြန်ဆန်တယ်။ Instruction set ဆိုတာ ပေးထားတဲ့ processor တစ်ခု execute လုပ်နိုင်တဲ့ machine instruction တွေရဲ့ အစုကို ဆိုလိုတယ်။ RISC ကို ဒီနေ့ခေတ် smartphone အများစုမှာ တွေ့နိုင်တယ်။ ဒါပေမယ့် CPU အားလုံးနီးပါးမှာ RISC ရဲ့ အစိတ်အပိုင်းတစ်ခုခု ပါတယ်။ RISC architecture တွေမှာ 32-bit နဲ့ 64-bit ဆိုပြီး instruction တွေရဲ့ အလျား fixed ဖြစ်တယ်။

### CISC

RISC နဲ့ ဆန့်ကျင်ဘက်အနေနဲ့ Complex Instruction Set Computer (CISC) က ကျယ်ပြန့်ပြီး ရှုပ်ထွေးတဲ့ instruction set ပါတဲ့ processor architecture တစ်ခုပါ။ ကွန်ပျူတာတွေရဲ့ သမိုင်းကြောင်းအရ recurring instruction sequences တွေကို ဒုတိယမျိုးဆက် ကွန်ပျူတာတွေမှာ ရှုပ်ထွေးတဲ့ instruction တွေအဖြစ် ပေါင်းစပ်ခဲ့ကြတယ်။ CISC architecture မှာ addressing က RISC နဲ့မတူဘဲ 32-bit ဒါမှမဟုတ် 64-bit မလိုအပ်ဘဲ 8-bit mode နဲ့လည်း လုပ်လို့ရတယ်။

### Instruction Cycle

Instruction set က processor တစ်ခုရဲ့ machine instruction တွေ အားလုံးပေါင်းစုကို ဖော်ပြတယ်။ Instruction set ရဲ့ နယ်ပယ်က processor type ပေါ်မူတည်ပြီး သိသိသာသာ ကွာခြားနိုင်တယ်။ CPU တစ်ခုချင်းစီမှာ မတူညီတဲ့ instruction cycles နဲ့ instruction sets ရှိနိုင်ပေမယ့် structure အားလုံးက ဆင်တူပြီး အောက်ပါအတိုင်း ခြုံပြောလို့ရတယ်-

| Instruction | ရှင်းလင်းချက် |
|---|---|
| 1. FETCH | Instruction Address Register (IAR) ကနေ နောက် machine instruction address ကိုဖတ်တယ်။ ပြီးရင် Cache ဒါမှမဟုတ် RAM ကနေ Instruction Register (IR) ထဲကို load လုပ်တယ်။ |
| 2. DECODE | Instruction decoder က instruction တွေကို ပြောင်းလဲပေးပြီး instruction ကို execute လုပ်ဖို့ လိုအပ်တဲ့ circuits တွေကို စတင်တယ်။ |
| 3. FETCH OPERANDS | Execute လုပ်ဖို့ နောက်ထပ် data တွေ ထပ်မံထည့်သွင်းရမယ်ဆိုရင် အဲဒီ data တွေကို cache ဒါမှမဟုတ် RAM ကနေ working registers ထဲကို load လုပ်တယ်။ |
| 4. EXECUTE | Instruction ကို execute လုပ်တယ်။ ဒါက ဥပမာ ALU ထဲက operations၊ program ထဲက jump တစ်ခု၊ result တွေကို working registers ထဲ ပြန်ရေးခြင်း၊ ဒါမှမဟုတ် peripheral devices တွေကို ထိန်းချုပ်ခြင်းတို့ ဖြစ်နိုင်တယ်။ တချို့ instruction တွေရဲ့ result ပေါ်မူတည်ပြီး status register ကို set လုပ်တယ်။ ဒါကို နောက်ထပ် instruction တွေက အကဲဖြတ်နိုင်တယ်။ |
| 5. UPDATE INSTRUCTION POINTER | EXECUTE အဆင့်မှာ jump instruction ကို execute မလုပ်ခဲ့ရင် IAR ကို instruction ရဲ့ အလျားအတိုင်း တိုးပေးလိုက်တယ်။ ဒါမှ နောက် machine instruction ကို ညွှန်ပြလိုက်တာပေါ့။ |

```
1. Fetch (ညွှန်ကြားချက်ကို RAM ကနေ ဆွဲယူ)
   ↓
2. Decode (ဘာလုပ်ရမလဲဆိုတာ စက်)

3. Execute (တွက်ချက်ခြင်း / ရွှေ့ခြင်း)

4. Store (ရလဒ်ကို ပြန်သိမ်း)
```