

SIB = Scale (အဆ) + Index (ညွှန်း) + Base (အခြေခံ)
SIB Byte က x86 instruction မှာ `ModRM` ပြီးရင် လိုက်တဲ့ 1 byte 
ဒါက complex memory addressing အတွက်သုံးတယ်

```
Instruction Format:
┌──────────┬─────────┬─────────┬──────────┬──────────┐
│  Opcode  │ ModRM   │   SIB   │ Displacement │ Immediate │
└──────────┴─────────┴─────────┴──────────┴──────────┘
                      ↑
                   SIB Byte (ModRM.R/M = 100 မှသာ)
```

ဘယ်အချိန်မှာ SIB လိုအပ်သလဲ -
- `ModRM.R/M` = `100` (4) ဖြစ်နေရင်
- Complex addressing ပုံစံတွေအတွက် - `[EAX + ECX*4]`, `[EBX + ESI*8 + 10]` 


 SIB Byte ရဲ့ ဖွဲ့စည်းပုံ (8 bits)
```
SIB Byte (8 bits)
┌──────────────┬───────────────┬───────────────┐
│  Scale (2 bits) │ Index (3 bits) │  Base (3 bits)  │
│   bits 7-6      │  bits 5-3      │  bits 2-0       │
└──────────────┴───────────────┴───────────────┘
```

| အပိုင်း   | Bit နေရာ | အလုပ်                                     |
| --------- | -------- | ----------------------------------------- |
| **Scale** | bits 7-6 | Index ကို ဘယ်နှစ်ဆမြှောက်မလဲ (1, 2, 4, 8) |
| **Index** | bits 5-3 | ညွှန်းတဲ့ register (ESI, EDI, ECX, etc.)  |
| **Base**  | bits 2-0 | အခြေခံ register (EAX, EBX, EBP, etc.)     |


##### - Scale (2 bits) 

Scale က Index register ရဲ့တန်ဖိုးကို ဘယ်နှစ်ဆမြှောက်မလဲ ဆိုတာကို သတ်မှတ်တယ်။

| Scale (binary) | Scale (hex) | မြှောက်ကိန်း | ဥပမာ |
|----------------|-------------|--------------|--------|
| 00 | 0 | ×1 | `[EAX + ECX]` |
| 01 | 1 | ×2 | `[EAX + ECX*2]` |
| 10 | 2 | ×4 | `[EAX + ECX*4]` (array of 4-byte ints) |
| 11 | 3 | ×8 | `[EAX + ECX*8]` (array of 8-byte doubles) |

> **သတိထားရန်** - Scale ကို **00, 01, 10, 11** လို့ 2 bits နဲ့ သိမ်းတယ်။ ဒါပေမဲ့ မြှောက်ကိန်းက `2^scale` ဖြစ်တယ် ဒါမှမဟုတ် ဇယားအတိုင်း မှတ်ထား

---

##### - Index (3 bits) 

Index က ဘယ် register ကို ညွှန်းမလဲ ဆိုတာကို သတ်မှတ်တယ်။

| Index (binary) | Index (hex) | 32-bit Register                                  | 64-bit Register |
| -------------- | ----------- | ------------------------------------------------ | --------------- |
| 000            | 0           | EAX                                              | RAX             |
| 001            | 1           | ECX                                              | RCX             |
| 010            | 2           | EDX                                              | RDX             |
| 011            | 3           | EBX                                              | RBX             |
| 100            | 4           | **No Index** (special - means no index register) |                 |
| 101            | 5           | EBP                                              | RBP             |
| 110            | 6           | ESI                                              | RSI             |
| 111            | 7           | EDI                                              | RDI             |

	 Index = 100 (4) အထူးအနေအထား

Index = `100` (4) ဆိုရင် **index register မရှိဘူး**။ ဒါက `[EAX + ECX*4]` မှာ ECX မပါဘဲ `[EAX]` ပုံစံအတွက်သုံးတယ်။

---

##### - Base (3 bits) 

Base က ဘယ် register ကို အခြေခံအဖြစ်သုံးမလဲ ဆိုတာကို သတ်မှတ်တယ်။

| Base (binary) | Base (hex) | 32-bit Register | 64-bit Register |
|---------------|------------|-----------------|-----------------|
| 000 | 0 | EAX | RAX |
| 001 | 1 | ECX | RCX |
| 010 | 2 | EDX | RDX |
| 011 | 3 | EBX | RBX |
| 100 | 4 | ESP | RSP |
| 101 | 5 | EBP | RBP |
| 110 | 6 | ESI | RSI |
| 111 | 7 | EDI | RDI |

	 Base = 101 (5) အထူးအနေအထား (Mod=00 မှာ)

**Mod=00 (no displacement)** နဲ့ **Base=101 (EBP)** ဆိုရင် **base register မရှိဘူး**။ ဒါက `[disp32]` ပုံစံအတွက်သုံးတယ်။ (displacement တစ်ခုတည်းနဲ့)

```
Mod=00, R/M=100 (SIB), Base=101 → [disp32] (SIB without base)
```
---


SIB Byte Formula

```
Memory Address = Base + Index × (2^Scale) + Displacement

Scale = 0 → ×1
Scale = 1 → ×2
Scale = 2 → ×4
Scale = 3 → ×8
```

**ဥပမာ -** `[EBX + ECX*4 + 8]`
- Base = EBX
- Index = ECX
- Scale = 2 (×4)
- Displacement = 8



(Cheat Sheet)

| Assembly | Scale | Index | Base | SIB (hex) |
|----------|-------|-------|------|-----------|
| `[EAX + ECX]` | 00 (×1) | 001 (ECX) | 000 (EAX) | 08 |
| `[EAX + ECX*2]` | 01 (×2) | 001 (ECX) | 000 (EAX) | 48 |
| `[EAX + ECX*4]` | 10 (×4) | 001 (ECX) | 000 (EAX) | 88 |
| `[EAX + ECX*8]` | 11 (×8) | 001 (ECX) | 000 (EAX) | C8 |
| `[EBX + ESI*4]` | 10 (×4) | 110 (ESI) | 011 (EBX) | B3 |
| `[ECX*4]` (no base) | 10 (×4) | 001 (ECX) | 101 (no base) | 8D |
| `[EAX]` (no index) | 00 (×1) | 100 (no index) | 000 (EAX) | 04 |



----

#### Examples


- `mov eax, [ebx + ecx*4]`

```
Opcode: 8B (MOV r32, r/m32)
ModRM: Mod=00, Reg=000 (EAX), R/M=100 (SIB needed)
SIB: Scale=10 (×4), Index=001 (ECX), Base=011 (EBX)

ModRM = 00 000 100 = 04 (hex)
SIB   = 10 001 011 = 1000 1011 = 8B (hex)

Machine Code: 8B 04 8B
```

- `mov eax, [ebx + ecx*4 + 8]`

```
Opcode: 8B
ModRM: Mod=01 (disp8), Reg=000 (EAX), R/M=100 (SIB)
SIB: Scale=10 (×4), Index=001 (ECX), Base=011 (EBX)
Displacement: 8 (1 byte)

ModRM = 01 000 100 = 44 (hex)
SIB   = 8B (အပေါ်အတိုင်း)

Machine Code: 8B 44 8B 08
```

- `mov eax, [eax + ecx*2]` (array of 2-byte elements)

```
ModRM: Mod=00, Reg=000 (EAX), R/M=100
SIB: Scale=01 (×2), Index=001 (ECX), Base=000 (EAX)

SIB = 01 001 000 = 0100 1000 = 48 (hex)

Machine Code: 8B 04 48
```

- `mov eax, [esi*4]` (no base register)

```
ModRM: Mod=00, Reg=000 (EAX), R/M=100
SIB: Scale=10 (×4), Index=110 (ESI), Base=101 (special - no base)

SIB = 10 110 101 = 1011 0101 = B5 (hex)

Machine Code: 8B 34 B5 + displacement? Mod=00, R/M=100, Base=101 → disp32
             + 00 00 00 00 (if no displacement)
```

 - `mov al, [ebx + esi*8 + 1000]` (8-byte array)

```
8-bit instruction: Opcode 8A
ModRM: Mod=10 (disp32), Reg=000 (AL), R/M=100 (SIB)
SIB: Scale=11 (×8), Index=110 (ESI), Base=011 (EBX)
Displacement: 1000 (4 bytes, little-endian)

ModRM = 10 000 100 = 1000 0100 = 84 (hex)
SIB   = 11 110 011 = 1111 0011 = F3 (hex)
disp32 = E8 03 00 00 (1000 in little-endian)

Machine Code: 8A 84 F3 E8 03 00 00
```
---
