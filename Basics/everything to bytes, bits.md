
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
123 ဆိုရင် 0x7B




မင်း ထည့်တဲ့ data (123) က အခြား calculation  type ဆိုရင်

| Type (signed) | Size (bytes) | Range                           | 123 သိမ်းပုံ                            |
| ------------- | ------------ | ------------------------------- | --------------------------------------- |
| ` char`       | 1            | -128 to 127                     | 0x7B                                    |
| `short`       | 2            | -32,768 to 32,767               | 0x7B 0x00                               |
| `int`         | 4            | -2B to 2B                       | 0x7B 0x00 0x00 0x00                     |
| `long`        | 8            | -9 quintillion to 9 quintillion | 0x7B 0x00 0x00 0x00 0x00 0x00 0x00 0x00 |
**Range** ဆိုတာ အဲဒီ data type ထဲမှာ သိမ်းလို့ရတဲ့ အနိမ့်ဆုံးနဲ့ အမြင့်ဆုံး တန်ဖိုး
signed char ဆိုရင်

Signed char (Two's complement):
00000000 = 0
00000001 = 1

01111111 = 127 (အမြင့်ဆုံး)
10000000 = -128 (အနိမ့်ဆုံး)
10000001 = -127

11111111 = -1
Unsigned char:
00000000 = 0
00000001 = 1

11111110 = 254
11111111 = 255 (အမြင့်ဆုံး)


#### Endianness
**Endianness** က memory ထဲမှာ bytes တွေကို ဘယ်လိုအစီအစဉ်နဲ့ သိမ်းသလဲ ဆိုတာ ဖော်ပြ
4 bytes ရှိတဲ့ `0x12345678` မှာဆိုရင် 
0x12 က MSB (Most Significant Byte)
0x78 က LSB (Least Significant Byte)

Little Endian (x86, x86-64)
မှာဆိုရင် LSB က စသိမ်း
Big Endian  (Network, ARM some modes)
မှာဆိုရင် MSB က စသိမ်း

| Byte Position (Address)     | Value | Little Endian  | Big Endian     |
| --------------------------- | ----- | -------------- | -------------- |
| Byte 0 (အနိမ့်ဆုံး address) | 0x12  | **0x78** (LSB) | **0x12** (MSB) |
| Byte 1                      | 0x34  | **0x56**       | **0x34**       |
| Byte 2                      | 0x56  | **0x34**       | **0x56**       |
| Byte 3 (အမြင့်ဆုံး address) | 0x78  | **0x12** (MSB) | **0x78** (LSB) |

ဆိုတော့ int 0x7B ကိုသိမ်းချင်တယ်
int က 4bytes ဆိုတော့ 0x00 00 00 7B
Little Endian ဆိုတော့ 
memory မှာ 0x7B 0x00 0x00 0x00

---


| Key Combination | ASCII Code | Character Name            | Terminal Action          |
| --------------- | ---------- | ------------------------- | ------------------------ |
| **Ctrl + @**    | `0x00`     | NUL (Null)                | ဘာမှမလုပ်ဘူး             |
| **Ctrl + A**    | `0x01`     | SOH (Start of Heading)    | Cursor to line start     |
| **Ctrl + B**    | `0x02`     | STX (Start of Text)       | Cursor left              |
| **Ctrl + C**    | `0x03`     | ETX (End of Text)         | **SIGINT (Interrupt)**   |
| **Ctrl + D**    | `0x04`     | EOT (End of Transmission) | **EOF / Exit**           |
| **Ctrl + E**    | `0x05`     | ENQ (Enquiry)             | Cursor to line end       |
| **Ctrl + F**    | `0x06`     | ACK (Acknowledge)         | Cursor right             |
| **Ctrl + G**    | `0x07`     | BEL (Bell)                | Beep sound               |
| **Ctrl + H**    | `0x08`     | BS (Backspace)            | Delete previous char     |
| **Ctrl + I**    | `0x09`     | HT (Horizontal Tab)       | Tab                      |
| **Ctrl + J**    | `0x0A`     | LF (Line Feed)            | Newline (Enter)          |
| **Ctrl + K**    | `0x0B`     | VT (Vertical Tab)         | Kill line from cursor    |
| **Ctrl + L**    | `0x0C`     | FF (Form Feed)            | Clear screen             |
| **Ctrl + M**    | `0x0D`     | CR (Carriage Return)      | Enter                    |
| **Ctrl + N**    | `0x0E`     | SO (Shift Out)            | Next line in history     |
| **Ctrl + O**    | `0x0F`     | SI (Shift In)             |                          |
| **Ctrl + P**    | `0x10`     | DLE (Data Link Escape)    | Previous line in history |
| **Ctrl + Q**    | `0x11`     | DC1 (Device Control 1)    | **XON (Resume output)**  |
| **Ctrl + R**    | `0x12`     | DC2 (Device Control 2)    | Reverse search           |
| **Ctrl + S**    | `0x13`     | DC3 (Device Control 3)    | **XOFF (Pause output)**  |
| **Ctrl + T**    | `0x14`     | DC4 (Device Control 4)    | Transpose chars          |
| **Ctrl + U**    | `0x15`     | NAK (Negative Ack)        | Kill line before cursor  |
| **Ctrl + V**    | `0x16`     | SYN (Synchronous Idle)    | **Literal next char**    |
| **Ctrl + W**    | `0x17`     | ETB (End Trans. Block)    | Delete previous word     |
| **Ctrl + X**    | `0x18`     | CAN (Cancel)              | Various commands         |
| **Ctrl + Y**    | `0x19`     | EM (End of Medium)        | Yank killed text         |
| **Ctrl + Z**    | `0x1A`     | SUB (Substitute)          | **SIGTSTP (Suspend)**    |



Terminal က control characters တွေကို သူ့ရဲ့ function အဖြစ် သုံးတယ်၊ input အဖြစ် မပို့ဘူး!**
