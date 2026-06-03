
Opcode အုပ်စုများ (High Nibble အလိုက်)

| High Nibble | Hex Range | Instruction Group                       |
| ----------- | --------- | --------------------------------------- |
| 0000 (0)    | 00-0F     | ADD, OR, ADC, SBB, PUSH, POP            |
| 0001 (1)    | 10-1F     | ADC, SBB, PUSH, POP                     |
| 0010 (2)    | 20-2F     | SUB, XOR, CMP, PUSH, POP                |
| 0011 (3)    | 30-3F     | XOR, CMP, REP, MOVS                     |
| 0100 (4)    | 40-4F     | INC, DEC (32-bit) / REX prefix (64-bit) |
| 0101 (5)    | 50-5F     | PUSH, POP                               |
| 0110 (6)    | 60-6F     | PUSHA, POPA, PUSH imm, INS, OUTS        |
| 0111 (7)    | 70-7F     | Conditional Jumps (short)               |
| 1000 (8)    | 80-8F     | Group instructions (ModRM needed)       |
| 1001 (9)    | 90-9F     | NOP, XCHG, CBW, CWD, PUSHF, POPF        |
| 1010 (A)    | A0-AF     | MOV moffs, STOS, LODS, CMPS, SCAS       |
| 1011 (B)    | B0-BF     | MOV reg, imm                            |
| 1100 (C)    | C0-CF     | RET, JMP, CALL, INT, SHL, SHR           |
| 1101 (D)    | D0-DF     | SHL, SHR, SAL, SAR, FPU                 |
| 1110 (E)    | E0-EF     | LOOP, JMP, CALL, IN, OUT                |
| 1111 (F)    | F0-FF     | LOCK, REP, HLT, CLC, STC, CLI, STI      |



##### (Quick Reference)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    x86 OPCODE QUICK REFERENCE                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  00-05  ADD         40-4F  INC/DEC/REX    80-8F  Group Inst       │
│  08-0D  OR          50-57  PUSH reg       A0-A3  MOV moffs        │
│  10-15  ADC         58-5F  POP reg        A4-A5  MOVS             │
│  18-1D  SBB         68     PUSH imm32     A6-A7  CMPS             │
│  20-25  SUB         6A     PUSH imm8      AA-AB  STOS             │
│  28-2D  SUB (alt)   70-7F  Jcc (short)    AC-AD  LODS             │
│  30-35  XOR         88-8B  MOV            AE-AF  SCAS             │
│  38-3D  CMP         8D     LEA            B0-B7  MOV r8, imm8     │
│  50-57  PUSH        90     NOP            B8-BF  MOV r32, imm32   │
│  58-5F  POP         9C     PUSHF          C3     RET              │
│  68     PUSH imm32  9D     POPF           CC     INT 3            │
│  6A     PUSH imm8   A0-A3  MOV moffs      E8     CALL             │
│  70-7F  Jcc         A4-A5  MOVS           E9     JMP              │
│  74     JE/JZ       AA-AB  STOS           EB     JMP short        │
│  75     JNE/JNZ     AC-AD  LODS           F4     HLT              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---



---

##### - Opcode 00-0F (ADD, OR, ADC, SBB)

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| 00 | ADD r/m8, r8 | 8-bit ပေါင်း (mem ← reg) |
| 01 | ADD r/m16/32, r16/32 | 16/32-bit ပေါင်း |
| 02 | ADD r8, r/m8 | 8-bit ပေါင်း (reg ← mem) |
| 03 | ADD r16/32, r/m16/32 | 16/32-bit ပေါင်း |
| 04 | ADD AL, imm8 | AL ကို imm8 ပေါင်း |
| 05 | ADD EAX, imm32 | EAX ကို imm32 ပေါင်း |
| 06 | PUSH ES | (16-bit, obsolete) |
| 07 | POP ES | (16-bit, obsolete) |
| 08 | OR r/m8, r8 | OR operation (mem ← reg) |
| 09 | OR r/m16/32, r16/32 | OR operation |
| 0A | OR r8, r/m8 | OR operation (reg ← mem) |
| 0B | OR r16/32, r/m16/32 | OR operation |
| 0C | OR AL, imm8 | AL ကို imm8 OR |
| 0D | OR EAX, imm32 | EAX ကို imm32 OR |
| 0E | PUSH CS | (16-bit, obsolete) |
| 0F | (Opcode map extension) | Two-byte opcodes start here |

---

##### - Opcode 10-1F (ADC, SBB)

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| 10 | ADC r/m8, r8 | Add with Carry (mem ← reg) |
| 11 | ADC r/m16/32, r16/32 | Add with Carry |
| 12 | ADC r8, r/m8 | Add with Carry (reg ← mem) |
| 13 | ADC r16/32, r/m16/32 | Add with Carry |
| 14 | ADC AL, imm8 | AL ကို imm8 + CF |
| 15 | ADC EAX, imm32 | EAX ကို imm32 + CF |
| 16 | PUSH SS | (16-bit, obsolete) |
| 17 | POP SS | (16-bit, obsolete) |
| 18 | SBB r/m8, r8 | Subtract with Borrow |
| 19 | SBB r/m16/32, r16/32 | Subtract with Borrow |
| 1A | SBB r8, r/m8 | Subtract with Borrow |
| 1B | SBB r16/32, r/m16/32 | Subtract with Borrow |
| 1C | SBB AL, imm8 | AL ကို imm8 နှုတ် (with borrow) |
| 1D | SBB EAX, imm32 | EAX ကို imm32 နှုတ် (with borrow) |
| 1E | PUSH DS | (16-bit, obsolete) |
| 1F | POP DS | (16-bit, obsolete) |

---

##### - Opcode 20-2F (SUB, XOR, CMP)

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| 20 | SUB r/m8, r8 | နှုတ် (mem ← reg) |
| 21 | SUB r/m16/32, r16/32 | နှုတ် |
| 22 | SUB r8, r/m8 | နှုတ် (reg ← mem) |
| 23 | SUB r16/32, r/m16/32 | နှုတ် |
| 24 | SUB AL, imm8 | AL ကို imm8 နှုတ် |
| 25 | SUB EAX, imm32 | EAX ကို imm32 နှုတ် |
| 26 | PUSH ES | (16-bit, obsolete) |
| 27 | POP ES | (16-bit, obsolete) |
| 28 | SUB r/m8, r8 | နှုတ် |
| 29 | SUB r/m16/32, r16/32 | နှုတ် |
| 2A | SUB r8, r/m8 | နှုတ် |
| 2B | SUB r16/32, r/m16/32 | နှုတ် |
| 2C | SUB AL, imm8 | AL ကို imm8 နှုတ် |
| 2D | SUB EAX, imm32 | EAX ကို imm32 နှုတ် |
| 2E | PUSH CS | (16-bit, obsolete) |
| 2F | POP CS | (16-bit, obsolete) |

---

##### - Opcode 30-3F (XOR, CMP, REP)

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| 30 | XOR r/m8, r8 | XOR operation |
| 31 | XOR r/m16/32, r16/32 | XOR operation |
| 32 | XOR r8, r/m8 | XOR operation |
| 33 | XOR r16/32, r/m16/32 | XOR operation |
| 34 | XOR AL, imm8 | AL ကို imm8 XOR |
| 35 | XOR EAX, imm32 | EAX ကို imm32 XOR |
| 36 | PUSH SS | (16-bit, obsolete) |
| 37 | POP SS | (16-bit, obsolete) |
| 38 | CMP r/m8, r8 | နှိုင်းယှဉ် |
| 39 | CMP r/m16/32, r16/32 | နှိုင်းယှဉ် |
| 3A | CMP r8, r/m8 | နှိုင်းယှဉ် |
| 3B | CMP r16/32, r/m16/32 | နှိုင်းယှဉ် |
| 3C | CMP AL, imm8 | AL နဲ့ imm8 နှိုင်းယှဉ် |
| 3D | CMP EAX, imm32 | EAX နဲ့ imm32 နှိုင်းယှဉ် |
| 3E | PUSH DS | (16-bit, obsolete) |
| 3F | POP DS | (16-bit, obsolete) |

---

##### - Opcode 40-4F (INC / DEC / REX Prefix)

| Opcode | 32-bit Mode (IA-32) | 64-bit Mode (x86-64) |
|--------|---------------------|----------------------|
| 40 | INC EAX | REX prefix (REX.W=0, REX.R=0, REX.X=0, REX.B=0) |
| 41 | INC ECX | REX prefix (REX.B=1) |
| 42 | INC EDX | REX prefix (REX.X=1) |
| 43 | INC EBX | REX prefix (REX.X=1, REX.B=1) |
| 44 | INC ESP | REX prefix (REX.R=1) |
| 45 | INC EBP | REX prefix (REX.R=1, REX.B=1) |
| 46 | INC ESI | REX prefix (REX.R=1, REX.X=1) |
| 47 | INC EDI | REX prefix (REX.R=1, REX.X=1, REX.B=1) |
| 48 | DEC EAX | REX prefix (REX.W=1) |
| 49 | DEC ECX | REX prefix (REX.W=1, REX.B=1) |
| 4A | DEC EDX | REX prefix (REX.W=1, REX.X=1) |
| 4B | DEC EBX | REX prefix (REX.W=1, REX.X=1, REX.B=1) |
| 4C | DEC ESP | REX prefix (REX.W=1, REX.R=1) |
| 4D | DEC EBP | REX prefix (REX.W=1, REX.R=1, REX.B=1) |
| 4E | DEC ESI | REX prefix (REX.W=1, REX.R=1, REX.X=1) |
| 4F | DEC EDI | REX prefix (REX.W=1, REX.R=1, REX.X=1, REX.B=1) |

---

##### - Opcode 50-5F (PUSH / POP)

| Opcode | 16-bit | 32-bit | 64-bit | အလုပ် |
|--------|--------|--------|--------|--------|
| 50 | PUSH AX | PUSH EAX | PUSH RAX | Stack ပေါ် push |
| 51 | PUSH CX | PUSH ECX | PUSH RCX | Stack ပေါ် push |
| 52 | PUSH DX | PUSH EDX | PUSH RDX | Stack ပေါ် push |
| 53 | PUSH BX | PUSH EBX | PUSH RBX | Stack ပေါ် push |
| 54 | PUSH SP | PUSH ESP | PUSH RSP | Stack ပေါ် push |
| 55 | PUSH BP | PUSH EBP | PUSH RBP | Stack ပေါ် push |
| 56 | PUSH SI | PUSH ESI | PUSH RSI | Stack ပေါ် push |
| 57 | PUSH DI | PUSH EDI | PUSH RDI | Stack ပေါ် push |
| 58 | POP AX | POP EAX | POP RAX | Stack ကနေ pop |
| 59 | POP CX | POP ECX | POP RCX | Stack ကနေ pop |
| 5A | POP DX | POP EDX | POP RDX | Stack ကနေ pop |
| 5B | POP BX | POP EBX | POP RBX | Stack ကနေ pop |
| 5C | POP SP | POP ESP | POP RSP | Stack ကနေ pop |
| 5D | POP BP | POP EBP | POP RBP | Stack ကနေ pop |
| 5E | POP SI | POP ESI | POP RSI | Stack ကနေ pop |
| 5F | POP DI | POP EDI | POP RDI | Stack ကနေ pop |

---

##### - Opcode 60-6F (PUSHA, POPA, PUSH imm, INS, OUTS)

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| 60 | PUSHA / PUSHAD | Push all registers (obsolete) |
| 61 | POPA / POPAD | Pop all registers (obsolete) |
| 62 | BOUND | Check array bounds (obsolete) |
| 63 | ARPL | Adjust RPL (obsolete) |
| 64 | FS segment prefix | FS segment override |
| 65 | GS segment prefix | GS segment override |
| 66 | Operand size prefix | 32-bit ↔ 16-bit switch |
| 67 | Address size prefix | Address size switch |
| 68 | PUSH imm16/32 | Push immediate (16/32-bit) |
| 69 | IMUL r16/32, r/m16/32, imm16/32 | Signed multiply |
| 6A | PUSH imm8 | Push sign-extended 8-bit |
| 6B | IMUL r16/32, r/m16/32, imm8 | Signed multiply (8-bit) |
| 6C | INS byte | Input from port to memory (string) |
| 6D | INS word/dword | Input from port |
| 6E | OUTS byte | Output from memory to port |
| 6F | OUTS word/dword | Output from memory to port |

---

##### - Opcode 70-7F (Conditional Jumps - Short)

| Opcode | Instruction | Condition |
|--------|-------------|-----------|
| 70 | JO | OF = 1 (Overflow) |
| 71 | JNO | OF = 0 (No Overflow) |
| 72 | JB / JNAE | CF = 1 (Below) |
| 73 | JAE / JNB | CF = 0 (Above or Equal) |
| 74 | JE / JZ | ZF = 1 (Equal / Zero) |
| 75 | JNE / JNZ | ZF = 0 (Not Equal / Not Zero) |
| 76 | JBE / JNA | CF = 1 or ZF = 1 |
| 77 | JA / JNBE | CF = 0 and ZF = 0 |
| 78 | JS | SF = 1 (Sign) |
| 79 | JNS | SF = 0 (Not Sign) |
| 7A | JP / JPE | PF = 1 (Parity Even) |
| 7B | JNP / JPO | PF = 0 (Parity Odd) |
| 7C | JL / JNGE | SF != OF (Less) |
| 7D | JGE / JNL | SF = OF (Greater or Equal) |
| 7E | JLE / JNG | ZF = 1 or SF != OF |
| 7F | JG / JNLE | ZF = 0 and SF = OF |

---

##### - Opcode 80-8F (Group Instructions - ModRM Needed)

| Opcode | Instruction Group | ရှင်းလင်းချက် |
|--------|------------------|---------------|
| 80 | ADD/SUB/CMP... r/m8, imm8 | 8-bit imm8 |
| 81 | ADD/SUB/CMP... r/m16/32, imm16/32 | 16/32-bit imm |
| 82 | (obsolete) | - |
| 83 | ADD/SUB/CMP... r/m16/32, imm8 | sign-extended imm8 |
| 84 | TEST r/m8, r8 | AND without storing result |
| 85 | TEST r/m16/32, r16/32 | AND without storing result |
| 86 | XCHG r8, r/m8 | Exchange |
| 87 | XCHG r16/32, r/m16/32 | Exchange |
| 88 | MOV r/m8, r8 | Register → Memory/Register |
| 89 | MOV r/m16/32, r16/32 | Register → Memory/Register |
| 8A | MOV r8, r/m8 | Memory/Register → Register |
| 8B | MOV r16/32, r/m16/32 | Memory/Register → Register |
| 8C | MOV r/m16, segreg | Segment register → Memory |
| 8D | LEA r16/32, m | Load Effective Address |
| 8E | MOV segreg, r/m16 | Memory → Segment register |
| 8F | POP r/m16/32 | Pop from stack to memory |

---

##### - Opcode 90-9F (NOP, XCHG, CBW, CWD, PUSHF, POPF)

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| 90 | NOP / XCHG EAX, EAX | No Operation |
| 91 | XCHG EAX, ECX | Swap EAX and ECX |
| 92 | XCHG EAX, EDX | Swap EAX and EDX |
| 93 | XCHG EAX, EBX | Swap EAX and EBX |
| 94 | XCHG EAX, ESP | Swap EAX and ESP |
| 95 | XCHG EAX, EBP | Swap EAX and EBP |
| 96 | XCHG EAX, ESI | Swap EAX and ESI |
| 97 | XCHG EAX, EDI | Swap EAX and EDI |
| 98 | CBW / CWDE | Convert Byte to Word |
| 99 | CWD / CDQ | Convert Word to Doubleword |
| 9A | CALL far pointer | (obsolete) |
| 9B | WAIT / FWAIT | Wait for FPU |
| 9C | PUSHF / PUSHFD | Push Flags |
| 9D | POPF / POPFD | Pop Flags |
| 9E | SAHF | Store AH into Flags |
| 9F | LAHF | Load AH from Flags |

---

##### - Opcode A0-AF (MOV moffs, STOS, LODS, CMPS, SCAS)

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| A0 | MOV AL, moffs8 | Memory → AL (absolute address) |
| A1 | MOV EAX, moffs32 | Memory → EAX |
| A2 | MOV moffs8, AL | AL → Memory |
| A3 | MOV moffs32, EAX | EAX → Memory |
| A4 | MOVSB | String move (byte) |
| A5 | MOVSW / MOVSD | String move (word/dword) |
| A6 | CMPSB | String compare (byte) |
| A7 | CMPSW / CMPSD | String compare |
| A8 | TEST AL, imm8 | AL AND imm8 (flags only) |
| A9 | TEST EAX, imm32 | EAX AND imm32 |
| AA | STOSB | Store string byte |
| AB | STOSW / STOSD | Store string word/dword |
| AC | LODSB | Load string byte |
| AD | LODSW / LODSD | Load string word/dword |
| AE | SCASB | Scan string byte |
| AF | SCASW / SCASD | Scan string word/dword |

---

##### - Opcode B0-BF (MOV reg, imm) - 
 B0-B7 (8-bit MOV)

| Opcode | Instruction  | Opcode | Instruction  |
| ------ | ------------ | ------ | ------------ |
| B0     | MOV AL, imm8 | B4     | MOV AH, imm8 |
| B1     | MOV CL, imm8 | B5     | MOV CH, imm8 |
| B2     | MOV DL, imm8 | B6     | MOV DH, imm8 |
| B3     | MOV BL, imm8 | B7     | MOV BH, imm8 |

 B8-BF (16/32/64-bit MOV)

| Opcode | 16-bit        | 32-bit         | 64-bit         |
| ------ | ------------- | -------------- | -------------- |
| B8     | MOV AX, imm16 | MOV EAX, imm32 | MOV RAX, imm64 |
| B9     | MOV CX, imm16 | MOV ECX, imm32 | MOV RCX, imm64 |
| BA     | MOV DX, imm16 | MOV EDX, imm32 | MOV RDX, imm64 |
| BB     | MOV BX, imm16 | MOV EBX, imm32 | MOV RBX, imm64 |
| BC     | MOV SP, imm16 | MOV ESP, imm32 | MOV RSP, imm64 |
| BD     | MOV BP, imm16 | MOV EBP, imm32 | MOV RBP, imm64 |
| BE     | MOV SI, imm16 | MOV ESI, imm32 | MOV RSI, imm64 |
| BF     | MOV DI, imm16 | MOV EDI, imm32 | MOV RDI, imm64 |

---

##### -  Opcode C0-CF (RET, JMP, CALL, INT, SHL, SHR)

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| C0 | ROL/ROR/RCL/RCR r/m8, imm8 | Rotate |
| C1 | ROL/ROR/RCL/RCR r/m16/32, imm8 | Rotate |
| C2 | RET imm16 | Return with pop |
| C3 | RET | Return |
| C4 | LES | Load ES (obsolete) |
| C5 | LDS | Load DS (obsolete) |
| C6 | MOV r/m8, imm8 | Memory ← imm8 |
| C7 | MOV r/m16/32, imm16/32 | Memory ← imm16/32 |
| C8 | ENTER imm16, imm8 | Create stack frame |
| C9 | LEAVE | Destroy stack frame |
| CA | RET far imm16 | (obsolete) |
| CB | RET far | (obsolete) |
| CC | INT 3 | Breakpoint |
| CD | INT imm8 | Software interrupt |
| CE | INTO | (obsolete) |
| CF | IRET | Return from interrupt |

---

##### - Opcode D0-DF (SHL, SHR, SAL, SAR, FPU)

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| D0 | SHL/SHR/SAL/SAR r/m8, 1 | Shift by 1 |
| D1 | SHL/SHR/SAL/SAR r/m16/32, 1 | Shift by 1 |
| D2 | SHL/SHR/SAL/SAR r/m8, CL | Shift by CL |
| D3 | SHL/SHR/SAL/SAR r/m16/32, CL | Shift by CL |
| D4 | AAM | (obsolete) |
| D5 | AAD | (obsolete) |
| D6 | SALC | (obsolete) |
| D7 | XLAT | Table lookup |
| D8-DF | FPU Instructions | Floating point operations |

---

##### - Opcode E0-EF (LOOP, JMP, CALL, IN, OUT)

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| E0 | LOOPNE / LOOPNZ | Loop while not zero |
| E1 | LOOPE / LOOPZ | Loop while zero |
| E2 | LOOP | Decrement RCX, loop if not zero |
| E3 | JCXZ / JECXZ | Jump if CX/ECX zero |
| E4 | IN AL, imm8 | Input from port (8-bit) |
| E5 | IN EAX, imm8 | Input from port (32-bit) |
| E6 | OUT imm8, AL | Output to port (8-bit) |
| E7 | OUT imm8, EAX | Output to port (32-bit) |
| E8 | CALL rel32 | Call near |
| E9 | JMP rel32 | Jump near |
| EA | JMP far | (obsolete) |
| EB | JMP rel8 | Short jump |
| EC | IN AL, DX | Input from port |
| ED | IN EAX, DX | Input from port |
| EE | OUT DX, AL | Output to port |
| EF | OUT DX, EAX | Output to port |

---

##### - Opcode F0-FF (LOCK, REP, HLT, CLC, STC, CLI, STI)

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| F0 | LOCK | Bus lock prefix |
| F1 | (reserved) | - |
| F2 | REPNE / REPNZ | Repeat not equal |
| F3 | REP / REPE | Repeat |
| F4 | HLT | Halt CPU |
| F5 | CMC | Complement carry flag |
| F6 | TEST/NOT/NEG/MUL/IMUL/DIV r/m8 | Group (8-bit) |
| F7 | TEST/NOT/NEG/MUL/IMUL/DIV r/m16/32 | Group (16/32-bit) |
| F8 | CLC | Clear carry flag |
| F9 | STC | Set carry flag |
| FA | CLI | Clear interrupt flag |
| FB | STI | Set interrupt flag |
| FC | CLD | Clear direction flag |
| FD | STD | Set direction flag |
| FE | INC/DEC r/m8 | Group (8-bit) |
| FF | INC/DEC/CALL/JMP/PUSH r/m16/32 | Group |

---

##### - 2-byte Opcodes (0F xx) -some examples

| Opcode | Instruction | ရှင်းလင်းချက် |
|--------|-------------|---------------|
| 0F 31 | RDTSC | Read Time Stamp Counter |
| 0F A2 | CPUID | CPU identification |
| 0F 05 | SYSCALL | System call (64-bit) |
| 0F 07 | SYSRET | Return from system call |
| 0F 30 | WRMSR | Write Model Specific Register |
| 0F 32 | RDMSR | Read Model Specific Register |

---
