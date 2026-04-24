
## The Memory 

Program တစ်ခုကို ခေါ်လိုက်တဲ့အခါ section တွေက process ထဲက segment တွေဆီ ပုံဖော်ခံရတယ်။ ပြီးရင် ELF file မှာ ဖော်ပြထားတဲ့အတိုင်း segment တွေကို memory ထဲကို load လုပ်တယ်။

အောက်ပါအတိုင်း memory ရဲ့ အပိုင်းတွေကို ကြည့်ရအောင်။

### Buffer အကြောင်း အရင်ပြောရရင်

Memory layout မှာ အောက်ပါအတိုင်း အပိုင်းတွေ ပါတယ်-
- .text
- .data
- .bss
- Heap
- empty space
- Stack

(Memory address တွေက 0x00000000 ကနေ 0xFFFFFFFF အထိရှိတယ်)

### .text

.text section မှာ program ရဲ့ တကယ့် assembler instruction တွေ ပါတယ်။ ဒီနေရာကို read-only အဖြစ် သတ်မှတ်ထားတတ်တယ်။ ဒါမှ process က သူ့ရဲ့ instruction တွေကို မတော်တဆ ပြင်မိမှာ ကာကွယ်ဖို့ပါ။ ဒီနေရာကို write လုပ်ဖို့ ကြိုးစားရင် segmentation fault ဖြစ်သွားမယ်။

### .data

.data section မှာ program က တန်းပြီး initialize လုပ်ထားတဲ့ global နဲ့ static variable တွေ ပါတယ်။

### .bss

Compiler နဲ့ linker အများစုက .bss section ကို data segment ရဲ့ အစိတ်အပိုင်းတစ်ခုအနေနဲ့ သုံးတယ်။ ဒီမှာ statically allocated variable တွေ ပါတယ်။ ဒီ variable တွေကို 0 bits နဲ့ပဲ ကိုယ်စားပြုတယ်။

### The Heap

Heap memory ကို ဒီနေရာကနေ ခွဲဝေပေးတယ်။ ဒီနေရာက ".bss" segment ရဲ့ အဆုံးကနေ စပြီး memory address ပိုမြင့်တဲ့နေရာတွေဆီ ကြီးထွားသွားတယ်။

### The Stack

Stack memory က Last-In-First-Out (LIFO) data structure တစ်ခုပါ။ ဒီမှာ return addresses, parameters, နဲ့ compiler option ပေါ်မူတည်ပြီး frame pointers တွေ သိမ်းဆည်းတယ်။ C/C++ local variable တွေကို ဒီမှာ သိမ်းတယ်။ code ကိုတောင် stack ထဲကို ကူးယူလို့ရတယ်။

Stack က RAM ထဲက သတ်မှတ်ထားတဲ့ နေရာတစ်ခုပါ။ Linker က ဒီနေရာကို ကြိုတင်သတ်မှတ်ပေးပြီး များသောအားဖြင့် RAM ရဲ့ အောက်ပိုင်း၊ global နဲ့ static variable တွေရဲ့ အပေါ်မှာ ထားတယ်။ Stack pointer ကနေ contents တွေကို access လုပ်တယ်။

Initialization လုပ်စဉ်တုန်းက stack pointer ကို stack ရဲ့ အပေါ်ဆုံးနေရာမှာ ထားတယ်။ Execution လုပ်စဉ်မှာ stack ရဲ့ allocated အပိုင်းက အောက် memory address တွေဆီ ကြီးထွားသွားတယ်။

### ခေတ်မီ Memory Protections (DEP/ASLR)

ခေတ်မီ memory protection တွေဖြစ်တဲ့ DEP နဲ့ ASLR က buffer overflow ကြောင့်ဖြစ်တဲ့ ပျက်စီးမှုတွေကို ကာကွယ်ပေးတယ်။

#### DEP (Data Execution Prevention)

DEP က memory ရဲ့ နေရာအချို့ကို "Read-Only" အဖြစ် မှတ်သားထားတယ်။ User input တွေ သိမ်းတဲ့နေရာ (ဥပမာ Stack) ကို read-only လုပ်ထားတယ်။ DEP ရဲ့ နောက်ကွယ်က စိတ်ကူးက user တွေ shellcode ကို memory ထဲတင်ပြီး instruction pointer ကို အဲဒီ shellcode ဆီ ညွှန်ပြခွင့် မပေးဖို့ပါ။

ဒါကြောင့် hacker တွေ ဒီအတားအဆီးကို ကျော်ဖို့ ROP (Return Oriented Programming) ကို စတင်သုံးလာကြတယ်။ ROP က shellcode ကို executable space ထဲ တင်ပြီး existing call တွေကို သုံးပြီး execute လုပ်ခွင့်ပေးတယ်။

#### ASLR (Address Space Layout Randomization)

ROP နဲ့ဆိုရင် attacker က memory address တွေ သိဖို့ လိုတယ်။ ဒါကြောင့် ASLR ဆိုတဲ့ ကာကွယ်ရေးနည်းလမ်းကို ထည့်သွင်းတယ်။ ASLR က အရာအားလုံး သိမ်းဆည်းတဲ့နေရာကို ကျပန်းဖြစ်အောင် လုပ်ပေးတယ်။ ဒါက ROP ကို ပိုခက်ခဲစေတယ်။

User တွေက memory address တွေ ပေါက်ကြားအောင် လုပ်ပြီး ASLR ကို ကျော်လွှားနိုင်တယ်။ ဒါပေမယ့် ဒါက exploit တွေကို ယုံကြည်ရမှု နည်းစေပြီး တစ်ခါတစ်လေ မဖြစ်နိုင်တော့ဘူး။

ဥပမာ - "Freefloat FTP Server" က Windows XP ပေါ်မှာ (DEP/ASLR မတိုင်ခင်) ကို exploit လုပ်ရတာ အရမ်းလွယ်တယ်။ ဒါပေမယ့် ဒီ application ကို ခေတ်မီ Windows operating system ပေါ်မှာ သုံးကြည့်ရင် buffer overflow က ရှိနေပေမယ့် DEP/ASLR ကြောင့် exploit လုပ်ရတာ ခက်ခဲသွားတယ် (memory address တွေ ပေါက်ကြားအောင် လုပ်နိုင်တဲ့ နည်းလမ်း မရှိသေးလို့ပါ)။