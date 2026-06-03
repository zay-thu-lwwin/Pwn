
### XOR (Exclusive OR)

Bit နှစ်ခုကြားမှာ လုပ်ဆောင်တယ်
ရလဒ်က တူရင် 0၊ မတူရင် 1 ဆိုတဲ့ စည်းမျဉ်းရှိတယ်

| Input A | Input B | Output (A XOR B) |
| ------- | ------- | ---------------- |
| 0       | 0       | 0                |
| 0       | 1       | 1                |
| 1       | 0       | 1                |
| 1       | 1       | 0                |


`A ^ B = C` ဖြစ်ခဲ့ရင် `C ^ A = B` ပြန်ဖြစ်ပါတယ်

```
mov eax, 0x11111111
xor eax, 0x1111112a  ; ရလဒ်က eax ထဲမှာ 0x3b (59) ဖြစ်သွားမယ်။ filter လွတ်ပါတယ်။
```


```
1010 1010  (0xAA)
XOR
1100 1100  (0xCC)
─── ─── ───
0110 0110  (0x66)
```


#### Properties of XOR
##### Identity Property
```
A XOR 0 = A
```
ဥပမာ: `5 XOR 0 = 5`

##### Self-Inverse Property 
```
A XOR A = 0
```
ဥပမာ: `5 XOR 5 = 0`

##### Commutative
```
A XOR B = B XOR A
```

##### Associative
```
(A XOR B) XOR C = A XOR (B XOR C)
```

##### Cancellation
```
A XOR B XOR B = A
```
ဘာလို့လဲဆိုတော့ B XOR B = 0, A XOR 0 = A



#### XOR in assembly
##### Making Register to Zero  (Most Common)

```asm
xor eax, eax    ; eax = 0
xor ebx, ebx    ; ebx = 0  
xor ecx, ecx    ; ecx = 0
xor edx, edx    ; edx = 0
```

Opcode: `31 c0` `(xor eax, eax)  `
→ Null byte လုံးဝမပါဘူး။

ဘာလို့ `mov eax, 0` မသုံးတာလဲ?
```asm
mov eax, 0      ; Opcode: b8 00 00 00 00 ← \x00 သုံးခုပါတယ် (ဆိုးတယ်)
xor eax, eax    ; Opcode: 31 c0 ← null byte မပါ (ကောင်းတယ်)
```

##### Immediate Value
```asm
xor eax, 0x12345678    ; eax ကို 0x12345678 နဲ့ XOR လုပ်
```

##### Memory XOR
```asm
xor byte [ebx], 0x55   ; memory address ebx မှာရှိတဲ့ byte ကို 0x55 နဲ့ XOR လုပ်
xor dword [esi], ecx   ; dword (4 bytes) ကို ecx နဲ့ XOR လုပ်
```

#### XOR usage in real world
##### Example 1: Basic XOR
```
ပုံမှန် data:    0x41 ('A')
XOR key:         0x55
Result:          0x14  (encoded)

ပြန် decode:
Encoded:         0x14
XOR key:         0x55
Original:        0x41  ('A') ← ပြန်ရတယ်
```

##### Example 2: Register Clearing
```asm
; မူရင်း eax မှာ ဘာရှိလဲ မသိဘူး
mov eax, 0xFFFFFFFF   ; eax = -1
xor eax, eax          ; eax = 0 (အားလုံးကို 0 ဖြစ်သွားစေတယ်)
```

##### Example 3: Simple Encryption
```c
// C မှာ XOR encryption လုပ်နည်း
char message[] = "secret";
char key = 0x42;

// Encrypt
for(int i=0; i<6; i++)
    message[i] ^= key;

// Decrypt (same operation)
for(int i=0; i<6; i++)
    message[i] ^= key;  // မူရင်းအတိုင်းပြန်ရမယ်
```

#### OR Techniques

##### XOR Swap (Swap Two Register)
```asm
; eax နဲ့ ebx ကို swap လုပ်မယ်
xor eax, ebx    ; eax = eax ^ ebx
xor ebx, eax    ; ebx = ebx ^ (eax ^ ebx) = eax
xor eax, ebx    ; eax = (eax ^ ebx) ^ eax = ebx
```

##### Finding the same
```asm
xor eax, ebx
test eax, eax
jz equal        ; eax == ebx ဆိုရင် zero flag set ဖြစ်မယ်
```


---
