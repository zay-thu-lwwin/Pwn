
PTR - an address to a location for storing a value.


SI - source index
DI - destination index

E - 32
R - 64

Assembly syntax - AT&T and Intel  (AT&T use GNU Assembler {gas}  and Intel use  NetWide Assembler{NASM})

```
NASM Format: CMD <destination>, <source> <;comments>
AT&T Format:  CMD <source>, <destination> <#comments> ($literal value, x%register)

```


operands -> 5 , $eax etc.. 

Jump

Just like Python, assembly programs are executed from top to bottom. Unlike Python, in assembly, you can jump to specific instructions, skipping everything in between them
jump goes by the function + offset given in angle brackets to the right of the address


static analysis -> read code
dynamic analysis -> run code to know what it does

break points

```

pwndbg> set disable-randomization off
pwndbg> break main
pwndbg> run

info funcions
info registers
info frame 
si (go into function call)
ni (skip function call)
set $rip
set $rax
disas main
continue
break *main+59
info break (i b)
delete break 1
run
info registers eax (register)
x/4xb $rbp-0x4 (memory)
x/s $rbp-0x30
x/s 0x003d3
x/10gx $rsp
pattern create
pattern offset 0x0a66a6a6a6a6a66 
x $ebp - 0xc
x/10x
(gdb) info symbol 0xf7e36d70
# printf in section .text of /lib/i386-linux-gnu/libc.so.6
telescope 20 (tel 20)
vmmap libc
info proc mappings

x/[COUNT][FORMAT][SIZE]

py exec("import gdb; gdb.execute('ni'); gdb.execute('telescope 20'); gdb.execute('x/30wx 0x804d1a0 - 16')")

command
define hook-stop

python -c "import sys; sys.stdout.buffer.write(b'\x13\xc4\x61\x56\xea\x56 %p %p %p %p %p %p')" | ./valley

```

```
(gdb) info registers           # All registers
(gdb) x/10gx $rsp             # Examine stack as 8-byte values  
(gdb) x/20bx $rsp             # Examine stack as 1-byte values
(gdb) info frame              # Current stack frame info
(gdb) backtrace               # Function call stack
(gdb) stepi                   # Single instruction step
(gdb) nexti                   # Step over function calls
(gdb) print $rsp              # Print RSP value
(gdb) print (void*)buffer     # Print buffer address
```

```

GDB မှာ memory address တစ်ခုကို breakpoint ချလိုက်တဲ့အခါ program execution က အဲဒီ memory address မှာရောက်တဲ့အခါ ရပ်သွားပါတယ်။

**ဒီနေရာမှာ သိထားရမှာက -**

- Program က breakpoint ချထားတဲ့ memory address **မှာ** ရပ်သွားတာပါ
    
- အဲဒီ address မှာရှိတဲ့ instruction **ကို မရှင်းသေးပါဘူး**
```

```
 
```

**`0xA1B2C3D4` (32-bit value)**
little endian - least significant bytes first (LSB)  
Address: 1000 1001 1002 1003
Data:    D4   C3   B2   A1
 
big endian - most significant bytes first (MSB)
Address: 1000 1001 1002 1003
Data:    A1   B2   C3   D4


- **Segmentation Fault** ဆိုတာ သင့်ရဲ့ program က **ခွင့်မပြုထားတဲ့ memory area** ကို ဝင်ရောက်ဖတ်တဲ့အခါ ဖြစ်တယ်
``
```
  void (*foo)(void) = (void (*)())val;
```


Buffer Overflow Steps

Fuzzing - 

```
# 36 'a' characters generate လုပ်မယ်
python3 -c "import sys; sys.stdout.buffer.write(b'a'*36)" | ./vuln
```



| Data Type        | Size (Bytes)                              | Range (Approximate)              |
| ---------------- | ----------------------------------------- | -------------------------------- |
| `bool`           | 1 byte                                    | true or false                    |
| `char`           | 1 byte                                    | -128 to 127 or 0 to 255          |
| `short`          | 2 bytes                                   | -32,768 to 32,767                |
| `unsigned short` | 2 bytes                                   | 0 to 65,535                      |
| `int`            | 4 bytes                                   | -2,147,483,648 to 2,147,483,647  |
| `unsigned int`   | 4 bytes                                   | 0 to 4,294,967,295               |
| `long`           | 4 bytes (on 32-bit) / 8 bytes (on 64-bit) | depends on system                |
| `unsigned long`  | 4 or 8 bytes                              | depends on system                |
| `long long`      | 8 bytes                                   | about ±9.22e18                   |
| `float`          | 4 bytes                                   | ~ ±3.4e38 (6–7 digits precision) |
| `double`         | 8 bytes                                   | ~ ±1.7e308 (15 digits precision) |
| `long double`    | 12 or 16 bytes                            | depends on compiler              |
| `wchar_t`        | 2 or 4 bytes                              | wide character type              |

what is save ebp??

Function တစ်ခု call လုပ်တိုင်း memory ထဲမှာ **stack frame** (သို့) **activation record** ဆိုတာ create လုပ်တယ်။



```
[ Typical Stack Frame ]
+-------------------+
| Local Variables   |  ← Function ထဲက variables တွေ
+-------------------+
| Saved EBP         |  ← ဒါကို မေးနေတာ!
+-------------------+
| Return Address    |  ← Function ပြီးရင် ဘယ်ကို return ပြန်သွားမလဲ
+-------------------+
| Parameters        |  ← Function arguments တွေ
+-------------------+

```

!! don't use strcpy() get()

```
64-bit LSB executable, x86-64
char input[16];  // 16 bytes
// Compiler adds 8 bytes padding here for alignment
int num = 64;    // 4 bytes  
// Another 4 bytes padding to maintain 8-byte alignment
```
*no padding in 32bit *



to see padding
```
gdb ./local-target
(gdb) break main
(gdb) run
(gdb) info frame
(gdb) x/20wx $rsp  # stack ကိုကြည့်မယ်
```



### **Compiler Optimization**



// Original code
int calculate(int a, int b) {
    int result = a + b;
    return result;
}

// Optimized (-O2) might become:
int calculate(int a, int b) {
    return a + b;  // stack operations removed
}

**သက်ရောက်မှု:** Function size ပြောင်းသွားမယ်၊ instructions တွေနည်းသွားမယ်

### **Padding & Alignment**



struct example {
    char a;      // 1 byte
    // 3 bytes padding (alignment အတွက်)
    int b;       // 4 bytes (must be 4-byte aligned)
};

**သက်ရောက်မှု:** Memory layout ပြောင်းသွားမယ်၊ expected offset တွေမှားသွားနိုင်တယ်



How data are stored in memory ??

for example ,  `int age = 34;` 

integer takes places 4 bytes so 34 is change into 4 bytes long binary  `00000000 00000000 00000000 00100010`

and it takes place like that :



|Memory Address|Stored Data (Binary)|Stored Data (Decimal)|Explanation|
|---|---|---|---|
|**1000**|`00100010`|34|**Byte 1** - LSB (Least Significant Byte)|
|**1001**|`00000000`|0|**Byte 2**|
|**1002**|`00000000`|0|**Byte 3**|
|**1003**|`00000000`|0|**Byte 4** - MSB (Most Significant Byte)|
(write first least significant bytes cuz we use little endian )

in 64 bit, only pointer size (memory address ) just increase like that 

**Memory Layout:**

| Memory Address     | Stored Data | Explanation      |
| ------------------ | ----------- | ---------------- |
| **0x7fff5a3b4c10** | `00100010`  | **Byte 1** - LSB |
| **0x7fff5a3b4c11** | `00000000`  | **Byte 2**       |
| **0x7fff5a3b4c12** | `00000000`  | **Byte 3**       |
| **0x7fff5a3b4c13** | `00000000`  | **Byte 4** - MSB |

what is pointer?
**Pointer ဆိုတာ Memory Address ကိုသိမ်းထားတဲ့ Variable တစ်ခုပါပဲ။**



	Program တစ်ခု Run တဲ့အခါ၊ OS က Memory (RAM) ထဲမှာ **အပိုင်းကြီး (၄) ပိုင်း** ခွဲပေးပါတယ်။ အပိုင်းတစ်ခုစီမှာ မတူညီတဲ့ Data အမျိုးအစားတွေ သိမ်းဆည်းထားပါတယ်။

```
High Address
+------------------+
|      Stack       |  ← Stack Pointer (SP) က ဒီဘက်ကိုညွှန်တယ်
|       |          |
|       v          |  (Stack က အောက်ကိုကြီးတယ်)
+------------------+
|                  |
|       Heap       |  ← Heap က အပေါ်ကိုကြီးတယ်
|       ^          |
|       |          |
+------------------+
|    Data Segment  |  (Global/Static Variables)
+------------------+
|    Text Segment  |  (Program Code)
+------------------+
Low Address
```

 Stack (လက်ရှိသုံးနေတဲ့ပစ္စည်း) **Local Variable** တွေ, **Function Parameter** တွေ
 Heap (သိမ်းထားတဲ့ပစ္စည်း) ခွဲယူတဲ့ Memory / `malloc()`, `calloc()` နဲ့ ခွဲယူ၊ `free()` နဲ့ ပြန်လွှတ်
 Data Segment (အမြဲသုံးပစ္စည်း) (**Global** နဲ့ **Static** variable)
 Text Segment (ချက်ပြုတ်နည်း) (read-only)


STACK - LIFO
HEAP -  NO order


![[Screenshot 2025-10-17 152147.png]]


![[general-memory-view.png]]


![[elf-sections.png]]


## Security Features များရှင်းလင်းချက်

- **RELRO** - (Relocation Read Only ) Global Offset Table (GOT) overwrite attacks မှ ကာကွယ်ပေးသည်
    
- **Stack Canary** - Stack buffer overflow မှ ကာကွယ်ပေးသည်
    
- **NX** - Stack/Heap ပေါ်တွင် code run ခွင့် ပိတ်ပေးသည်
    
- **PIE** - ASLR ကို enable လုပ်ပေးသည်


## **RELRO (Relocation Read-Only)**

**ဘာလုပ်ပေးလဲ** - GOT (Global Offset Table) overwrite attacks ကနေ ကာကွယ်ပေးတယ်

**အသေးစိတ်**:

- Binary runtime မှာ လိုအပ်တဲ့ function addresses တွေကို GOT ထဲမှာ သိမ်းထားတယ်
    
- RELRO မပါရင် hacker တွေက GOT ကို overwrite လုပ်ပြီး malicious code ကို run နိုင်တယ်
    
- RELRO ရှိရင် GOT ကို read-only လုပ်ပေးလိုက်တော့ overwrite မလုပ်နိုင်တော့ဘူး
    

**အမျိုးအစားများ**:

- **No RELRO** - ဘာ protection မှမရှိ
    
- **Partial RELRO** - GOT ကို read-only လုပ်ပေးတယ်
    
- **Full RELRO** - GOT ကို startup မှာတည်းက initialize လုပ်ပြီး read-only လုပ်ပေးတယ်

## **Stack Canary**

**ဘာလုပ်ပေးလဲ** - Stack buffer overflow attacks ကနေ ကာကွယ်ပေးတယ်

**အလုပ်လုပ်ပုံ**:

- Function တစ်ခု စခေါ်တိုင်း stack ရဲ့ အစွန်းမှာ "canary" လို့ခေါ်တဲ့ random value တစ်ခုထားပေးတယ်
    
- Function ပြန်မထွက်ခင် ဒီ canary value ပျက်သွားလားစစ်တယ်
    
- Buffer overflow ဖြစ်ရင် canary value ပျက်သွားပြီး program က crash ဖြစ်သွားတယ်
    

**နမူနာ**:

text

```
[Stack Layout]
[Local Variables][Canary][Return Address]

```
Buffer overflow ဖြစ်ရင် Canary ကိုအရင်ထိမယ်၊ ပြီးမှ Return Address ကိုထိမယ်


## **NX (No-eXecute) / DEP (Data Execution Prevention)**

**ဘာလုပ်ပေးလဲ** - Stack နဲ့ Heap ပေါ်မှာ code run ခွင့်ပိတ်ပေးတယ်

**အသေးစိတ်**:

- Memory ကို section (3) ခုခွဲထားတယ် - Code, Data, Stack
    
- NX မပါရင် hacker တွေက stack/heap ပေါ်မှာ shellcode တင်ပြီး run နိုင်တယ်
    
- NX ပါရင် stack/heap ကို "executable" flag မထားတော့ဘူး
    
- CPU က data memory (stack/heap) ပေါ်က code တွေကို run ခွင့်မပေးတော့ဘူး
    

## **PIE (Position Independent Executable)**

**ဘာလုပ်ပေးလဲ** - ASLR (Address Space Layout Randomization) ကို enable လုပ်ပေးတယ်

**အလုပ်လုပ်ပုံ**:

- PIE မပါရင် binary က memory ထဲမှာ အမြဲတမ်း fixed address မှာ load ဖြစ်တယ်
    
- PIE ပါရင် binary က memory ထဲမှာ random address တိုင်းမှာ load ဖြစ်နိုင်တယ်
    
- ဒါကြောင့် hacker တွေအတွက် specific addresses တွေကို ကြိုတင်မခန့်မှန်းနိုင်တော့ဘူး
    

**နမူနာ**:

- **No PIE**: Binary က 0x400000 မှာ အမြဲ load ဖြစ်တယ်
    
- **PIE Enabled**: Binary က 0x555555554000, 0x566666664000, စသဖြင့် random addresses တွေမှာ load ဖြစ်နိုင်တယ်


## **ဘယ်လို Attack တွေကို ကာကွယ်သလဲ？**

|Security Feature|ကာကွယ်တဲ့ Attack|ဘယ်လို ကာကွယ်သလဲ|
|---|---|---|
|**RELRO**|GOT overwrite|GOT ကို read-only လုပ်ပေး|
|**Stack Canary**|Stack buffer overflow|Stack မှာ canary value ထားပေး|
|**NX**|Shellcode injection|Stack/Heap ပေါ် code run မရအောင်|
|**PIE**|Return-to-libc, ROP|Memory addresses တွေကို randomize လုပ်ပေး|

others:


Symbols: 84 Symbols

- **အဓိပ္ပါယ်**: Binary ထဲမှာ function/variable names 84 ခု ရှိတယ် (debugging အတွက် useful)

## **FORTIFY_SOURCE ဆိုတာဘာလဲ？**

FORTIFY_SOURCE သည် GCC compiler ရဲ့ security feature တစ်ခုဖြစ်ပြီး buffer overflow vulnerabilities ကို compile-time မှာတည်းက ကာကွယ်ပေးပါတယ်



Compiling to test buffer overflow 
`gcc -fno-stack-protector -z execstack -no-pie -o output_file input_file.c`



## ELF format

ELF is a format commonly used by Unix systems.


`readelf -h /bin/bash` will show you the ELF header.

Example:

```
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x2f630
  Start of program headers:          64 (bytes into file)
  Start of section headers:          1166920 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         11
  Size of section headers:           64 (bytes)
  Number of section headers:         29
  Section header string table index: 28
```


# Regiesters




```
push rbp         ; save old RBP
mov rbp, rsp     ; RBP = current RSP (frame setup)
sub rsp, 16      ; allocate 16 bytes on stack
; ... function body ...
mov rsp, rbp     ; restore RSP
pop rbp          ; restore old RBP
```


Example Execution

```
int add(int a, int b) {
    int result = a + b;
    return result;
}

int main() {
    int x = add(5, 3);
    return 0;
}
```

```
; main function starts
main:
    push rbp            ; RSP -= 8, save RBP on stack
    mov rbp, rsp        ; RBP = RSP (new frame)
    sub rsp, 16         ; RSP -= 16 (allocate space)
    
    ; call add(5, 3)
    mov rdi, 5          ; RDI = 5 (1st arg)
    mov rsi, 3          ; RSI = 3 (2nd arg)
    call add            ; RIP jumps to add, return addr pushed
    
    ; add function
    add:
        push rbp        ; RSP -= 8, save RBP  
        mov rbp, rsp    ; RBP = RSP
        mov eax, edi    ; RAX = RDI (5)
        add eax, esi    ; RAX += RSI (3) = 8
        pop rbp         ; RBP = old value, RSP += 8
        ret             ; RIP = return address, RSP += 8
    
    ; back in main
    mov [rbp-4], eax    ; store result in local variable
    mov eax, 0          ; RAX = 0 (return value)
    mov rsp, rbp        ; RSP = RBP (clean stack)
    pop rbp             ; RBP = old value, RSP += 8
    ret                 ; return from main
```


# RIP 

```
; Initial: RIP = 0x400500
mov rax, 10      ; RIP → 0x400500 (executing this)
                 ; After execution: RIP = 0x400507

mov rbx, 20      ; RIP → 0x400507 (executing this)  
                 ; After execution: RIP = 0x40050E

add rax, rbx     ; RIP → 0x40050E (executing this)
                 ; After execution: RIP = 0x400510

push rax         ; RIP → 0x400510 (executing this)
                 ; After execution: RIP = 0x400511

call function    ; RIP → 0x400511 (executing this)
                 ; After execution: RIP = function_address
```



**No!** RBP နဲ့ RSP က **ဒီ code memory addresses (0x400500, 0x400507, စသည်) မှာ မရှိပါဘူး**။

ဒီမှာရှင်းပြပါမယ်:

## Memory Segments (Memory အပိုင်းအခြားများ)

### **1. Code Segment (Text Segment)**
```
Address Range: 0x400000 - 0x401000 (example)
Contains: Instructions (mov, cmp, jmp, call, etc.)
What's there: သင့်ရဲ့ program code တွေ
Example: 0x400500, 0x400507, 0x40050B - ဒါတွေက INSTRUCTIONS တွေရှိတဲ့နေရာ
```

### **2. Stack Segment** 
```
Address Range: 0x7fffffffe000 - 0x7ffffffff000 (example) 
Contains: Stack data (local variables, return addresses, etc.)
What's there: RBP and RSP point to THIS region
Example: RBP = 0x7fffffffe000, RSP = 0x7fffffffdff0
```

## ဘာကြောင့် မတူကြတာလဲ?

### **RIP** vs **RBP/RSP**

| Register | Points to | Purpose |
|----------|-----------|---------|
| **RIP** | **Code Segment** (0x400500, ...) | Next instruction to execute |
| **RBP/RSP** | **Stack Segment** (0x7fffffffe000, ...) | Stack frame management |

## Real Memory Layout (လက်တွေ့ Memory ဖြန့်ကျက်ပုံ)

```
HIGH MEMORY
0x7ffffffff000     | Stack Top
0x7fffffffe000     | RBP points somewhere here  ← RBP/RSP ARE HERE!
0x7fffffffdff0     | RSP points somewhere here
...                | Stack grows downward
0x7fffffffd000     | Stack Bottom

... (other memory segments)

0x401000           | Code Segment End
0x400530           | end: add rax, rbx         ← RIP GOES HERE!
0x400520           | greater: mov rbx, 1       ← RIP GOES HERE!  
0x400500           | main: mov rax, 10         ← RIP GOES HERE!
0x400000           | Code Segment Start
LOW MEMORY
```

## Execution Example (RIP, RBP, RSP တွေဘယ်လိုအလုပ်လုပ်လဲ)

```asm
; Code Segment (RIP moves here)
0x400500        mov rax, 10          ; RIP = 0x400500
0x400507        cmp rax, 5           ; RIP = 0x400507  
0x40050B        jg greater           ; RIP = 0x40050B

; Stack Segment (RBP/RSP point here)
; Assume: RBP = 0x7fffffffe000, RSP = 0x7fffffffdff0
;
; Memory at stack:
; 0x7fffffffe000     [saved RBP]     ← RBP points here
; 0x7fffffffdff8     [return address]
; 0x7fffffffdff0     [local vars]    ← RSP points here
```

## Function Call မှာ ဘယ်လိုဆက်စပ်လဲ?

```asm
; Code Segment
0x400500        call my_function     ; RIP = 0x400500

; What happens:
; 1. RIP က my_function ကိုသွားမယ် (e.g., 0x400600)
; 2. RSP က 8 bytes ကျသွားမယ် (RSP = RSP - 8)  
; 3. Return address (0x400505) က stack မှာသိမ်းမယ်

; Stack changes:
; Before call: RSP = 0x7fffffffdff8
; After call:  RSP = 0x7fffffffdff0
; Memory at 0x7fffffffdff0: 0x400505 (return address)
```

## GDB မှာ ကြည့်ကြရအောင်

```gdb
(gdb) info registers
rax            0x0                 0
rbx            0x0                 0  
rip            0x400500            0x400500 <main>    ← CODE SEGMENT
rbp            0x7fffffffe000      0x7fffffffe000     ← STACK SEGMENT
rsp            0x7fffffffdff0      0x7fffffffdff0     ← STACK SEGMENT

(gdb) x/i $rip
=> 0x400500 <main>: mov rax, 10     ← RIP points to CODE

(gdb) x/gx $rsp  
0x7fffffffdff0: 0x0000000000400505  ← RSP points to STACK (return address)
```

## Summary (အနှစ်ချုပ်)

- **RIP** က **code segment** (0x400500, 0x400507, ...) ကိုညွှန်ပြတယ်
- **RBP/RSP** က **stack segment** (0x7fffffffe000, ...) ကိုညွှန်ပြတယ်  
- **နှစ်ခုလုံးက memory addresses ပဲ**၊ ဒါပေမယ့် **မတူညီတဲ့ memory regions** မှာရှိတယ်
- Program တစ်ခုမှာ **code တွေ run ဖို့** RIP က code segment မှာရွေ့လျားနေပြီး၊ **data သိမ်းဖို့** RBP/RSP က stack segment ကိုညွှန်ပြနေတယ်

ဒါကြောင့် "RBP နဲ့ RSP က 0x400500 မှာရှိလား" မေးရင် **မရှိပါဘူး**။ သူတို့က stack segment မှာရှိပြီး RIP ကသာ code segment မှာရှိပါတယ်။


---
I'll show you a concrete example of how the stack changes during function calls with register values. Let me create a simple C program and show you the assembly with stack visualization.

## Example C Program
```c
#include <stdio.h>

int add(int a, int b) {
    int sum = a + b;
    return sum;
}

int main() {
    int x = 5;
    int y = 10;
    int result = add(x, y);
    return 0;
}
```

## Step-by-Step Stack Visualization (x86-64)

Let's assume these memory addresses (simplified):
- Stack grows **downward** (higher addresses → lower addresses)
- Base pointer (RBP) and stack pointer (RSP) are 64-bit registers
- Each cell is 8 bytes (64 bits)

### **Step 1: Initial State - Before main() executes**
```
Registers:
RIP = 0x400500 (main function start)
RBP = 0x7fffffffe520 (some initial value)
RSP = 0x7fffffffe520 (stack is empty at this level)

Memory (Stack):
Address       Contents
0x7fffffffe520  [previous frame]
0x7fffffffe518  [...]
0x7fffffffe510  [...]
...
```

### **Step 2: main() Prologue - Setting up stack frame**
```assembly
push rbp            ; Save old base pointer
mov rbp, rsp        ; Set new base pointer
sub rsp, 32         ; Allocate space for locals
```

```
After "push rbp":
RIP = 0x400501
RBP = 0x7fffffffe520 (unchanged)
RSP = 0x7fffffffe518 (decreased by 8)

Stack:
0x7fffffffe520  [old RBP value will go here later]
0x7fffffffe518  [old RBP = 0x7fffffffe550] ← RSP points here
0x7fffffffe510  [...]

After "mov rbp, rsp" and "sub rsp, 32":
RIP = 0x400505
RBP = 0x7fffffffe518 (now points to saved RBP)
RSP = 0x7fffffffe4f8 (32 bytes lower)
```

### **Step 3: main() Local Variables**
```assembly
mov DWORD PTR [rbp-4], 5    ; x = 5
mov DWORD PTR [rbp-8], 10   ; y = 10
```

```
Stack Layout (main's frame):
Address       Contents                Notes
0x7fffffffe518  [saved RBP]           ← RBP points here
0x7fffffffe514  [???]
0x7fffffffe510  [x = 5]               (rbp-8)
0x7fffffffe50c  [y = 10]              (rbp-12)
0x7fffffffe508  [unused]
0x7fffffffe504  [unused]
0x7fffffffe500  [unused]
0x7fffffffe4fc  [unused]
0x7fffffffe4f8  [???]                 ← RSP points here
```

### **Step 4: Preparing to Call add()**
```assembly
mov edx, DWORD PTR [rbp-8]  ; Load y
mov eax, DWORD PTR [rbp-4]  ; Load x
mov esi, edx                ; Second arg (b = y)
mov edi, eax                ; First arg (a = x)
call add                    ; RIP jumps to add
```

```
Before call:
RIP = 0x40051a (address of 'call' instruction)
RSP = 0x7fffffffe4f8

During 'call add':
1. Push return address (next instruction after call)
2. RIP jumps to add() function

After call:
RIP = 0x400400 (start of add() function)
RSP = 0x7fffffffe4f0 (decreased by 8 for return address)

Stack now:
0x7fffffffe4f8  [return address = 0x40051f]
0x7fffffffe4f0  [???]                 ← RSP points here
...
0x7fffffffe518  [saved RBP]           ← RBP still points here
```

### **Step 5: add() Function Prologue**
```assembly
push rbp
mov rbp, rsp
sub rsp, 16
```

```
After add() prologue:
RIP = 0x400405
RBP = 0x7fffffffe4f0 (new frame pointer)
RSP = 0x7fffffffe4e0

Stack (add's frame):
0x7fffffffe4f8  [return address]
0x7fffffffe4f0  [saved RBP = 0x7fffffffe518] ← RBP points here
0x7fffffffe4e8  [local variable space]
0x7fffffffe4e0  [???]                 ← RSP points here

Arguments are in registers (not on stack for x86-64 System V):
EDI = 5 (a), ESI = 10 (b)
```

### **Step 6: add() Execution**
```assembly
mov DWORD PTR [rbp-4], edi  ; Store a
mov DWORD PTR [rbp-8], esi  ; Store b
mov edx, DWORD PTR [rbp-4]  ; Load a
mov eax, DWORD PTR [rbp-8]  ; Load b
add eax, edx                ; sum = a + b
mov DWORD PTR [rbp-12], eax ; Store sum
```

```
Stack (add's frame with locals):
Address       Contents
0x7fffffffe4f8  [return address]
0x7fffffffe4f0  [saved RBP]
0x7fffffffe4ec  [a = 5]      (rbp-4)
0x7fffffffe4e8  [b = 10]     (rbp-8)
0x7fffffffe4e4  [sum = 15]   (rbp-12)
0x7fffffffe4e0  [unused]     ← RSP
```

### **Step 7: add() Epilogue - Returning**
```assembly
mov eax, DWORD PTR [rbp-12]  ; Return value in eax
leave                       ; mov rsp, rbp; pop rbp
ret                         ; pop return address, jump to it
```

```
After "leave":
RSP = 0x7fffffffe4f8 (restored to before add's prologue)
RBP = 0x7fffffffe518 (restored to main's frame)

After "ret":
RIP = 0x40051f (back in main, after call)
RSP = 0x7fffffffe4f0 (popped return address)
```

### **Step 8: Back in main() - After add() returns**
```assembly
mov DWORD PTR [rbp-12], eax  ; result = add(x, y)
```

```
Stack (main's frame):
0x7fffffffe518  [saved RBP]
0x7fffffffe514  [result = 15]  (rbp-12)
0x7fffffffe510  [x = 5]
0x7fffffffe50c  [y = 10]
0x7fffffffe508  [unused]
0x7fffffffe4f8  [???]         ← RSP points here
```

### **Step 9: main() Epilogue**
```assembly
mov eax, 0      ; return 0
leave          ; restore RSP and RBP
ret           ; return to caller
```

```
Final state:
RIP = back to libc_start_main or OS
RBP = restored to previous frame
RSP = restored to original position
```

## Key Points:
1. **RBP (Base Pointer)**: Points to the current stack frame
2. **RSP (Stack Pointer)**: Points to the top of the stack
3. **RIP (Instruction Pointer)**: Points to next instruction
4. **Stack grows downward**: Higher addresses → lower addresses
5. **call instruction**: Pushes return address, jumps to function
6. **leave instruction**: `mov rsp, rbp; pop rbp` (cleans up frame)
7. **ret instruction**: Pops return address into RIP

Would you like me to explain any specific part in more detail or show a different architecture (like 32-bit x86)?