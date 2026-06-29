
1 byte = 8bit
1 nibble = 4 bit
so 1 byte = 2 nibble 
1 nibble = 1 hex character (0-9) or (A-F)
0000 -> 11111 = 0 -> F

32 bit system  = 4 byte
x86 system
(CPU က တစ်ကြိမ်မှာ 4 byte ကို process လုပ်နိုင်)

64 bit system = 8 byte  
x64 system
(CPU က တစ်ကြိမ်မှာ 8 byte ကို process လုပ်နိုင်)


| Type              | 32-bit Size (bytes) | 64-bit Size (bytes) | Example Value                                         | Stack Alignment |
| ----------------- | ------------------- | ------------------- | ----------------------------------------------------- | --------------- |
| `char`            | 1                   | 1                   | `'A'` (0x41)                                          | 1-byte          |
| `unsigned char`   | 1                   | 1                   | `255` (0xFF)                                          | 1-byte          |
| `bool`            | 1                   | 1                   | `1` (0x01)                                            | 1-byte          |
| `short`           | 2                   | 2                   | `0x1234`                                              | 2-byte          |
| `unsigned short`  | 2                   | 2                   | `0xFFFF`                                              | 2-byte          |
| `int`             | 4                   | 4                   | `0x12345678`                                          | 4-byte          |
| `unsigned int`    | 4                   | 4                   | `0xFFFFFFFF`                                          | 4-byte          |
| `float`           | 4                   | 4                   | `3.14` (0x4048F5C3)                                   | 4-byte          |
| `long`            | **4**               | **8**               | `0x12345678` (32-bit) / `0x123456789ABCDEF0` (64-bit) | 4/8-byte        |
| `unsigned long`   | **4**               | **8**               | `0xFFFFFFFF` (32-bit) / `0xFFFFFFFFFFFFFFFF` (64-bit) | 4/8-byte        |
| `long long`       | 8                   | 8                   | `0x123456789ABCDEF0`                                  | 8-byte          |
| `double`          | 8                   | 8                   | `3.14159`                                             | 8-byte          |
| `void*` (pointer) | **4**               | **8**               | `0x7fffffff` (32-bit) / `0x7fffffffdb00` (64-bit)     | 4/8-byte        |
| `size_t`          | **4**               | **8**               | `0x64` (100)                                          | 4/8-byte        |
#### input

ငါတို့ keyboard ကနေ စာရိုက်တယ်
ရီုက်တဲ့ကောင်တွေက ဘာရိုက်ရိုက် ASCII ဘဲ
ASCII ဆိုတာ `character တွေကို ဂဏန်းနဲ့ ကိုယ်စားပြုတာ`
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

![](ASCII.gif)



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

example, for char byte
Signed char (Two's complement):
00000000 = 0
00000001 = 1
...
01111111 = 127 (အမြင့်ဆုံး)

10000000 = -128 (အနိမ့်ဆုံး)
10000001 = -127
...
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

#### byte-by-byte

ဆိုတော့ နောက်ထပ် တကယ် stack မှာ addressတွေ ကို ဘယ်လိုသိမ်းလဲ
```stack
0d:0068│-008     0x7fffffffdb68 ◂— 0xaca2f1b1956a3100
0e:0070│ rbp     0x7fffffffdb70 —▸ 0x7fffffffdb80 ◂— 1
0f:0078│+008     0x7fffffffdb78 —▸ 0x555555555413 (main+18) ◂— mov eax, 0
10:0080│+010     0x7fffffffdb80 ◂— 1
11:0088│+018     0x7fffffffdb88 —▸ 0x7ffff7c29f68 (__libc_start_call_main+120) ◂— mov edi, eax
12:0090│+020     0x7fffffffdb90 ◂— 0

ဒီနေရာမှာတော့ debugger က little endain အနေနဲ့မပြဘဲရိုးရိုးဖတ်လို့ရအောင်ပြပေးတယ်
```

ဒီမှာ  0x7fffffffdb78 မှာ 0x555555555413 ကိုသိမ်းထားတယ်ဆိုပါဆို့ 
တကယ်တမ်းက 0x7fffffffdb78 ကနေ 0x7fffffffdb80 အထိ 8 byte စာအတွက် ယူတယ်
အစောကပြောတဲ့အတိုင်း little endian နဲ့သိမ်မယ်
break down လုပ်မယ်

```
Value: 0x555555555413 (8 bytes)

Byte breakdown:
0x55 0x55 0x55 0x55 0x55 0x41 0x30
```

ပြီးရင် little endian အတိုင်းသိမ်းမယ်
ဆိုတော့ဒီအတိုင်းဘဲဖြစ်မယ်

| Address        | Byte Value   |
| -------------- | ------------ |
| 0x7fffffffdb78 | `0x13` (LSB) |
| 0x7fffffffdb79 | `0x54`       |
| 0x7fffffffdb7a | `0x55`       |
| 0x7fffffffdb7b | `0x55`       |
| 0x7fffffffdb7c | `0x55`       |
| 0x7fffffffdb7d | `0x55`       |
| 0x7fffffffdb7e | `0x55`       |
| 0x7fffffffdb7f | `0x55`       |




---


##### Control Characters (ASCII 0-31)

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
Terminal က control characters တွေကို သူ့ရဲ့ function အဖြစ် သုံးတယ်၊ input အဖြစ် မပို့ဘူး


### Keyboard Input Flow

```
Keyboard မှာ စာရိုက်
         ↓
[Kernel: TTY Driver]   ← ဒီမှာ raw scan code → ASCII ပြောင်း
         ↓
[Line Discipline]      ← ဒီမှာ processing (cooked/raw mode)
         ↓
[Program's stdin]      ← program လက်ခံတယ်
         ↓
[Program's parsing]    ← strcpy, gets, scanf, etc.
         ↓
[Memory/Segfault]      ← exploit ဖြစ်တယ်/မဖြစ်ဘူး
```

```
Terminal Emulator (xterm, gnome-terminal, ssh)
         ↓
   Pseudo-Terminal (PTY)
         ↓
    TTY Driver (kernel)
         ↓
   Line Discipline (kernel)
         ↓
       Program
```

#### TTY System (TeleTYpewriter) 

**TTY** က သင်ရိုက်တဲ့ စာတွေကို စုဆောင်းပြီး program ဆီ ပို့ပေးတဲ့ ကြားထဲက process
Linux မှာ terminal input/output ကို စီမံခန့်ခွဲတဲ့ kernel subsystem

`Hello<backspace>` လို့ရိုက်လိုက်ရင် terminal မှာ `Hello\x0f` လို့ဝင်သွားတယ်
အဲ့အခါ **\x0f** ကြောင့် ရှေ့တစ်လုံးကို tty က ဖြတ်ပေးသွား
`Hell`
ပြီးမှ program ကို ပို့ပေး

ဘယ်လို bypass မလဲဆို `\x16` _(escape character)_ သုံး
`\x16` နောက်မှာပါတဲ့ byte က Control character အနေနဲ့ အလုပ် မလုပ်တော့

```
┌─────────────────────────────────────────────────────────────┐
│                      KERNEL SPACE                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                    TTY DRIVER                        │    │
│  │  (Hardware abstraction layer)                       │    │
│  │  • Keyboard/Mouse events ကို လက်ခံတယ်                │    │
│  │  • Raw bytes/scancodes တွေကို စုဆောင်းတယ်            │    │
│  │  • Hardware-specific processing                      │    │
│  └─────────────────────┬───────────────────────────────┘    │
│                        ↓                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              LINE DISCIPLINE (ldisc)                │    │
│  │  (Protocol/processing layer)                        │    │
│  │  • Character processing (backspace, erase)          │    │
│  │  • Signal generation (Ctrl+C, Ctrl+Z)               │    │
│  │  • Line editing (canonical vs raw)                  │    │
│  │  • Echo control                                     │    │
│  └─────────────────────┬───────────────────────────────┘    │
│                        ↓                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              USER SPACE (Program)                    │    │
│  │  • read() → processed bytes                         │    │
│  │  • strcpy(), gets(), etc.                           │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```


> TTY Driver = Hardware layer (raw data)  
> Line Discipline = Processing layer (ဘယ်လို interpret လုပ်မလဲ)


#### Line Discipline

Line discipline is a MODULE that ATTACHES to a TTY driver

```
Keyboard/Mouse Input
         ↓
   [TTY Driver]
         ↓
[Line Discipline] ← Canonical Mode သို့ Non-Canonical Mode ရွေးချယ်နိုင်
         ↓
      Program
```


```
TTY Driver (Hardware layer):
  - Handles: Serial ports, USB keyboards, virtual terminals
  - Doesn't care: What you do with the data
  - Can be: uart, usb, pty, console

Line Discipline (Processing layer):
  - Handles: Character processing, signals, line editing
  - Doesn't care: How data arrives (serial, USB, network)
  - Can be: n_tty (default), n_null (null), custom
```


Line discipline  has modes 
```
You can change line discipline without changing TTY driver:
  stty raw    → n_tty in raw mode
  stty cooked → n_tty in canonical mode
  
But you CANNOT change TTY driver without changing hardware!
```

#### Raw Mode
Terminal က ရိုက်လိုက်တဲ့ ဘာမဆိုကို program ဆီ တန်းပို့ပေးတယ် ဘာ processing မှ မလုပ်ဘူး

```shell
$ stty raw
```

- ဘာ processing မှမရှိ
- ခလုတ်တိုင်းကို program ဆီ တန်းပို့
- Signal chars တောင် raw byte အဖြစ်ပို့

```
# သင်ရိုက်တယ်: Ctrl+C
# Raw mode: program ဆီ 0x03 ရောက်တယ် (signal မပို့ဘူး)
# Program က 0x03 ကို data အဖြစ်ယူတယ်
```


#### Cooked Mode (Normal) 
Terminal က special keys တွေကို ကြိုစစ်ပြီး သူ့ဘာသာ အဓိပ္ပာယ်ဖွင့်တယ်

```bash
 $ stty cooked
```

- Line buffer ထဲမှာ စုတယ် (Enter နှိပ်မှ program ဆီပို့)
- Backspace, arrows ကို interpret လုပ်
- Signal chars (Ctrl+C, Ctrl+Z) ကို catch လုပ်

```
# သင်ရိုက်တယ်: "hello\x08\x08world"  (two backspaces)
# Line discipline: "hellworld" (ပြင်ပေးတယ်)
# Program ဆီရောက်တာ: "hellworld\n"
```



|Feature|Cooked Mode (ပုံမှန်)|Raw Mode|
|---|---|---|
|**Ctrl+C**|SIGINT signal ပို့တယ်|`0x03` byte အဖြစ်ရောက်တယ်|
|**Ctrl+Z**|SIGTSTP signal ပို့တယ်|`0x1A` byte အဖြစ်ရောက်တယ်|
|**Backspace**|Character ကိုဖျက်တယ်|`0x7F` byte အဖြစ်ရောက်တယ်|
|**Enter**|`\n` (0x0A) ပို့တယ်|`\r` (0x0D) ပို့တယ်|
|**Line editing**|ရှိတယ် (backspace, arrows)|မရှိဘူး (program ကိုယ်တိုင်လုပ်ရတယ်)|
|**Echo**|သင်ရိုက်တာ screen ပေါ်ပြတယ်|မပြဘူး (program ကိုယ်တိုင်ပြရတယ်)|

#### Cbreak Mode (Raw နဲ့ Cooked ကြား)

```bash
$ stty cbreak
```

- Signal processing ရှိတယ် (Ctrl+C အလုပ်လုပ်တယ်)
- ဒါပေမယ့် line editing မရှိဘူး
- ခလုတ်တိုင်းကို ချက်ချင်းပို့တယ်
- Echo ရှိတယ်

##### Terminal Modes Comparison Table

|Feature|Cooked|Cbreak|Raw|
|---|---|---|---|
|**Line editing**|✅|❌|❌|
|**Enter မနှိပ်ခင် data**|မပို့ဘူး|ချက်ချင်းပို့|ချက်ချင်းပို့|
|**Signal processing** (Ctrl+C)|✅|✅|❌|
|**Echo (display typed chars)**|✅|✅|❌|
|**Backspace editing**|✅|❌|❌|
|**Use case**|Shell, simple programs|Password input, games|Vim, editors, games|

#### Bypass Techniques

```
# Problem: TTY in cooked mode, Ctrl+C (0x03) sends signal
# Solution: Escape with Ctrl+V (0x16)

# Without bypass:
payload = b"\x03"  # Ctrl+C
# TTY sees: SIGINT → kills program!

# With bypass:
payload = b"\x16\x03"  # Ctrl+V then Ctrl+C
# TTY sees: 0x16 (literal next) → passes 0x03 as data
# Program receives: 0x03 (just a byte, no signal)
```

| Key         | Hex  | Name | TTY Action       | Can Bypass with 0x16?    | Is it Bad for Program? |
| ----------- | ---- | ---- | ---------------- | ------------------------ | ---------------------- |
| Ctrl+@      | 0x00 | NUL  | None             | Yes → becomes data       | **YES (strcpy kills)** |
| Ctrl+A      | 0x01 | SOH  | Line start       | Yes → becomes data       | Depends on program     |
| Ctrl+B      | 0x02 | STX  | Cursor left      | Yes → becomes data       | Depends                |
| **Ctrl+C**  | 0x03 | ETX  | **SIGINT**       | Yes → becomes data       | Depends                |
| **Ctrl+D**  | 0x04 | EOT  | **EOF**          | Yes → becomes data       | Depends                |
| Ctrl+E      | 0x05 | ENQ  | Line end         | Yes                      | Depends                |
| Ctrl+F      | 0x06 | ACK  | Cursor right     | Yes                      | Depends                |
| Ctrl+G      | 0x07 | BEL  | Beep             | Yes                      | Depends                |
| **Ctrl+H**  | 0x08 | BS   | Backspace        | Yes → becomes 0x08       | Depends                |
| Ctrl+I      | 0x09 | TAB  | Tab              | Yes                      | **YES (scanf kills)**  |
| **Ctrl+J**  | 0x0A | LF   | Newline          | Yes → becomes data       | **YES (gets kills)**   |
| Ctrl+K      | 0x0B | VT   | Kill line        | Yes                      | Depends                |
| Ctrl+L      | 0x0C | FF   | Clear screen     | Yes                      | Depends                |
| **Ctrl+M**  | 0x0D | CR   | Carriage Return  | Yes → becomes data       | **YES (gets kills)**   |
| Ctrl+N      | 0x0E | SO   | Next history     | Yes                      | Depends                |
| Ctrl+O      | 0x0F | SI   | None             | Yes                      | Depends                |
| Ctrl+P      | 0x10 | DLE  | Prev history     | Yes                      | Depends                |
| **Ctrl+Q**  | 0x11 | DC1  | **XON**          | Yes                      | Depends                |
| Ctrl+R      | 0x12 | DC2  | Reverse search   | Yes                      | Depends                |
| **Ctrl+S**  | 0x13 | DC3  | **XOFF**         | Yes                      | Depends                |
| Ctrl+T      | 0x14 | DC4  | Transpose        | Yes                      | Depends                |
| Ctrl+U      | 0x15 | NAK  | Kill line start  | Yes                      | Depends                |
| **Ctrl+V**  | 0x16 | SYN  | **LITERAL NEXT** | N/A (this is the escape) | Depends                |
| Ctrl+W      | 0x17 | ETB  | Delete word      | Yes                      | Depends                |
| Ctrl+X      | 0x18 | CAN  | Various          | Yes                      | Depends                |
| Ctrl+Y      | 0x19 | EM   | Yank             | Yes                      | Depends                |
| **Ctrl+Z**  | 0x1A | SUB  | **SIGTSTP**      | Yes → becomes data       | Depends                |
| Ctrl+[      | 0x1B | ESC  | Escape           | Yes                      | Depends                |
| Ctrl+\|0x1C | FS   | Quit | Yes              | Depends                  |                        |
| Ctrl+]      | 0x1D | GS   | None             | Yes                      | Depends                |
| Ctrl+^      | 0x1E | RS   | None             | Yes                      | Depends                |
| Ctrl+_      | 0x1F | US   | None             | Yes                      | Depends                |
| Backspace   | 0x7F | DEL  | Delete char      | Yes                      | Depends                |


##### Commands

` Check Current TTY Settings`
```bash
# View all settings
stty -a

# Check if signals are enabled
stty -a | grep isig

# Check echo
stty -a | grep echo
```


` Change TTY Modes`
```
# Go raw (for exploit delivery)
stty raw

# Go back to cooked
stty cooked

# Disable just signals (keep line editing)
stty -isig

# Disable echo (for password/payload)
stty -echo
```

`Test TTY Bypass`
```python
# Test if TTY interprets Ctrl+C
python -c "print('\x03')" | cat -A
# Output: ^C (if TTY processes it)

# Test with escape
python -c "print('\x16\x03')" | cat -A
# Output: ^C (still? depends on TTY settings)
```



----




##### To learn

```
အဆင့် 1: TTY Basics
├── TTY ဆိုတာဘာလဲ
├── Canonical vs Raw Mode
├── stty commands
└── Line discipline

အဆင့် 2: Terminal Emulators
├── PTY (Pseudo Terminal) အလုပ်လုပ်ပုံ
├── Terminal types (xterm, vt100, vt520, etc.)
├── Terminfo database
└── TERM environment variable

အဆင့် 3: Character Encoding
├── ASCII
├── UTF-8 (မြန်မာ၊ တရုတ်၊ ဂျပန်)
├── Wide characters (wchar_t)
└── Unicode normalization

အဆင့် 4: Input Processing Chain
├── Kernel → TTY Driver → Line Discipline
├── Signal handling (SIGINT, SIGTSTP, SIGQUIT)
├── Job control (foreground/background)
└── Process groups and sessions

အဆင့် 5: Advanced Terminal Features
├── Escape sequences (ANSI, CSI)
├── Colors and formatting
├── Mouse input in terminal
├── Bracketed paste
└── Alternate screen buffer

အဆင့် 6: Programming Terminal Apps
├── termios.h (full API)
├── ncurses library
├── readline library
├── termcap/terminfo
└── Building TUI applications

အဆင့် 7: Remote Terminals
├── SSH PTY forwarding
├── Telnet (legacy)
├── Serial terminals (RS-232)
└── Network virtual terminals

အဆင့် 8: Security Aspects
├── Input injection attacks
├── Escape sequence vulnerabilities
├── Terminal hijacking
├── Shellshock and related
└── TTY sandboxing
```


---
