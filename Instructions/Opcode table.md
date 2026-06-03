
##### 1.1 MOV reg8, imm8 (Register ← Immediate)

|Opcode|Register|ဥပမာ|
|---|---|---|
|**B0**|AL|`mov al, 0x41`|
|**B1**|CL|`mov cl, 0x41`|
|**B2**|DL|`mov dl, 0x41`|
|**B3**|BL|`mov bl, 0x41`|
|**B4**|AH|`mov ah, 0x41`|
|**B5**|CH|`mov ch, 0x41`|
|**B6**|DH|`mov dh, 0x41`|
|**B7**|BH|`mov bh, 0x41`|

##### 1.2 MOV r16/r32/r64, imm (Register ← Immediate)

|Opcode|16-bit|32-bit|64-bit (with REX)|
|---|---|---|---|
|**B8**|AX|EAX|RAX|
|**B9**|CX|ECX|RCX|
|**BA**|DX|EDX|RDX|
|**BB**|BX|EBX|RBX|
|**BC**|SP|ESP|RSP|
|**BD**|BP|EBP|RBP|
|**BE**|SI|ESI|RSI|
|**BF**|DI|EDI|RDI|

##### 1.3 MOV r/m8, r8 (Memory/Register ← Register) - 

|Opcode|Direction|ရှင်းလင်းချက်|
|---|---|---|
|**88**|r/m8 ← r8|Register ကနေ Memory/Register ကိုကူး|
|**8A**|r8 ← r/m8|Memory/Register ကနေ Register ကိုကူး|

##### 1.4 MOV r/m16/32/64, r16/32/64 (Memory/Register ← Register)

|Opcode|Direction|ရှင်းလင်းချက်|
|---|---|---|
|**89**|r/m16/32/64 ← r16/32/64|Register ကနေ Memory/Register ကိုကူး|
|**8B**|r16/32/64 ← r/m16/32/64|Memory/Register ကနေ Register ကိုကူး|

##### 1.5 MOV r/m8, imm8 (Memory ← Immediate)

|Opcode|ရှင်းလင်းချက်|
|---|---|
|**C6**|Memory/Register ← imm8|

##### 1.6 MOV r/m16/32/64, imm16/32 (Memory ← Immediate)

|Opcode|ရှင်းလင်းချက်|
|---|---|
|**C7**|Memory/Register ← imm16/32|

##### 1.7 MOV AL/AX/EAX/RAX, moffs (Memory ← Accumulator)

|Opcode|ရှင်းလင်းချက်|
|---|---|
|**A0**|AL ← [moffs8]|
|**A1**|AX/EAX/RAX ← [moffs16/32/64]|

##### 1.8 MOV `moffs`, AL/AX/EAX/RAX (Accumulator ← Memory)

|Opcode|ရှင်းလင်းချက်|
|---|---|
|**A2**|[moffs8] ← AL|
|**A3**|[moffs16/32/64] ← AX/EAX/RAX|


---


##### 2.1 PUSH reg16/32/64 (Register Push)

| Opcode | 16-bit Register | 32-bit Register | 64-bit Register |
| ------ | --------------- | --------------- | --------------- |
| **50** | AX              | EAX             | RAX             |
| **51** | CX              | ECX             | RCX             |
| **52** | DX              | EDX             | RDX             |
| **53** | BX              | EBX             | RBX             |
| **54** | SP              | ESP             | RSP             |
| **55** | BP              | EBP             | RBP             |
| **56** | SI              | ESI             | RSI             |
| **57** | DI              | EDI             | RDI             |

##### 2.2 PUSH imm8 / imm32 (Immediate Push)

| Opcode | Immediate Size                 | ဥပမာ              |
| ------ | ------------------------------ | ----------------- |
| **6A** | imm8 (sign-extended to 32-bit) | `push 5`          |
| **68** | imm32                          | `push 0x12345678` |


##### 2.3 PUSH segreg (Segment Register Push)

| Opcode    | Segment Register           |
| --------- | -------------------------- |
| **06**    | PUSH ES (16-bit, obsolete) |
| **0E**    | PUSH CS (16-bit, obsolete) |
| **16**    | PUSH SS (16-bit, obsolete) |
| **1E**    | PUSH DS (16-bit, obsolete) |
| **0F A0** | PUSH FS                    |
| **0F A8** | PUSH GS                    |

##### 3.1 POP reg16/32/64 (Register Pop)

| Opcode | 16-bit Register | 32-bit Register | 64-bit Register |
| ------ | --------------- | --------------- | --------------- |
| **58** | AX              | EAX             | RAX             |
| **59** | CX              | ECX             | RCX             |
| **5A** | DX              | EDX             | RDX             |
| **5B** | BX              | EBX             | RBX             |
| **5C** | SP              | ESP             | RSP             |
| **5D** | BP              | EBP             | RBP             |
| **5E** | SI              | ESI             | RSI             |
| **5F** | DI              | EDI             | RDI             |
|        |                 |                 |                 |


##### 3.2 POP `segreg` (Segment Register Pop)

|Opcode|Segment Register|
|---|---|
|**07**|POP ES (16-bit, obsolete)|
|**17**|POP SS (16-bit, obsolete)|
|**1F**|POP DS (16-bit, obsolete)|
|**0F A1**|POP FS|
|**0F A9**|POP GS|

---
