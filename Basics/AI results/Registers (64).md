
**x86-64 Architecture Registers အပြည့်အစုံနှင့် အသုံးပြုပုံများ**

## 1. **General Purpose Registers** (16-bit, 32-bit, 64-bit versions ရှိ)

### **RAX (Accumulator Register)**
- **အဓိကအသုံး**: Arithmetic operations, function return values
- **Sizes**: AL (8-bit), AX (16-bit), EAX (32-bit), RAX (64-bit)
- **ဥပမာ**: 
  ```asm
  mov rax, 5    ; store 5 in RAX
  add rax, 10   ; RAX = 15
  ```

### **RBX (Base Register)**
- **အဓိကအသုံး**: Base pointer for memory addressing
- **Sizes**: BL, BX, EBX, RBX
- **ဥပမာ**:
  ```asm
  mov rbx, [array]  ; array base address
  mov rax, [rbx+8]  ; access array[2]
  ```

### **RCX (Counter Register)**
- **အဓိကအသုံး**: Loop counters, string operations
- **Sizes**: CL, CX, ECX, RCX
- **ဥပမာ**:
  ```asm
  mov rcx, 10
  loop_start:
      ; do something
      dec rcx
      jnz loop_start
  ```

### **RDX (Data Register)**
- **အဓိကအသုံး**: I/O operations, multiply/divide operations
- **Sizes**: DL, DX, EDX, RDX
- **ဥပမာ**:
  ```asm
  mul rdx    ; multiply with RDX
  ```

### **RSI (Source Index)**
- **အဓိကအသုံး**: String/memory operations (source), function arguments (2nd)
- **Sizes**: SIL, SI, ESI, RSI
- **ဥပမာ**:
  ```asm
  mov rsi, source_string
  mov rdi, destination_string
  rep movsb  ; copy string
  ```

### **RDI (Destination Index)**
- **အဓိကအသုံး**: String/memory operations (destination), function arguments (1st)
- **Sizes**: DIL, DI, EDI, RDI
- **ဥပမာ**:
  ```asm
  mov rdi, buffer  ; destination for input
  ```

### **RBP (Base Pointer)**
- **အဓိကအသုံး**: Stack frame base pointer
- **Sizes**: BPL, BP, EBP, RBP
- **ဥပမာ**:
  ```asm
  push rbp
  mov rbp, rsp  ; create stack frame
  ```

### **RSP (Stack Pointer)**
- **အဓိကအသုံး**: Current stack position
- **Sizes**: SPL, SP, ESP, RSP
- **ဥပမာ**:
  ```asm
  push rax  ; RSP decreases by 8
  pop rbx   ; RSP increases by 8
  ```

## 2. **Extended Registers** (64-bit only)

### **R8 - R15**
- **အဓိကအသုံး**: Additional general purpose registers
- **Sizes**: R8B-R15B (8-bit), R8W-R15W (16-bit), R8D-R15D (32-bit), R8-R15 (64-bit)
- **ဥပမာ**:
  ```asm
  mov r8, 100
  mov r9, 200
  add r10, r8, r9  ; R10 = 300
  ```

## 3. **Special Purpose Registers**

### **RIP (Instruction Pointer)**
- **အဓိကအသုံး**: Next instruction to execute
- **အရေးကြီးဆုံး** register တစ်ခု
- **ဥပမာ**:
  ```asm
  call function  ; RIP changes to function address
  ret           ; RIP returns to caller
  ```

### **RFLAGS (Flags Register)**
- **အဓိကအသုံး**: Status and control flags
- **အရေးကြီး flags များ**:
  - **ZF (Zero Flag)**: Result was zero
  - **CF (Carry Flag)**: Arithmetic carry/borrow
  - **SF (Sign Flag)**: Result was negative
  - **OF (Overflow Flag)**: Arithmetic overflow

## 4. **Segment Registers** (16-bit)

### **CS (Code Segment)**
- **အဓိကအသုံး**: Code segment (instructions)

### **DS (Data Segment)**
- **အဓိကအသုံး**: Data segment

### **SS (Stack Segment)**
- **အဓိကအသုံး**: Stack segment

### **ES, FS, GS**
- **အဓိကအသုံး**: Extra data segments
- **FS/GS**: Thread-local storage, OS-specific uses

## 5. **System Registers**

### **CR0-CR4 (Control Registers)**
- **အဓိကအသုံး**: System control (paging, protection, etc.)

### **GDTR, LDTR, IDTR**
- **အဓိကအသုံး**: Descriptor table registers

## Register Usage in Function Calls (System V AMD64 ABI)

### **Function Arguments**:
1. **RDI** - 1st argument
2. **RSI** - 2nd argument  
3. **RDX** - 3rd argument
4. **RCX** - 4th argument
5. **R8** - 5th argument
6. **R9** - 6th argument
7. **Stack** - Additional arguments

### **Return Value**:
- **RAX** - Integer return value
- **RDX:RAX** - 128-bit return value
- **XMM0** - Floating point return value

### **Caller-saved** (caller must save):
- RAX, RCX, RDX, RSI, RDI, R8-R11

### **Callee-saved** (callee must save):
- RBX, RBP, R12-R15

## Practical Example

```asm
; Function: add_numbers(a, b)
; RDI = a, RSI = b
add_numbers:
    mov rax, rdi    ; RAX = a
    add rax, rsi    ; RAX = a + b
    ret             ; Return result in RAX

; Calling the function
main:
    mov rdi, 10     ; 1st argument = 10
    mov rsi, 20     ; 2nd argument = 20
    call add_numbers ; Result in RAX = 30
```

**အတိုချုပ်**: x86-64 architecture မှာ registers တွေက data storage, memory addressing, control flow အတွက်အဓိကသုံးပြီး programming language တိုင်းနီးပါးက eventually assembly instructions အဖြစ်ပြောင်းတဲ့အခါမှာ registers တွေကိုသုံးကြတယ်။