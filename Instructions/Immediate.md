
**Immediate** = တိုက်ရိုက်တန်ဖိုး (instruction ထဲမှာ ပါဝင်တဲ့ ကိန်းသေ)

Immediate က instruction ရဲ့ အနောက်ဆုံးမှာ လိုက်တဲ့ အပိုင်းဖြစ်တယ်
ဒါက memory ကနေ ဖတ်စရာမလိုဘဲ တန်ဖိုးကို တိုက်ရိုက်သုံးနိုင်တယ်

```
Instruction Format:
┌──────────┬─────────┬─────────┬──────────┬──────────────┐
│  Opcode  │ ModRM   │   SIB   │ Displacement │ Immediate   │
└──────────┴─────────┴─────────┴──────────┴──────────────┘
                                                    ↑
                                              Immediate Value
```

example 
```
mov al, 0x41     ; 0x41 က immediate value
add eax, 10      ; 10 က immediate value
mov [ebx], 100   ; 100 က immediate value
```


| Type      | short term | size    | value width             | example                       |
| --------- | ---------- | ------- | ----------------------- | ----------------------------- |
| **imm8**  | imm8       | 1 byte  | 0 to 255 (-128 to +127) | `add al, 5`                   |
| **imm16** | imm16      | 2 bytes | 0 to 65535              | `mov ax, 0x1234`              |
| **imm32** | imm32      | 4 bytes | 0 to 4.29B              | `mov eax, 0x12345678`         |
| **imm64** | imm64      | 8 bytes | 0 to 2⁶⁴-1              | `mov rax, 0x1122334455667788` |


#### Instruction with immediate
##### MOV reg, `imm`

|Instruction|Opcode|Immediate Size|ဥပမာ Machine Code|
|---|---|---|---|
|`mov al, imm8`|B0|1 byte|`B0 41`|
|`mov ax, imm16`|B8|2 bytes|`B8 34 12`|
|`mov eax, imm32`|B8|4 bytes|`B8 78 56 34 12`|
|`mov rax, imm64`|48 B8|8 bytes|`48 B8 88 77 66 55 44 33 22 11`|

##### 3.2 ADD/SUB reg, `imm`

|Instruction|Opcode|Immediate Size|ဥပမာ|
|---|---|---|---|
|`add al, imm8`|04|1 byte|`04 01`|
|`add eax, imm32`|05|4 bytes|`05 01 00 00 00`|
|`sub al, imm8`|2C|1 byte|`2C 01`|
|`sub eax, imm32`|2D|4 bytes|`2D 01 00 00 00`|

##### CMP reg, `imm`

|Instruction|Opcode|Immediate Size|ဥပမာ|
|---|---|---|---|
|`cmp al, imm8`|3C|1 byte|`3C 0A`|
|`cmp eax, imm32`|3D|4 bytes|`3D 0A 00 00 00`|

##### MOV [memory], `imm`

|Instruction|Opcode|Immediate Size|ဥပမာ|
|---|---|---|---|
|`mov byte [ebx], imm8`|C6|1 byte|`C6 03 41`|
|`mov word [ebx], imm16`|C7|2 bytes|`C7 03 34 12`|
|`mov dword [ebx], imm32`|C7|4 bytes|`C7 03 78 56 34 12`|

##### PUSH `imm`

|Instruction|Opcode|Immediate Size|ဥပမာ|
|---|---|---|---|
|`push imm8`|6A|1 byte (sign-extended)|`6A 05`|
|`push imm32`|68|4 bytes|`68 78 56 34 12`|


```
┌─────────────────────────────────────────────────────────────────────┐
│                    IMMEDIATE SUMMARY                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. Immediate က instruction ရဲ့ နောက်ဆုံးမှာပါတယ်                 │
│                                                                     │
│  2. Immediate အရွယ်အစား - imm8 (1 byte), imm16 (2 bytes),         │
│     imm32 (4 bytes), imm64 (8 bytes)                                │
│                                                                     │
│  3. x86 က little-endian → အနိမ့်ဆုံး byte ကို အရင်သိမ်း             │
│     0x1234 → 34 12                                                  │
│     0x12345678 → 78 56 34 12                                        │
│                                                                     │
│  4. imm8 ကို sign-extend လုပ်ပြီး 32-bit အဖြစ်သုံးနိုင်            │
│     push 5 → 6A 05 (0x00000005)                                    │
│     push -5 → 6A FB (0xFFFFFFFB)                                   │
│                                                                     │
│  5. သိပြီးသား Opcodes:                                          │
│     B0-B7 = MOV reg8, imm8                                          │
│     B8-BF = MOV r16/r32/r64, imm16/imm32/imm64                     │
│     04 = ADD AL, imm8                                               │
│     05 = ADD EAX, imm32                                             │
│     2C = SUB AL, imm8                                               │
│     2D = SUB EAX, imm32                                             │
│     3C = CMP AL, imm8                                               │
│     3D = CMP EAX, imm32                                             │
│     C6 = MOV r/m8, imm8                                             │
│     C7 = MOV r/m16/32, imm16/32                                     │
│     6A = PUSH imm8 (sign-extended)                                  │
│     68 = PUSH imm32                                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```