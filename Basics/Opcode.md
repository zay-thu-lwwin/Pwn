
Opcode = Operation Code
ဒါကတော့ CPU ကို ဘာလုပ်ရမလဲဆိုတာ ညွှန်ကြားတဲ့ machine language ရဲ့ တစ်စိတ်တစ်ပိုင်း
လူတွေရေးတဲ့ Assembly ကို CPU နားလည်တဲ့ hexadecimal (သို့) binary ပုံစံပြောင်းလိုက်တဲ့အခါ ရလာတဲ့ byte(s) တွေကို opcode လို့ခေါ်


https://pnx.tf/files/x86_opcode_structure_and_instruction_overview.pdf

### အဆင့် 1: Opcode table ထားပါ (reference)

### အဆင့် 2: Instruction ကိုခွဲကြည့်ပါ

### အဆင့် 3: တစ်ပိုင်းချင်းတွက်ပါ

### အဆင့် 4: စုစည်းပါ



x86 Opcode Reference Table (Shellcode ရေးဖို့ လိုအပ်သလောက်)
ဒါက အပြည့်အစုံမဟုတ်ဘူး။ ဒါပေမယ့် shellcode ရေးတဲ့အခါ လိုအပ်တဲ့ opcode တွေအားလုံး ပါတယ်

##### 1. PUSH and POP (1 byte)

| Instruction | Opcode |
|-------------|--------|
| push eax | 50 |
| push ecx | 51 |
| push edx | 52 |
| push ebx | 53 |
| push esp | 54 |
| push ebp | 55 |
| push esi | 56 |
| push edi | 57 |
| pop eax | 58 |
| pop ecx | 59 |
| pop edx | 5A |
| pop ebx | 5B |
| pop esp | 5C |
| pop ebp | 5D |
| pop esi | 5E |
| pop edi | 5F |
| pusha (all) | 60 |
| popa (all) | 61 |
| push imm8 | 6A |
| push imm32 | 68 |

---

##### 2. XOR (2 bytes)

| Instruction | Opcode |
|-------------|--------|
| xor eax, eax | 31 C0 |
| xor ecx, ecx | 31 C9 |
| xor edx, edx | 31 D2 |
| xor ebx, ebx | 31 DB |
| xor esp, esp | 31 E4 |
| xor ebp, ebp | 31 ED |
| xor esi, esi | 31 F6 |
| xor edi, edi | 31 FF |
| xor al, al | 30 C0 |
| xor cl, cl | 30 C9 |
| xor dl, dl | 30 D2 |
| xor bl, bl | 30 DB |

---

##### 3. MOV Immediate to Register

##### 8-bit (2 bytes)

| Instruction | Opcode |
|-------------|--------|
| mov al, imm8 | B0 imm8 |
| mov cl, imm8 | B1 imm8 |
| mov dl, imm8 | B2 imm8 |
| mov bl, imm8 | B3 imm8 |
| mov ah, imm8 | B4 imm8 |
| mov ch, imm8 | B5 imm8 |
| mov dh, imm8 | B6 imm8 |
| mov bh, imm8 | B7 imm8 |

##### 32-bit (5 bytes - null bytes ပါတတ်တယ်)

| Instruction | Opcode |
|-------------|--------|
| mov eax, imm32 | B8 imm32 |
| mov ecx, imm32 | B9 imm32 |
| mov edx, imm32 | BA imm32 |
| mov ebx, imm32 | BB imm32 |
| mov esp, imm32 | BC imm32 |
| mov ebp, imm32 | BD imm32 |
| mov esi, imm32 | BE imm32 |
| mov edi, imm32 | BF imm32 |

---

##### 4. MOV Register to Register (2 bytes)

| Instruction | Opcode |
|-------------|--------|
| mov eax, ebx | 89 D8 |
| mov eax, ecx | 89 C8 |
| mov eax, edx | 89 D0 |
| mov eax, esp | 89 E0 |
| mov ebx, eax | 89 C3 |
| mov ebx, ecx | 89 CB |
| mov ebx, edx | 89 D3 |
| mov ebx, esp | 89 E3 |
| mov ecx, eax | 89 C1 |
| mov ecx, ebx | 89 D9 |
| mov ecx, esp | 89 E1 |
| mov edx, eax | 89 C2 |
| mov edx, ebx | 89 DA |
| mov edx, esp | 89 E2 |
| mov esp, ebx | 89 DC |
| mov esp, eax | 89 C4 |

---

##### 5. INC and DEC (1 byte)

| Instruction | Opcode | Instruction | Opcode |
|-------------|--------|-------------|--------|
| inc eax | 40 | dec eax | 48 |
| inc ecx | 41 | dec ecx | 49 |
| inc edx | 42 | dec edx | 4A |
| inc ebx | 43 | dec ebx | 4B |
| inc esp | 44 | dec esp | 4C |
| inc ebp | 45 | dec ebp | 4D |
| inc esi | 46 | dec esi | 4E |
| inc edi | 47 | dec edi | 4F |

---

##### 6. ADD and SUB (2 bytes)

| Instruction | Opcode |
|-------------|--------|
| add eax, ebx | 01 D8 |
| add eax, ecx | 01 C8 |
| add ebx, eax | 01 C3 |
| add al, imm8 | 04 imm8 |
| add eax, imm32 | 05 imm32 |
| sub eax, ebx | 29 D8 |
| sub eax, ecx | 29 C8 |
| sub ebx, eax | 29 C3 |
| sub al, imm8 | 2C imm8 |
| sub eax, imm32 | 2D imm32 |

---

##### 7. CMP and TEST (2 bytes)

| Instruction | Opcode |
|-------------|--------|
| cmp al, imm8 | 3C imm8 |
| cmp eax, imm32 | 3D imm32 |
| cmp eax, ebx | 39 D8 |
| cmp eax, ecx | 39 C8 |
| test eax, eax | 85 C0 |
| test ebx, ebx | 85 DB |
| test ecx, ecx | 85 C9 |
| test edx, edx | 85 D2 |

---

##### 8. System Calls and Interrupts

| Instruction | Opcode |
|-------------|--------|
| int 0x80 | CD 80 |
| syscall (x64) | 0F 05 |
| int3 (breakpoint) | CC |
| ret | C3 |
| retf (far return) | CB |
| iret (interrupt return) | CF |

---

##### 9. Jumps (Conditional and Unconditional)

##### Unconditional Jumps

| Instruction | Opcode |
|-------------|--------|
| jmp short rel8 | EB rel8 |
| jmp near rel32 | E9 rel32 |
| jmp far | EA |
| jmp eax | FF E0 |
| jmp ebx | FF E3 |
| call rel32 | E8 rel32 |
| call eax | FF D0 |
| call ebx | FF D3 |

##### Conditional Jumps (short)

| Instruction | Opcode | Condition |
|-------------|--------|-----------|
| je / jz | 74 rel8 | equal / zero |
| jne / jnz | 75 rel8 | not equal / not zero |
| jg / jnle | 7F rel8 | greater (signed) |
| jl / jnge | 7C rel8 | less (signed) |
| jge / jnl | 7D rel8 | greater or equal |
| jle / jng | 7E rel8 | less or equal |
| ja / jnbe | 77 rel8 | above (unsigned) |
| jb / jnae | 72 rel8 | below (unsigned) |
| jae / jnb | 73 rel8 | above or equal |
| jbe / jna | 76 rel8 | below or equal |
| js | 78 rel8 | sign (negative) |
| jns | 79 rel8 | not sign |
| jo | 70 rel8 | overflow |
| jno | 71 rel8 | no overflow |
| jp / jpe | 7A rel8 | parity odd |
| jnp / jpo | 7B rel8 | parity even |
| jcxz | E3 rel8 | cx register zero |

---

##### 10. NOP and Others

| Instruction | Opcode | မှတ်ချက် |
|-------------|--------|----------|
| nop | 90 | no operation |
| hlt | F4 | halt CPU |
| cli | FA | clear interrupt |
| sti | FB | set interrupt |
| cld | FC | clear direction flag |
| std | FD | set direction flag |
| cdq | 99 | sign extend eax to edx:eax |
| cbw | 98 | sign extend al to ax |
| cwde | 98 | sign extend ax to eax |
| xchg eax, ebx | 93 | exchange |
| xchg eax, ecx | 91 | |
| xchg eax, edx | 92 | |

---

##### 11. String Instructions

| Instruction | Opcode | လုပ်ဆောင်ချက် |
|-------------|--------|----------------|
| movsb | A4 | [edi] = [esi], esi++, edi++ |
| movsw | A5 | word |
| movsd | A5 | dword |
| stosb | AA | [edi] = al, edi++ |
| stosd | AB | [edi] = eax, edi+=4 |
| lodsb | AC | al = [esi], esi++ |
| lodsd | AD | eax = [esi], esi+=4 |
| scasb | AE | cmp al, [edi], edi++ |
| scasd | AF | cmp eax, [edi], edi+=4 |
| rep movsb | F3 A4 | repeat movsb (ecx times) |
| rep stosb | F3 AA | repeat stosb |

---

##### 12. Cheat Sheet (most common)

```
50 51 52 53     push eax,ecx,edx,ebx
58 59 5A 5B     pop eax,ecx,edx,ebx
31 C0 31 C9     xor eax,eax / xor ecx,ecx
31 D2 31 DB     xor edx,edx / xor ebx,ebx
B0 B1 B2 B3     mov al,cl,dl,bl
68 6A           push imm32 / push imm8
89 E3 89 E1     mov ebx,esp / mov ecx,esp
CD 80           int 0x80
90              nop
C3              ret
EB              jmp short
74 75           je / jne
E8 E9           call / jmp near
```




---

#### ModR/M

```
ModR/M Byte = 8 bits (1 byte)
Bits: [7-6] [5-3] [2-0]
      MOD   REG   R/M
```




##### 1. MOD (2 bits) - Memory လား Register လား

|MOD|ဘာကိုဆိုလိုသလဲ|
|---|---|
|11|register to register (အများဆုံးသုံး)|
|00|memory, no offset|
|01|memory + 1-byte offset|
|10|memory + 4-byte offset|

##### 2. REG (3 bits) - Register no

|ဂဏန်း|Register|
|---|---|
|000|eax|
|001|ecx|
|010|edx|
|011|ebx|
|100|esp|
|101|ebp|
|110|esi|
|111|edi|

##### 3. R/M (3 bits) - another Register no

REG နဲ့ အတူတူပါပဲ


---
