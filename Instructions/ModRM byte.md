
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


##### - Mod (2 bits) - defining mode

|Mod (binary)|Mod (hex)|အဓိပ္ပါယ်|ဥပမာ|
|---|---|---|---|
|**00**|0|[R/M] (memory, no displacement)|`mov al, [ebx]`|
|**01**|1|[R/M + disp8] (memory + 1-byte offset)|`mov al, [ebx+5]`|
|**10**|2|[R/M + disp32] (memory + 4-byte offset)|`mov al, [ebx+1000]`|
|**11**|3|R/M က register (no memory)|`mov al, bl`|

---

##### - Reg (3 bits) - defining Register (first operand)

4.1 8-bit Registers (for 8-bit instructions)

|Reg (binary)|Reg (hex)|8-bit Register|
|---|---|---|
|000|0|AL|
|001|1|CL|
|010|2|DL|
|011|3|BL|
|100|4|AH|
|101|5|CH|
|110|6|DH|
|111|7|BH|



 4.2 16/32/64-bit Registers (for 16/32/64-bit instructions)

|Reg (binary)|Reg (hex)|16-bit|32-bit|64-bit (with REX)|
|---|---|---|---|---|
|000|0|AX|EAX|RAX|
|001|1|CX|ECX|RCX|
|010|2|DX|EDX|RDX|
|011|3|BX|EBX|RBX|
|100|4|SP|ESP|RSP|
|101|5|BP|EBP|RBP|
|110|6|SI|ESI|RSI|
|111|7|DI|EDI|RDI|

---

##### - R/M (3 bits) - second operand (Register/Memory)

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