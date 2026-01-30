x86 (32-bit) Architecture Registers အပြည့်အစုံ

## 1. **General Purpose Registers** (8-bit, 16-bit, 32-bit versions ရှိ)

### **EAX (Accumulator Register)**
- **အဓိကအသုံး**: Arithmetic operations, function return values
- **Sizes**: AL (8-bit), AX (16-bit), EAX (32-bit)
- **Structure**:
  ```
  EAX (32-bit): [XXXXXXXX][XXXXXXXX][XXXXXXXX][XXXXXXXX]
                  AH (8-bit)    AL (8-bit)
                 AX (16-bit)
  ```
- **ဥပမာ**: 
  ```asm
  mov eax, 5      ; store 5 in EAX
  add eax, 10     ; EAX = 15
  ```

### **EBX (Base Register)**
- **အဓိကအသုံး**: Base pointer for memory addressing
- **Sizes**: BL, BX, EBX
- **ဥပမာ**:
  ```asm
  mov ebx, offset array  ; array base address
  mov eax, [ebx+4]       ; access array[1]
  ```

### **ECX (Counter Register)**
- **အဓိကအသုံး**: Loop counters, string operations
- **Sizes**: CL, CX, ECX
- **ဥပမာ**:
  ```asm
  mov ecx, 10
  loop_start:
      ; do something
      dec ecx
      jnz loop_start
  ```

### **EDX (Data Register)**
- **အဓိကအသုံး**: I/O operations, multiply/divide operations
- **Sizes**: DL, DX, EDX
- **ဥပမာ**:
  ```asm
  mov eax, 5
  mov edx, 0      ; clear EDX for division
  mov ecx, 2
  div ecx         ; EAX = 2, EDX = 1 (remainder)
  ```

### **ESI (Source Index)**
- **အဓိကအသုံး**: String/memory operations (source)
- **Sizes**: SI, ESI
- **ဥပမာ**:
  ```asm
  mov esi, source_string
  mov edi, destination_string
  mov ecx, length
  rep movsb       ; copy string
  ```

### **EDI (Destination Index)**
- **အဓိကအသုံး**: String/memory operations (destination)
- **Sizes**: DI, EDI
- **ဥပမာ**:
  ```asm
  mov edi, buffer    ; destination for input
  ```

### **EBP (Base Pointer)**
- **အဓိကအသုံး**: Stack frame base pointer
- **Sizes**: BP, EBP
- **ဥပမာ**:
  ```asm
  push ebp
  mov ebp, esp      ; create stack frame
  sub esp, 16       ; allocate local variables
  ```

### **ESP (Stack Pointer)**
- **အဓိကအသုံး**: Current stack position
- **Sizes**: SP, ESP
- **ဥပမာ**:
  ```asm
  push eax      ; ESP decreases by 4
  pop ebx       ; ESP increases by 4
  ```

## 2. **Special Purpose Registers**

### **EIP (Instruction Pointer)**
- **အဓိကအသုံး**: Next instruction to execute
- **Modify လုပ်လို့မရ** (directly)
- **ဥပမာ**:
  ```asm
  call function  ; EIP changes to function address
  ret           ; EIP returns to caller
  jmp label     ; EIP jumps to label
  ```

### **EFLAGS (Flags Register)**
- **အဓိကအသုံး**: Status and control flags (32-bit)
- **အရေးကြီး flags များ**:
  - **CF (Carry Flag)**: Arithmetic carry/borrow
  - **PF (Parity Flag)**: Even parity in result
  - **AF (Auxiliary Flag)**: BCD operations
  - **ZF (Zero Flag)**: Result was zero
  - **SF (Sign Flag)**: Result was negative
  - **TF (Trap Flag)**: Single-step mode
  - **IF (Interrupt Flag)**: Interrupts enabled
  - **DF (Direction Flag)**: String direction (0=forward, 1=backward)
  - **OF (Overflow Flag)**: Arithmetic overflow

## 3. **Segment Registers** (16-bit)

### **CS (Code Segment)**
- **အဓိကအသုံး**: Code segment (instructions)

### **DS (Data Segment)**
- **အဓိကအသုံး**: Data segment

### **SS (Stack Segment)**
- **အဓိကအသုံး**: Stack segment

### **ES (Extra Segment)**
- **အဓိကအသုံး**: Extra data segment

### **FS, GS**
- **အဓိကအသုံး**: Additional extra segments
- **OS-specific uses**: Thread-local storage in Windows/Linux

## 4. **System Registers**

### **CR0-CR4 (Control Registers)**
- **CR0**: Paging, protection, monitoring
- **CR1**: Reserved
- **CR2**: Page fault linear address
- **CR3**: Page directory base
- **CR4**: Extensions enable

### **GDTR, LDTR, IDTR**
- **အဓိကအသုံး**: Descriptor table registers

## 32-bit vs 64-bit Differences

| Feature | 32-bit (x86) | 64-bit (x86-64) |
|---------|-------------|-----------------|
| **General Registers** | 8 (EAX, EBX, ECX, EDX, ESI, EDI, EBP, ESP) | 16 (RAX-R15) |
| **Register Size** | 32-bit | 64-bit |
| **Function Arguments** | Stack | Registers (RDI, RSI, RDX, RCX, R8, R9) |
| **Return Value** | EAX | RAX |
| **Instruction Pointer** | EIP | RIP |
| **Flags Register** | EFLAGS | RFLAGS |

## Register Usage in Function Calls (cdecl convention)

### **Function Arguments**:
```asm
; Arguments passed on stack (right to left)
push arg3
push arg2  
push arg1
call function
add esp, 12    ; cleanup stack (3 args * 4 bytes)
```

### **Return Value**:
- **EAX** - Integer return value
- **EDX:EAX** - 64-bit return value

### **Caller-saved** (caller must save):
- EAX, ECX, EDX

### **Callee-saved** (callee must save):
- EBX, ESI, EDI, EBP

## Practical 32-bit Example

```asm
; Function: add_numbers(a, b)
; Stack: [ebp+8] = a, [ebp+12] = b
add_numbers:
    push ebp
    mov ebp, esp        ; setup stack frame
    
    mov eax, [ebp+8]    ; EAX = a
    add eax, [ebp+12]   ; EAX = a + b
    
    mov esp, ebp        ; cleanup stack
    pop ebp
    ret                 ; Return result in EAX

; Calling the function
main:
    push 20             ; 2nd argument = 20
    push 10             ; 1st argument = 10
    call add_numbers    ; Result in EAX = 30
    add esp, 8          ; cleanup stack
```

## Memory Addressing Modes (32-bit)

```asm
; Direct
mov eax, [0x8048000]    ; address directly

; Register indirect
mov eax, [ebx]          ; address in EBX

; Based indexed
mov eax, [ebx + esi]    ; EBX + ESI

; Based indexed with displacement
mov eax, [ebx + esi*4 + 16]  ; array access
```

## Segment:Offset Addressing (Real Mode)
```
Physical Address = (Segment × 16) + Offset
Example: CS:IP = 0x1000:0x0200 = 0x10200
```

**အတိုချုပ်**: 32-bit architecture မှာ registers 8ခုသာရှိပြီး 64-bit ထက်ပိုရှုပ်ထွေးတဲ့ memory addressing နဲ့ stack-based function calls တွေသုံးရတယ်။ ဒါပေမယ့် embedded systems, legacy applications တွေမှာ အခုထိအသုံးများဆဲဖြစ်တယ်။