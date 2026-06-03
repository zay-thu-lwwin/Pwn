                           
`ModRM` = Mod (Mode) + Reg (Register) + R/M (Register/Memory)

`ModRM` byte က x86 instruction ရဲ့ ပထမ opcode ပြီးရင် လိုက်တဲ့ တစ် byte ပါ ဒါက operand တွေကို သတ်မှတ်တယ်

```
Instruction Format:
┌──────────┬─────────┬──────────┬──────────┬──────────┐
│  Opcode  │ ModRM   │ SIB      │ Displacement │ Immediate │
│  (1-3 bytes)│ (1 byte) │ (optional) │ (optional)  │ (optional) │
└──────────┴─────────┴──────────┴──────────┴──────────┘
```

`ModRM` are used when 
- Memory operand ပါတဲ့ instruction တွေ
- Register-to-register operation တွေ (mov al, bl စသည်)
- သတ်မှတ်ထားတဲ့ opcode တွေ (88, 89, 8A, 8B, C7, etc.)

##### Structure
```
ModRM Byte (8 bits)
┌──────────────┬───────────────┬───────────────┐
│   Mod (2 bits) │  Reg (3 bits)  │  R/M (3 bits)  │
│   bits 7-6     │  bits 5-3      │  bits 2-0      │
└──────────────┴───────────────┴───────────────┘
```

| အပိုင်း | Bit နေရာ | အလုပ်                                           |
| ------- | -------- | ----------------------------------------------- |
| **Mod** | bits 7-6 | ဒုတိယ operand က memory လား register လား သတ်မှတ် |
| **Reg** | bits 5-3 | ပထမ operand (register) ကို သတ်မှတ်              |
| **R/M** | bits 2-0 | ဒုတိယ operand (register/memory) ကို သတ်မှတ်     |



```
formula 1
ModRM = (1 << 6) | (2 << 3) | 3
= 64 | 16 | 3
= 64 + 16 + 3 = 83



fourmula 2
ModRM = (1 × 64) + (2 × 8) + 3
= 64 + 16 + 3 = 83



formula 3
Mod=1 → 01
Reg=2 → 010
R/M=3 → 011
ModRM = 01 010 011 = 01010011 = 83
```

#### how to know Mod value

 Step 1 - ဒုတိယ operand က register လား memory လား ဆုံးဖြတ်

| Second operand Type              | Mod Value                   |
| -------------------------------- | --------------------------- |
| **Register** (eg. `mov al, bl`)  | **11**                      |
| **Memory** (eg. `mov al, [ebx]`) | 00, 01, 10 (အောက်မှာဆက်ဖတ်) |

> **အဓိက** - ဒုတိယ operand က register ဆိုရင် **Mod = 11** အမြဲဖြစ်တယ်။

---

Step 2 - Memory ဆိုရင် displacement ပါလား ဆုံးဖြတ်

Memory operand အတွက် -

| Displacement ?                     | Mod Value                   |
| ---------------------------------- | --------------------------- |
| NO (eg. `[ebx]`)                   | **00**                      |
| YES  (eg. `[ebx+5]`, `[ebx+1000]`) | 01 သို့ 10 (အောက်မှာဆက်ဖတ်) |

---

Step 3 - displacement ပါရင် ဘယ်လောက်လဲ ဆုံးဖြတ်

| Displacement Size    | Mod Value | ဥပမာ                            |
| -------------------- | --------- | ------------------------------- |
| **1 byte** (disp8)   | **01**    | `[ebx + 5]` (5 က 1 byte)        |
| **4 bytes** (disp32) | **10**    | `[ebx + 1000]` (1000 က 4 bytes) |


```
                    ဒုတိယ operand က ဘာလဲ?
                           │
            ┌──────────────┴──────────────┐
            │                             │
         Register                       Memory
            │                             │
            ▼                             ▼
        Mod = 11                displacement ပါလား?
                                      │
                        ┌─────────────┴─────────────┐
                        │                           │
                     မပါဘူး                      ပါတယ်
                        │                           │
                        ▼                           ▼
                    Mod = 00              displacement ဘယ်လောက်?
                                                │
                                    ┌───────────┴───────────┐
                                    │                       │
                                   1 byte                 4 bytes
                                    │                       │
                                    ▼                       ▼
                                Mod = 01                Mod = 10
```



#### Examples
- `mov al, bl`
```
ဒုတိယ operand = bl (REGISTER)
→ Mod = 11
```

 - `mov al, [ebx]`
```
ဒုတိယ operand = [ebx] (MEMORY)
displacement မပါ
→ Mod = 00
```

- `mov al, [ebx + 5]`
```
ဒုတိယ operand = [ebx + 5] (MEMORY)
displacement = 5 (1 byte ထဲဝင်)
→ Mod = 01
```

 - `mov al, [ebx + 1000]`
```
ဒုတိယ operand = [ebx + 1000] (MEMORY)
displacement = 1000 (1 byte ထဲမဝင်၊ 4 bytes လို)
→ Mod = 10
```

 - `mov al, [ebx - 3]`
```
ဒုတိယ operand = [ebx - 3] (MEMORY)
displacement = -3 (1 byte ထဲဝင် - 0xFD)
→ Mod = 01
```



| Mode       | Mod အလုပ်လုပ်ပုံ                                          |
| ---------- | --------------------------------------------------------- |
| **32-bit** | အပေါ်ကအတိုင်း (EAX, EBX, ECX, EDX, ESI, EDI, EBP, ESP)    |
| **64-bit** | အတူတူပဲ (RAX, RBX, RCX, RDX, RSI, RDI, RBP, RSP + R8-R15) |

64-bit မှာ ထူးခြားချက် - R8-R15 သုံးရင် REX prefix လိုတယ်။ ဒါပေမဲ့ Mod ဆုံးဖြတ်ပုံက အတူတူပဲ

---

#### Mod

##### - Mod (2 bits) - defining mode

| Mod (binary) | Mod (hex) | အဓိပ္ပါယ်                               | ဥပမာ                 |
| ------------ | --------- | --------------------------------------- | -------------------- |
| **00**       | 0         | [R/M] (memory, no displacement)         | `mov al, [ebx]`      |
| **01**       | 1         | [R/M + disp8] (memory + 1-byte offset)  | `mov al, [ebx+5]`    |
| **10**       | 2         | [R/M + disp32] (memory + 4-byte offset) | `mov al, [ebx+1000]` |
| **11**       | 3         | R/M က register (no memory)              | `mov al, bl`         |

Mod = 11 → R/M က REGISTER ကို ပြတယ်
Mod ≠ 11 → R/M က MEMORY ADDRESSING MODE ကို ပြတယ်



##### Mod = 11 (Register Mode) 
 Mod=11 ဆိုရင် R/M က register ကို သတ်မှတ်တယ်

##### - 8-bit Registers (Mod=11)

| R/M (binary) | R/M (hex) | 8-bit Register |
| ------------ | --------- | -------------- |
| 000          | 0         | **AL**         |
| 001          | 1         | **CL**         |
| 010          | 2         | **DL**         |
| 011          | 3         | **BL**         |
| 100          | 4         | **AH**         |
| 101          | 5         | **CH**         |
| 110          | 6         | **DH**         |
| 111          | 7         | **BH**         |

##### - 16/32/64-bit Registers (Mod=11)

| R/M (binary) | R/M (hex) | 16-bit | 32-bit | 64-bit |
|--------------|-----------|--------|--------|--------|
| 000 | 0 | AX | EAX | RAX |
| 001 | 1 | CX | ECX | RCX |
| 010 | 2 | DX | EDX | RDX |
| 011 | 3 | BX | EBX | RBX |
| 100 | 4 | SP | ESP | RSP |
| 101 | 5 | BP | EBP | RBP |
| 110 | 6 | SI | ESI | RSI |
| 111 | 7 | DI | EDI | RDI |

(Mod=11)
```asm
mov al, bl    ; Mod=11, Reg=000(AL), R/M=011(BL) → ModRM = C3
mov eax, ebx  ; Mod=11, Reg=011(EBX), R/M=000(EAX) → ModRM = D8
```


#### Mod ≠ 11

Mod=00, 01, 10 တွေအတွက် R/M က memory addressing mode ကို သတ်မှတ်တယ်


```
ဥပမာ - R/M = 000
- Mod=00 → [EAX]
- Mod=01 → [EAX + disp8]
- Mod=10 → [EAX + disp32]
- Mod=11 → AL (register)

R/M တစ်ခုတည်းက Mod မတူရင် လုံးဝကွဲပြားတဲ့ အဓိပ္ပါယ်တွေ ရှိတယ်
```


##### Mod=00 (No Displacement) 

| R/M     | Mod=00 အဓိပ္ပါယ် | ရှင်းလင်းချက်                                     |
| ------- | ---------------- | ------------------------------------------------- |
| 000     | [EAX]            | EAX ထဲမှာရှိတဲ့ address                           |
| 001     | [ECX]            | ECX ထဲမှာရှိတဲ့ address                           |
| 010     | [EDX]            | EDX ထဲမှာရှိတဲ့ address                           |
| 011     | [EBX]            | EBX ထဲမှာရှိတဲ့ address                           |
| **100** | **[SIB]**        | **SIB byte လိုတယ်** (အဆင့်မြင့် addressing)       |
| **101** | **[disp32]**     | **32-bit address တိုက်ရိုက်** (base register မပါ) |
| 110     | [ESI]            | ESI ထဲမှာရှိတဲ့ address                           |
| 111     | [EDI]            | EDI ထဲမှာရှိတဲ့ address                           |


- **R/M=100** → SIB byte လိုတယ် (scale + index + base) - ဥပမာ `[EAX + ECX*4]`
- **R/M=101** → displacement တစ်ခုတည်း (base register မပါ) - ဥပမာ `[0x00401000]`


##### Mod=01 (8-bit Displacement) 

ဒါက **base address + 1-byte displacement** ကိုသုံးတယ်။

| R/M | Mod=01 အဓိပ္ပါယ် | ရှင်းလင်းချက် |
|-----|------------------|---------------|
| 000 | [EAX + disp8] | EAX + 1-byte offset |
| 001 | [ECX + disp8] | ECX + 1-byte offset |
| 010 | [EDX + disp8] | EDX + 1-byte offset |
| 011 | [EBX + disp8] | EBX + 1-byte offset |
| 100 | [SIB + disp8] | SIB + 1-byte offset |
| **101** | **[EBP + disp8]** | **EBP + 1-byte offset** (သတိထား - Mod=00 နဲ့မတူ) |
| 110 | [ESI + disp8] | ESI + 1-byte offset |
| 111 | [EDI + disp8] | EDI + 1-byte offset |

 - R/M=101 အတွက် Mod=00 နဲ့ Mod=01 မှာ လုံးဝမတူ
```
R/M=101
- Mod=00 → [disp32] (address တိုက်ရိုက်)
- Mod=01 → [EBP + disp8] (EBP base + 1-byte offset)
```



##### Mod=10 (32-bit Displacement)

ဒါက **base address + 4-byte displacement** ကိုသုံးတယ်

| R/M     | Mod=10 အဓိပ္ပါယ်   | ရှင်းလင်းချာ            |
| ------- | ------------------ | ----------------------- |
| 000     | [EAX + disp32]     | EAX + 4-byte offset     |
| 001     | [ECX + disp32]     | ECX + 4-byte offset     |
| 010     | [EDX + disp32]     | EDX + 4-byte offset     |
| 011     | [EBX + disp32]     | EBX + 4-byte offset     |
| 100     | [SIB + disp32]     | SIB + 4-byte offset     |
| **101** | **[EBP + disp32]** | **EBP + 4-byte offset** |
| 110     | [ESI + disp32]     | ESI + 4-byte offset     |
| 111     | [EDI + disp32]     | EDI + 4-byte offset     |



---

#### - Reg (3 bits) - defining Register (first operand)

4.1 8-bit Registers (for 8-bit instructions)

| Reg (binary) | Reg (hex) | 8-bit Register |
| ------------ | --------- | -------------- |
| 000          | 0         | AL             |
| 001          | 1         | CL             |
| 010          | 2         | DL             |
| 011          | 3         | BL             |
| 100          | 4         | AH             |
| 101          | 5         | CH             |
| 110          | 6         | DH             |
| 111          | 7         | BH             |



 4.2 16/32/64-bit Registers (for 16/32/64-bit instructions)

| Reg (binary) | Reg (hex) | 16-bit | 32-bit | 64-bit (with REX) |
| ------------ | --------- | ------ | ------ | ----------------- |
| 000          | 0         | AX     | EAX    | RAX               |
| 001          | 1         | CX     | ECX    | RCX               |
| 010          | 2         | DX     | EDX    | RDX               |
| 011          | 3         | BX     | EBX    | RBX               |
| 100          | 4         | SP     | ESP    | RSP               |
| 101          | 5         | BP     | EBP    | RBP               |
| 110          | 6         | SI     | ESI    | RSI               |
| 111          | 7         | DI     | EDI    | RDI               |

---

#### - R/M (3 bits) - second operand (Register/Memory)

5.1 Mod = 11 (Register mode) အတွက် R/M

Mod=11 ဆိုရင် R/M က **register** ကို သတ်မှတ်တယ်။

R/M (3 bits) ဟာ Mod ရဲ့တန်ဖိုးပေါ်မူတည်ပြီး အဓိပ္ပါယ် လုံးဝကွဲသွားတယ်။

Mod = 11 → R/M က REGISTER ကို ပြတယ်။
Mod ≠ 11 → R/M က MEMORY ADDRESSING MODE ကို ပြတယ်။

|R/M (binary)|R/M (hex)|8-bit|16-bit|32-bit|64-bit|
|---|---|---|---|---|---|
|000|0|AL|AX|EAX|RAX|
|001|1|CL|CX|ECX|RCX|
|010|2|DL|DX|EDX|RDX|
|011|3|BL|BX|EBX|RBX|
|100|4|AH|SP|ESP|RSP|
|101|5|CH|BP|EBP|RBP|
|110|6|DH|SI|ESI|RSI|
|111|7|BH|DI|EDI|RDI|

 5.2 Mod != 11 (Memory mode) အတွက် R/M

ဒီအခါ R/M က **memory addressing mode** ကို သတ်မှတ်တယ်။

**Mod = 00 (no displacement) -**

|R/M|Mod=00 အဓိပ္ပါယ်|
|---|---|
|000|[EAX]|
|001|[ECX]|
|010|[EDX]|
|011|[EBX]|
|100|[SIB] (SIB byte လိုတယ်)|
|101|[disp32] (32-bit displacement)|
|110|[ESI]|
|111|[EDI]|

**Mod = 01 (8-bit displacement) -**

|R/M|Mod=01 အဓိပ္ပါယ်|
|---|---|
|000|[EAX + disp8]|
|001|[ECX + disp8]|
|010|[EDX + disp8]|
|011|[EBX + disp8]|
|100|[SIB + disp8]|
|101|[EBP + disp8]|
|110|[ESI + disp8]|
|111|[EDI + disp8]|

**Mod = 10 (32-bit displacement) -**

|R/M|Mod=10 အဓိပ္ပါယ်|
|---|---|
|000|[EAX + disp32]|
|001|[ECX + disp32]|
|010|[EDX + disp32]|
|011|[EBX + disp32]|
|100|[SIB + disp32]|
|101|[EBP + disp32]|
|110|[ESI + disp32]|
|111|[EDI + disp32]|


### example  1 - `mov al, [ebx]`
```
Mod = 00 (no displacement)
R/M = 011 (EBX)
→ [EBX]
```

### ဥပမာ example 2 - `mov al, [ebx + 5]`
```
Mod = 01 (8-bit displacement)
R/M = 011 (EBX)
→ [EBX + disp8]
```
### example 3 - `mov al, [ebx + 1000]`
```
Mod = 10 (32-bit displacement)
R/M = 011 (EBX)
→ [EBX + disp32]
```

### example 4 - `mov al, bl`
```
Mod = 11 (register mode)
R/M = 011 (BL)
→ BL (register)
```

### example 5 - `mov al, [0x00401000]`
```
Mod = 00 (no displacement, but special case)
R/M = 101 (special)
→ [disp32] (absolute address)
```




```
┌─────────────────────────────────────────────────────────────────────┐
│                    R/M SUMMARY (3 bits)                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  R/M ၏ အဓိပ္ပါယ်သည် Mod ပေါ်မူတည်သည် -                             │
│                                                                     │
│  Mod = 11 → R/M က REGISTER                                         │
│  Mod ≠ 11 → R/M က MEMORY ADDRESSING MODE                           │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Mod=00 │ Mod=01    │ Mod=10     │ Mod=11                     │   │
│  ├────────┼───────────┼────────────┼────────────────────────────┤   │
│  │ [EAX]  │ [EAX+d8]  │ [EAX+d32]  │ AL/AX/EAX/RAX (000)        │   │
│  │ [ECX]  │ [ECX+d8]  │ [ECX+d32]  │ CL/CX/ECX/RCX (001)        │   │
│  │ [EDX]  │ [EDX+d8]  │ [EDX+d32]  │ DL/DX/EDX/RDX (010)        │   │
│  │ [EBX]  │ [EBX+d8]  │ [EBX+d32]  │ BL/BX/EBX/RBX (011)        │   │
│  │ [SIB]  │ [SIB+d8]  │ [SIB+d32]  │ AH/SP/ESP/RSP (100)        │   │
│  │ [d32]  │ [EBP+d8]  │ [EBP+d32]  │ CH/BP/EBP/RBP (101)        │   │
│  │ [ESI]  │ [ESI+d8]  │ [ESI+d32]  │ DH/SI/ESI/RSI (110)        │   │
│  │ [EDI]  │ [EDI+d8]  │ [EDI+d32]  │ BH/DI/EDI/RDI (111)        │   │
│  └────────┴───────────┴────────────┴────────────────────────────┘   │
│                                                                     │
│  သတိထားရန် -                                                         │
│  R/M=100 က Mod ဘယ်လောက်ပဲဖြစ်ဖြစ် SIB byte လိုတယ်                 │
│  R/M=101 က Mod=00 မှာ [disp32] (base မပါ)၊ Mod=01/10 မှာ [EBP]    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```
