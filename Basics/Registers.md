

Register ဆိုတာ **CPU (Central Processing Unit)** ထဲမှာရှိတဲ့ သိပ်မြန်တဲ့ မှတ်ဉာဏ်အလွန်သေးသေးလေး 
အချက်အလက်တွေ ခဏလောက် သိမ်းထားဖို့ သုံးတယ်
မြန်တယ် - RAM ထက်တောင် အဆပေါင်းများစွာ မြန်တယ်
သေးတယ် - တစ်လုံးချင်းစီက 8 bit, 16 bit, 32 bit, ဒါမှမဟုတ် 64 bit လောက်ပဲ သိမ်းနိုင်တယ် 
ဈေးကြီးတယ် - သိပ်မြန်လွန်းလို့ CPU ထဲမှာ အလုံး ၁၀-၂၀ လောက်ပဲ ထည့်ထားနိုင်တယ်
 x86-64 CPU မှာ အဓိက ၁၆ ခုရှိတယ်
CPU က RAM ထဲက data တွေကို တိုက်ရိုက် တွက်ချက်လို့ မရဘူး 
အဲဒီအတွက် CPU က RAM ထဲက data ကို Register ထဲ အရင်ကူးယူပြီးမှ တွက်ချက်တယ်



#### Register Structure

Register တစ်ခုဟာ Flip-Flop ခေါ်တဲ့ အီလက်ထရွန်းနစ် circuit အသေးစား အများအပြား စုစည်းထားတာဖြစ်တယ်
- တစ်ခုချင်းစီက **1 bit** (0 သို့မဟုတ် 1) ကို သိမ်းနိုင်တယ်    
- 8-bit register ဆိုရင် flip-flop 8 ခု တွဲထားတာ
- 64-bit register ဆိုရင် flip-flop 64 ခု တွဲထားတာ

```c
  bit 7   bit 6   bit 5   bit 4   bit 3   bit 2   bit 1   bit 0
+-------+-------+-------+-------+-------+-------+-------+-------+
|  FF7  |  FF6  |  FF5  |  FF4  |  FF3  |  FF2  |  FF1  |  FF0  |
+-------+-------+-------+-------+-------+-------+-------+-------+
   MSB                                    LSB
   
FF = Flip-Flop
MSB = Most Significant Bit (အကြီးဆုံးနေရာ)
LSB = Least Significant Bit (အငယ်ဆုံးနေရာ)
```



#### Register Internal Bus Connections

Register တစ်ခုမှာ အဓိက အစိတ်အပိုင်း 3 မျိုးရှိတယ်

| အစိတ်အပိုင်း      | အလုပ်                                                                                                                           |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Data Input Lines  | အပြင်က လာတဲ့ data ကို register ထဲထည့်တဲ့လိုင်း (ဘယ်နှစ် bit လက်ခံလဲဆိုတာ register အရွယ်ပေါ်မူတည်)                               |
| Data Output Lines | register ထဲက data ကို အပြင်ကို ထုတ်ပေးတဲ့လိုင်း                                                                                 |
| Control Lines     | ဘယ်အချိန်မှာ ထည့်မယ်၊ ဘယ်အချိန်မှာ ထုတ်မယ်၊ ဘယ်အချိန်မှာ သိမ်းမယ်ဆိုတဲ့ command ပေးတဲ့လိုင်း (Clock, Write Enable, Read Enable) |

```c
             Data Input (8 bits)
                  ↓↓↓↓↓↓↓↓
        ┌─────────────────────┐
        │                     │
Clock ──│     8-bit Register  │──→ Data Output (8 bits)
        │                     │
        └─────────────────────┘
              ↑
         Write Enable
```

#### Register Operation Cycle

##### Load (Write) -

1. Data input lines ပေါ်ကို တန်ဖိုးတွေ ရောက်လာ
2. **Write Enable** signal ကို 1 ပေး
3. Clock pulse တစ်ခု ရောက်လာတဲ့အခါ flip-flop တွေ အလုပ်လုပ်ပြီး data ကို သိမ်းတယ်
    
#####  Read (Store) 
1. **Read Enable** signal ကို 1 ပေး
2. Register ထဲက တန်ဖိုးတွေ data output lines ပေါ်ကို ထွက်လာ


----

#### Register Naming Convention

 8086 CPU (1978) ကစတယ်
AMD64 (2003) မှာ register အသစ် ၈ လုံးထပ်ထည့်တယ်။ နာမည်အသစ်မတီထွင်တော့ဘဲ R8 ကနေ R15 လို့ ရိုးရိုးရှင်းရှင်းခေါ်

|Register|မူလရည်ရွယ်ချက်|
|---|---|
|**AX** (Accumulator)|တွက်ချက်မှုရလဒ် accumulate လုပ်ရန်|
|**BX** (Base)|memory address base အဖြစ်သုံးရန်|
|**CX** (Counter)|loop ရေတွက်ရန်|
|**DX** (Data)|I/O port address, စားခြင်းရလဒ်အတွက်|


#### Register Size


8-bit registers (Smallest)

|Register|အမည်အပြည့်|အသုံး|
|---|---|---|
|AL|Accumulator Low|တွက်ချက်ရန်|
|BL|Base Low|memory address အတွက်|
|CL|Counter Low|loop ရေတွက်ရန်|
|DL|Data Low|data သိမ်းရန်|

##### AL (Accumulator Low) 

AL က AX register ရဲ့ အောက်ခြေ 8 bits (low byte) ပါ။

```
AX (16 bits)
┌─────────────────┬─────────────────┐
│      AH         │       AL        │
│  (High Byte)    │   (Low Byte)    │
│   bits 15-8     │    bits 7-0     │
└─────────────────┴─────────────────┘
```

Example - AX = `0x1234` ဆိုပါစို့
```
AH = 0x12 (18 decimal)
AL = 0x34 (52 decimal)
```

AL ကို ဘာတွေအတွက် အဓိကသုံးလဲ
1. ဂဏန်းသင်္ချာ - ပေါင်းခြင်း၊ နှုတ်ခြင်း (add al, bl)
2. ရွှေ့ခြင်း - data ကူးယူခြင်း (mov al, [address])
3. ပြန်ပို့ခြင်း - function ရဲ့အဖြေကို AL မှာ ပြန်ပို့တယ်

Example
```asm
mov al, 0x41    ; AL ထဲကို 'A' (ASCII 65) ထည့်
add al, 0x01    ; AL = 0x42 ('B')
sub al, 0x02    ; AL = 0x40 ('@')
```

---

##### BL (Base Low) 

BL က BX register ရဲ့ low byte ပါ။ BX က memory address တွေအတွက် အခြေခံ အနေနဲ့ အဓိကသုံးတယ်။

```
BX (16 bits)
┌─────────────────┬─────────────────┐
│      BH         │       BL        │
└─────────────────┴─────────────────┘
```

BL ကို ဘာအတွက်သုံးလဲ
1. Memory addressing - `[BX + constant]` ပုံစံနဲ့ array သွားရန်
2. Base pointer - data structure ရဲ့ အခြေခံနေရာ

Example
```asm
mov bx, 0x1000  ; BX ထဲကို address 0x1000 ထည့်
mov al, [bx]    ; RAM address 0x1000 မှာ ရှိတဲ့ value ကို AL ထဲထည့်
mov bl, 0x05    ; BL ကို 5 ထည့် (ဒါက BX ရဲ့ low byte ကိုပြောင်း)
```

> **သတိပြုရန်** - BL ကို ပြောင်းလိုက်ရင် BX ရဲ့ အနိမ့်ပိုင်းပဲပြောင်းတယ်။ BH ကတော့ အတိုင်းပဲရှိတယ်။

---

##### CL (Counter Low) 

CL က CX register ရဲ့ low byte ပါ။ CX က loop counter (အကြိမ်ရေရေတွက်သူ) ဖြစ်တယ်။

```
CX (16 bits)
┌─────────────────┬─────────────────┐
│      CH         │       CL        │
└─────────────────┴─────────────────┘
```

CL ကို ဘာအတွက်သုံးလဲ
1. Loop counter - `loop` instruction က CX ကိုသုံးတယ် (သို့မဟုတ် ECX, RCX)
2. Shift/Rotate counter - `shl al, cl` (CL က ဘယ်နှစ် bit ရွှေ့မလဲဆိုတာ သတ်မှတ်)

Example
```asm
mov cl, 0x04    ; CL = 4
mov al, 0x01    ; AL = 00000001
shl al, cl      ; AL ကို ဘယ်ဘက် 4 bit ရွှေ့ → AL = 00010000 (0x10)

; Loop ဥပမာ
mov cx, 0x0005  ; 5 ကြိမ်လှည့်မယ်
mov al, 0x00
my_loop:
    add al, 0x01    ; AL ကို 1 တိုး
    loop my_loop    ; CX ကို 1 လျှော့၊ CX≠0 ဆိုရင် my_loop ကိုပြန်ခုန်
; ဒီမှာ AL = 5
```

---

##### DL (Data Low) 

DL က DX register ရဲ့ low byte ပါ။ DX က general data သိမ်းရန် နဲ့ I/O port အတွက် အဓိကသုံးတယ်။

```
DX (16 bits)
┌─────────────────┬─────────────────┐
│      DH         │       DL        │
└─────────────────┴─────────────────┘
```

DL ကို ဘာအတွက်သုံးလဲ
1. I/O operations - `in al, dx` (DX က port number)
2. Division/Multiplication - 32-bit division မှာ DX:AX အတွဲသုံး
3. General data storage

Example
```asm
; I/O port ဥပမာ (DOS/BIOS မှာ)
mov dx, 0x3F8   ; COM1 serial port address
mov al, 0x41    ; 'A' ကို ပို့မယ်
out dx, al      ; port ကို data ထုတ်

; Division ဥပမာ (32-bit)
mov dx, 0x0000  ; high 16 bits
mov ax, 0x2710  ; low 16 bits (10000 decimal)
mov bx, 0x000A  ; 10 နဲ့စား
div bx          ; AX = quotient (1000), DX = remainder (0)
```

---

##### A, B, C, D  growth

##### 8-bit  (8080, Z80 - 1974)
```
A, B, C, D, E, H, L (သီးသန့် 8-bit registers)
```

##### 16-bit ခေတ် (8086 - 1978)
```
AX (A-high + A-low) = AH + AL
BX = BH + BL
CX = CH + CL
DX = DH + DL
(ထပ်တိုး: SI, DI, BP, SP)
```

##### 32-bit  (80386 - 1985)
```
EAX = Extended AX (32-bit)
EBX, ECX, EDX
(အပေါ်ဆုံး 16-bit က အသစ်၊ အောက် 16-bit က old AX)
```

##### 64-bit  (AMD64 - 2003)
```
RAX = Register AX (64-bit)
RBX, RCX, RDX
(ထပ်တိုး: R8-R15)
```


```
64-bit ခေတ် (RAX)
┌────────────────────────────────────────────────────────────────┐
│                         RAX (64 bits)                          │
├───────────────────────────────┬────────────────────────────────┤
│        Upper 32 bits          │          EAX (32 bits)          │
│        (bits 63-32)           │  ┌────────────────────────────┐  │
│                               │  │      Upper 16 bits         │  │
│                               │  │      of EAX (bits 31-16)   │  │
│                               │  │  ┌────────────┬───────────┐ │  │
│                               │  │  │    AH      │    AL     │ │  │
│                               │  │  │ (bits15-8) │ (bits7-0) │ │  │
│                               │  │  └────────────┴───────────┘ │  │
│                               │  │          AX (16 bits)        │  │
│                               │  └────────────────────────────┘  │
└───────────────────────────────┴────────────────────────────────┘
```


AH ,AL , AX , EAX , RAX တွေက ဘယ်လိုအတူတကွအလုပ်လုပ်လဲ?
 Overlapping Behavior


 RAX ထည့်ပြီး EAX, AX, AH, AL ကြည့်ခြင်း
```c
; RAX ထဲကို 64-bit တန်ဖိုးထည့်
mov rax, 0x1122334455667788

; ရလဒ်
; RAX = 0x1122334455667788
; EAX = 0x55667788    (အောက်ဆုံး 32 bits)
; AX  = 0x7788        (အောက်ဆုံး 16 bits)
; AH  = 0x77          (AX ရဲ့ high byte)
; AL  = 0x88          (AX ရဲ့ low byte)
```


EAX ထည့်ပြီး RAX ပြောင်းလဲခြင်း 
```c

; အစပိုင်း
mov rax, 0x1122334455667788   ; RAX = 0x1122334455667788

; EAX ကိုပြောင်း (32-bit)
mov eax, 0xAABBCCDD

; RAX ပြောင်းသွားပြီ (အပေါ်ဆုံး 32 bits သုညဖြစ်သွား)
; RAX = 0x00000000AABBCCDD
; EAX = 0xAABBCCDD
; AX  = 0xCCDD
; AH  = 0xCC
; AL  = 0xDD
```

32-bit register (EAX) ကိုထိရင် 64-bit register (RAX) ရဲ့ အပေါ်ဆုံး 32 bits သုညဖြစ်သွားတယ်


= AX ထည့်ပြီး EAX/RAX ပြောင်းလဲခြင်း
```c
; အစပိုင်း
mov rax, 0x1122334455667788   ; RAX = 0x1122334455667788

; AX ကိုပြောင်း (16-bit)
mov ax, 0x1234

; RAX ရဲ့ အောက်ဆုံး 16 bits ပဲပြောင်းတယ် (အပေါ်ပိုင်းအတိုင်းပဲ)
; RAX = 0x1122334455661234
; EAX = 0x55661234
; AX  = 0x1234
; AH  = 0x12
; AL  = 0x34
```


 AL ထည့်ပြီး အပေါ်ပိုင်းများ သက်ရောက်မှု
```c
; အစပိုင်း
mov rax, 0x1122334455667788   ; RAX = 0x1122334455667788

; AL ကိုပြောင်း (8-bit)
mov al, 0xFF

; အောက်ဆုံး 8 bits ပဲပြောင်း
; RAX = 0x11223344556677FF
; EAX = 0x556677FF
; AX  = 0x77FF
; AH  = 0x77
; AL  = 0xFF
```


 AH ထည့်ပြီး အပေါ်ပိုင်းများ သက်ရောက်မှု
```c
; အစပိုင်း
mov rax, 0x1122334455667788   ; RAX = 0x1122334455667788

; AH ကိုပြောင်း (8-bit high byte of AX)
mov ah, 0xAA

; bits 15-8 ပဲပြောင်း
; RAX = 0x112233445566AA88
; EAX = 0x5566AA88
; AX  = 0xAA88
; AH  = 0xAA
; AL  = 0x88
```


 8-bit, 16-bit, 32-bit ရောနှောသုံးခြင်း
 ```c
 ; multi-precision arithmetic ဥပမာ
; 32-bit တန်ဖိုးကို 8-bit ပိုင်းပိုင်းနဲ့ တည်ဆောက်ခြင်း

mov ah, 0x12      ; AH = 0x12
mov al, 0x34      ; AL = 0x34
; AX ယခု = 0x1234

mov bx, 0x5678    ; BX = 0x5678

; AX နဲ့ BX ကိုပေါင်း
add ax, bx        ; AX = 0x1234 + 0x5678 = 0x68AC
; CF = 0, ZF = 0

; ရလဒ်ကို EAX ထဲသို့ချဲ့
movzx eax, ax     ; EAX = 0x000068AC
 ```


 Byte, Word, Doubleword တို့ ရောနှောခြင်း
```c
; ASCII character တွေကို တွဲဆက်ခြင်း
mov al, 'H'       ; AL = 'H' (0x48)
mov ah, 'i'       ; AH = 'i' (0x69)
; AX = "Hi" (0x6948) - little endian!

; EAX ထဲကို ထပ်ထည့်
mov eax, 0x00000000
mov ax, 0x6948    ; EAX = 0x00006948

; RAX ထဲကို ချဲ့
mov rax, 0x0000000000000000
mov eax, 0x00006948  ; RAX = 0x0000000000006948
```

 Function Return Value (အရေးကြီးဆုံး)
```c
; C calling convention (System V AMD64)
; return value ကို RAX မှာပြန်တယ်

my_function:
    mov eax, 42       ; return 42 (32-bit)
    ret               ; caller က RAX ကိုကြည့်မယ်
    
; main function မှာ
call my_function
; RAX = 42, EAX = 42, AX = 42, AL = 42, AH = 0

; 64-bit return value
my_func_64:
    mov rax, 0x1122334455667788
    ret
    
; caller က RAX တစ်ခုလုံးကိုဖတ်
; EAX ကို ဖတ်ရင် 0x55667788 ပဲရမယ်
```

---


#### Choosing Register

##### By value size

|လိုအပ်ချက်|သုံးသင့်တဲ့ register|ဥပမာ|
|---|---|---|
|0-255 အတွင်း တန်ဖိုး|AL, BL, CL, DL|`mov al, 0xFF`|
|0-65535 အတွင်း တန်ဖိုး|AX, BX, CX, DX|`mov ax, 0xFFFF`|
|0-4.29 ဘီလီယံအတွင်း|EAX, EBX, ECX, EDX|`mov eax, 0xFFFFFFFF`|
|0-2⁶⁴-1 အတွင်း|RAX, RBX, RCX, RDX|`mov rax, 0xFFFFFFFFFFFFFFFF`|

---

##### for memory  Address 

|လိုအပ်ချက်|သုံးသင့်တဲ့ register|ဥပမာ|
|---|---|---|
|Memory address (16-bit mode)|BX|`mov [bx], al`|
|Memory address (32-bit mode)|EBX|`mov [ebx], al`|
|Memory address (64-bit mode)|RBX|`mov [rbx], al`|

---

##### for looping

|လိုအပ်ချက်|သုံးသင့်တဲ့ register|ဥပမာ|
|---|---|---|
|Loop (16-bit)|CX|`loop label`|
|Loop (32-bit)|ECX|`loop label`|
|Loop (64-bit)|RCX|`loop label`|

---

##### other necessary

|လိုအပ်ချက်|သုံးသင့်တဲ့ register|ဥပမာ|
|---|---|---|
|Multiplication/Division (32-bit)|EAX, EDX (တွဲ)|`mul ebx` (result in EDX:EAX)|
|Multiplication/Division (64-bit)|RAX, RDX (တွဲ)|`mul rbx` (result in RDX:RAX)|
|I/O Port (16-bit)|DX|`out dx, al`|
|I/O Port (32-bit)|EDX|`out dx, eax`|
|String Source|SI (16-bit), ESI (32-bit), RSI (64-bit)|`movsb`|
|String Destination|DI (16-bit), EDI (32-bit), RDI (64-bit)|`movsb`|
|Function return value (small)|AL|`mov al, 0x01` (return 1)|
|Function return value (32-bit)|EAX|`mov eax, 0x12345678`|
|Function return value (64-bit)|RAX|`mov rax, 0x1122334455667788`|

---

##### Default Address Registers by Modes

|Mode|Default Address Register|ဥပမာ|
|---|---|---|
|Real Mode (16-bit)|BX, BP, SI, DI|`mov al, [bx+si]`|
|Protected Mode (32-bit)|EBX, EBP, ESI, EDI|`mov al, [ebx+esi]`|
|Long Mode (64-bit)|RBX, RBP, RSI, RDI|`mov al, [rbx+rsi]`|

---

##### Caution

|လုပ်ဆောင်ချက်|မသုံးသင့်တဲ့ register|အကြောင်းရင်း|
|---|---|---|
|General computation|RSP|Stack pointer ကိုထိရင် program crash ဖြစ်နိုင်|
|General computation|RIP|Instruction pointer ကို mov နဲ့မပြောင်းရ (jmp သုံးရမယ်)|
|8-bit operation in 64-bit mode|AH, BH, CH, DH (တချို့ instruction တွေ)|တချို့ CPU မှာ slow ဖြစ်နိုင်|
|Loop with CX in 64-bit mode|CX (အကယ်၍ 64-bit address သုံးရင်)|Address ကြီးရင် RCX သုံးသင့်တယ်|

---

#### In Examples
##### Choosing value size
```
; 0-255 အတွင်း တန်ဖိုး
mov al, 0xFF        ; AL = 255
; 0-65535 အတွင်း တန်ဖိုး
mov ax, 0xFFFF      ; AX = 65535
; 0-4.29 ဘီလီယံအတွင်း
mov eax, 0xFFFFFFFF ; EAX = 4294967295
; 0-2⁶⁴-1 အတွင်း
mov rax, 0xFFFFFFFFFFFFFFFF ; RAX = 18446744073709551615
```

##### Memory Address 
```
; 16-bit mode (Real Mode)
mov bx, 0x1000      ; BX = address
mov al, [bx]        ; RAM address 0x1000 မှ value ကိုဖတ်
; 32-bit mode (Protected Mode)
mov ebx, 0x00401000 ; EBX = address
mov al, [ebx]       ; RAM address 0x00401000 မှ value ကိုဖတ်
; 64-bit mode (Long Mode)
mov rbx, 0x7FFD00001000 ; RBX = address
mov al, [rbx]           ; RAM address မှ value ကိုဖတ်
```

##### Loop

```
; 16-bit loop
mov cx, 0x000A       ; CX = 10
loop_16:
    ; something here
    loop loop_16     ; CX--, if CX != 0 jump
; 32-bit loop
mov ecx, 0x000186A0  ; ECX = 100000
loop_32:
    ; something here
    loop loop_32
; 64-bit loop
mov rcx, 0x0000000005F5E100 ; RCX = 100000000
loop_64:
    ; something here
    loop loop_64
```


##### Multiplication , Division

```
; 32-bit multiplication (EAX * EBX = EDX:EAX)
mov eax, 0x000186A0  ; EAX = 100000
mov ebx, 0x00000064  ; EBX = 100
mul ebx              ; EDX:EAX = 100000 * 100 = 10,000,000
; 64-bit multiplication (RAX * RBX = RDX:RAX)
mov rax, 0x0000000005F5E100 ; RAX = 100,000,000
mov rbx, 0x0000000000000064 ; RBX = 100
mul rbx                     ; RDX:RAX = 10,000,000,000
```
##### - I/O Port 
```
; 16-bit I/O
mov dx, 0x3F8   ; COM1 port
mov al, 'A'
out dx, al
; 32-bit I/O (some systems)
mov edx, 0x3F8
mov eax, 0x41414141  ; 'AAAA'
out dx, eax
```

---

##### (Quick Reference Card)
```
┌─────────────────────────────────────────────────────────────────┐
│                    REGISTER SELECTION GUIDE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  VALUE SIZE:                                                     │
│    0-255         → AL, BL, CL, DL                               │
│    0-65535       → AX, BX, CX, DX                               │
│    0-4.29B       → EAX, EBX, ECX, EDX                           │
│    0-2⁶⁴-1       → RAX, RBX, RCX, RDX                           │
│                                                                  │
│  MEMORY ADDRESS:                                                 │
│    16-bit mode   → BX, BP, SI, DI                               │
│    32-bit mode   → EBX, EBP, ESI, EDI                           │
│    64-bit mode   → RBX, RBP, RSI, RDI                           │
│                                                                  │
│  LOOP COUNTER:                                                   │
│    16-bit loop   → CX                                           │
│    32-bit loop   → ECX                                          │
│    64-bit loop   → RCX                                          │
│                                                                  │
│  MATH (MUL/DIV):                                                 │
│    32-bit        → EAX (result), EDX (remainder)                │
│    64-bit        → RAX (result), RDX (remainder)                │
│                                                                  │
│  I/O:                                                            │
│    16-bit port   → DX                                           │
│    32-bit port   → EDX                                          │
│                                                                  │
│  STRING OPS:                                                     │
│    Source        → SI (16), ESI (32), RSI (64)                  │
│    Destination   → DI (16), EDI (32), RDI (64)                  │
│                                                                  │
│  FUNCTION RETURN:                                                │
│    Small value   → AL                                           │
│    32-bit value  → EAX                                          │
│    64-bit value  → RAX                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

