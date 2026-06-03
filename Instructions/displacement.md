
Displacement = ရွေ့လျှားမှုအကွာအဝေး (Offset)

ဒါက memory address ကို အခြေခံ address (base) ကနေ ဘယ်လောက်ဝေးတယ် ဆိုတာကို ပြတဲ့ တန်ဖိုး ဖြစ်တယ်

```
Memory Address ဖော်မြူလာ = Base Address + Displacement

ဥပမာ - [EBX + 5] ဆိုရင်
- Base Address = EBX ထဲမှာရှိတဲ့ တန်ဖိုး
- Displacement = 5
- Final Address = EBX + 5
```

```
Memory Array တစ်ခုလို စဉ်းစားပါ:

Address:   1000   1001   1002   1003   1004   1005   1006   ...
          ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
Value:    │  A  │  B  │  C  │  D  │  E  │  F  │  G  │ ...
          └─────┴─────┴─────┴─────┴─────┴─────┴─────┘
            ↑
            │
        Base = 1000 (EBX ထဲမှာ 1000 ရှိတယ်)

[EBX]     → Address 1000 (အခြေခံနေရာ)
[EBX + 1] → Address 1001 (ဘေးတစ်နေရာ)
[EBX + 2] → Address 1002 (နောက်ထပ်တစ်နေရာ)
[EBX + 5] → Address 1005 (အကွာအဝေး 5)


ဒီမှာ 1, 2, 5 ဆိုတာတွေက Displacement တွေ
```


| Types      | Size                               | Mod Value | Example        |
| ---------- | ---------------------------------- | --------- | -------------- |
| **disp8**  | 1 byte (0 to 255, or -128 to +127) | Mod = 01  | `[EBX + 5]`    |
| **disp32** | 4 bytes (-2³¹ to 2³¹-1)            | Mod = 10  | `[EBX + 1000]` |


##### 3.1 disp8 (8-bit displacement)

```
အသုံးပြုပုံ: [Base + disp8]
အရွယ်အစား: 1 byte
တန်ဖိုးနယ်နိမိတ်: -128 မှ +127 (signed) သို့ 0 မှ 255 (unsigned)

ဘယ်အချိန်သုံးလဲ: အနီးကပ် offset တွေအတွက် (memory မှာ နည်းနည်းပဲရွှေ့ချင်ရင်)
```


##### 3.2 disp32 (32-bit displacement)


```
အသုံးပြုပုံ: [Base + disp32]
အရွယ်အစား: 4 bytes
တန်ဖိုးနယ်နိမိတ်: -2,147,483,648 မှ +2,147,483,647

ဘယ်အချိန်သုံးလဲ: အဝေးကြီးကို ရွှေ့ချင်ရင် (သို့) absolute address အတွက်
```



#### Examples

#####  - no Displacement  (Mod=00)

```asm
mov al, [ebx]     ; EBX ထဲမှာရှိတဲ့ address ကိုပဲသုံး
                  ; Displacement မလိုဘူး (0)
                  
Machine Code: 8A 03
              (Mod=00, R/M=011 → [EBX])
```

#####  - disp8 (Mod=01) - 1 byte displacement

```asm
mov al, [ebx + 5]     ; EBX + 5 က address
                      ; Displacement = 5 (1 byte)
                      
Machine Code: 8A 43 05
              ├─┬─┤  │
              │ │   └─ disp8 = 05
              │ └─ ModRM (43 = 01 000 011)
              └─ Opcode (8A)
```

#####  - disp8 minus value (signed)

```asm
mov al, [ebx - 3]     ; EBX - 3 က address
                      ; Displacement = -3 (0xFD)
                      
Machine Code: 8A 43 FD
              ├─┬─┤  │
              │ │   └─ disp8 = FD (-3 in signed)
              │ └─ ModRM (43)
              └─ Opcode (8A)
```

#####  - disp32 (Mod=10) - 4 byte displacement

```asm
mov al, [ebx + 1000]  ; EBX + 1000 က address
                      ; Displacement = 1000 (0x3E8)
                      
Machine Code: 8A 83 E8 03 00 00
              ├─┬─┤  └─────┬─────┘
              │ │        disp32 = E8 03 00 00 (little-endian = 0x3E8)
              │ └─ ModRM (83 = 10 000 011)
              └─ Opcode (8A)
```




| Mod | Displacement Size | Instruction ဥပမာ | Machine Code ပုံစံ |
|-----|------------------|------------------|-------------------|
| 00 | None (0) | `mov al, [ebx]` | Opcode + ModRM |
| 01 | 1 byte (disp8) | `mov al, [ebx+5]` | Opcode + ModRM + disp8 |
| 10 | 4 bytes (disp32) | `mov al, [ebx+1000]` | Opcode + ModRM + disp32 |
| 11 | (register mode, no displacement) | `mov al, bl` | Opcode + ModRM |



---

##### SIB

**SIB** = Scale + Index + Base

R/M = 100 (4) ဆိုရင် SIB byte လိုအပ်တယ် ဒါက `[EAX + ECX*4]` လိုမျိုး **complex addressing** အတွက်ပါ

```
ပုံစံ: [Base + Index * Scale + Displacement]

ဥပမာ - [EBX + ECX*4 + 8]

ဒါကို encoding လုပ်ရန်:
- ModRM မှာ R/M=100 (SIB byte လိုတယ်)
- SIB byte က Base, Index, Scale ကိုသတ်မှတ်
- Displacement က optional
```



---

#####  R/M=101 (disp32) (special situation in Mod=00)

Mod=00, R/M=101 ဆိုရင် base register မရှိဘဲ displacement တစ်ခုတည်း

```
Mod=00, R/M=101 → [disp32] (absolute address)

ဥပမာ - mov al, [0x00401000]
         
Machine Code: 8A 05 00 10 40 00
              ├─┬─┤ └─────┬─────┘
              │ │     disp32 = 0x00401000
              │ └─ ModRM (05 = 00 000 101)
              └─ Opcode (8A)
```

ဒါက base register မလိုဘဲ တိုက်ရိုက် address ကိုသုံးတဲ့အခါ

---

#####  - how to store displacement in machine Code 
 Little-Endian 
x86 CPU က little-endian သုံးတယ်။ ဆိုလိုတာက အနိမ့်ဆုံး byte ကို အရင်သိမ်းတယ်

```
disp32 = 0x12345678 ဆိုရင်:

Memory မှာ: 78 56 34 12 (နောက်ပြန်)

ဥပမာ - mov al, [ebx + 0x12345678]
Machine Code: 8A 83 78 56 34 12
                    └──┬───┘
                 disp32 in little-endian
```

---

##### (Displacement Summary)

| Q                                   | Ans                                                      |
| ----------------------------------- | -------------------------------------------------------- |
| Displacement ဆိုတာဘာလဲ              | Base address ကနေ ရွေ့လျှားတဲ့ အကွာအဝေး                   |
| disp8 ဆိုတာဘာလဲ                     | 1-byte displacement (Mod=01)                             |
| disp32 ဆိုတာဘာလဲ                    | 4-byte displacement (Mod=10)                             |
| Mod=00 မှာ displacement ရှိလား      | မရှိ (R/M=101 မှလွဲရင်)                                  |
| Mod=01 မှာ ဘယ်လောက် displacement လဲ | 1 byte (disp8)                                           |
| Mod=10 မှာ ဘယ်လောက် displacement လဲ | 4 bytes (disp32)                                         |
| ဘယ်အချိန်မှာ displacement သုံးလဲ    | array access, struct field access, stack local variables |


---
