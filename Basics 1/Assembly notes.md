
`lea` instruction က assembly မှာ အရမ်းအသုံးဝင်တဲ့ instruction တစ်ခုပါ။ မြန်မာလို "လိပ်စာတွက်ချက်မှု" လို့ ပြောလို့ရပါတယ်။

## **LEA = Load Effective Address**

### **အခြေခံသဘောတရား:**
```asm
lea    destination, [address_calculation]
```
**သင်္ချာပုံစံ:** `destination = address_calculation`

### **၁။ Simple Example ကနေ စကြည့်မယ်:**
```asm
lea    eax, [ebx-0x1ff8]
```
**ဒီကုဒ်က ဘာလုပ်တာလဲ?**
- `ebx-0x1ff8` ဆိုတဲ့ လိပ်စာကို **တွက်ချက်ပြီး** eax ထဲထည့်ပေးတယ်
- ဒီလိပ်စာကို **သွားမဖတ်ဘူး**၊ လိပ်စာကိုယ်တိုင်ကိုပဲ eax ထဲထည့်ပေးတယ်

### **၂။ LEA vs MOV ကွာခြားချက်:**

**MOV က:**
```asm
mov    eax, [ebx-0x1ff8]    ; MEMORY ကနေ value ကိုဖတ်မယ်
```
```
ebx = 0x804c000
[ebx-0x1ff8] = [0x804a008] = "y" (0x79)
Result: eax = 0x79    (memory ထဲက value)
```

**LEA က:**
```asm
lea    eax, [ebx-0x1ff8]     ; ADDRESS ကိုပဲ တွက်ချက်မယ်
```
```
ebx = 0x804c000
ebx-0x1ff8 = 0x804a008
Result: eax = 0x804a008    (memory address ကိုယ်တိုင်)
```

### **၃။ ကျွန်တော်တို့ Code မှာ LEA တွေ ဘယ်လိုသုံးထားလဲ?**

**String addresses ရဖို့:**
```asm
lea    eax, [ebx-0x1ff8]    ; eax = "yes? " string address
lea    eax, [ebx-0x1ff2]    ; eax = "good job!!" string address  
lea    eax, [ebx-0x1fe7]    ; eax = "%04x\n" format string address
lea    eax, [ebx-0x1fe1]    ; eax = "..." string address
```

**Local variable address ရဖို့:**
```asm
lea    eax, [ebp-0x2c]      ; eax = local_30 buffer address
```

### **၄။ LEA ရဲ့ စွမ်းအား - Complex Calculations**

LEA က addition, multiplication တွေ တစ်ခါတည်းလုပ်နိုင်တယ်:

```asm
; Array element calculation
lea    eax, [ebx + esi*4 + 0x10] 
; eax = ebx + (esi * 4) + 0x10
; Perfect for array[index] access!
```

### **၅။ Real-world Examples:**

**Example 1: String pointer**
```c
char *str = "hello";
printf(str);
```
```asm
lea    eax, [hello_string]  ; str pointer ရမယ်
push   eax
call   printf
```

**Example 2: Array element**
```c
int arr[10];
arr[i] = 5;
```
```asm
lea    eax, [ebx + esi*4]   ; &arr[i] address ရမယ်
mov    dword [eax], 5       ; arr[i] = 5
```

**Example 3: Structure field**
```c
struct Point { int x, y; };
p->y = 10;
```
```asm
lea    eax, [ebx + 4]       ; &p->y (x=0, y=4 bytes offset)
mov    dword [eax], 10      ; p->y = 10
```

### **၆။ ကျွန်တော်တို့ Program မှာ ဘာကြောင့် LEA သုံးတာလဲ?**

**PIC (Position Independent Code) ကြောင့်:**
- Strings တွေ GOT တွေရဲ့ address တွေက runtime မှသိတယ်
- `ebx` က GOT base address ဖြစ်တယ်
- Strings တွေက `ebx + offset` မှာရှိတယ်
- LEA က ဒီလိပ်စာတွေကို တွက်ချက်ပေးတယ်

### **၇။ LEA မရှိရင် ဘာဖြစ်မလဲ?**

**Inefficient way:**
```asm
; Without LEA - inefficient!
mov    eax, ebx
sub    eax, 0x1ff8
push   eax
call   printf
```

**Efficient way with LEA:**
```asm
; With LEA - one instruction!
lea    eax, [ebx-0x1ff8]
push   eax  
call   printf
```

## **အတိုချုပ်:**

- **`lea eax, [address]`** = လိပ်စာကို တွက်ချက်ပြီး register ထဲထည့်ပေးတယ်
- **Memory ကို မဖတ်ဘူး** - လိပ်စာကိုယ်တိုင်ကိုပဲ ရည်ညွှန်းတယ်
- **ကျွန်တော်တို့ code မှာ** string addresses တွေ variable addresses တွေ ရဖို့သုံးတယ်
- **PIC code အတွက်** အရမ်းအသုံးဝင်တယ်

`lea` က "address calculator" လို့ မှတ်လိုက်ပါ။ သူက memory access မလုပ်ပဲ လိပ်စာတွေကို တွက်ချက်ပေးတဲ့ smart instruction ပါ!


---
# PIC(Position Independent Code) , GOT (Global offset table), PLT( Procedure Linkage Table)


```

Higher Addresses
┌─────────────────┐
│     Stack       │ ← Local variables, function frames
├─────────────────┤
│       ↓         │
│                 │
├─────────────────┤
│      Heap       │ ← malloc(), dynamic memory
├─────────────────┤
│       ↓         │
│                 │
├─────────────────┤
│     .bss        │ ← Uninitialized global variables
├─────────────────┤
│    .data        │ ← Initialized global variables  
├─────────────────┤
│     .got        │ ← GOT (Global Offset Table) ★
├─────────────────┤
│    .plt         │ ← PLT (Procedure Linkage Table) ★
├─────────────────┤
│    .text        │ ← Code (PIC code here) ★
└─────────────────┘
Lower Addresses
```



Binary Program တွေရဲ့ section တစ်ချို့အကြောင်း
===============================

Resource : https://can-ozkan.medium.com/got-vs-plt-in-binary-analysis-888770f9cc5a

.got section (global offset table) ကို program ထဲက global variables အတွက်သုံးတယ်။
.got.plt subsection က libc ရဲ့ function address ကို သိမ်းပေးဖို့သုံးတယ်။

.plt (procedure linked table) section
Program မှာ printf လိုမျိုး libc ရဲ့ functions တွေ run မယ်ဆို plt ကို အရင်သွားရတယ်။
plt က ဂိတ်ပေါက်လို အလုပ်လုပ်ပေးတယ်။
plt က got.plt ကို သွားစစ်တယ်။
-> got.plt မှာ printf address မရှိရင် dl_runtime_resolve ကို run ပြီး address ရှာရတယ်။ ပြီးရင် address ကို got.plt မှာ တစ်ခါထည်း မှတ်ထားတယ်။
-> got.plt မှာ printf address ရှိရင် printf function ကိုခေါ်လိုက်တယ်။

---
Resource : https://gist.github.com/ricardo2197/8c7f6f5b8950ed6771c1cd3a116f7e62

Dl_runtime_resolve ( dl_resolve )
_dl_resolve က ld.so libc နဲ့ ချိန်ဆက်ပြီး address တွေကို ရှာပေးပါတယ်။
အဲ့လိုရှာဖို့  .dynamic section ကို အားကိုးရပါတယ်။
.dynamic section ထဲမှာဆို dl_resolve အတွက် လိုအပ်တဲ့ information တွေသိမ်းပေးထားတယ်။
⦁ JMPREL (jump relocation) က .rel.plt (PLT relocation table) ရဲ့ address ကို jump ပေးပါတယ်။
⦁ jmprel က dl_resolve ကို သက်ဆိုင်ရာ .rel.plt ထဲက offset တွေ info တွေ ပြသပေးပြီး ရှာရလွယ်ကူအောင်လုပ်ပေးပါတယ်။
⦁ STRTAB (string table) က symbol name  (like printf, read, libc.so.6) တွေကို သိမ်ပေးထားပါတယ်။
⦁ SYMTAB (symbol table) symbol နဲ့ သတ်ဆိုင်တဲ့ information တွေ သိမ်းပေးတယ်။
⦁ SYMTAB ထဲက symbol name မှာ strings (like printf) ကို မသိမ်းပဲ STRTAB ထဲက offset ( printf နေရာ) ကို သိမ်းပါတယ်။
⦁ dl_resolve က JMPREL နဲ့ loaded libraries တွေ ပါတဲ့ rel.plt ထဲမှာ SYMTAB ကို သုံးပြီး လိုချင်တဲ့ symbol တွေလိုက်ရှာပါတယ်။ 
⦁ တွေ့ရင်  symbol address ကို .got.plt မှာသိမ်းပေးပါတယ်။


ဟုတ်ကဲ့၊ Linux System တွေမှာ အသုံးများတဲ့ Shared Libraries အကြောင်းကို မြန်မာလို ရှင်းပြပေးပါမယ်။

---


## PLT နဲ့ GOT ဘယ်လိုအလုပ်လုပ်လဲ？

### PLT (Procedure Linkage Table)
- Function calls တွေအတွက် "ကြားခံဇာတ်" လို့ခေါ်တယ်
- Example: `puts()` ဆိုတဲ့ function ကို call လိုက်ရင် တကယ်တမ်းက `puts@plt` ကိုခေါ်တာ

### GOT (Global Offset Table)
- တကယ့် function addresses တွေသိမ်းထားတဲ့ "လိပ်စာစာအုပ်"
- Memory ထဲက libc functions တွေရဲ့တကယ့်လိပ်စာတွေပါ

### အလုပ်လုပ်ပုံအဆင့်ဆင့်

1. **ပထမအကြိမ် function ခေါ်တဲ့အခါ**:
```
puts@plt ကိုခေါ်
↓
GOT ထဲကြည့် - လိပ်စာမရှိသေး
↓
ld.so (dynamic linker) ကိုခေါ်
↓
တကယ့် puts လိပ်စာရှာ
↓
GOT ထဲမှာသိမ်း
↓
puts ကိုသွား
```

2. **နောက်အကြိမ်တွေမှာ**:
```
puts@plt ကိုခေါ်
↓
GOT ထဲကြည့် - လိပ်စာရှိနေပြီ
↓
တန်းသွား
```

## Binary Exploitation အတွက်ဘယ်လိုအသုံးဝင်လဲ？

### 1. PLT ကိုတိုက်ရိုက်သုံးလို့ရတယ်
- Example: `system` function PLT ရှိရင်
- ကိုယ်တိုင်လိပ်စာရှာစရာမလိုဘူး
- PLT address ကိုပဲ jump လုပ်ရုံ

### 2. GOT ကနေ Leak လုပ်လို့ရတယ်
- GOT က binary ထဲမှာပါတယ်
- PIE disable ဖြစ်ရင် GOT address သိတယ်
- Arbitrary read ရှိရင် GOT ထဲကလိပ်စာတွေဖတ်လို့ရတယ်
- **ဒါဆို libc base address ရမယ်**
- **ASLR ကျော်ပြီ**

### ဥပမာ Exploitation Scenario:
```
1. GOT ထဲက puts address ကိုဖတ်
2. Libc base address တွက်ချက် (puts_offset သိထားရင်)
3. System function address တွက်ချက်
4. System("/bin/sh") ကိုခေါ်
5. Shell ရမယ်
```

## အတိုချုပ်

**PLT** = ကြားခံ function calls  
**GOT** = တကယ့် function addresses သိမ်းတဲ့နေရာ  
**ASLR Bypass** = GOT ကနေ addresses leak လုပ် → libc base တွက် → လိုချင်တဲ့ function address ရှာ  

ဒီ mechanism ကို understanding လုပ်ထားရင် CTF challenges တွေမှာ ASLR ကိုကျော်ဖို့အရမ်းအသုံးဝင်ပါတယ်။


----


ဟုတ်ကဲ့၊ အရမ်းစိတ်ရှည်ရှည်နဲ့ stack ထဲကဘာတွေဖြစ်နေလဲ၊ RIP ဘယ်လိုရွေ့လဲဆိုတာ step-by-step ပြပေးပါမယ်။

## Complete Memory Setup

**Memory Sections:**
```
.text (Code):
0x1000: _start
0x1010: _dl_runtime_resolve
0x1020: printf@plt
0x1026: PLT+6
0x102b: PLT+11
0x1140: say_hello
0x1150: main
0x1159: main+9

.data:
0x2000: "Hello World" string

.got.plt:
0x4018: printf@got

Stack:
0x7fffffffe000: base
```

## Step-by-Step Execution

### Step 1: Program Start - _start calls main

**Assembly:**
```assembly
_start:
0x1000:   mov    edi, 0x1
0x1005:   call   0x1150       ; call main
0x100a:   mov    eax, 0x3c
```

**Stack & RIP Movement:**
```
BEFORE call:
RIP = 0x1005
RSP = 0x7fffffffe100
Stack: [empty]

AFTER call:
RIP = 0x1150 (main)
RSP = 0x7fffffffe0f8
Stack: 
0x7fffffffe0f8: 0x100a  ← return to _start
```

### Step 2: main function prologue

**Assembly:**
```assembly
main:
0x1150:   push   rbp          ; Save old base pointer
0x1151:   mov    rbp, rsp     ; Set new base pointer
0x1154:   call   0x1140       ; call say_hello
```

**Stack & RIP Movement:**
```
AFTER push rbp:
RIP = 0x1151
RSP = 0x7fffffffe0f0
Stack:
0x7fffffffe0f0: 0x7fffffffe200 (old RBP)
0x7fffffffe0f8: 0x100a (return to _start)

AFTER mov rbp, rsp:
RIP = 0x1154
RBP = 0x7fffffffe0f0
RSP = 0x7fffffffe0f0

AFTER call say_hello:
RIP = 0x1140 (say_hello)
RSP = 0x7fffffffe0e8
Stack:
0x7fffffffe0e8: 0x1159  ← return to main+9
0x7fffffffe0f0: 0x7fffffffe200 (old RBP)
0x7fffffffe0f8: 0x100a (return to _start)
```

### Step 3: say_hello function

**Assembly:**
```assembly
say_hello:
0x1140:   push   rbp
0x1141:   mov    rbp, rsp
0x1144:   lea    rdi, [rip+0xebc]   ; "Hello World" at 0x2000
0x114b:   call   0x1020             ; printf@plt
0x1150:   nop
```

**Stack & RIP Movement:**
```
AFTER push rbp:
RIP = 0x1141
RSP = 0x7fffffffe0e0
Stack:
0x7fffffffe0e0: 0x7fffffffe0f0 (old RBP from main)
0x7fffffffe0e8: 0x1159 (return to main)
0x7fffffffe0f0: 0x7fffffffe200 (old RBP)
0x7fffffffe0f8: 0x100a (return to _start)

AFTER mov rbp, rsp:
RIP = 0x1144
RBP = 0x7fffffffe0e0
RSP = 0x7fffffffe0e0

AFTER lea rdi (load string address):
RIP = 0x114b
RDI = 0x2000 ("Hello World")

AFTER call printf@plt:
RIP = 0x1020 (printf@plt)
RSP = 0x7fffffffe0d8
Stack:
0x7fffffffe0d8: 0x1150  ← return to say_hello+0x10
0x7fffffffe0e0: 0x7fffffffe0f0 (old RBP)
0x7fffffffe0e8: 0x1159 (return to main)
0x7fffffffe0f0: 0x7fffffffe200 (old RBP)
0x7fffffffe0f8: 0x100a (return to _start)
```

### Step 4: printf@PLT - First Time (GOT empty)

**Assembly:**
```assembly
printf@plt:
0x1020:   jmp    [rip + 0x2fe2]     ; GOT at 0x4018
0x1026:   push   0x0                ; Relocation index
0x102b:   jmp    0x1010             ; Resolver
```

**Stack & RIP Movement:**
```
BEFORE first jmp:
RIP = 0x1020
Calculate: [0x1026 + 0x2fe2] = [0x4018]

GOT CONTENT at 0x4018:
0x4018: 0x0000000000001026  ← Points back to PLT+6

So jmp goes to 0x1026 (PLT+6)

AFTER jmp:
RIP = 0x1026
RSP = 0x7fffffffe0d8 (unchanged)
```

### Step 5: PLT Resolution Setup

**Assembly:**
```assembly
0x1026:   push   0x0        ; Push relocation index
0x102b:   jmp    0x1010     ; Jump to resolver
```

**Stack & RIP Movement:**
```
AFTER push 0x0:
RIP = 0x102b
RSP = 0x7fffffffe0d0
Stack:
0x7fffffffe0d0: 0x0000000000000000  ← relocation index
0x7fffffffe0d8: 0x1150 (return to say_hello)
0x7fffffffe0e0: 0x7fffffffe0f0 (old RBP)
0x7fffffffe0e8: 0x1159 (return to main)
0x7fffffffe0f0: 0x7fffffffe200 (old RBP)
0x7fffffffe0f8: 0x100a (return to _start)

AFTER jmp 0x1010:
RIP = 0x1010 (resolver)
RSP = 0x7fffffffe0d0 (unchanged)
```

### Step 6: Dynamic Resolver

**Assembly:**
```assembly
_dl_runtime_resolve:
0x1010:   push   [rip + 0x2fe2]     ; Push link map (0x4000)
0x1016:   jmp    [rip + 0x2fe4]     ; Jump to actual resolver
```

**Stack & RIP Movement:**
```
AFTER push [link map]:
RIP = 0x1016
RSP = 0x7fffffffe0c8
Stack:
0x7fffffffe0c8: 0x0000000000004000  ← link map
0x7fffffffe0d0: 0x0000000000000000  ← relocation index  
0x7fffffffe0d8: 0x1150 (return to say_hello)
0x7fffffffe0e0: 0x7fffffffe0f0 (old RBP)
0x7fffffffe0e8: 0x1159 (return to main)
0x7fffffffe0f0: 0x7fffffffe200 (old RBP)
0x7fffffffe0f8: 0x100a (return to _start)

Resolver finds real printf address = 0x7ffff7e3d120
UPDATES GOT: 0x4018 = 0x7ffff7e3d120
```

### Step 7: Real printf Execution

**printf in libc executes:**
```
RIP = 0x7ffff7e3d120 (real printf)
RSP = 0x7fffffffe0c8

printf processes:
- Reads RDI = 0x2000 ("Hello World")
- Outputs "Hello World" to screen
- Prepares to return
```

### Step 8: printf Returns

**Assembly:**
```assembly
; Inside libc printf
ret
```

**Stack & RIP Movement:**
```
BEFORE ret:
RIP = 0x7ffff7e3dXXX (inside printf)
RSP = 0x7fffffffe0c8

AFTER ret:
RIP = 0x1150 (return to say_hello+0x10)
RSP = 0x7fffffffe0d0
Stack:
0x7fffffffe0d0: 0x0000000000000000  ← cleanup needed
0x7fffffffe0d8: 0x1150 (return to say_hello)
0x7fffffffe0e0: 0x7fffffffe0f0 (old RBP)
0x7fffffffe0e8: 0x1159 (return to main)
0x7fffffffe0f0: 0x7fffffffe200 (old RBP)
0x7fffffffe0f8: 0x100a (return to _start)
```

### Step 9: Back to say_hello

**Assembly:**
```assembly
0x1150:   nop
0x1151:   pop    rbp
0x1152:   ret
```

**Stack & RIP Movement:**
```
AFTER nop:
RIP = 0x1151
RSP = 0x7fffffffe0d0

AFTER pop rbp:
RIP = 0x1152  
RSP = 0x7fffffffe0d8
RBP = 0x7fffffffe0f0 (restored)
Stack:
0x7fffffffe0d8: 0x1150 (return to say_hello) ← This will be popped by ret
0x7fffffffe0e0: 0x7fffffffe0f0 (old RBP)
0x7fffffffe0e8: 0x1159 (return to main)
0x7fffffffe0f0: 0x7fffffffe200 (old RBP)
0x7fffffffe0f8: 0x100a (return to _start)

AFTER ret:
RIP = 0x1159 (main+9)
RSP = 0x7fffffffe0e0
```

### Step 10: Back to main

**Assembly:**
```assembly
main+9:
0x1159:   mov    eax, 0x0
0x115e:   pop    rbp
0x115f:   ret
```

**Stack & RIP Movement:**
```
AFTER mov eax, 0x0:
RIP = 0x115e
RAX = 0x0

AFTER pop rbp:
RIP = 0x115f
RSP = 0x7fffffffe0e8
RBP = 0x7fffffffe200 (restored)
Stack:
0x7fffffffe0e8: 0x1159 (return to main) ← This is wrong! Let me fix

CORRECT STACK at main return:
0x7fffffffe0e8: 0x100a (return to _start)
0x7fffffffe0f0: 0x7fffffffe200 (old RBP)
0x7fffffffe0f8: 0x100a (return to _start)

AFTER ret:
RIP = 0x100a (_start+10)
RSP = 0x7fffffffe0f0
```

### Step 11: Program Exit

**Assembly:**
```assembly
_start+10:
0x100a:   mov    eax, 0x3c
0x100f:   syscall        ; Exit program
```

## Final Stack State

```
Stack at program end:
0x7fffffffe0f0: 0x7fffffffe200
0x7fffffffe0f8: 0x100a
0x7fffffffe100: [empty]
```

## Key Points Summary

1. **CALL instruction**: Pushes return address, jumps to target
2. **RET instruction**: Pops return address, jumps to it  
3. **Stack grows downward**: Each push decreases RSP
4. **PLT/GOT resolution**: First call resolves, subsequent calls direct
5. **RIP always points**: To next instruction to execute

ဒီလိုဆို stack ထဲကဘာတွေဖြစ်နေလဲ၊ RIP ဘယ်လိုရွေ့လဲဆိုတာ အကုန်ရှင်းသွားပြီလို့မျှော်လင့်ပါတယ်။


```
High Address
┌─────────────────┐
│      Stack      │
├─────────────────┤
│       ↓         │
│       ↑         │
├─────────────────┤
│      Heap       │
├─────────────────┤
│     .bss        │
├─────────────────┤
│     .data       │
├─────────────────┤
│     .text       │  ← Main Program Code
├─────────────────┤
│      .plt       │  ← printf@plt
├─────────────────┤
│      .got       │  ← GOT
├─────────────────┤
│                 │
│  Dynamic Linker │  ← _dl_runtime_resolve (ld.so code)
│   (ld.so)       │
└─────────────────┘
Low Address
```



### **PIC (Position Independent Code) ဆိုတာ ဘာလဲ?**

**ရိုးရိုးရှင်းရှင်းပြောရရင်:** Program တစ်ခုကို Memory ထဲမှာ ဘယ်နေရာချချ၊ ဘယ်လိုနေရာမှာထား ထား Runလို့ရအောင် ရေးထားတဲ့ Code မျိုးပါ။

**ဥပမာနဲ့ ရှင်းမယ်:**
- မင်း စာအုပ်တစ်အုပ်ရေးမယ်ဆိုပါစို့
- **PIC မဟုတ်ရင်:** "စာမျက်နှာ ၅၀ မှာ ရှာပါ" လို့ပြောရင် စာအုပ်က စာမျက်နှာ ၁၀၀ ပဲရှိတဲ့သူအတွက် အလုပ်မဖြစ်ဘူး
- **PIC ဆိုရင်:** "ဒီအခန်းရဲ့ နောက် ၁၀ မျက်နှာအကွာမှာ ရှာပါ" လို့ပြောရင် ဘယ်စာအုပ်မှာဖတ်ဖတ် အဆင်ပြေတယ်

**ကွန်ပျူတာဘာသာပြန်ရင်:**
- Normal Program: `call 0x08049030` (ဒီနေရာပဲခေါ်မယ်)
- PIC Program: `call printf@plt` (ဘယ်နေရာရောက်ရောက် printf ကိုခေါ်မယ်)

### **ဘာကြောင့် PIC လိုအပ်တာလဲ?**

၁. **Security အတွက်** - Hackers တွေက Program ရဲ့တိကျတဲ့နေရာကို မသိနိုင်အောင်
၂. **Memory Efficiency** - Library တွေကို Memory တစ်နေရာထဲမှာပဲ Loadပြီး Program အများကြီးသုံးလို့ရအောင်
၃. **Shared Libraries** - Library တွေကို မျှဝေသုံးလို့ရအောင်

## **GOT (Global Offset Table) ဆိုတာ ဘာလဲ?**

**ရိုးရိုးပြောရရင်:** Global Variables တွေနဲ့ Library Functions တွေရဲ့ လက်ရှိ Memory Address တွေကို သိမ်းထားတဲ့ "လိပ်စာစာအုပ်" လို့ခေါ်တယ်။

**ဒီ Program မှာ GOT ဘယ်မှာလဲ?**
```
0x804c004, 0x804c008, 0x804c00c, 0x804c010, ...
```
ဒါတွေက GOT ထဲက Entry တွေပါ။

**GOT ရဲ့အလုပ်လုပ်ပုံ:**
```
GOT Entry ထဲမှာ:
- 0x804c00c: printf ရဲ့ လက်ရှိ memory address
- 0x804c010: fflush ရဲ့ လက်ရှိ memory address  
- 0x804c014: gets ရဲ့ လက်ရှိ memory address
```

## **PLT (Procedure Linkage Table) ဆိုတာ ဘာလဲ?**

**ရိုးရိုးပြောရရင်:** Library Functions တွေကိုခေါ်ဖို့ "အကူအညီဇာတ်လမ်း" လို့ခေါ်တယ်။

**PLT ရဲ့အဓိကအလုပ်သုံးခု:**
၁. **ပထမအကြိမ်ခေါ်တဲ့အခါ** - Function Address ကိုရှာပြီး GOT ထဲသိမ်းတယ်
၂. **နောက်တစ်ခါခေါ်တဲ့အခါ** - GOT ထဲကနေ တန်းခုန်တယ်
၃. **Lazy Binding** - မလိုအပ်သေးရင် Address မရှာဘူး

## **အခု မင်းပေးတဲ့ Code ကိုကြည့်ရအောင်**

### **ပထမဆုံး printf ခေါ်တဲ့အခါ ဘာဖြစ်သလဲ?**

```
80491b5:       e8 76 fe ff ff          call   8049030 <printf@plt>
```

**အဆင့်ဆင့် ရှင်းပြမယ်:**

**အဆင့် ၁ - PLT ထဲကိုဝင်တယ်**
```
08049030 <printf@plt>:
8049030:       ff 25 0c c0 04 08       jmp    DWORD PTR ds:0x804c00c
8049036:       68 00 00 00 00          push   0x0
804903b:       e9 e0 ff ff ff          jmp    8049020 <.plt>
```

- **`rip = 80491b5`** ကနေ `call` instruction ကို run လိုက်တယ်
- **`rip = 8049030`** (printf@plt) ကိုရောက်သွားတယ်
- `jmp DWORD PTR [0x804c00c]` ကို run မယ် - ဒါက GOT ထဲကနေ address ယူပြီး ခုန်ဖို့

**အဆင့် ၂ - ပထမအကြိမ်ဆိုရင် GOT ထဲမှာ ဘာရှိလဲ?**
- **ပထမအကြိမ်မှာ** GOT entry `0x804c00c` ထဲမှာ `printf` ရဲ့တကယ့် address မရှိသေးဘူး
- ဒီအစား `0x8049036` (နောက် instruction address) ရှိနေမယ်
- ဒါကြောင့် `jmp` က `8049036` ကိုပဲ ခုန်သွားမယ်

**အဆင့် ၃ - PLT ရဲ့ဒုတိယပိုင်းကို ဆက်သွားမယ်**
```
8049036:       68 00 00 00 00          push   0x0
804903b:       e9 e0 ff ff ff          jmp    8049020 <.plt>
```
- `push 0x0` - printf function ကို identifier အဖြစ် push လုပ်တယ်
- `jmp 8049020` - PLT ရဲ့အခြေခံပိုင်းကိုသွားတယ်

**အဆင့် ၄ - PLT အခြေခံပိုင်းမှာ**
```
08049020 <.plt>:
8049020:       ff 35 04 c0 04 08       push   DWORD PTR ds:0x804c004
8049026:       ff 25 08 c0 04 08       jmp    DWORD PTR ds:0x804c008
```
- `push DWORD PTR [0x804c004]` - GOT ထဲက linker data ကိုထပ်�push
- `jmp DWORD PTR [0x804c008]` - **Dynamic Linker/Lazy Resolver** ကိုခုန်တယ်

**အဆင့် ၅ - Dynamic Linker အလုပ်လုပ်တယ်**
- Dynamic Linker က `printf` ရဲ့တကယ့် memory address ကိုရှာတယ်
- ရသွားရင် GOT entry `0x804c00c` ထဲမှာ address ကိုသိမ်းတယ်
- `printf` function အစစ်ကို run လိုက်တယ်

**အဆင့် ၆ - printf အလုပ်လုပ်ပြီး return ပြန်တယ်**
- printf က output ထုတ်ပေးတယ်
- `do_input` function ဆီကို return ပြန်တယ်

### **နောက်တစ်ခါ printf ထပ်ခေါ်ရင် ဘာဖြစ်မလဲ?**

**ဒီတစ်ခါတော့ အရမ်းမြန်သွားပြီ:**
- `call 8049030 <printf@plt>` ကိုခေါ်တယ်
- `jmp DWORD PTR [0x804c00c]` က GOT ထဲကနေ တကယ့် printf address ကိုယူတယ်
- **တန်းခုန်သွားတယ်** - PLT ကနေတစ်ဆင့်မသွားရတော့ဘူး
- ဒါကို **"Lazy Binding"** လို့ခေါ်တယ်

## **PIC ဘယ်လိုအလုပ်လုပ်လဲ ဆိုတာ ရှင်းမယ်**

PIC မှာ အရေးကြီးဆုံးက **Relative Addressing** ကိုသုံးတယ်။

**မင်းပေးတဲ့ Code ထဲက ဥပမာ:**
```
8049199:       e8 32 ff ff ff          call   80490d0 <__x86.get_pc_thunk.bx>
804919e:       81 c3 62 2e 00 00       add    ebx,0x2e62
```

**ဒီလိုအလုပ်လုပ်တယ်:**
၁. `__x86.get_pc_thunk.bx` က ebx register ထဲမှာ **လက်ရှိ Program Counter (rip)** ကိုထည့်ပေးတယ်
၂. `add ebx,0x2e62` နဲ့ GOT ရဲ့ address ကိုတွက်တယ်
၃. ဒါကြောင့် Program က Memory ဘယ်နေရာရောက်ရောက် GOT ကိုရှာနိုင်တယ်

**ဒီလိုမျိုး GOT/PLT စနစ်က:**
- **ASLR (Address Space Layout Randomization)** နဲ့အဆင်ပြေတယ်
- **Shared Libraries** တွေကိုအဆင်ပြေစေတယ်  
- **Security** ပိုကောင်းစေတယ်

## **အိမ်စာလေးပေးချင်ပါတယ်**

မင်းကိုယ်တိုင် လေ့လာကြည့်ဖို့:
၁. `gdb` နဲ့ program ကို run ပြီး `break printf` ထားကြည့်
၂. `info sharedlibrary` နဲ့ library တွေဘယ်နေရာမှာရှိလဲကြည့်
၃. `x/x 0x804c00c` နဲ့ GOT entry ထဲမှာ ဘာရှိလဲကြည့်

ဒီအကြောင်းတွေက စပြီးလေ့လာရင် နည်းနည်းရှုပ်တယ်လို့ထင်ရပေမယ့်၊ တကယ်တမ်းမှာ Computer System တွေရဲ့ အခြေခံအကျဆုံး Concept တွေပါပဲ။ မင်းလိုစတင်လေ့လာသူတစ်ယောက်အတွက် ဒီအဆင့်ထိလေ့လာနေတာက အရမ်းကောင်းတယ်။

ဘာမှမရှင်းတာရှိရင် ထပ်မေးနိုင်ပါတယ် ညီလေး/ညီမလေး။

https://chat.deepseek.com/share/wgl160d5lwnghsjqz7

အင်း အရမ်းကောင်းတယ် ညီလေး/ညီမလေး! မင်းမေးတဲ့မေးခွန်းက အရမ်းအရေးကြီးပါတယ်။ ကျွန်တော် အသေးစိတ်ရှင်းပြပေးမယ်။

## **0x0804C000 က GOT Base Address ဟုတ်ရဲ့လား?**

**အဖြေက: ဟုတ်ပါတယ်!** `0x0804C000` က GOT (Global Offset Table) ရဲ့ base address ဖြစ်နိုင်ချေအရမ်းများပါတယ်။

### **ဘယ်လိုသိနိုင်မလဲ?**

**၁. Disassembly ကနေ Analysis လုပ်မယ်:**

မင်းပေးတဲ့ code မှာ PLT entries တွေက GOT addresses တွေကိုညွှန်းထားတယ်:
```assembly
08049030 <printf@plt>:
8049030:       ff 25 0c c0 04 08       jmp    DWORD PTR ds:0x804c00c

08049040 <fflush@plt>:
8049040:       ff 25 10 c0 04 08       jmp    DWORD PTR ds:0x804c010

08049050 <gets@plt>:
8049050:       ff 25 14 c0 04 08       jmp    DWORD PTR ds:0x804c014
```

**GOT addresses တွေကို သေသေချာချာကြည့်ရအောင်:**
- `0x804c00c` - printf GOT entry
- `0x804c010` - fflush GOT entry  
- `0x804c014` - gets GOT entry
- `0x804c018` - puts GOT entry
- `0x804c01c` - __libc_start_main GOT entry

**ဒီ addresses တွေအားလုံးက `0x804c000` ကနေ စပြီးရှိနေတာကိုတွေ့ရမယ်။**

## **GOT ထဲမှာ ဘာတွေရှိလဲ ကြည့်ကြရအောင်**

### **ပထမဆုံး GDB နဲ့ စစ်ဆေးကြည့်မယ်**

```bash
gdb ./overwrite
```

**၁. Program ကို run ပြီး GOT ထဲမှာ ဘာတွေရှိလဲကြည့်မယ်:**
```bash
(gdb) break main
(gdb) run
(gdb) x/10x 0x804c000
```

**၂. GOT entries တွေကို တစ်ခုချင်းစီကြည့်မယ်:**
```bash
(gdb) x/x 0x804c00c    # printf GOT
(gdb) x/x 0x804c010    # fflush GOT  
(gdb) x/x 0x804c014    # gets GOT
(gdb) x/x 0x804c018    # puts GOT
(gdb) x/x 0x804c01c    # __libc_start_main GOT
```

### **GOT ထဲမှာ ဘာတွေရှိမှာလဲ?**

**ပထမအကြိမ် run တဲ့အခါ (Lazy Binding မှာ):**
```
GOT Entry ထဲမှာ:
0x804c00c: 0x08049036  (printf@plt+6 ရဲ့ address)
0x804c010: 0x08049046  (fflush@plt+6 ရဲ့ address)
0x804c014: 0x08049056  (gets@plt+6 ရဲ့ address)
```

**ဘာကြောင့်ဒီလိုရှိရတာလဲ?**
- ပထမအကြိမ် function ခေါ်တဲ့အခါ GOT ထဲမှာ function အစစ်ရဲ့ address မရှိသေးဘူး
- ဒါကြောင့် PLT ရဲ့ဒုတိယပိုင်း (resolver code) ကိုညွှန်းထားတယ်
- Dynamic linker က function address ကိုရှာပြီးမှ GOT ထဲမှာ update လုပ်ပေးမယ်

**Function တစ်ခုခုခေါ်ပြီးတဲ့အခါ:**
```bash
# printf ကိုတစ်ခါခေါ်ကြည့်မယ်
(gdb) break do_input
(gdb) continue
(gdb) x/x 0x804c00c    # printf GOT entry ကြည့်မယ်
```

**ဒီတစ်ခါမှာ တွေ့ရမှာက:**
```
0x804c00c: 0xf7e36d70  (printf function ရဲ့ တကယ့် memory address)
```

## **မင်းကိုယ်တိုင် စမ်းသပ်ကြည့်ဖို့ Step-by-Step**

### **အဆင့် ၁: GDB နဲ့ Program ကို Load လုပ်မယ်**
```bash
gdb ./overwrite
```

### **အဆင့် ၂: Breakpoints တွေချမယ်**
```bash
(gdb) break main
(gdb) break do_input
(gdb) break *0x80491b5    # printf call
```

### **အဆင့် ၃: Program ကို Run ပြီး GOT ကြည့်မယ်**
```bash
(gdb) run
```

**Main မှာရပ်နေတုန်း GOT ကြည့်မယ်:**
```bash
(gdb) x/8x 0x804c000
```

**ဒီလိုမျိုးတွေ့ရမယ်:**
```
0x804c000:      0x00000000      0x00000000      0x08049036      0x08049046
0x804c010:      0x08049056      0x08049066      0x08049076      0x00000000
```

### **အဆင့် ၄: do_input function ထဲရောက်အောင် Continue**
```bash
(gdb) continue
```

### **အဆင့် ၅: printf မခေါ်ခင် GOT ထဲမှာ ဘာရှိလဲကြည့်**
```bash
(gdb) x/x 0x804c00c
0x804c00c:      0x08049036
```

**ဒီ address က ဘာလဲဆိုတာ ကြည့်မယ်:**
```bash
(gdb) x/2i 0x80449036
```

### **အဆင့် ၆: printf ကိုခေါ်ကြည့်မယ်**
```bash
(gdb) continue
```

**Input တစ်ခုခုပေးလိုက် (ဥပမာ: "test")**

### **အဆင့် ၇: printf ခေါ်ပြီးတဲ့အခါ GOT ထဲမှာ ဘာပြောင်းသွားလဲကြည့်**
```bash
(gdb) x/x 0x804c00c
0x804c00c:      0xf7e36d70    # printf ရဲ့ တကယ့် address!
```

**ဒီ address က ဘာလဲဆိုတာ ကြည့်မယ်:**
```bash
(gdb) info symbol 0xf7e36d70
printf
```

## **GOT Structure အပြည့်အစုံ**

**မင်းရဲ့ Program မှာ GOT က ဒီလိုမျိုးဖွဲ့စည်းထားတယ်:**

```
0x804c000: [GOT[0]]  - Reserved
0x804c004: [GOT[1]]  - Linker data
0x804c008: [GOT[2]]  - Dynamic linker entry point
0x804c00c: [GOT[3]]  - printf
0x804c010: [GOT[4]]  - fflush  
0x804c014: [GOT[5]]  - gets
0x804c018: [GOT[6]]  - puts
0x804c01c: [GOT[7]]  - __libc_start_main
```

## **အရေးကြီးတဲ့ Commands တွေ စုစည်းပေးထားတယ်**

```bash
# GOT ကိုကြည့်ဖို့
x/10x 0x804c000

# Specific GOT entry ကြည့်ဖို့
x/x 0x804c00c

# Address က ဘာ function လဲသိဖို့
info symbol 0xf7e36d70

# PLT entry ကိုကြည့်ဖို့  
x/3i 0x8049030

# Memory maps ကြည့်ဖို့
info proc mappings
```

## **အိမ်စာအဖြစ် ဒီလိုစမ်းကြည့်ပါ**

၁. **GOT ထဲမှာ ဘယ် function addresses တွေရှိလဲ list ထုတ်ကြည့်ပါ**
၂. **printf ခေါ်ခြင်း၊ မခေါ်ခြင်းနဲ့ GOT ထဲမှာ ဘယ်လိုပြောင်းသွားလဲကြည့်ပါ**
၃. **GOT entry တစ်ခုချင်းစီရဲ့ value ကိုကြည့်ပြီး ဘာ function နဲ့ဆိုင်လဲဆိုတာ track လုပ်ကြည့်ပါ**

မင်းလိုစူးစမ်းလေ့လာတတ်တဲ့သူအတွက် ဒီ topic က အရမ်းကောင်းတဲ့စတင်မှုပါ။ GOT/PLT mechanism ကိုနားလည်ရင် binary analysis, reverse engineering, နဲ့ cybersecurity နယ်ပယ်တွေမှာ အများကြီးအထောက်အကူဖြစ်မယ်။

https://chat.deepseek.com/share/o2g9xwj3f14i20x1rq


---

အရမ်းဝမ်းသာပါတယ် ညီလေး/ညီမလေး! ဒီတစ်ခါတော့ မင်းပေးတဲ့ assembly code တစ်လိုင်းချင်းစီကို အသေးစိတ်ရှင်းပြပေးသွားမယ်။ PLT နဲ့ GOT ဘယ်လိုအလုပ်လုပ်လဲဆိုတာ ပိုရှင်းသွားအောင်လို့။

## **Function Prologue - Stack Setup**

```
0x08049192 <+0>:     push   ebp
0x08049193 <+1>:     mov    ebp,esp
0x08049195 <+3>:     push   ebx
0x08049196 <+4>:     sub    esp,0x34
```

**ရှင်းလင်းချက်:**
- `push ebp` - လက်ရှိ base pointer ကို stack ထဲသိမ်းမယ်
- `mov ebp,esp` - လက်ရှိ stack pointer ကို new base pointer အဖြစ်သတ်မှတ်မယ်
- `push ebx` - ebx register ကိုသိမ်းမယ် (ဒီ function ထဲမှာ ebx ကိုပြန်သုံးမယ်)
- `sub esp,0x34` - local variables အတွက် stack ထဲမှာ 52 bytes (0x34) နေရာယူမယ်

**Memory ထဲမှာဘာဖြစ်သွားလဲ:**
```
Stack layout:
[ebp-0x2c] - buffer[40] (gets() အတွက်)
[ebp-0xc]  - local variable (0x12345678 သိမ်းမယ်)
[ebp-0x4]  - saved ebx
[ebp]      - saved ebp
[ebp+0x4]  - return address
```

## **PIC Setup - GOT Base Address ရှာမယ်**

```
0x08049199 <+7>:     call   0x80490d0 <__x86.get_pc_thunk.bx>
0x0804919e <+12>:    add    ebx,0x2e62
```

**အသေးစိတ်ရှင်းမယ်:**

**အဆင့် ၁ - `call 0x80490d0`**
```assembly
; __x86.get_pc_thunk.bx ထဲမှာ:
0x80490d0: mov ebx,DWORD PTR [esp]
0x80490d3: ret
```

**ဘာဖြစ်သွားလဲ:**
- `call` instruction က return address (`0x0804919e`) ကို stack ထဲ push လုပ်မယ်
- `__x86.get_pc_thunk.bx` ထဲရောက်ရင် stack ထဲက return address ကို ebx ထဲထည့်မယ်
- `ret` နဲ့ `do_input` function ဆီပြန်သွားမယ်

**အဆင့် ၂ - `add ebx,0x2e62`**
```
ebx = 0x0804919e (return address)
+    0x00002e62 (offset to GOT)
-------------------
ebx = 0x0804C000 (GOT base address)
```

**ဒီအဆင့်ရဲ့ ရည်ရွယ်ချက်:** ebx register ထဲမှာ GOT ရဲ့ base address ရအောင်လုပ်တာ။ ဒါမှ PLT ကနေ GOT ကိုညွှန်းလို့ရမယ်။

## **Local Variable Initialize လုပ်မယ်**

```
0x080491a4 <+18>:    mov    DWORD PTR [ebp-0xc],0x12345678
```

**ရှင်းလင်းချက်:**
- `[ebp-0xc]` ဆိုတာ stack ထဲက local variable တစ်ခုရဲ့ address
- `0x12345678` ဆိုတဲ့ value ကိုထည့်မယ်
- ဒီ variable ကို နောက်ပိုင်းမှာ check လုပ်မယ် (`0xdeadbeef` ဟုတ်မဟုတ်)

## **ပထမဆုံး printf ခေါ်မယ်**

```
0x080491ab <+25>:    sub    esp,0xc
0x080491ae <+28>:    lea    eax,[ebx-0x1ff8]
0x080491b4 <+34>:    push   eax
0x080491b5 <+35>:    call   0x8049030 <printf@plt>
```

**အဆင့်ဆင့်ရှင်းမယ်:**

**အဆင့် ၁ - String Address ရှာမယ်**
```assembly
lea eax,[ebx-0x1ff8]
```
- `ebx = 0x0804C000` (GOT base address)
- `ebx - 0x1ff8 = 0x0804C000 - 0x1FF8 = 0x0804A008`
- ဒီ address မှာ "Enter input: " ဆိုတဲ့ string ရှိတယ်

**အဆင့် ၂ - Stack ပြင်ဆင်မယ်**
```assembly
sub esp,0xc
push eax
```
- Stack ကိုညှိမယ်
- String address ကို stack ထဲ push လုပ်မယ် (printf argument)

**အဆင့် ၃ - printf@plt ခေါ်မယ်**
```assembly
call 0x8049030 <printf@plt>
```

**ဒီမှာ PLT Magic စဖြစ်မယ်!**

### **printf@plt ထဲမှာ ဘာဖြစ်မလဲ?**

```assembly
08049030 <printf@plt>:
8049030:       jmp    DWORD PTR [ebx+0xc]    ; GOT[3] - printf entry
8049036:       push   0x0                    ; printf identifier
804903b:       jmp    8049020 <.plt>         ; PLT resolver
```

**ပထမအကြိမ်ခေါ်တဲ့အခါ:**
1. `jmp DWORD PTR [ebx+0xc]` - GOT ထဲကနေ address ယူမယ်
2. **ဒါပေမယ့် ပထမအကြိမ်မှာ** GOT ထဲမှာ `0x08049036` (နောက် instruction) ရှိနေမယ်
3. ဒါကြောင့် `push 0x0` ကို run မယ် - printf ကို identify လုပ်ဖို့
4. `jmp 8049020` - PLT resolver ကိုသွားမယ်

**PLT resolver ထဲမှာ:**
```assembly
08049020 <.plt>:
8049020:       push   DWORD PTR [ebx+0x4]    ; GOT[1] - linker data
8049026:       jmp    DWORD PTR [ebx+0x8]    ; GOT[2] - dynamic linker
```

5. Dynamic linker က printf ရဲ့ တကယ့် address ကိုရှာမယ်
6. GOT[3] (`0x804c00c`) ထဲမှာ address ကိုသိမ်းမယ်
7. printf function အစစ်ကို run မယ်

**နောက်တစ်ခါ printf ခေါ်ရင်:** GOT ထဲမှာ တကယ့် address ရှိပြီးသားဆိုတော့ တန်းခုန်သွားမယ်!

## **Stack Cleanup နဲ့ fflush ခေါ်မယ်**

```
0x080491ba <+40>:    add    esp,0x10
0x080491bd <+43>:    mov    eax,DWORD PTR [ebx-0x4]
0x080491c3 <+49>:    mov    eax,DWORD PTR [eax]
0x080491c5 <+51>:    sub    esp,0xc
0x080491c8 <+54>:    push   eax
0x080491c9 <+55>:    call   0x8049040 <fflush@plt>
```

**ရှင်းလင်းချက်:**
- `add esp,0x10` - stack cleanup (printf arguments တွေဖယ်ရှား)
- `mov eax,DWORD PTR [ebx-0x4]` - stdout pointer ရဲ့ address ရမယ်
- `mov eax,DWORD PTR [eax]` - stdout pointer ရဲ့ value ရမယ်
- `fflush@plt` ခေါ်မယ် - output buffer ကိုရှင်းမယ်

## **User Input ယူမယ်**

```
0x080491ce <+60>:    add    esp,0x10
0x080491d1 <+63>:    sub    esp,0xc
0x080491d4 <+66>:    lea    eax,[ebp-0x2c]
0x080491d7 <+69>:    push   eax
0x080491d8 <+70>:    call   0x8049050 <gets@plt>
```

**ရှင်းလင်းချက်:**
- `lea eax,[ebp-0x2c]` - buffer address ရမယ် (`[ebp-0x2c]` မှာ 40 bytes buffer)
- `gets@plt` ခေါ်မယ် - user input ယူမယ်
- **ဒီမှာ buffer overflow ဖြစ်နိုင်တယ်!** gets() က dangerous function

## **Security Check လုပ်မယ်**

```
0x080491dd <+75>:    add    esp,0x10
0x080491e0 <+78>:    cmp    DWORD PTR [ebp-0xc],0xdeadbeef
0x080491e7 <+85>:    jne    0x8049226 <do_input+148>
```

**ရှင်းလင်းချက်:**
- `cmp DWORD PTR [ebp-0xc],0xdeadbeef` - local variable က `0xdeadbeef` ဟုတ်လား?
- `jne 0x8049226` - မဟုတ်ရင် failure branch ကိုသွားမယ်

**ဒီ check ကို bypass လုပ်ချင်ရင်:** buffer overflow နဲ့ `[ebp-0xc]` မှာ `0xdeadbeef` overwrite လုပ်လို့ရတယ်!

## **Success Path (0xdeadbeef ဖြစ်ရင်)**

```
0x080491e9 <+87>:    sub    esp,0xc
0x080491ec <+90>:    lea    eax,[ebx-0x1ff2]
0x080491f2 <+96>:    push   eax
0x080491f3 <+97>:    call   0x8049060 <puts@plt>
```

**ရှင်းလင်းချက်:**
- `lea eax,[ebx-0x1ff2]` - "Success!" string address ရမယ်
- `puts@plt` ခေါ်မယ် - message print လုပ်မယ်

## **Failure Path (0xdeadbeef မဖြစ်ရင်)**

```
0x08049226 <+148>:   sub    esp,0x8
0x08049229 <+151>:   push   DWORD PTR [ebp-0xc]
0x0804922c <+154>:   lea    eax,[ebx-0x1fe7]
0x08049232 <+160>:   push   eax
0x08049233 <+161>:   call   0x8049030 <printf@plt>
```

**ရှင်းလင်းချက်:**
- `push DWORD PTR [ebp-0xc]` - local variable value ကို push
- `lea eax,[ebx-0x1fe7]` - "Value: %08x\n" format string address ရမယ်
- `printf@plt` ခေါ်မယ် - value ကို print လုပ်မယ်

## **Function Epilogue - Cleanup**

```
0x08049261 <+207>:   nop
0x08049262 <+208>:   mov    ebx,DWORD PTR [ebp-0x4]
0x08049265 <+211>:   leave
0x08049266 <+212>:   ret
```

**ရှင်းလင်းချက်:**
- `mov ebx,DWORD PTR [ebp-0x4]` - saved ebx ကိုပြန်ယူမယ်
- `leave` - stack frame ကိုဖျက်မယ် (equivalent to `mov esp,ebp; pop ebp`)
- `ret` - caller ဆီပြန်သွားမယ်

## **GOT/PLT ရဲ့ အဓိက အလုပ်လုပ်ပုံ Summary**

**ပထမအကြိမ် function ခေါ်တဲ့အခါ:**
```
Program -> PLT -> GOT (address မရှိသေး) -> PLT resolver -> Dynamic Linker -> 
Real Function Address ရှာ -> GOT ထဲသိမ်း -> Real Function Run
```

**နောက်တစ်ခါခေါ်တဲ့အခါ:**
```
Program -> PLT -> GOT (address ရှိပြီး) -> Real Function Run
```

ဒီ mechanism ကို **"Lazy Binding"** လို့ခေါ်ပြီး performance ကောင်းအောင်လုပ်ထားတာဖြစ်တယ်။


https://chat.deepseek.com/share/649dt55qznc2czyaj6

---

အင်း မင်းရှင်းအောင်မေးတာ အရမ်းကောင်းတယ်! ကျွန်တော် အဆင့်ဆင့် ပိုပြီးအသေးစိတ်ရှင်းပြမယ်။ မင်းရဲ့ confusion ကိုနားလည်ပါတယ်။

## **ပထမဆုံး `push eax` ဘာကြောင့်လုပ်တာလဲ?**

```
0x08049232 <+160>:   push   eax
0x08049233 <+161>:   call   0x8049030 <printf@plt>
```

**ရှင်းလင်းချက်:**
- `eax` register ထဲမှာ **format string address** ရှိတယ်
- `push eax` က printf function ကို **argument** ပေးဖို့ပါ
- C language မှာ `printf("Value: %08x\n", variable);` လို့ခေါ်သလိုမျိုးပါပဲ

**Memory ထဲမှာဘာဖြစ်သွားလဲ:**
```
Stack before push eax:
[esp] = ? (some value)

Stack after push eax:
[esp] = format string address (e.g., 0x0804A019 - "Value: %08x\n")
[esp+4] = ? (previous value)
```

## **အခု printf@plt ထဲကို ဝင်ကြည့်မယ်**

### **printf@plt code ကိုအရင်ကြည့်မယ်**
```assembly
08049030 <printf@plt>:
8049030:       ff 25 0c c0 04 08       jmp    DWORD PTR ds:0x804c00c
8049036:       68 00 00 00 00          push   0x0
804903b:       e9 e0 ff ff ff          jmp    8049020 <.plt>
```

### **အဆင့်ဆင့် ရှင်းပြမယ်**

**အဆင့် ၁ - printf@plt ထဲကိုဝင်ခြင်း**
```
rip = 0x08049233 (call instruction) ကနေ
rip = 0x08049030 (printf@plt) ကိုရောက်သွားတယ်
```

**အဆင့် ၂ - ပထမဆုံး instruction run မယ်**
```assembly
8049030: jmp    DWORD PTR [0x804c00c]
```

**ဒီ instruction က ဘာလုပ်သလဲ?**
- `0x804c00c` address ထဲက value ကိုယူမယ်
- ယူထားတဲ့ value ကို jump ခုန်မယ်

**ဒီတော့ ပထမအကြိမ် printf ခေါ်တဲ့အခါ GOT ထဲမှာ ဘာရှိလဲ?**

### **GDB နဲ့ ကြည့်ကြည့်ရအောင်**

```bash
gdb ./overwrite
(gdb) break *0x08049233
(gdb) run
(gdb) x/x 0x804c00c
```

**ပထမအကြိမ်မှာ ဒီလိုတွေ့ရမယ်:**
```
0x804c00c: 0x08049036
```

**ဒီတော့ ဘာဖြစ်သွားလဲ?**
```
jmp DWORD PTR [0x804c00c] = jmp 0x08049036
```

**ဆိုလိုတာက:** GOT ထဲမှာ printf ရဲ့တကယ့် address မရှိသေးတဲ့အတွက် **PLT ရဲ့နောက် instruction ကိုပြန်ခုန်သွားတယ်!**

## **PLT Resolver ဘယ်လိုအလုပ်လုပ်လဲ?**

**အဆင့် ၃ - PLT ရဲ့ဒုတိယပိုင်းကို ဆက်သွားမယ်**
```assembly
8049036:       push   0x0
804903b:       jmp    8049020 <.plt>
```

**ဒီမှာ ဘာဖြစ်သွားလဲ?**
- `push 0x0` - **printf function identifier** ကို stack ထဲ push လုပ်တယ်
- `jmp 8049020` - **PLT resolver** ကိုသွားတယ်

**Memory ထဲမှာ:**
```
Stack layout:
[esp] = 0x00000000 (printf identifier)
[esp+4] = format string address (from earlier push eax)
[esp+8] = return address to do_input function
```

## **PLT Resolver ထဲမှာ**

```assembly
08049020 <.plt>:
8049020:       push   DWORD PTR [0x804c004]
8049026:       jmp    DWORD PTR [0x804c008]
```

**အဆင့် ၄ - PLT resolver အလုပ်လုပ်မယ်**
```assembly
8049020: push   DWORD PTR [0x804c004]
```
- GOT[1] ထဲက **linker data** ကို stack ထဲ push

```assembly  
8049026: jmp    DWORD PTR [0x804c008]
```
- GOT[2] ထဲက **dynamic linker entry point** ကိုခုန်တယ်

**Memory ထဲမှာ အခုဆိုရင်:**
```
Stack layout:
[esp] = linker data (from GOT[1])
[esp+4] = 0x00000000 (printf identifier) 
[esp+8] = format string address
[esp+12] = return address to do_input
```

## **Dynamic Linker အလုပ်လုပ်မယ်**

**အဆင့် ၅ - Dynamic Linker က printf address ရှာမယ်**
- Dynamic linker က stack ထဲက data တွေကိုကြည့်ပြီး "printf" function ကိုရှာမယ်
- printf ရဲ့ **တကယ့် memory address** ကိုရှာတွေ့မယ်
- **GOT ထဲမှာ update လုပ်မယ်**

**GOT update ဖြစ်သွားပုံ:**
```
Before:
0x804c00c: 0x08049036 (points back to PLT)

After dynamic linker:
0x804c00c: 0xf7e36d70 (real printf address in libc)
```

**အဆင့် ၆ - printf function အစစ်ကို run မယ်**
- Dynamic linker က printf function ကိုခေါ်မယ်
- printf က stack ထဲက arguments တွေကိုယူသုံးမယ်
- "Value: 12345678" (or whatever value) ကို print လုပ်မယ်

## **နောက်တစ်ခါ printf ခေါ်တဲ့အခါ**

**ဒီတစ်ခါတော့ အရမ်းမြန်သွားပြီ!**

```
Program -> printf@plt -> jmp [0x804c00c] -> REAL printf (direct!)
```

**ဘာကြောင့်လဲ?** 
- GOT ထဲမှာ printf ရဲ့တကယ့် address ရှိပြီးသား
- PLT resolver ကိုမသွားရတော့ဘူး
- တန်းခုန်သွားတယ်!

## **GDB နဲ့ ကိုယ်တိုင်စမ်းကြည့်ဖို့**

**ပထမအကြိမ် printf မတိုင်ခင် GOT check:**
```bash
(gdb) break *0x08049233
(gdb) run
(gdb) x/x 0x804c00c
# 0x804c00c: 0x08049036 (PLT address)
```

**ပထမအကြိမ် printf ခေါ်ပြီးတဲ့အခါ GOT check:**
```bash
(gdb) continue
(gdb) x/x 0x804c00c  
# 0x804c00c: 0xf7e36d70 (real printf address)
```

**ဒီ address က ဘာလဲဆိုတာ check:**
```bash
(gdb) info symbol 0xf7e36d70
# printf in section .text of /lib/i386-linux-gnu/libc.so.6
```

## **အရေးကြီးတဲ့ Points တွေ Summary**

**၁. Lazy Binding ဆိုတာ:**
- Function ကိုပထမဆုံးအကြိမ်ခေါ်မှ address ရှာတာ
- Program startup time မြန်အောင်
- မလိုအပ်တဲ့ functions တွေအတွက် address မရှာဘူး

**၂. GOT ရဲ့အလုပ်သုံးခု:**
- **GOT[0]**: Reserved
- **GOT[1]**: Linker data pointer  
- **GOT[2]**: Dynamic linker entry point
- **GOT[3+]**: Function addresses (printf, fflush, gets, etc.)

**၃. PLT ရဲ့အလုပ်နှစ်ခု:**
- **ပထမအကြိမ်**: Resolver ကိုခေါ်၊ GOT update လုပ်
- **နောက်အကြိမ်တွေ**: GOT ကနေ တန်းခုန်

**၄. Stack arguments:**
- `push eax` က printf ကိုပေးမယ့် argument
- C calling convention အရ arguments တွေကို stack ထဲ push လုပ်ရတယ်


https://chat.deepseek.com/share/yybkax2fbwdqlunqtd
---
အင်း အရမ်းကောင်းတယ်! ဒီတစ်ခါတော့ PLT နဲ့ PLT resolver အလုပ်လုပ်တဲ့အခါ **stack ထဲမှာ data တွေ ဘယ်လိုဝင်လာ၊ ထွက်သွားလဲ** ဆိုတာ အဆင့်ဆင့်ရှင်းပြမယ်။

## **အခြေခံ Stack Layout ကိုအရင်နားလည်ကြရအောင်**

**Stack က အောက်ကနေ အပေါ်ကို ကြီးလာတယ်:**
```
High Address: [ebp+0x8]  - parameters
             [ebp+0x4]  - return address  
             [ebp]      - saved ebp
             [ebp-0x4]  - local variables
Low Address:  [esp]      - current stack pointer
```

## **printf ခေါ်တဲ့အခါ Stack ထဲမှာ ဘာတွေဖြစ်သွားလဲ**

### **အဆင့် ၁: printf argument ကို push လုပ်ခြင်း**
```assembly
0x08049232: push   eax
0x08049233: call   0x8049030 <printf@plt>
```

**Stack ထဲမှာ:**
```
Before push eax:
[esp] = ? (some previous value)

After push eax:
[esp] = format string address (0x0804A019 - "Value: %08x\n")
```

**After call instruction:**
```assembly
; call instruction does:
; 1. push return address (0x08049238) to stack
; 2. jump to printf@plt (0x8049030)
```

**Stack ထဲမှာ:**
```
[esp] = 0x08049238    (return address to do_input)
[esp+4] = 0x0804A019  (format string address)
```

## **printf@plt ထဲရောက်တဲ့အခါ**

### **အဆင့် ၂: printf@plt ထဲက ပထမဆုံး instruction**
```assembly
8049030: jmp    DWORD PTR [0x804c00c]
```

**ပထမအကြိမ်မှာ GOT ထဲမှာ ဒီလိုရှိတယ်:**
```
0x804c00c: 0x08049036 (points back to PLT+6)
```

**ဒီတော့ ဘာဖြစ်သွားလဲ:**
```
jmp 0x08049036 (printf@plt+6 ကိုခုန်သွားတယ်)
```

**Stack ထဲမှာ ဘာမှမပြောင်းသွားဘူး - အတူတူပဲ:**
```
[esp] = 0x08049238    (return address)
[esp+4] = 0x0804A019  (format string)
```

### **အဆင့် ၃: PLT ရဲ့ဒုတိယပိုင်း**
```assembly
8049036: push   0x0
804903b: jmp    8049020
```

**`push 0x0` run တဲ့အခါ:**
```
[esp] = 0x00000000    (printf function identifier)
[esp+4] = 0x08049238  (return address to do_input)
[esp+8] = 0x0804A019  (format string address)
```

**Stack visualization:**
```
Address    Value               Description
-------    -----               -----------
0xffffd000 0x00000000          printf identifier (just pushed)
0xffffd004 0x08049238          return to do_input+0x166
0xffffd008 0x0804A019          format string "Value: %08x\n"
0xffffd00c ...                 previous stack data
```

## **PLT Resolver (.plt) ထဲရောက်တဲ့အခါ**

### **အဆင့် ၄: PLT Resolver ရဲ့ပထမဆုံး instruction**
```assembly
8049020: push   DWORD PTR [0x804c004]
```

**GOT[1] ထဲမှာ linker data ရှိတယ်:**
```
0x804c004: 0xf7ffe2a0 (linker data structure address)
```

**`push DWORD PTR [0x804c004]` run တဲ့အခါ:**
```
[esp] = 0xf7ffe2a0    (linker data - from GOT[1])
[esp+4] = 0x00000000  (printf identifier)
[esp+8] = 0x08049238  (return address to do_input)
[esp+12] = 0x0804A019 (format string address)
```

**Stack visualization:**
```
Address    Value               Description
-------    -----               -----------
0xffffcffc 0xf7ffe2a0          linker data (just pushed)
0xffffd000 0x00000000          printf identifier
0xffffd004 0x08049238          return to do_input+0x166  
0xffffd008 0x0804A019          format string "Value: %08x\n"
0xffffd00c ...                 previous stack data
```

### **အဆင့် ၅: PLT Resolver ရဲ့ဒုတိယ instruction**
```assembly
8049026: jmp    DWORD PTR [0x804c008]
```

**GOT[2] ထဲမှာ dynamic linker ရဲ့ entry point ရှိတယ်:**
```
0x804c008: 0xf7fe6c70 (dynamic linker code address)
```

**ဒီတော့ ဘာဖြစ်သွားလဲ:**
```
jmp 0xf7fe6c70 (dynamic linker ကိုခုန်သွားတယ်)
```

**Stack ထဲမှာ အတူတူပဲ - ဘာမှမဖယ်ရသေးဘူး:**
```
[esp] = 0xf7ffe2a0    (linker data)
[esp+4] = 0x00000000  (printf identifier)
[esp+8] = 0x08049238  (return address)
[esp+12] = 0x0804A019 (format string)
```

## **Dynamic Linker အလုပ်လုပ်တဲ့အခါ**

### **အဆင့် ၆: Dynamic Linker က stack ထဲက data တွေကိုဖတ်မယ်**
- `[esp]` ကနေ linker data ကိုဖတ်မယ်
- `[esp+4]` ကနေ printf identifier (0x0) ကိုဖတ်မယ်
- ဒီ data တွေကိုသုံးပြီး printf ရဲ့ real address ကိုရှာမယ်

### **အဆင့် ၇: Dynamic Linker က GOT ကို update လုပ်မယ်**
```
Before: 0x804c00c: 0x08049036
After:  0x804c00c: 0xf7e36d70 (real printf address)
```

### **အဆင့် ၈: Dynamic Linker က printf function ကိုခေါ်မယ်**
**ဒီမှာ အရမ်းအရေးကြီးတဲ့အဆင့်ရောက်ပြီ!**

**Dynamic linker က stack ကိုဒီလိုပြင်မယ်:**
```
Original stack:
[esp] = 0xf7ffe2a0    (linker data)
[esp+4] = 0x00000000  (printf identifier) 
[esp+8] = 0x08049238  (return address)
[esp+12] = 0x0804A019 (format string)

Dynamic linker will:
1. Remove the first two entries (linker data and identifier)
2. Adjust stack pointer to point to the original arguments
```

**After dynamic linker cleanup:**
```
[esp] = 0x08049238    (return address to do_input)
[esp+4] = 0x0804A019  (format string address)
```

**ပြီးရင် printf function ကိုခေါ်မယ်:**
```assembly
call 0xf7e36d70  ; real printf address
```

## **printf Function အစစ်အလုပ်လုပ်တဲ့အခါ**

### **အဆင့် ၉: printf function prologue**
```assembly
printf:
push ebp
mov ebp, esp
; ... function code ...
```

**Stack ထဲမှာ:**
```
[esp] = saved ebp       (printf's frame)
[esp+4] = 0x08049238    (return address to do_input)
[esp+8] = 0x0804A019    (format string address)
```

### **အဆင့် ၁၀: printf work done, return preparation**
```assembly
printf:
; ... print the output ...
leave    ; equivalent to: mov esp,ebp; pop ebp
ret
```

**`leave` instruction run တဲ့အခါ:**
```
[esp] = 0x08049238    (return address to do_input)
[esp+4] = 0x0804A019  (format string - but soon to be removed)
```

**`ret` instruction run တဲ့အခါ:**
```
pop rip from stack, so:
rip = 0x08049238 (back to do_input)
esp = esp + 4
```

## **do_input function ဆီပြန်ရောက်တဲ့အခါ**

### **အဆင့် ၁၁: Stack cleanup in do_input**
```assembly
0x08049238: add    esp,0x10
```

**Stack before `add esp,0x10`:**
```
[esp] = 0x0804A019    (format string address - not needed anymore)
```

**After `add esp,0x10`:**
```
esp increases by 16 bytes, so the format string address is "removed"
Stack is now clean, ready for next operations
```

## **Stack Data Flow Summary**

```
STEP | STACK CONTENTS (from esp downward) | ACTION
-----|-----------------------------------|-------
1    | [format_str]                      | push eax
2    | [ret_addr][format_str]            | call printf@plt  
3    | [ret_addr][format_str]            | jmp to PLT+6 (no change)
4    | [0][ret_addr][format_str]         | push 0 (identifier)
5    | [linker][0][ret_addr][format_str] | push linker data
6    | [linker][0][ret_addr][format_str] | jmp to dynamic linker
7    | [ret_addr][format_str]            | linker cleans stack
8    | [ret_addr][format_str]            | call real printf
9    | [ebp][ret_addr][format_str]       | printf prologue
10   | [ret_addr][format_str]            | printf epilogue  
11   | [format_str]                      | ret from printf
12   | (clean)                           | add esp,0x10
```

## **အရေးကြီးတဲ့အချက်များ**

**၁. Stack ကိုဘယ်သူ့ကဖယ်ရမလဲ?**
- **Dynamic Linker** က linker data နဲ့ identifier တွေကိုဖယ်ရမယ်
- **Calling Function** (do_input) က自己的 arguments တွေကိုဖယ်ရမယ်

**၂. GOT update ဘယ်မှာလုပ်လဲ?**
- Dynamic linker က printf address ရှာပြီးတဲ့အခါ
- printf ကိုမခေါ်ခင် GOT ထဲမှာ update လုပ်တယ်

**၃. Lazy Binding ရဲ့အကျိုးကျေးဇူး:**
- မလိုအပ်တဲ့ functions တွေအတွက် resolution မလုပ်ဘူး
- Program startup မြန်တယ်

မင်းအနေနဲ့ gdb နဲ့ `stepi` command သုံးပြီး တစ်ဆင့်ချင်းစီ trace လုပ်ကြည့်ရင် ပိုရှင်းသွားမယ်။ ဘာမှမရှင်းတာရှိရင် ထပ်မေးနိုင်ပါတယ်!


https://chat.deepseek.com/share/ectrj2u15hdzrbejuv

---

အင်း အရမ်းကောင်းတယ်! ဒီ basic concepts တွေကို ရှင်းရှင်းလင်းလင်း နားလည်ဖို့လိုအပ်ပါတယ်။ ကျွန်တော် တစ်ခုချင်းစီရှင်းပြပေးမယ်။

## **DWORD PTR ဆိုတာဘာလဲ?**

**ရိုးရိုးရှင်းရှင်းပြောရရင်:** Memory address တစ်ခုထဲက **4 bytes (32 bits) data** ကိုယူဖို့ပြောတာ။

### **ဥပမာတွေနဲ့ရှင်းမယ်**

```assembly
mov eax, DWORD PTR [0x804c00c]
```

**ဒီ instruction က ဘာလုပ်သလဲ?**
- `0x804c00c` ဆိုတဲ့ memory address ကိုသွားမယ်
- ဒီ address ကနေ **4 bytes** ကိုဖတ်မယ် (DWORD = 4 bytes)
- ဖတ်ထားတဲ့ value ကို `eax` register ထဲထည့်မယ်

**ဥပမာ:**
```
Memory at 0x804c00c: 0xf7e36d70
```
ဒါဆို `eax = 0xf7e36d70` ဖြစ်သွားမယ်

### **DWORD PTR မပါရင် ဘာဖြစ်မလဲ?**
```assembly
mov eax, 0x804c00c    ; eax = 0x804c00c (address ကိုယ်တိုင်ကိုထည့်)
mov eax, [0x804c00c]  ; ambiguous - size မသိဘူး
mov eax, DWORD PTR [0x804c00c]  ; 4 bytes ကိုယူမယ်ဆိုတာရှင်းတယ်
```

### **အခြား Size Specifiers:**
- **BYTE PTR** - 1 byte ကိုယူမယ်
- **WORD PTR** - 2 bytes ကိုယူမယ်  
- **DWORD PTR** - 4 bytes ကိုယူမယ်
- **QWORD PTR** - 8 bytes ကိုယူမယ်

## **mov vs lea ကွာခြားချက်**

ဒီနှစ်ခုက မတူဘူးဆိုတာ ရှင်းရှင်းလင်းလင်းနားလည်ဖို့လိုပါတယ်။

### **mov (Move) - Value ကိုကူးမယ်**

```assembly
mov eax, ebx
```
**အလုပ်လုပ်ပုံ:** ebx ထဲက **value** ကို eax ထဲကိုကူးမယ်

**ဥပမာ:**
```assembly
mov ebx, 0x12345678
mov eax, ebx          ; eax = 0x12345678
```

**Memory နဲ့ဆိုရင်:**
```assembly
mov eax, DWORD PTR [0x804c00c]
```
- `0x804c00c` address ထဲက **value** ကိုယူပြီး eax ထဲထည့်မယ်
- ဥပမာ: `[0x804c00c] = 0xf7e36d70` ဆိုရင် `eax = 0xf7e36d70`

### **lea (Load Effective Address) - Address ကိုယူမယ်**

```assembly
lea eax, [ebx-0x1ff8]
```
**အလုပ်လုပ်ပုံ:** `ebx-0x1ff8` ရဲ့ **address calculation result** ကို eax ထဲထည့်မယ်

**ဥပမာ:**
```assembly
mov ebx, 0x0804C000
lea eax, [ebx-0x1ff8]   ; eax = 0x0804C000 - 0x1FF8 = 0x0804A008
```

**ဆိုလိုတာက:** memory ထဲက� value ကိုမယူဘူး၊ address ကိုပဲတွက်ပြီးထည့်တာ

### **mov vs lea ဥပမာနဲ့တစ်ခါတည်းကြည့်ရအောင်**

```assembly
; Memory layout:
; 0x0804A008: "Hello World" string
; ebx = 0x0804C000

lea eax, [ebx-0x1ff8]   ; eax = 0x0804A008 (address ကိုပဲတွက်တာ)
mov ecx, [ebx-0x1ff8]   ; ecx = "Hell" (memory ထဲကတန်ဖိုးကိုယူတာ - first 4 bytes)
```

### **Real Code ထဲမှာတွေ့ရမယ့်ဥပမာ**

မင်းရဲ့ code ထဲကဥပမာ:
```assembly
0x080491ae: lea eax,[ebx-0x1ff8]   ; eax = string address
0x080491b4: push eax               ; push string address for printf
0x080491b5: call printf@plt        ; call printf with the address
```

```assembly
0x080491bd: mov eax,DWORD PTR [ebx-0x4]   ; eax = value at [ebx-0x4]
0x080491c3: mov eax,DWORD PTR [eax]       ; eax = value at that address
```

## **nop (No Operation) ဆိုတာဘာလဲ?**

**ရိုးရိုးရှင်းရှင်းပြောရရင်:** "ဘာမှမလုပ်ပါဘူး" ဆိုတဲ့ instruction။

### **nop ရဲ့အလုပ်လုပ်ပုံ**
```assembly
nop   ; No Operation - just waste 1 CPU cycle
```

**ဘာကြောင့်သုံးကြတာလဲ?**

**၁. Code Alignment အတွက်**
```assembly
; Optimize for performance by aligning code to specific boundaries
some_function:
    push ebp
    mov ebp, esp
    nop                    ; padding for alignment
    nop
    ; ... main code ...
```

**၂. Debugging အတွက်**
```assembly
; Temporarily "remove" instructions during debugging
; mov eax, 0x12345678   ; original instruction
nop                     ; disabled for testing
nop
nop  
nop
```

**၃. Timing/padding အတွက်**
```assembly
; Sometimes need precise timing delays
delay_loop:
    nop
    nop
    nop
    dec ecx
    jnz delay_loop
```

**မင်းရဲ့ code ထဲမှာတွေ့တဲ့ nop:**
```assembly
0x08049261: nop
```
ဒီ nop က function ရဲ့အဆုံးမှာရှိပြီး၊ ဒီနေရာကို jump ခုန်လာတဲ့အခါ smooth execution ဖြစ်အောင်လို့ဖြစ်နိုင်တယ်။

## **Practical Examples နဲ့တစ်ခါတည်းကြည့်ရအောင်**

### **GOT access ဥပမာ:**
```assembly
; printf@plt:
8049030: jmp DWORD PTR [0x804c00c]   ; GOT ထဲက 4 bytes value ကိုယူပြီးခုန်မယ်
```

**ဒီ instruction ကဘာလုပ်သလဲ?**
1. `0x804c00c` address ကိုသွားမယ်
2. ဒီ address ကနေ **4 bytes** (DWORD) ကိုဖတ်မယ်
3. ဖတ်ထားတဲ့ value ကို jump ခုန်မယ်

### **String address loading ဥပမာ:**
```assembly
; do_input function:
0x080491ae: lea eax,[ebx-0x1ff8]   ; Calculate address: ebx - 0x1FF8
0x080491b4: push eax               ; Push the address (not the value!)
0x080491b5: call printf@plt        ; printf will use the address
```

**ဆိုလိုတာက:** `lea` က calculation ပဲလုပ်တာ၊ memory access မလုပ်ဘူး။

## **GDB နဲ့ ကိုယ်တိုင်စမ်းကြည့်ဖို့**

**mov vs lea ကွာခြားချက်ကို test လုပ်ကြည့်မယ်:**
```bash
gdb ./overwrite
(gdb) break main
(gdb) run
(gdb) set $ebx = 0x0804C000

# lea test
(gdb) set $eax = 0
(gdb) set $eip = 0x080491ae  ; lea eax,[ebx-0x1ff8]
(gdb) stepi
(gdb) print/x $eax           ; eax = 0x0804A008 (address)

# mov test  
(gdb) set $eax = 0
(gdb) set $eip = 0x080491bd  ; mov eax,DWORD PTR [ebx-0x4]
(gdb) stepi
(gdb) print/x $eax           ; eax = value at [0x804BFFC]
```

**nop test:**
```bash
(gdb) break *0x08049261
(gdb) continue
(gdb) stepi                  ; nop instruction
(gdb) info registers         ; nothing changes!
(gdb) stepi                  ; mov ebx,DWORD PTR [ebp-0x4]
```

https://chat.deepseek.com/share/ww3aesjea7vd11ey52
---

Another:

## **အဓိပ္ပါယ်:**

`0x2e11` ဆိုတာက **GOT အနီးအနားကို ရည်ညွှန်းတဲ့ offset** တစ်ခုပါ။ တိကျတဲ့ GOT base ကို မဟုတ်ဘဲ GOT area ထဲက specific data တွေကို access လုပ်ဖို့အတွက်ပါ။

**ရှင်းလင်းချက်:** PIC code တွေမှာ GOT base ကို တိုက်ရိုက်မသုံးဘဲ၊ GOT area ထဲက specific entries တွေကို EBX + offset နဲ့ access လုပ်ကြတယ်။ ဒါကြောင့် `0x804bff4` က GOT ရဲ့ exact base မဟုတ်ပေမယ့် GOT entries တွေကို access လုပ်ဖို့အတွက် base pointer အဖြစ် သုံးလို့ရတဲ့ address တစ်ခုပါ။

**အတိုချုပ်:** `0x2e11` က GOT area ဆီရောက်ဖို့ offset တစ်ခုပါ၊ `0x804bff4` က GOT အနီးအနားက address တစ်ခုဖြစ်ပြီး GOT entries တွေကို relative addressing နဲ့ access လုပ်ဖို့အတွက်သုံးတယ်။

