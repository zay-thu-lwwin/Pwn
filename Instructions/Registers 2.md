


#### Register Types


| အမျိုးအစား                           | အလုပ်                                           | ဥပမာ                                           | သုံးစွဲခွင့်                 |
| ------------------------------------ | ----------------------------------------------- | ---------------------------------------------- | ---------------------------- |
| **General Purpose Registers (GPRs)** | တွက်ချက်ခြင်း၊ data သိမ်းခြင်း                  | RAX, RBX, RCX, RDX, RSI, RDI, RBP, RSP, R8-R15 | User (Ring 3)                |
| **Pointer/Index Registers**          | memory address ညွှန်ခြင်း                       | RSP, RBP, RSI, RDI                             | User                         |
| **Instruction Pointer**              | နောက် execute လုပ်မယ့် instruction ရဲ့ address  | RIP                                            | User                         |
| **Status/Flags Register**            | ရလဒ်ရဲ့ အခြေအနေ (zero, carry, etc.)             | RFLAGS (32-bit: EFLAGS, 16-bit: FLAGS)         | User                         |
| **Segment Registers**                | memory segment ကိုညွှန်ပြခြင်း (protected mode) | CS, DS, SS, ES, FS, GS                         | User (mostly read-only)      |
| **Control Registers**                | CPU ရဲ့ အခြေခံအလုပ်လုပ်ပုံကိုထိန်းချုပ်         | CR0, CR2, CR3, CR4                             | Kernel (Ring 0)              |
| **Debug Registers**                  | hardware breakpoints အတွက်                      | DR0, DR1, DR2, DR3, DR6, DR7                   | Kernel/Debugger              |
| **Test Registers** (legacy)          | စမ်းသပ်ခြင်း ( obsolete )                       | TR3-TR7 (ခေတ်နောက်ကျ)                          | Kernel                       |
| **Model-Specific Registers (MSRs)**  | CPU-specific features                           | IA32_EFER, STAR, LSTAR, etc.                   | Kernel (via `rdmsr`/`wrmsr`) |
| **Memory Management Registers**      | paging, GDT, IDT, task state                    | GDTR, IDTR, LDTR, TR                           | Kernel                       |

| အမျိုးအစား                           | အလုပ်                                          | ဥပမာ                                   | သုံးစွဲခွင့်    |
| ------------------------------------ | ---------------------------------------------- | -------------------------------------- | --------------- |
| **General Purpose Registers (GPRs)** | တွက်ချက်ခြင်း၊ data သိမ်းခြင်း                 | EAX, EBX, ECX, EDX, ESI, EDI, EBP, ESP | User (Ring 3)   |
| **Pointer/Index Registers**          | memory address ညွှန်ခြင်း                      | ESP, EBP, ESI, EDI                     | User            |
| **Instruction Pointer**              | နောက် execute လုပ်မယ့် instruction ရဲ့ address | EIP                                    | User            |
| **Status/Flags Register**            | ရလဒ်ရဲ့ အခြေအနေ (zero, carry, etc.)            | EFLAGS                                 | User            |
| **Segment Registers**                | memory segment ကိုညွှန်ပြခြင်း                 | CS, DS, SS, ES, FS, GS                 | User            |
| **Control Registers**                | CPU ရဲ့ အခြေခံအလုပ်လုပ်ပုံကိုထိန်းချုပ်        | CR0, CR2, CR3, CR4                     | Kernel (Ring 0) |
| **Debug Registers**                  | hardware breakpoints အတွက်                     | DR0, DR1, DR2, DR3, DR6, DR7           | Kernel/Debugger |
| **Memory Management Registers**      | GDT, IDT, TSS                                  | GDTR, IDTR, LDTR, TR                   | Kernel          |
| **Task Register**                    | task state segment                             | TR                                     |                 |


ဘာလို့  RSP (Stack Pointer), RBP (Base Pointer), RSI (Source Index), RDI (Destination Index) ဒီကောင်တွေက  general Purpose registerတွေထဲမှာရောပါတာလဲဆိုတော့ သူတို့ကို လိုအပ်ရင် သာမန် GPRs လိုလည်း သုံးလို့ရလို့  အထူးအလုပ်တွေအတွက် "ပုံမှန်အားဖြင့်" သုံးပေမဲ့ လိုအပ်ရင် တခြားတန်ဖိုးတွေလည်း သိမ်းလို့ရတယ်


```
┌────────────────────────────────────────────────────────────────────┐
│                    REGISTER TYPES (x86-64)                         │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  GENERAL PURPOSE (GPRs) - 16 registers                            │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ RAX, RBX, RCX, RDX  │  A,B,C,D family                        │  │
│  │ RSI, RDI, RBP, RSP  │  Pointer/Index                         │  │
│  │ R8, R9, R10, R11    │  Extended (64-bit only)                │  │
│  │ R12, R13, R14, R15  │  Extended (64-bit only)                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  INSTRUCTION POINTER - 1 register                                  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ RIP  │ Next instruction address (cannot mov directly)        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  FLAGS REGISTER - 1 register                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ RFLAGS │ CF, ZF, SF, OF, AF, PF, TF, IF, DF, etc.           │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  SEGMENT REGISTERS - 6 registers (legacy)                          │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ CS, DS, SS, ES, FS, GS  │ Segment selectors                  │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  CONTROL REGISTERS - 5 registers (kernel only)                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ CR0, CR2, CR3, CR4  │ Control paging, protection, etc.      │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  DEBUG REGISTERS - 8 registers (debugger only)                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ DR0, DR1, DR2, DR3, DR6, DR7  │ Hardware breakpoints        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

##### 16-bit ခေတ် (8086 - 1978)
Register တွေက အလွန်ရှားပါးတယ် ( 8 registers ပဲရှိတယ်

|Register|အထူးအလုပ်|GPR အဖြစ်|
|---|---|---|
|AX, BX, CX, DX|yes|yes|
|SI, DI|string operations|yes|
|BP, SP|stack frame|yes (SP ကို သတိထားရ)|
ဒီအချိန်တုန်းက ရှားပါးလွန်းလို့ register အားလုံးကို GPRs လို ပြန်သုံးနိုင်အောင် လုပ်ထားတယ်





| အမျိုးအစား              | အရေအတွက်               | သုံးစွဲခွင့်အဆင့်       | အသုံးများသော instruction           |
| ----------------------- | ---------------------- | ----------------------- | ---------------------------------- |
| **GPRs**                | 16 (RAX-R15)           | User (Ring 3)           | mov, add, sub, mul, div            |
| **Pointer/Index**       | 4 (RSP, RBP, RSI, RDI) | User                    | push, pop, movsb, rep              |
| **Instruction Pointer** | 1 (RIP)                | User (indirectly)       | jmp, call, ret                     |
| **Flags**               | 1 (RFLAGS)             | User                    | cmp, test, jcc (conditional jumps) |
| **Segment**             | 6 (CS, DS, etc.)       | User (mostly read-only) | mov to/from segment (rare)         |
| **Control**             | 5 (CR0-CR4)            | Kernel (Ring 0)         | mov cr0, rax                       |
| **Debug**               | 8 (DR0-DR7)            | Kernel/Debugger         | mov dr0, rax                       |


##### General Purpose Registers (GPRs)  (64 bit mode)

| 64-bit | 32-bit | 16-bit | 8-bit (low) | 8-bit (high) | အထူးသုံးဆောင်ချက် |
| ------ | ------ | ------ | ----------- | ------------ | ----------------- |
| RAX    | EAX    | AX     | AL          | AH           | တွက်ချက်မှုရလဒ်   |
| RBX    | EBX    | BX     | BL          | BH           | base pointer      |
| RCX    | ECX    | CX     | CL          | CH           | loop counter      |
| RDX    | EDX    | DX     | DL          | DH           | I/O, division     |
| RSI    | ESI    | SI     | SIL         | -            | source index      |
| RDI    | EDI    | DI     | DIL         | -            | destination index |
| RBP    | EBP    | BP     | BPL         | -            | stack frame base  |
| RSP    | ESP    | SP     | SPL         | -            | stack pointer     |
| R8     | R8D    | R8W    | R8B         | -            | လိုသလိုသုံး       |
| R9     | R9D    | R9W    | R9B         | -            | လိုသလိုသုံး       |
| R10    | R10D   | R10W   | R10B        | -            | လိုသလိုသုံး       |
| R11    | R11D   | R11W   | R11B        | -            | လိုသလိုသုံး       |
| R12    | R12D   | R12W   | R12B        | -            | လိုသလိုသုံး       |
| R13    | R13D   | R13W   | R13B        | -            | လိုသလိုသုံး       |
| R14    | R14D   | R14W   | R14B        | -            | လိုသလိုသုံး       |
| R15    | R15D   | R15W   | R15B        | -            | လိုသလိုသုံး       |

##### General Purpose Registers (GPRs)  (32 bit mode)

| 32-bit     | 16-bit | 8-bit (low) | 8-bit (high) | မှတ်ချက်          |
| ---------- | ------ | ----------- | ------------ | ----------------- |
| EAX        | AX     | AL          | AH           | A family          |
| EBX        | BX     | BL          | BH           | B family          |
| ECX        | CX     | CL          | CH           | C family          |
| EDX        | DX     | DL          | DH           | D family          |
| ESI        | SI     | SIL         | -            | Source index      |
| EDI        | DI     | DIL         | -            | Destination index |
| EBP        | BP     | BPL         | -            | Base pointer      |
| ESP        | SP     | SPL         | -            | Stack pointer     |
| **R8-R15** | -      | -           | -            | ❌ **မရှိ**        |


> **မှတ်ချက်** - AH, BH, CH, DH က 16-bit ခေတ်အမွေ။ SIL, DIL စတာတွေက 64-bit ခေတ်မှအသစ်။

##### 32bit  Vs 64bit

| 64-bit (x86-64)          | 32-bit (IA-32)      | ကွာခြားချက်                    |
| ------------------------ | ------------------- | ------------------------------ |
| RAX (64-bit)             | EAX (32-bit)        | 64-bit က နှစ်ဆကြီးတယ်          |
| RBX                      | EBX                 |                                |
| RCX                      | ECX                 |                                |
| RDX                      | EDX                 |                                |
| RSI                      | ESI                 |                                |
| RDI                      | EDI                 |                                |
| RBP                      | EBP                 |                                |
| RSP                      | ESP                 |                                |
| **R8-R15** (8 registers) | **မရှိ**            | 32-bit မှာ ဒီ register မရှိဘူး |
| **RIP** (64-bit)         | **EIP** (32-bit)    | Instruction pointer            |
| **RFLAGS** (64-bit)      | **EFLAGS** (32-bit) | Flags register                 |

##### Special Purpose Registers  (for both)

နောက်ထပ် special purpose register အနေနဲ့ ဒီကောာင်တွေပါတယ်
ဒီ register တွေကို ကိုယ်တိုင် တိုက်ရိုက်သုံးတတ်ဖို့လိုတယ်

| Register                 | အမည်                | အလုပ်လုပ်ပုံ                                                   |
| ------------------------ | ------------------- | -------------------------------------------------------------- |
| **RIP** (EIP for 32 bit) | Instruction Pointer | နောက်ထပ် execute လုပ်မယ့် instruction ရဲ့ address ကို ညွှန်တယ် |
| **RFLAGS**               | Status Flags        | carry, zero, sign, overflow စတဲ့ condition တွေသိမ်း            |
| **CR0-CR4**              | Control Registers   | protected mode, paging စတာကိုထိန်း                             |
| **DR0-DR7**              | Debug Registers     | breakpoint အတွက်                                               |


| Mode       | Register အမည် | Size    | မှတ်ချက်                   |
| ---------- | ------------- | ------- | -------------------------- |
| **64-bit** | **RFLAGS**    | 64 bits | Upper 32 bits are reserved |
| **32-bit** | **EFLAGS**    | 32 bits | -                          |
| **16-bit** | **FLAGS**     | 16 bits | Lower 16 bits of EFLAGS    |



#####  Status/Flags Register (RFLAGS / EFLAGS) 

Flag တွေကို "ဆုံးဖြတ်ချက်ချဖို့" သုံးတယ် CPU က တွက်ချက်ပြီးတဲ့အခါ "ဘာဖြစ်သွားလဲ" ဆိုတာကို flag တွေထဲမှာ သိမ်းထားတယ်။ ပြီးရင် ဒီ flag တွေကိုကြည့်ပြီး ဘယ်လိုဆက်လုပ်မလဲ ဆုံးဖြတ်တယ်

```
 "5 ကနေ 3 ကိုနှုတ်" လို့ CPU ကိုပြောလိုက်တယ်။
CPU တွက်လိုက်တယ် → ရလဒ် 2 ရတယ်။
ဒါပေမဲ့ အရေးကြီးတာက -

1. ရလဒ် သုညလား? → No → ZF = 0
2. ရလဒ် အနုတ်လား? → No (2 က positive) → SF = 0
3. ငှားလိုက်ရလား? (borrow) → No → CF = 0
4. Overflow ဖြစ်လား? → No → OF = 0

ဒီ flag တွေကို ကြည့်ပြီး "အနုတ်ဖြစ်ရင် ဒီကိုသွား၊ သုညဖြစ်ရင် ဒီကိုသွား" စသဖြင့် ဆုံးဖြတ်တယ်
```

| Mode       | Register အမည် | Size    | မှတ်ချက်                   |
| ---------- | ------------- | ------- | -------------------------- |
| **64-bit** | **RFLAGS**    | 64 bits | Upper 32 bits are reserved |
| **32-bit** | **EFLAGS**    | 32 bits | -                          |
| **16-bit** | **FLAGS**     | 16 bits | Lower 16 bits of EFLAGS    |

```c
64-bit RFLAGS ပုံစံ
┌─────────────────────────────────────────────────────────────────────────────┐
│ Bit 63 ─────────────────────────────────────────────── Bit 32 │ Bit 31 ── Bit 0 │
│                                                                │               │
│                    Reserved (must be 0)                        │    EFLAGS     │
│                                                                │   (32 bits)   │
└────────────────────────────────────────────────────────────────┴───────────────┘

ဆိုလိုတာက - RFLAGS ရဲ့ အောက်ဆုံး 32 bits က EFLAGS နဲ့ အတူတူပဲ။
အပေါ်ဆုံး 32 bits က သုံးမထားဘူး (reserved)။
```

```text
RFLAGS (64-bit, အောက်ဆုံး 32-bit ကို EFLAGS လို့ခေါ်)
┌─────────────────────────────────────────────────────────────┐
│bit... 22 21 20 19 18 17 16 15 14 13 12 11 10 9 8 7 6 5 4 3 2 1 0│
│        │  │  │  │  │  │  │  │  │  │  │  │  │  │  │ │ │ │ │ │ │ │
│        CF  PF  AF  ZF  SF  TF  IF  DF  OF
└─────────────────────────────────────────────────────────────┘
```


| Flag     | Bit   | အမည်                      | ဘယ်အချိန်မှာ 1 ဖြစ်မလဲ                                       | ဥပမာ                                |
| -------- | ----- | ------------------------- | ------------------------------------------------------------ | ----------------------------------- |
| **CF**   | 0     | Carry Flag                | ပေါင်းလို့ အပေါ်ကို သယ်သွားရင် (unsigned overflow)           | `0xFF + 0x01 = 0x00`                |
| **PF**   | 2     | Parity Flag               | ရလဒ်ရဲ့ အောက်ဆုံး 8 bits မှာ 1 က အရေအတွက် **စုံ** (even) ရင် | `0x03` (00000011) → 1 နှစ်ခု → PF=1 |
| **AF**   | 4     | Auxiliary Carry Flag      | BCD တွက်ရင် bit 3 ကနေ bit 4 ကိုသယ်ရင်                        | BCD arithmetic                      |
| **ZF**   | 6     | Zero Flag                 | ရလဒ် **0** ဖြစ်ရင်                                           | `0x05 - 0x05 = 0x00`                |
| **SF**   | 7     | Sign Flag                 | ရလဒ် **အနုတ်** (negative) ဖြစ်ရင် (MSB=1)                    | `0x01 - 0x02 = 0xFF`                |
| **TF**   | 8     | Trap Flag                 | single-step debugging အတွက်                                  | (debugger သုံး)                     |
| **IF**   | 9     | Interrupt Flag            | interrupts ကို enable/disable                                | CLI/STI instructions                |
| **DF**   | 10    | Direction Flag            | string operation direction (up/down)                         | CLD/STD instructions                |
| **OF**   | 11    | Overflow Flag             | signed တွက်ရင် ပြည့်လျှံရင်                                  | `0x7F + 0x01 = 0x80`                |
| **IOPL** | 12-13 | I/O Privilege Level       | I/O port access အဆင့်                                        | (kernel use)                        |
| **NT**   | 14    | Nested Task Flag          | task nesting အတွက်                                           | (obsolete)                          |
| **RF**   | 16    | Resume Flag               | debug exception ပြီးရင် resume                               | (debugger use)                      |
| **VM**   | 17    | Virtual 8086 Mode Flag    | virtual 8086 mode ဖွင့်ရန်                                   | (obsolete)                          |
| **AC**   | 18    | Alignment Check Flag      | memory alignment စစ်ရန်                                      | (kernel use)                        |
| **VIF**  | 19    | Virtual Interrupt Flag    | virtual interrupt                                            | (obsolete)                          |
| **VIP**  | 20    | Virtual Interrupt Pending | virtual interrupt pending                                    | (obsolete)                          |
| **ID**   | 21    | ID Flag                   | CPUID instruction support ရှိမရှိ                            | CPU feature detection               |


**ဥပမာ** - `mov al, 0xFF` ပြီးရင် `add al, 0x01`

```
AL = 0xFF (11111111) + 1 = 256 → 8-bit မှာ 00000000 ဖြစ်သွား
CF = 1 (သယ်သွားလို့)
ZF = 1 (ရလဒ်က 0 ဖြစ်လို့)
```


```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        STATUS FLAGS QUICK REFERENCE                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Flag  Bit  အမည်              ဘာအတွက်                       စစ်တဲ့ jump   │
│  ─────────────────────────────────────────────────────────────────────────  │
│  CF    0    Carry Flag        Unsigned overflow              JA, JB, JAE, JBE│
│  PF    2    Parity Flag       Even parity (low byte)         (rare)         │
│  AF    4    Aux Carry         BCD arithmetic                 (rare)         │
│  ZF    6    Zero Flag         Result is zero                 JE, JZ, JNE    │
│  SF    7    Sign Flag         Result is negative             JS, JNS        │
│  TF    8    Trap Flag         Single-step debug              (debugger)     │
│  IF    9    Interrupt Flag    Interrupt enable/disable       (kernel)       │
│  DF    10   Direction Flag    String direction (up/down)     (rare)         │
│  OF    11   Overflow Flag     Signed overflow                JO, JNO, JG, JL│
│                                                                             │
│  32-bit: EFLAGS (32 bits)                                                   │
│  64-bit: RFLAGS (64 bits) - lower 32 bits = EFLAGS                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```



#### Most important 6 flag

##### 1. CF (Carry Flag) - Bit 0

 Unsigned integer ပေါင်းခြင်း/နှုတ်ခြင်းမှာ အပေါ်ကိုသယ်သွားရင် (carry) သို့မဟုတ် ငှားရင် (borrow) 1 ဖြစ်တယ်

```asm
; Carry Flag ဥပမာ (8-bit)
mov al, 0xFF        ; AL = 255
add al, 0x01        ; AL = 0, CF = 1 (256 → 0 ဖြစ်သွားလို့)

; နှုတ်တဲ့အခါ (borrow)
mov al, 0x00
sub al, 0x01        ; AL = 0xFF, CF = 1 (ငှားလိုက်ရလို့)
```

**ဘာအတွက်သုံးလဲ**
- Multi-precision arithmetic (128-bit, 256-bit တွက်ရန်)
- Unsigned comparison (CF=1 ဆိုရင် ငယ်တယ်)



##### 2. ZF (Zero Flag) - Bit 6

 ရလဒ် 0 ဖြစ်ရင် 1 ဖြစ်တယ်

```asm
mov al, 0x05
sub al, 0x05        ; AL = 0 → ZF = 1

xor eax, eax        ; EAX = 0 → ZF = 1 (ဒါက zero register လုပ်နည်း)

cmp eax, ebx        ; EAX - EBX, result = 0 → ZF = 1 (equal)
```

**ဘာအတွက်သုံးလဲ**
- တန်ဖိုးနှစ်ခု တူမတူ စစ်ရန် (`je`, `jz` conditional jumps)
- Loop ထွက်ရန်



##### 3. SF (Sign Flag) - Bit 7

 - ရလဒ်ရဲ့ **Most Significant Bit (MSB)** ကို ပြတယ်။ signed integer အတွက် အနုတ်/အပြင်း စစ်ရန်

```asm
; Signed interpretation
mov al, 0x7F        ; AL = 127 (positive)
add al, 0x01        ; AL = 0x80 = -128 (negative) → SF = 1

mov al, 0x01
sub al, 0x02        ; AL = 0xFF = -1 (negative) → SF = 1

mov al, 0x10
sub al, 0x05        ; AL = 0x0B = 11 (positive) → SF = 0
```

**ဘာအတွက်သုံးလဲ**
- Signed comparison (`jg`, `jl`, `jge`, `jle`)
- ရလဒ် အနုတ်လား စစ်ရန်



##### 4. OF (Overflow Flag) - Bit 11

 - Signed integer ပေါင်းခြင်း/နှုတ်ခြင်းမှာ **ပြည့်လျှံမှု (overflow)** ဖြစ်ရင် 1 ဖြစ်တယ်

```asm
; Overflow ဥပမာ (8-bit signed)
mov al, 0x7F        ; AL = 127 (maximum positive)
add al, 0x01        ; AL = 0x80 = -128 → OF = 1 (overflow!)
                    ; CF = 0, OF = 1

; နောက်ထပ် overflow ဥပမာ
mov al, 0x80        ; AL = -128 (minimum negative)
sub al, 0x01        ; AL = 0x7F = 127 → OF = 1 (overflow again!)
```

CF vs OF 

| Flag | ဘာအတွက် | ဥပမာ |
|------|---------|-------|
| **CF** | Unsigned arithmetic | 255 + 1 = 0 (carry) |
| **OF** | Signed arithmetic | 127 + 1 = -128 (overflow) |

```asm
; တစ်ခါတလေ နှစ်ခုလုံးဖြစ်တယ်
mov al, 0xFF        ; AL = 255 (unsigned) / -1 (signed)
add al, 0x80        ; AL = 0x7F = 127
                    ; CF = 1 (255+128=383 → 127, carry)
                    ; OF = ? ( -1 + -128 = -129 → 127, overflow!)
```



##### 5. DF (Direction Flag) - Bit 10

 String instruction (`movsb`, `cmpsb`, `scasb`, etc.) တွေရဲ့ **ဦးတည်ချက်** ကိုသတ်မှတ်တယ်။

| DF တန်ဖိုး | ဦးတည်ချက် | အလုပ်လုပ်ပုံ |
|-----------|-----------|-------------|
| **DF = 0** | Up (increment) | RSI/RSI နဲ့ RDI/RDI တိုး (← ဘယ်မှညာ) |
| **DF = 1** | Down (decrement) | RSI/RSI နဲ့ RDI/RDI လျှော့ (→ ညာမှဘယ်) |

```asm
; DF = 0 (up) - အသုံးအများဆုံး
cld                 ; DF = 0 (clear direction flag)
mov rsi, src_addr
mov rdi, dst_addr
mov rcx, 100
rep movsb           ; RSI တိုး၊ RDI တိုး

; DF = 1 (down) - memory overlap ရှိရင် သုံး
std                 ; DF = 1 (set direction flag)
mov rsi, src_end
mov rdi, dst_end
mov rcx, 100
rep movsb           ; RSI လျှော့၊ RDI လျှော့
```



##### 6. IF (Interrupt Flag) - Bit 9

 Maskable interrupts (keyboard, timer, etc.) ကို **enable/disable** လုပ်တယ်။

```asm
cli                 ; IF = 0 (Clear Interrupts) → interrupts disabled
sti                 ; IF = 1 (Set Interrupts) → interrupts enabled
```

**ဘာအတွက်သုံးလဲ**
- Critical section (atomic operation) လုပ်ရန်
- Kernel code မှာ interrupt မလာစေရန်
- User mode မှာ မပြောင်းလဲနိုင် (privileged instruction)


#### how Flag works
##### Addition

```asm
; 8-bit addition
mov al, 0x7F        ; AL = 127
add al, 0x01        ; AL = 128 (0x80)
                    
; ရလဒ် Flag တွေ
; CF = 0 (unsigned မှာ 127+1=128, no carry)
; OF = 1 (signed မှာ 127+1 = -128, overflow)
; SF = 1 (result 0x80 = negative)
; ZF = 0 (result not zero)
```

##### Comparison

```asm
mov al, 0x05
cmp al, 0x05        ; AL - 0x05 = 0
; ZF = 1 (equal)

cmp al, 0x03        ; AL - 0x03 = 2
; ZF = 0 (not equal)
; CF = 0 (no borrow)
; SF = 0 (positive result)

cmp al, 0x08        ; AL - 0x08 = -3
; ZF = 0
; CF = 1 (borrow)
; SF = 1 (negative result)
```

##### Conditional Jumps (check flag before jump)

```asm
cmp eax, ebx

; Unsigned comparison
je  equal_label     ; Jumps if ZF = 1
ja  above_label     ; Jumps if CF = 0 and ZF = 0 (unsigned >)
jb  below_label     ; Jumps if CF = 1 (unsigned <)
jae above_equal     ; Jumps if CF = 0 (unsigned >=)
jbe below_equal     ; Jumps if CF = 1 or ZF = 1 (unsigned <=)

; Signed comparison
je  equal_label     ; Jumps if ZF = 1
jg  greater_label   ; Jumps if ZF = 0 and SF = OF (signed >)
jl  less_label      ; Jumps if SF != OF (signed <)
jge greater_equal   ; Jumps if SF = OF (signed >=)
jle less_equal      ; Jumps if ZF = 1 or SF != OF (signed <=)
```




----

