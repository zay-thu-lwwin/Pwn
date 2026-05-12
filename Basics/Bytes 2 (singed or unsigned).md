

	Bit တစ်ခုက 0 သို့မဟုတ် 1
	Byte ဆိုတာ 8 bits
	bit system 

### Signed Number

Signed = အနုတ် (-) နဲ့ အပေါင်း (+) နှစ်မျိုးလုံးပါ

signed မှာ (+) value နဲ့ (-) value သိဖို့က
အရင်ဆုံး ငါတို့ data type တွေက byte ဘယ်လောက်ယူတာလဲဆိုတာသိထားရမယ်
အဲဒါမှ  bits width , bits အကျယ်ကိုသိမယ်



| Type              | 32-bit Size | 64-bit Size | Min Limit      | Max Limit       | Example Value                                         |
| ----------------- | ----------- | ----------- | -------------- | --------------- | ----------------------------------------------------- |
| `char`            | 1           | 1           | -128           | +127            | `'A'` (65)                                            |
| `unsigned char`   | 1           | 1           | 0              | 255             | `255` (0xFF)                                          |
| `bool`            | 1           | 1           | 0              | 1               | `1` (true)                                            |
| `short`           | 2           | 2           | -32,768        | +32,767         | `0x7FFF` (32,767)                                     |
| `unsigned short`  | 2           | 2           | 0              | 65,535          | `0xFFFF` (65,535)                                     |
| `int`             | 4           | 4           | -2,147,483,648 | +2,147,483,647  | `0x7FFFFFFF` (2,147,483,647)                          |
| `unsigned int`    | 4           | 4           | 0              | 4,294,967,295   | `0xFFFFFFFF` (4,294,967,295)                          |
| `float`           | 4           | 4           | ~1.18×10⁻³⁸    | ~3.4×10³⁸       | `3.14159`                                             |
| `long`            | **4**       | **8**       | -2³¹ / -2⁶³    | +2³¹-1 / +2⁶³-1 | `0x7FFFFFFF` (32-bit) / `0x7FFFFFFFFFFFFFFF` (64-bit) |
| `unsigned long`   | **4**       | **8**       | 0              | 4B / 1.8×10¹⁹   | `0xFFFFFFFF` (32-bit) / `0xFFFFFFFFFFFFFFFF` (64-bit) |
| `long long`       | 8           | 8           | -9.22×10¹⁸     | +9.22×10¹⁸      | `0x7FFFFFFFFFFFFFFF`                                  |
| `double`          | 8           | 8           | ~2.2×10⁻³⁰⁸    | ~1.8×10³⁰⁸      | `3.14159265358979`                                    |
| `void*` (pointer) | **4**       | **8**       | 0              | 4GB / 16EB      | `0x7fffffff` (32-bit) / `0x7fffffffdb00` (64-bit)     |
| `size_t`          | **4**       | **8**       | 0              | 4GB / 16EB      | `0x64` (100 bytes)                                    |

| Type                     | Size (bytes) | **Alignment Required** | Example Value        | Notes            |
| ------------------------ | ------------ | ---------------------- | -------------------- | ---------------- |
| `char`                   | 1            | **1-byte**             | `'A'`                | anywhere         |
| `unsigned char`          | 1            | **1-byte**             | `255`                | anywhere         |
| `bool`                   | 1            | **1-byte**             | `1`                  | anywhere         |
| `short`                  | 2            | **2-byte**             | `0x1234`             | address % 2 == 0 |
| `unsigned short`         | 2            | **2-byte**             | `0xFFFF`             | address % 2 == 0 |
| `int`                    | 4            | **4-byte**             | `0x12345678`         | address % 4 == 0 |
| `unsigned int`           | 4            | **4-byte**             | `0xFFFFFFFF`         | address % 4 == 0 |
| `float`                  | 4            | **4-byte**             | `3.14`               | address % 4 == 0 |
| `long` (32-bit)          | 4            | **4-byte**             | `0x12345678`         | address % 4 == 0 |
| `long` (64-bit)          | 8            | **8-byte**             | `0x123456789ABCDEF0` | address % 8 == 0 |
| `unsigned long` (32-bit) | 4            | **4-byte**             | `0xFFFFFFFF`         | address % 4 == 0 |
| `unsigned long` (64-bit) | 8            | **8-byte**             | `0xFFFFFFFFFFFFFFFF` | address % 8 == 0 |
| `long long`              | 8            | **8-byte**             | `0x123456789ABCDEF0` | address % 8 == 0 |
| `double`                 | 8            | **8-byte**             | `3.14159`            | address % 8 == 0 |
| `void*` (pointer 32-bit) | 4            | **4-byte**             | `0x7fffffff`         | address % 4 == 0 |
| `void*` (pointer 64-bit) | 8            | **8-byte**             | `0x7fffffffdb00`     | address % 8 == 0 |
| `size_t` (32-bit)        | 4            | **4-byte**             | `0x64`               | address % 4 == 0 |
| `size_t` (64-bit)        | 8            | **8-byte**             | `0x64`               | address % 8 == 0 |




ဥပမာပြောရရင်
```
Decimal: 3
Binary:  ... 0 0 1 1  (အောက်ဆုံး 2 bits က 11)
Hex: 0x3
```

```

3 ကို 1 byte နဲ့ပြရင်: 00000011 (binary)
3 ကို 4 bytes (32-bit) နဲ့ပြရင်: 00000000 00000000 00000000 00000011
3 ကို 8 bytes (64-bit) နဲ့ပြရင်: 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000011

```
ဒီမှာ ဒီကောင်မှာ bytes (bit width ) ပေါ်မူတည်ပြီး ကိန်းရေပါဝင်နိုင်နှုန်းလည်းကွဲသလို 
(-) value တွက်တဲ့အခါလည်း တစ်ခုနဲ့ တစ်ခု မတူဘူး
ဘာလို့ဆို
unsigned မှာ အကုန်လုံးက (+) value တွေဖြစ်ပြီး
signedမှာ ရှေတစ်ဝက်က အကုန် (+) value တွေဖြစ်ပြီး
နောက်တစ်ဝက်က (-) value တွေဖြစ်တယ်
ပြီးတော့ (-) value တွေက နောက်ကနေစရေတယ်

ဥပမာ 
4 bytes မှာ
	unsigned မှာဆိုရင် 11111111 11111111 11111111 11111111 (hex value = 0xFF FF FF FF) က 4,294,967,295 နဲ့ညီပြီးတော့
	signed မှာဆိုရင် 11111111 11111111 11111111 11111111 (hex value = 0xFF FF FF FF) က -1 နဲ့ညီတယ်
	အလယ်ကို သွားစုတယ်
ဆိုတော့ stackမှာ 0xFF FF FF FF မြင်တိုင်း 4,294,967,295 လား -1 လားဆိုတာ ခွဲလို့မရဘူး 
ငါတို့ variable datatype ကိုသွားကြည့်ပြီး signed လား unsinged လား ခွဲလို့ရမှာ



ပိုပြီးရှင်းရအောင် အခြား example တွေနဲ့ကြည့်ကြည့်မယ်


##### with logic (own idea)
```
singed (+ and -)
အရင် ဆုံး bit representation နဲ့  စဉ်းစားကြည့်ရအောင်
byte presentation မှာ 1 byte ဆိုတာ အငယ်ဆုံး
ဆိုတော့ 1byte presentation နဲ့ကြည့်ရအောင်
1 byte မှာဆို 8 bits ရှိတယ်
00 00 00 00 ကို  decimal  value အနေနဲ့ 0 
00 00 00 01  =  1
00 00 00 10  =  2
...
01 11 11 11 = 127

အထိရှိတယ် စုစုပေါင်း 0 အပါအဝင် 128 လုံး (0 - 127)
singed မှာဆို ဒီအထိဘဲ
ရှေ့ဆုံးက bit ကို 0 ချန်ထားတယ်
ရှေ့ဆုံး က bit ကို 1 ပြောင်းလိုက်တဲ့ အချိန်က  (-) တန်ဖိုးဖြစ်သွားပြီ
10 00 00 00 = -128
10 00 00 01 = -127 
...
11 11 11 11 = -1 (hexဆို)
and ဒီကောင်ကိုကျော်သွားရင် 
00 00 00 00 ပြန်ဖြစ်
00 00 00 00  ဆိုတော့  0
00 00 00 01  =  1
00 00 00 10  =  2

အဲလိုမျိုး wrap around ဖြစ်နေတာ
ဆိုတော့ + နဲ့ - ဆို စုစုပေါင်း 128 + 128 ဆိုတော့ 256

unsinged (only +) မှာ ဆို (-) မရှိတော့
01 11 11 11 = 127  
ဒီကောင်ပြီးရင်
10 00 00 00  = 128 ဒီကောင်လာ
10 00 00 01  = 129
...
11 11 11 11 = 255 
ဆိုတော့ အားလုံးစုစုပေါင်းက 256 လုံး

```

##### with formula

negative ဘယ်လိုတွက်မလဲကြည့်ရအောင် 

first way
```
Negative number ရဲ့ bit pattern = 2^N - (positive version)

N = bits အရေအတွက် (8/1byte, 16/2bytes, 32/4bytes, 64/8bytes)
```

```
8-bit မှာ ဖြစ်နိုင်တဲ့ အကွာအဝေး: -128 to +127

-1 ကို 8-bit နဲ့ သိမ်းပုံ:
= 2^8 - 1
= 256 - 1 = 255
255 ရဲ့ binary = 11111111
255 ရဲ့ hex = 0xFF

ဒါကြောင့် 8-bit မှာ 0xFF = -1
```

အပေါ်ကကောင်က wrap around  ကိုနားလည်အောင်ပြထားတာ 
stack မှာတကယ် store လုပ်ရင် integer အတွက် 4 byteဘဲသုံးတယ် 
Data Type က ဘယ်လောက် bytes သုံးမယ်ဆိုတာ သတ်မှတ်တယ်

```
-1 ကို 4byte မှာသိမ်းမယ်ဆို
00 00 00 0xFF
```

second way 
positive integer ရဲ့ bitတွေသိလို့ပြောင်းချင်ရင်
```
အရင်ဆုံး +300 ရဲ့ bit pattern: 
00000000 00000000 00000001 00101100  (0x0000012C)

-300 ရဲ့ bit pattern (two's complement):
Step 1: +300 ကို bitwise NOT လုပ် (အကုန်ပြောင်း)
11111111 11111111 11111110 11010011

Step 2: +1 ထပ်ပေါင်း
11111111 11111111 11111110 11010100

= 0xFFFFFED4 (ဒါက -300 ရဲ့ bit pattern)
```

ဆို‌တော့ ဒီမှာ data store လုပ်မယ် memory address အနေနဲ့
သေချာကြည့်ရင် 1 byte  (8bit) က 256 လောက်ထိဘဲ store လုပ်နိုင်တယ် 
256 ကျော်ရင်မရတော့ဘူး
ဆိုတော့ အဲကြ 4 byte လာတယ်
ဆိုတော့ 8 byte ဆို 4 byte ထက်များများဆန့်ပြီး store  လုပ်နိုင်မယ်

4 byte 8 byte တို့ဆိုတာ က အကျယ်အဝန်းဘဲ
ဒါမဲ့ character(1byte) ကို storeလုပ်မယ်ဆို stack memory address 1 byte ကိုဘဲသုံး (အပေါ်က table ကိုကြည့်)

##### singed integers

| Data Type   | Size (bits) | 32-bit System Range                 | 64-bit System Range             | Calculation                                        |
| ----------- | ----------- | ----------------------------------- | ------------------------------- | -------------------------------------------------- |
| `char`      | 8           | -128 to 127                         | -128 to 127                     | -2⁷ to 2⁷-1                                        |
| `short`     | 16          | -32,768 to 32,767                   | -32,768 to 32,767               | -2¹⁵ to 2¹⁵-1                                      |
| `int`       | 32          | -2,147,483,648 to 2,147,483,647     | -2,147,483,648 to 2,147,483,647 | -2³¹ to 2³¹-1                                      |
| `long`      | 32 or 64    | **-2,147,483,648 to 2,147,483,647** | **-9.22×10¹⁸ to 9.22×10¹⁸**     | -2³¹ to 2³¹-1 (32-bit)  <br>-2⁶³ to 2⁶³-1 (64-bit) |
| `long long` | 64          | -9.22×10¹⁸ to 9.22×10¹⁸             | -9.22×10¹⁸ to 9.22×10¹⁸         | -2⁶³ to 2⁶³-1                                      |

|Data Type|32-bit Min|32-bit Max|64-bit Min|64-bit Max|
|---|---|---|---|---|
|`long`|-2,147,483,648|2,147,483,647|-9,223,372,036,854,775,808|9,223,372,036,854,775,807|
|`long long`|-9,223,372,036,854,775,808|9,223,372,036,854,775,807|-9,223,372,036,854,775,808|9,223,372,036,854,775,807|

##### unsigned  integers


| Data Type            | Size (bits) | 32-bit System Range             | 64-bit System Range                 | Calculation                                  |
| -------------------- | ----------- | ------------------------------- | ----------------------------------- | -------------------------------------------- |
| `unsigned char`      | 8           | 0 to 255                        | 0 to 255                            | 0 to 2⁸-1                                    |
| `unsigned short`     | 16          | 0 to 65,535                     | 0 to 65,535                         | 0 to 2¹⁶-1                                   |
| `unsigned int`       | 32          | 0 to 4,294,967,295              | 0 to 4,294,967,295                  | 0 to 2³²-1                                   |
| `unsigned long`      | 32 or 64    | **0 to 4,294,967,295**          | **0 to 18,446,744,073,709,551,615** | 0 to 2³²-1 (32-bit)  <br>0 to 2⁶⁴-1 (64-bit) |
| `unsigned long long` | 64          | 0 to 18,446,744,073,709,551,615 | 0 to 18,446,744,073,709,551,615     | 0 to 2⁶⁴-1                                   |

| Data Type            | 32-bit Max                 | 64-bit Max                 |
| -------------------- | -------------------------- | -------------------------- |
| `unsigned long`      | 4,294,967,295              | 18,446,744,073,709,551,615 |
| `unsigned long long` | 18,446,744,073,709,551,615 | 18,446,744,073,709,551,615 |

---

### Terminology


### Masking

 Masking ဆိုတာ bits တွေကို စစ်ထုတ်ခြင်း (filtering)
 AND (`&`) operation ကို အသုံးပြုပြီး မလိုချင်တဲ့ bits တွေကို 0 ဖြစ်အောင်လုပ်တယ်
 လိုချင်တဲ့ bits တွေကို အတိုင်းထားတယ်


##### 8bit masking

```c
Value:  1 0 1 0 1 0 1 1
Mask:   1 1 1 1 0 0 0 0
─────────────────────── AND
Result: 1 0 1 0 0 0 0 0
        ↑ ဒီ 4 bits ပဲကျန်တယ် ↑
```

```c
uint8_t value = 0xAB;  // 10101011
uint8_t mask  = 0xF0;  // 11110000 (အပေါ် 4 bits ကို ယူမယ်)

uint8_t result = value & mask;  // 10100000 (0xA0)
```
note - hex 0xF အဆုံးထိဆို 1111
ဒါကြောင့် hex ဆိုတာဖြစ်လာတာ 1111 နဲ့ကိုက်ညီတဲ့ကိန်းကိုယူချင်လို့
octal (hex ရဲ့တစ်ဝက်) ဆို 17 ဒီကောင်လည်း အဆုံးဘဲ ဒါပြီးရင် 20 ဘဲလာတယ်


```c
// အောက်ဆုံး 4 bits ကိုယူမယ်
uint8_t value = 0xAB;  // 10101011
uint8_t mask  = 0x0F;  // 00001111
uint8_t result = value & mask;  // 00001011 (0x0B)
```


#### Common use of masking

##### Extract Lower Bits

```c
// အောက်ဆုံး 4 bits ကိုယူမယ်
uint8_t value = 0xAB;  // 10101011
uint8_t mask  = 0x0F;  // 00001111
uint8_t result = value & mask;  // 00001011 (0x0B)
```

real world example: RGB color ထဲက Blue component ကိုယူတာ
```c
uint32_t rgb = 0x00FF80AA;  // R=0xFF, G=0x80, B=0xAA
uint8_t blue = rgb & 0xFF;   // 0xAA
```

##### Extract Upper Bit

```c
// အပေါ် 4 bits ကိုယူမယ်
uint8_t value = 0xAB;  // 10101011
uint8_t result = value & 0xF0;  // 10100000
```
ဒါပေမယ့် အပေါ် bits ကို shift လုပ်ဖို့လိုတယ်
```c
uint8_t upper_nibble = (value & 0xF0) >> 4;  // 1010 (0xA)
```


##### Check if a bit is set

```c
uint8_t value = 0xAB;  // 10101011

// bit 3 (4th bit) က 1 လား?
if (value & 0x08) {  // 0x08 = 00001000 (bit 3 ကိုစစ်)
    printf("Bit 3 is set!\n");
}
```

##### Set/Clear Bits

```c
uint8_t value = 0xAB;

// bit 2 ကို 1 ထားမယ် (set)
value |= 0x04;   // 0x04 = 00000100

// bit 4 ကို 0 ထားမယ် (clear)
value &= ~0x10;  // 0x10 = 00010000, ~0x10 = 11101111
```


| Mask         | Binary        | usage                       |
| ------------ | ------------- | --------------------------- |
| `0x1`        | `00000001`    | အောက်ဆုံး 1 bit             |
| `0x3`        | `00000011`    | အောက်ဆုံး 2 bits            |
| `0x7`        | `00000111`    | အောက်ဆုံး 3 bits            |
| `0xF`        | `00001111`    | အောက်ဆုံး 4 bits (nibble)   |
| `0xFF`       | `11111111`    | အောက်ဆုံး 8 bits (1 byte)   |
| `0xFFFF`     | 16 bits အကုန် | အောက်ဆုံး 16 bits (2 bytes) |
| `0xFFFFFFFF` | 32 bits အကုန် | အောက်ဆုံး 32 bits (4 bytes) |


### Truncation

Truncation ဆိုတာ data တစ်ခုရဲ့ အရွယ်အစားကို လျှော့ချတာကိုခေါ်တယ်
ပိုကြီးတဲ့ data type (64-bit) ကနေ ပိုသေးတဲ့ data type (32-bit) ကို ပြောင်းတဲ့အခါ အပေါ်ပိုင်း bits တွေကို ဖြတ်ပစ်လိုက်တယ်

```
ပုံကြီး → အပိုင်းအစဖြတ် → ပုံသေး
(64-bit)    (upper bits ပျက်)   (32-bit)
```

 for simple example: 4-bit ကနေ 2-bit
```
Original 4-bit: 1011 (11 in decimal)
                  ↑↑
Truncate to 2-bit: 11 (3 in decimal)
                     ↑ အောက် 2-bit ပဲကျန်
```

##### example 1: 32-bit ကနေ 8-bit ကိုဖြတ် 
```c
int large = 0x12345678;  // 32-bit value
char small = large;       // 8-bit truncation

printf("large: 0x%08X\n", large);   // 0x12345678
printf("small: 0x%02X\n", small);   // 0x78 (အောက် 8-bit ပဲကျန်)
```

##### example 2: 64-bit ကနေ 32-bit ကိုဖြတ်
```c
long long big = 0x123456789ABCDEF0;  // 64-bit
int medium = big;                     // 32-bit truncation

printf("big:    0x%016llX\n", big);     // 0x123456789ABCDEF0
printf("medium: 0x%08X\n", medium);     // 0x9ABCDEF0 (အောက် 32-bit)
```


##### When Truncation Happens

| Situation               | example Code                  | explanation               |
| ----------------------- | ----------------------------- | ------------------------- |
| **Implicit Conversion** | `int x = 300; char c = x;`    | compiler က auto truncate  |
| **Explicit Cast**       | `char c = (char)300;`         | programmer က တမင်လုပ်     |
| **Function Argument**   | `void foo(char c); foo(300);` | argument ကို truncate     |
| **Return Statement**    | `char foo() { return 300; }`  | return value ကို truncate |
| **Overflow**            | `char c = 300;`               | value က range ထက်ကြီး     |

#### Side Effects

##### Data Loss 

```c
int big = 0xFFFFFFFF;   // 4,294,967,295
short small = big;       // truncate to 16-bit

printf("%d\n", small);   // -1 (ဘာဖြစ်သွားလဲ?)
```
Memory ထဲမှာ
```
32-bit big:  [0xFF] [0xFF] [0xFF] [0xFF]
              ↑ MSB            ↑ LSB

Truncate to 16-bit (အောက် 2 bytes ပဲယူ):
16-bit small:            [0xFF] [0xFF] = -1
```

##### Sign Change

```c
unsigned int u = 0xFFFFFFFF;   // 4,294,967,295 (unsigned)
int s = u;                      // truncation? မဟုတ်ဘူး! conversion ပြောင်း

printf("%d\n", s);              // -1 (အနုတ်ဖြစ်သွား)
```
ဒါက type conversion ဖြစ်တယ် — truncation မဟုတ်ဘူး (size အတူတူ)။

#####  Wrap Around 
```c
unsigned char c = 300;  // 300 က range (0-255) ထက်ကြီး
printf("%u\n", c);      // 44 (300 - 256 = 44)
```
တွက်နည်း `result = value % (max + 1)`  
`300 % 256 = 44`



> [!NOTE]
> Truncation က တစ်ခါတစ်လေ လိုအပ်ပေမယ့် သတိထားရမယ် — data loss ဖြစ်နိုင်တယ်၊ security vulnerability ဖြစ်နိုင်တယ်။ ဖြစ်နိုင်ရင် maskingကို သုံးပြီး explicit လုပ်



###  Sign Extension

 Sign Extension ဆိုတာ signed integer (အနုတ်အပေါင်းပါတဲ့ ကိန်း) ကို bit အရေအတွက် ပိုကြီးတဲ့ type ကို တန်ဖိုးမပြောင်းလဲဘဲ ပြောင်းတဲ့ နည်းလမ်း

MSB (Most Significant Bit) ကို အပေါ်ဘက် bits တွေဆီကို ပွားကူးယူတယ် (copy)

```
မူရင်း MSB = 0 → Positive → အပေါ်ကို 0 ဖြည့်
မူရင်း MSB = 1 → Negative → အပေါ်ကို 1 ဖြည့်
```


#####  8-bit → 32-bit

 Positive Number (+42)
```c
8-bit +42:  00101010  (MSB = 0 → positive)

Sign Extension (MSB 0 ကို အပေါ်ကို ပွား):
32-bit: 00000000 00000000 00000000 00101010
        ↑ 0 တွေ အကုန်ဖြည့် ↑

Result: +42 (တန်ဖိုးအတူတူ)
```

 Negative Number (-42)
```c
8-bit -42:  11010110  (MSB = 1 → negative)

Sign Extension (MSB 1 ကို အပေါ်ကို ပွား):
32-bit: 11111111 11111111 11111111 11010110
        ↑ 1 တွေ အကုန်ဖြည့် ↑

Result: -42 (တန်ဖိုးအတူတူ)
```

```c
8-bit -128 (min):  10000000
Sign Extension → 11111111 11111111 11111111 10000000 (32-bit -128)

8-bit +127 (max):  01111111
Sign Extension → 00000000 00000000 00000000 01111111 (32-bit +127)
```

C code
```c
#include <stdio.h>
#include <stdint.h>

int main() {
    // Signed (sign extension)
    int8_t  s8  = -42;
    int32_t s32 = s8;  // sign extension
    
    // Unsigned (zero extension)
    uint8_t  u8  = 214;  // 214 = -42 ရဲ့ unsigned version?
    uint32_t u32 = u8;   // zero extension
    
    printf("=== Sign Extension ===\n");
    printf("s8 (8-bit -42)    = 0x%02X\n", (uint8_t)s8);   // 0xD6
    printf("s32 (sign ext)    = 0x%08X = %d\n", s32, s32); // 0xFFFFFFD6 = -42
    
    printf("\n=== Zero Extension ===\n");
    printf("u8 (8-bit 214)    = 0x%02X\n", u8);            // 0xD6
    printf("u32 (zero ext)    = 0x%08X = %u\n", u32, u32); // 0x000000D6 = 214
    
    return 0;
}
```

output
```
=== Sign Extension ===
s8 (8-bit -42)    = 0xD6
s32 (sign ext)    = 0xFFFFFFD6 = -42

=== Zero Extension ===
u8 (8-bit 214)    = 0xD6
u32 (zero ext)    = 0x000000D6 = 214
```


#### Hardware Implementation

CPU မှာ MOVSX (Move with Sign Extension) ဆိုတဲ့ instruction ရှိတယ်
```
; x86 assembly
mov al, 0xFF        ; al = 0xFF (-1)
movsx ebx, al       ; ebx = 0xFFFFFFFF (-1) (sign extension)
movzx ecx, al       ; ecx = 0x000000FF (255)   (zero extension)
```



### Zero Extension

Zero Extension ဆိုတာ unsigned integer (အနုတ်မပါတဲ့ ကိန်း) ကို bit အရေအတွက် ပိုကြီးတဲ့ type ကို အပေါ်ဘက် bits တွေကို 0 ဖြည့်ပြီး ပြောင်းတဲ့ နည်းလမ်း


```
8-bit 255:  11111111  (MSB = 1)

Zero Extension:
           အောက် 8-bit ကို ထားပြီး အပေါ် 8-bit ကို 0 ဖြည့်

16-bit 255: 00000000 11111111
            ↑ အပေါ်ကို 0 ↑

Result: 255 (တန်ဖိုးအတူတူ)
```
MSB က 1 ဖြစ်နေပေမယ့် အပေါ်ကို 0 ဖြည့်



```
8-bit 42:    00101010

Zero Extension:
32-bit: 00000000 00000000 00000000 00101010
        ↑ အကုန် 0 ↑

Result: 42 (အတူတူ)
```

```
8-bit 200:   11001000

Zero Extension:
32-bit: 00000000 00000000 00000000 11001000
        ↑ အကုန် 0 ↑

Result: 200 (အတူတူ)
```


C code
```c
#include <stdio.h>
#include <stdint.h>

int main() {
    // Zero Extension (unsigned)
    uint8_t  u8  = 0xFF;      // 255
    uint32_t u32 = u8;        // zero extension
    
    // Sign Extension (signed) — နှိုင်းယှဉ်ဖို့
    int8_t   s8  = 0xFF;      // -1
    int32_t  s32 = s8;        // sign extension
    
    printf("=== Zero Extension ===\n");
    printf("u8 (8-bit)  = 0x%02X = %u\n", u8, u8);      // 0xFF = 255
    printf("u32 (32-bit) = 0x%08X = %u\n", u32, u32);   // 0x000000FF = 255
    
    printf("\n=== Sign Extension (for comparison) ===\n");
    printf("s8 (8-bit)  = 0x%02X = %d\n", (uint8_t)s8, s8);  // 0xFF = -1
    printf("s32 (32-bit) = 0x%08X = %d\n", s32, s32);        // 0xFFFFFFFF = -1
    
    return 0;
}
```

output
```
=== Zero Extension ===
u8 (8-bit)  = 0xFF = 255
u32 (32-bit) = 0x000000FF = 255

=== Sign Extension (for comparison) ===
s8 (8-bit)  = 0xFF = -1
s32 (32-bit) = 0xFFFFFFFF = -1
```

#### Why Need Zero Extension?

##### To keep Unsigned value

```c
unsigned char uc = 255;
unsigned int ui = uc;  // zero extension ကြောင့် ui = 255
```

##### For Array Index 

```c
long long user_input = -1;
unsigned int index = user_input & 0xffffffff;  // zero extension
// index = 4294967295 (positive)
```

##### For Memory Address 

```c
void *ptr = (void*)0x80000000;
unsigned long addr = (unsigned long)ptr;  // zero extension
```


#### Implementation


##### Implicit Conversion (Compiler ကလုပ်)

```c
unsigned char small = 200;
unsigned int large = small;  // compiler က zero extension လုပ်တယ်
```

##### Explicit Casting

```c
unsigned int large = (unsigned int)small;
```

#####  Masking 

```c
unsigned int truncated = value & 0xFFFFFFFF;
```

##### CPU Instruction (MOVZX)

```assembly
; x86 assembly
mov al, 0xFF        ; al = 0xFF
movzx ebx, al       ; ebx = 0x000000FF (zero extension)
```


#### Sign Extension vs Zero Extension 

Positive အတွက် sign extension နဲ့ zero extension တူတယ်

| Feature               | Sign Extension                                        | Zero Extension                                     |
| --------------------- | ----------------------------------------------------- | -------------------------------------------------- |
| ဘယ်အခါသုံးလဲ          | Signed integer ကို ပိုကြီးတဲ့ signed type သို့        | Unsigned integer ကို ပိုကြီးတဲ့ unsigned type သို့ |
| ဘယ်လိုဖြည့်လဲ         | **Sign bit (MSB)** ကို အပေါ်က bits တွေဆီ ကူးယူ (copy) | **0** တွေ ဖြည့်ပေး                                 |
| တန်ဖိုး ထိန်းသိမ်းမှု | တူညီတဲ့ signed value ကို ထိန်းသိမ်း                   | တူညီတဲ့ unsigned value ကို ထိန်းသိမ်း              |
| သင်္ချာအဓိပ္ပာယ်      | `-1` → `-1` (အတူတူပဲ)                                 | `255` → `255` (အတူတူပဲ)                            |
| အသုံးပြုပုံ           | `int8_t → int32_t`                                    | `uint8_t → uint32_t`                               |

| Sign Extension | Zero Extension                |                                  |
| -------------- | ----------------------------- | -------------------------------- |
| သုံးတဲ့နေရာ    | Signed types                  | Unsigned types                   |
| ဖြည့်တဲ့ပုံ    | Sign bit ကို copy             | 0 ဖြည့်                          |
| Negative value | ထိန်းသိမ်းတယ် (e.g., -1 → -1) | ပျက်စီးသွားတယ် (e.g., -1 → 255)  |
| Positive value | ထိန်းသိမ်းတယ်                 | ထိန်းသိမ်းတယ်                    |
| Assembly       | `MOVSX` / `MOVSXD`            | `MOVZX`                          |
| Use case       | သင်္ချာ၊ signed arithmetic    | Memory address, bit manipulation |

---


##### To learn

| Category          | Operation                | လုပ်ဆောင်ချက်                           |
| ----------------- | ------------------------ | --------------------------------------- |
| **Extraction**    | Masking                  | Specific bits ကို ထုတ်ယူ                |
|                   | Truncation               | Upper bits ဖြတ်ပစ်                      |
|                   | Sign Extension           | Bit width တိုးတဲ့အခါ sign ထိန်း         |
|                   | Zero Extension           | Bit width တိုးတဲ့အခါ 0 ဖြည့်            |
| **Movement**      | Left Shift               | ဘယ်ဘက်ရွှေ့ (×2)                        |
|                   | Right Shift (Logical)    | ညာဘက်ရွှေ့ (÷2, unsigned)               |
|                   | Right Shift (Arithmetic) | ညာဘက်ရွှေ့ (÷2, signed, sign preserved) |
|                   | Rotation                 | လှည့်ပတ်                                |
| **Modification**  | Set / Clear / Toggle     | တစ်နေရာချင်းစီ ပြောင်း                  |
| **Conversion**    | Integer Promotion        | Small → int                             |
|                   | Implicit Conversion      | Compiler အလိုအလျောက်                    |
|                   | Explicit Cast            | လူက တိုက်ရိုက်ပြောင်း                   |
| **Overflow**      | Wrap-around              | Unsigned overflow                       |
|                   | Saturation               | Clipping to min/max                     |
|                   | Undefined Behavior       | Signed overflow                         |
| **Memory Layout** | Endianness               | Byte order                              |
|                   | Padding                  | Structure alignment                     |
|                   | Alignment                | Stack frame alignment                   |

Level 1: Data Representation (အခြေခံ)
   └─ Binary, Hex, Decimal
   └─ Bits and Bytes
   
Level 2: Integer Representation (အရေးကြီးဆုံး)
   └─ Signed vs Unsigned
   └─ Two's Complement ← မင်းရှုပ်နေတာက ဒီမှာ!
   └─ Sign Extension, Zero Extension
   
Level 3: Memory and Pointers
   └─ Array access, Pointer arithmetic
   └─ 64-bit vs 32-bit
   
Level 4: Security
   └─ Integer overflow attacks
   └─ Masking as defense



**လက်တွေ့ programming မှာ ဒါတွေလည်း သိထားရမယ်**:

1. **Bit Shifting** (logic vs arithmetic right shift)
    
2. **Integer Promotion** (C language ရဲ့ ထူးခြားချက်)
    
3. **Padding/Alignment** (memory layout အတွက်)
    
4. **Endianness** (networking, binary file parsing)