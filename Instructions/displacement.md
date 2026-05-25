
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


### 3.1 disp8 (8-bit displacement)

```
အသုံးပြုပုံ: [Base + disp8]
အရွယ်အစား: 1 byte
တန်ဖိုးနယ်နိမိတ်: -128 မှ +127 (signed) သို့ 0 မှ 255 (unsigned)

ဘယ်အချိန်သုံးလဲ: အနီးကပ် offset တွေအတွက် (memory မှာ နည်းနည်းပဲရွှေ့ချင်ရင်)
```


### 3.2 disp32 (32-bit displacement)


```
အသုံးပြုပုံ: [Base + disp32]
အရွယ်အစား: 4 bytes
တန်ဖိုးနယ်နိမိတ်: -2,147,483,648 မှ +2,147,483,647

ဘယ်အချိန်သုံးလဲ: အဝေးကြီးကို ရွှေ့ချင်ရင် (သို့) absolute address အတွက်
```