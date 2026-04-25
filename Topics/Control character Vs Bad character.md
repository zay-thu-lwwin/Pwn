


```
                    COMPLETE INPUT FLOW (from keyboard)
                    ===================================

Keyboard: [Ctrl+V][Ctrl+C]
          ↓
TTY Driver: Raw scan codes → ASCII
          ↓
Line Discipline: 
   - Sees 0x16 (SYN) → "literal next" mode
   - Sees 0x03 → passes as data (not SIGINT)
          ↓
Program stdin: Receives b"\x16\x03"
          ↓
Program Function: (e.g., strcpy)
   - Copies b"\x16\x03" to buffer
   - No null byte → full copy
          ↓
Memory: Both bytes arrive intact
          ↓
Exploit: Works!

But if program had gets() instead of read():
   - gets() sees b"\x16\x03" (no newline)
   - Still works! (no \x0a or \x0d)
```


```
┌─────────────────────────────────────────────────────────┐
│                    Application Layer                    │
│  (strcpy, gets, scanf, JSON parser, XML parser, etc.)   │
│  → Bad Characters ကို program logic ကသတ်မှတ်တယ်        │
│  → ဒီ bypass က မဖြေရှင်းနိုင်ဘူး                              │
├─────────────────────────────────────────────────────────┤
│                    Transport Layer                      │
│  (VPN, Serial, TCP, Network filters, etc.)              │
│  → Bad Characters ကို data transmission ကသတ်မှတ်တယ်    │
│  → ဒီ bypass က ဖြေရှင်းနိုင်တယ်                             │
└─────────────────────────────────────────────────────────┘
```


#### Application Layer Problems (Bad Characters)

Program ရဲ့ code ထဲက function တွေ (strcpy, gets, scanf, custom parser) ကြောင့်ဖြစ်တဲ့ bad characters

##### strcpy() Problem
```c
char buffer[100];
strcpy(buffer, user_input);  // Stops at \x00
```

```python
payload = "AAA" + "\x00" + "BBB" + shellcode
# strcpy က AAA ပြီးတာနဲ့ ရပ်သွားတယ်
# BBB ရော shellcode ပါ မရောက်ဘူး
# Result: Exploit fails
```

```python
# Try to escape \x00
escaped = b"\x16\x00"  # Send this
# strcpy က \x16 ကိုမြင်တယ် → တစ်ခါတလေ \x16 ကိုလည်း string ရဲ့အစိတ်အပိုင်းအဖြစ်ယူတယ်
# ဒါပေမယ့် \x00 ကို ဘယ်လိုမှ မလွှဲနိုင်ဘူး
# strcpy က \x00 ရောက်တာနဲ့ ရပ်တယ်
# So: NO, can't bypass with this method
```

##### gets() Problem
```c
gets(buffer);  // Stops at \x0a (newline)
```

```python
payload = "AAA" + "\x0a" + "BBB" + shellcode
# gets က AAA ပြီးတာနဲ့ ရပ် (newline က input end အဖြစ်ယူ)
# Result: Exploit fails
```

```python
escaped = b"\x16\x0a"  # Send escape + newline
# gets() က \x16 ကိုမြင်တယ် (printable? no, it's control char)
# \x16 ကိုလည်း control char အဖြစ်ယူတယ်
# Still might stop or misinterpret
# So: NO, can't reliably bypass
```


#### Transport Layer Problems (Control Character Filtering)

Data က network/connection ကိုဖြတ်ကူးတဲ့အခါ ဖြစ်တဲ့ bad characters

##### VPN Control Character Filtering
```python
# VPN က security အတွက် control chars တွေကို strip လုပ်တယ်
original = b"\x31\xc0\x50\x68\x2f\x2f\x73\x68"  # Shellcode start
# VPN sees: 0x31 (printable), 0xc0 (control? maybe), etc.
# Some VPNs: drop any byte < 0x20
# Result: Shellcode corrupted

payload = b"\x0b"  # Vertical tab
# VPN drops this byte completely
# Result: Missing byte in payload
```

```python
# Escape the control char
escaped = b"\x16\x0b"  # Send this instead of \x0b
# VPN sees: 0x16 (allowed, >0x20? no, but some VPNs allow)
# 0x16 is also control char, but VPN might allow it as "escape"
# If VPN passes both bytes through:
# Target decoder: sees 0x16, knows next byte is literal
# Decodes back to: \x0b
# Result: YES, bypass works!
```
##### Serial Line Discipline
```bash
# Serial line က control chars တွေကို interpret လုပ်တယ်
echo -e "\x03" > /dev/ttyS0  
# 0x03 = Ctrl+C → sends interrupt signal!
# Not what we want

echo -e "\x11" > /dev/ttyS0
# 0x11 = XON (flow control) → may pause transmission!
```

```python
escaped = b"\x16\x03"  # Send Ctrl+C as escaped
# Serial line: sees 0x16 = literal next
# Doesn't interpret 0x03 as Ctrl+C
# Passes both bytes through
# Target: receives 0x03 as data, not as signal
# Result: YES, bypass works!
```

#### How to Solve Each

##### Application Layer Problem
```python
# Problem: strcpy() stops at \x00
# Solution 1: Use address without \x00
address = 0x080491a3  # No null bytes
payload = b'A'*offset + p32(address)
# Solution 2: XOR encode
key = 0x55
encoded = bytes([b ^ key for b in payload])
# Add decoder stub
```
##### Transport Layer Problem
```python
# Problem: VPN strips \x0a
# Solution: Escape encoding
if byte == 0x0a:
    payload += b"\x16\x0a"  # Escape it
else:
    payload += bytes([byte])
```



|Aspect|Application Layer Problem|Transport Layer Problem|
|---|---|---|
|**Source**|Program's code (strcpy, gets, etc.)|Network/VPN/Serial filtering|
|**When occurs**|During processing on target|During transmission|
|**Affects**|How program interprets bytes|Which bytes arrive|
|**Can bypass with 0x16?**|❌ No (unless program has decoder)|✅ Yes (if target has decoder)|
|**Solution**|Change payload (XOR, alphanum, ROP)|Escape encoding|
|**Example**|`\x00` kills strcpy()|`\x0a` stripped by VPN|
|**Testing**|Check with local gdb|Check with network capture|

| Layer                | Problem                                       | Bypass Method               |
| -------------------- | --------------------------------------------- | --------------------------- |
| **TTY**              | Control chars interpreted as signals/commands | **Escape with 0x16**        |
| **Program (strcpy)** | Null byte terminates string                   | **XOR encoding + decoder**  |
| **Program (gets)**   | Newline terminates input                      | **Use raw mode or encode**  |
| **Program (scanf)**  | Whitespace separates args                     | **Change input method**     |
| **Network/VPN**      | Control chars stripped                        | **Base64/Quoted-printable** |

```
┌─────────────────────────────────────────────────────────────┐
│  CONTROL CHARACTERS (ASCII Definition)                      │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  • Fixed by ASCII standard (1963)                           │
│  • 33 characters: 0x00-0x1F + 0x7F                         │
│  • Purpose: Control devices, formatting, communication      │
│  • Examples: NUL, SOH, STX, ETX, EOT, ENQ, ACK, BEL, etc.   │
│  • Same on EVERY system (Unix, Windows, embedded)          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  BAD CHARACTERS (Exploitation Definition)                   │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  • Dynamic (depends on program, input method, environment)  │
│  • ANY byte from 0x00 to 0xFF can be bad                    │
│  • Purpose: Bytes that break exploit payload                │
│  • Examples: \x00 (null), \x20 (space), \x3c (<), \x22 (")  │
│  • Changes per target                                       │
└─────────────────────────────────────────────────────────────┘
```

> Transport bypass fixes **how data travels**  
> Application fixes are still needed for **how data is processed**


---
