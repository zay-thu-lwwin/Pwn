
1 byte = 8bit
32 bit system  = 4 byte
x86 system
(CPU က တစ်ကြိမ်မှာ 4 byte ကို process လုပ်နိုင်)

64 bit system = 8 byte  
x64 system
(CPU က တစ်ကြိမ်မှာ 8 byte ကို process လုပ်နိုင်)



|Type|Size (bytes)|Example Value|Stack Alignment|
|---|---|---|---|
|`char`|1|`'A'` (0x41)|1-byte|
|`unsigned char`|1|`255` (0xFF)|1-byte|
|`bool`|1|`1` (0x01)|1-byte|
|`short`|2|`0x1234`|2-byte|
|`unsigned short`|2|`0xFFFF`|2-byte|
|`int`|4|`0x12345678`|4-byte|
|`unsigned int`|4|`0xFFFFFFFF`|4-byte|
|`float`|4|`3.14` (0x4048F5C3)|4-byte|
|`long`|8|`0x123456789ABCDEF0`|8-byte|
|`unsigned long`|8|`0xFFFFFFFFFFFFFFFF`|8-byte|
|`long long`|8|`0x123456789ABCDEF0`|8-byte|
|`double`|8|`3.14159`|8-byte|
|`void*` (pointer)|8|`0x7fffffffdb00`|8-byte|
|`size_t`|8|`0x64` (100)|8-byte|

#### input
ငါတို့ keyboard ကနေ စာရိုက်တယ်
ရီုက်တဲ့ကောင်တွေက ဘာရိုက်ရိုက် ASCII ဘဲ
အခြားဘာသာစကားတွေကို တစ်ညီတစ်ညွတ်တည်းသုံးဖို့  UTF-8 (Unicode Transformation Format - 8 bit) ကိုသုံးတယ်
ASCII က 'A' = 0x41
UTF-8 က 'A' = 0x41 (အတူတူပဲ)
ASCII file တိုင်းက UTF-8 file လည်းဖြစ်တယ်
UTF-8 က စာလုံးတစ်လုံးကို 1 byte ကနေ 4 bytes နဲ့ ကိုယ်စားပြု

| Bytes       | Pattern                               | what for                 | example                      |
| ----------- | ------------------------------------- | ------------------------ | ---------------------------- |
| **1 byte**  | `0xxxxxxx`                            | ASCII (U+0000 to U+007F) | 'A' → `01000001`             |
| **2 bytes** | `110xxxxx 10xxxxxx`                   | U+0080 to U+07FF         | 'ع' (Arabic)                 |
| **3 bytes** | `1110xxxx 10xxxxxx 10xxxxxx`          | U+0800 to U+FFFF         | 'မ' (Burmese), '中' (Chinese) |
| **4 bytes** | `11110xxx 10xxxxxx 10xxxxxx 10xxxxxx` | U+10000 to U+10FFFF      | '😀' (Emoji)                 |

![](./pictures/ASCII.GIF)



ဆိုတော့ ဆက်ပြောရရင်
ငါတို့က  input ကို ဘာဘဲရိုက်ရိုက် ASCII or UTF-8 အနေနဲ့ဝင်သွားတယ်
ပြီးမှ program က လိုအပ်သလို data type ပြောင်းလဲသတ်မှတ်တယ်

နောက်တစ်ခု က Convension
စာသားပုံစံလိုမျိုးမဟုတ်ဘဲ calculation တွေတွက်ဖို့ဆိုရင် Convension ကိုသုံးတယ်
123 ဆိုပြီး ရိုက်ပေမယ့် ဒီကောင်က char မဟုတ်ဘဲ အခြား calculation data type အတွက်ဆိုရင်
123 (decimal /base 10 ) ကို base (16) အဖြစ်ပြောင်းပြီး သူ့ type ပုံစံအတိုင်းသိမ်းမယ်





မင်း ထည့်တဲ့ data (123) က အခြား calculation  type ဆိုရင်

| Type    | Size (bytes) | Range                           | 123 သိမ်းပုံ                            |
| ------- | ------------ | ------------------------------- | --------------------------------------- |
| `char`  | 1            | -128 to 127                     | 0x7B                                    |
| `short` | 2            | -32,768 to 32,767               | 0x7B 0x00                               |
| `int`   | 4            | -2B to 2B                       | 0x7B 0x00 0x00 0x00                     |
| `long`  | 8            | -9 quintillion to 9 quintillion | 0x7B 0x00 0x00 0x00 0x00 0x00 0x00 0x00 |


01111111 = 127 (အမြင့်ဆုံး)
10000000 = -128 (အနိမ့်ဆုံး)
10000001 = -127