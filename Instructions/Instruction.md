



basic syntax
```c
label:    instruction    operand1, operand2    ; comment
```

```c
start:    mov eax, 42     ; EAX = 42
          add ebx, eax    ; EBX = EBX + EAX
          jmp start       ; jump back to start
```


##### 3 Types of operand

| Type          | ရှင်းလင်းချက်            | ဥပမာ                |
| ------------- | ------------------------ | ------------------- |
| **Register**  | CPU register ထဲက တန်ဖိုး | `mov eax, ebx`      |
| **Immediate** | တန်ဖိုး တိုက်ရိုက်       | `mov eax, 42`       |
| **Memory**    | RAM address က တန်ဖိုး    | `mov eax, [0x1000]` |

#### Basic 10 instructions

##### 1. MOV (Move) 

```asm
mov destination, source    ; destination = source
```

examples
```asm
; Register to register
mov eax, ebx        ; EAX = EBX

; Immediate to register
mov eax, 42         ; EAX = 42

; Memory to register
mov eax, [0x1000]   ; EAX = value at address 0x1000

; Register to memory
mov [0x1000], eax   ; address 0x1000 = EAX

; 8-bit, 16-bit, 32-bit, 64-bit
mov al, 0xFF        ; 8-bit
mov ax, 0xFFFF      ; 16-bit
mov eax, 0xFFFFFFFF ; 32-bit
mov rax, 0xFFFFFFFFFFFFFFFF ; 64-bit
```

လေ့ကျင့်ခန်း ၃.၁
```asm
; ဒီ code ပြီးရင် EAX နဲ့ EBX က ဘာဖြစ်မလဲ
mov eax, 10
mov ebx, eax
mov eax, 20
```
(အဖြေ: EAX=20, EBX=10)

---

##### 2. ADD (Addition)

```asm
add destination, source    ; destination = destination + source
```

ဥပမာများ
```asm
mov eax, 10
add eax, 5          ; EAX = 15

mov eax, 10
mov ebx, 20
add eax, ebx        ; EAX = 30

add al, 1           ; AL = AL + 1
add [0x1000], 5     ; memory တန်ဖိုး +5
```

Flag သက်ရောက်မှု - CF, ZF, SF, OF, AF, PF

```asm
; ဥပမာ - flag တွေကြည့်
mov al, 0xFF
add al, 0x01        ; AL=0, CF=1 (carry), ZF=1 (zero)
```

---

##### 3. SUB (Subtraction) 

```asm
sub destination, source    ; destination = destination - source
```

examples
```asm
mov eax, 20
sub eax, 5          ; EAX = 15

mov eax, 10
sub eax, 20         ; EAX = -10 (0xFFFFFFF6), SF=1

sub al, 1           ; AL = AL - 1
sub [0x1000], 5     ; memory တန်ဖိုး -5
```

Flag သက်ရောက်မှု - CF, ZF, SF, OF, AF, PF

```asm
; ဥပမာ - borrow detection
mov al, 0x00
sub al, 0x01        ; AL=0xFF, CF=1 (borrow)
```

---

##### 4. INC / DEC (Increment / Decrement) 

```asm
inc destination    ; destination = destination + 1
dec destination    ; destination = destination - 1
```


```asm
mov eax, 10
inc eax             ; EAX = 11
dec eax             ; EAX = 10

inc al              ; AL = AL + 1
dec [0x1000]        ; memory ကို 1 လျှော့

; loop counter အတွက် အသုံးများ
mov ecx, 10
loop_start:
    ; do something
    dec ecx
    jnz loop_start  ; ECX သုညမဖြစ်ရင် ပြန်ခုန်
```

Flag သက်ရောက်မှု - ZF, SF, OF, AF, PF (CF မပြောင်း)

---

##### 5. CMP (Compare) 

```asm
cmp operand1, operand2    ; operand1 - operand2 (result not stored)
```

အလုပ်လုပ်ပုံ - နှုတ်တွက်တယ်၊ ရလဒ်ကို မသိမ်းဘူး Flag တွေကိုပဲ ပြောင်းတယ်

examples:
```asm
cmp eax, ebx        ; compare EAX and EBX

; result flags based on EAX - EBX
; EAX == EBX → ZF = 1
; EAX < EBX (signed) → SF != OF
; EAX > EBX (signed) → ZF=0 and SF=OF
; EAX < EBX (unsigned) → CF = 1
; EAX > EBX (unsigned) → CF = 0 and ZF = 0
```


```asm
mov eax, 10
mov ebx, 20
cmp eax, ebx        ; 10 - 20 = -10 → SF=1, ZF=0, CF=1
; EAX က EBX ထက် ငယ်တယ်
```

---

##### 6. JMP (Unconditional Jump) 

```asm
jmp label    ; jump to label
```


```asm
start:
    mov eax, 1
    jmp start       ; infinite loop (အဆုံးမရှိ)

; forward jump
    jmp end_program
    mov eax, 0      ; ဒီ code က ဘယ်တော့မှ run မှာမဟုတ်ဘူး
end_program:
    mov eax, 1
```

---

##### 7. Conditional Jumps 

ZF (Zero Flag) ကိုစစ်တဲ့ Jumps

| Instruction | Condition | ဘယ်အချိန်ခုန်မလဲ |
|-------------|-----------|-------------------|
| **JE / JZ** | ZF = 1 | တူရင် (equal / zero) |
| **JNE / JNZ** | ZF = 0 | မတူရင် (not equal / not zero) |

```asm
cmp eax, ebx
je  equal_label     ; if EAX == EBX, jump
jne not_equal_label ; if EAX != EBX, jump
```

CF (Carry Flag) ကိုစစ်တဲ့ Jumps (Unsigned)

| Instruction | Condition | ဘယ်အချိန်ခုန်မလဲ |
|-------------|-----------|-------------------|
| **JA / JNBE** | CF=0 and ZF=0 | အကြီးဆုံး (unsigned >) |
| **JAE / JNB** | CF=0 | အကြီးး သို့ တူ (unsigned >=) |
| **JB / JNAE** | CF=1 | အငယ် (unsigned <) |
| **JBE / JNA** | CF=1 or ZF=1 | အငယ် သို့ တူ (unsigned <=) |

```asm
cmp eax, ebx
ja  above_label     ; if EAX > EBX (unsigned)
jb  below_label     ; if EAX < EBX (unsigned)
```

SF နဲ့ OF ကိုစစ်တဲ့ Jumps (Signed)

| Instruction | Condition | ဘယ်အချိန်ခုန်မလဲ |
|-------------|-----------|-------------------|
| **JG / JNLE** | ZF=0 and SF=OF | အကြီး (signed >) |
| **JGE / JNL** | SF=OF | အကြီး သို့ တူ (signed >=) |
| **JL / JNGE** | SF != OF | အငယ် (signed <) |
| **JLE / JNG** | ZF=1 or SF!=OF | အငယ် သို့ တူ (signed <=) |

```asm
cmp eax, ebx
jg  greater_label   ; if EAX > EBX (signed)
jl  less_label      ; if EAX < EBX (signed)
```

အခြား Flag Jumps

| Instruction | Condition |
|-------------|-----------|
| **JC** | CF = 1 |
| **JNC** | CF = 0 |
| **JO** | OF = 1 |
| **JNO** | OF = 0 |
| **JS** | SF = 1 |
| **JNS** | SF = 0 |
| **JP / JPE** | PF = 1 |
| **JNP / JPO** | PF = 0 |

---

##### 8. TEST - Bitwise AND (checking Flag)

```asm
test operand1, operand2    ; operand1 AND operand2 (result not stored)
```

အလုပ်လုပ်ပုံ - AND တွက်တယ်၊ ရလဒ်ကို မသိမ်းဘူး Flag တွေကိုပဲ ပြောင်းတယ်

```asm
test eax, eax       ; EAX သုညလားစစ်ဖို့
jz  zero_label      ; if EAX == 0

test al, 0x01       ; AL ရဲ့ bit 0 (LSB) က 1 လားစစ်
jnz odd_label       ; if LSB = 1 → odd number
```

---

##### 9. XOR - Bitwise Exclusive OR

```asm
xor destination, source    ; destination = destination XOR source
```

အသုံးဝင်ပုံ - Register ကို Zero လုပ်ရန် (MOV ထက် မြန်တယ်)

```asm
xor eax, eax        ; EAX = 0 (fastest way to zero a register)
xor ebx, ebx        ; EBX = 0
xor rcx, rcx        ; RCX = 0
```

Flag သက်ရောက်မှု - ZF=1 (if result zero), CF=0, OF=0

---

##### 10. NOP - No Operation (do nothing)
```asm
nop     ; do nothing, just take time
```

ဘာအတွက်သုံးလဲ
- Code alignment အတွက်
- Patching/Modifying code အတွက် (placeholders)
- Timing/delay အတွက်

```asm
; သုံးစွဲပုံ
nop                 ; 1 byte instruction (0x90)
nop
nop                 ; 3 bytes padding
```


##### Instruction Set Reference Table

| Instruction | အလုပ်                    | ဥပမာ            | Flag Effect                          |
| ----------- | ------------------------ | --------------- | ------------------------------------ |
| **MOV**     | ကူးယူ                    | `mov eax, 5`    | None                                 |
| **ADD**     | ပေါင်း                   | `add eax, ebx`  | CF, ZF, SF, OF, AF, PF               |
| **SUB**     | နှုတ်                    | `sub eax, 5`    | CF, ZF, SF, OF, AF, PF               |
| **INC**     | 1 တိုး                   | `inc eax`       | ZF, SF, OF, AF, PF (no CF)           |
| **DEC**     | 1 လျှော့                 | `dec eax`       | ZF, SF, OF, AF, PF (no CF)           |
| **CMP**     | နှိုင်းယှဉ်              | `cmp eax, ebx`  | CF, ZF, SF, OF, AF, PF               |
| **TEST**    | AND စစ်                  | `test eax, eax` | CF=0, ZF, SF, OF=0, PF, AF undefined |
| **XOR**     | Exclusive OR             | `xor eax, eax`  | ZF=1 if zero, CF=0, OF=0             |
| **JMP**     | unconditional jump       | `jmp label`     | None                                 |
| **JE/JZ**   | jump if equal/zero       | `je label`      | None                                 |
| **JNE/JNZ** | jump if not equal        | `jne label`     | None                                 |
| **JG**      | jump if greater (signed) | `jg label`      | None                                 |
| **JL**      | jump if less (signed)    | `jl label`      | None                                 |
| **JA**      | jump if above (unsigned) | `ja label`      | None                                 |
| **JB**      | jump if below (unsigned) | `jb label`      | None                                 |
| **NOP**     | nothing                  | `nop`           | None                                 |


##### Example 1 - Simple Addition Program

```asm
; add_two_numbers.asm
section .text
global _start

_start:
    ; Initialize values
    mov eax, 10     ; EAX = 10
    mov ebx, 20     ; EBX = 20
    
    ; Add them
    add eax, ebx    ; EAX = 30
    
    ; Exit program (Linux syscall)
    mov eax, 60     ; sys_exit
    xor edi, edi    ; status = 0
    syscall
```

**Compile & Run**
```bash
nasm -f elf64 add_two_numbers.asm -o add.o
ld add.o -o add
./add
echo $?    # should print 0
```

---

##### Example 2 - Compare Two Numbers

```asm
; compare.asm - find which is bigger
section .text
global _start

_start:
    mov eax, 15     ; first number
    mov ebx, 10     ; second number
    
    cmp eax, ebx    ; compare
    jg  eax_bigger  ; if EAX > EBX
    jl  ebx_bigger  ; if EAX < EBX
    ; if equal
    mov edi, 0      ; return 0 for equal
    jmp exit
    
eax_bigger:
    mov edi, 1      ; return 1 if EAX bigger
    jmp exit
    
ebx_bigger:
    mov edi, 2      ; return 2 if EBX bigger
    
exit:
    mov eax, 60     ; sys_exit
    syscall
```

---

##### Example 3 - Loop (1 to 10 sum)

```asm
; sum_loop.asm - sum 1 to 10
section .text
global _start

_start:
    xor eax, eax    ; sum = 0
    mov ecx, 10     ; counter = 10
    
loop_start:
    add eax, ecx    ; sum += counter
    dec ecx         ; counter--
    jnz loop_start  ; if counter != 0, loop again
    
    ; EAX now contains 55 (1+2+...+10)
    
    mov edi, eax    ; return sum as exit code
    mov eax, 60     ; sys_exit
    syscall
```

---

##### Example 4 - Check Even/Odd Number

```asm
; even_odd.asm
section .text
global _start

_start:
    mov eax, 7      ; number to check
    test eax, 1     ; check LSB (bit 0)
    jz  even_label  ; if LSB = 0 → even
    jmp odd_label   ; if LSB = 1 → odd
    
even_label:
    mov edi, 0      ; return 0 for even
    jmp exit
    
odd_label:
    mov edi, 1      ; return 1 for odd
    
exit:
    mov eax, 60
    syscall
```

---

##### Example 5 - Find Maximum in Array

```asm
; max_array.asm - find maximum value in array
section .data
    arr dd 5, 12, 8, 20, 3, 15, 7    ; array of 7 numbers
    arr_len equ 7

section .text
global _start

_start:
    mov esi, arr        ; ESI points to array start
    mov ecx, arr_len    ; counter = 7
    mov eax, [esi]      ; max = arr[0]
    
    dec ecx             ; skip first element (already in EAX)
    add esi, 4          ; move to next element (4 bytes each)
    
loop_start:
    cmp ecx, 0
    je  done            ; if counter == 0, done
    
    mov ebx, [esi]      ; ebx = current value
    cmp ebx, eax        ; compare with max
    jle next            ; if <=, skip
    
    mov eax, ebx        ; new max found
    
next:
    add esi, 4          ; move to next element
    dec ecx             ; counter--
    jmp loop_start
    
done:
    ; EAX contains max value (20)
    mov edi, eax
    mov eax, 60
    syscall
```



---
