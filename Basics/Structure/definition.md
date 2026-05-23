

`String`

| String Type                                           | Storage          | can be Modify ? |
| ----------------------------------------------------- | ---------------- | --------------- |
| **String literal** (ဥပမာ `"hello"`)                   | `.rodata`        | ❌ မရဘူး (crash) |
| **Stack array** (ဥပမာ `char arr[] = "hello"`)         | **Stack**        | ✅ ရတယ်          |
| **Heap string** (ဥပမာ `malloc()` နဲ့ခွဲတာ)            | **Heap**         | ✅ ရတယ်          |
| **Global array** (ဥပမာ `char arr[] = "hello"` global) | **Data Segment** | ✅ ရတယ်          |

| Declaration | Location | Read-only? | Example |
|-------------|----------|------------|---------|
| `"literal"` (direct usage) | `.rodata` | ✅ Yes | `printf("hello")` |
| `char *p = "literal"` | `.rodata` | ✅ Yes | `p[0] = 'x'` → crash |
| `const char *p = "literal"` | `.rodata` | ✅ Yes | Same as above |
| `char arr[] = "literal"` | **Stack** | ❌ No | Copy, can modify |
| `char *p = malloc(); strcpy(p, "literal")` | **Heap** | ❌ No | Copy to heap |
| `char arr[] = "literal"` (global) | **Data** | ❌ No | Global, modifiable |
| `static char arr[] = "literal"` | **Data** | ❌ No | Static, modifiable |


```
HIGH MEMORY ADDRESS (0x7fffffffffff)
╔══════════════════════════════════════════════════════════════════╗
║                          STACK                                   ║
║  ┌─────────────────────────────────────────────────────────┐    ║
║  │  main() stack frame                                      │    ║
║  │  ├─ str1[] = ['H','e','l','l','o','\0']  (0x7ffd1234)   │    ║
║  │  ├─ str2[] = ['W','o','r','l','d','\0']  (0x7ffd1228)   │    ║
║  │  ├─ p1 (pointer) = 0x400600              (0x7ffd1220)   │    ║
║  │  ├─ p2 (pointer) = 0x400606              (0x7ffd1218)   │    ║
║  │  ├─ arr[] = [0x400610, 0x400615]         (0x7ffd1210)   │    ║
║  │  └─ heap_str (pointer) = 0x601010        (0x7ffd1208)   │    ║
║  └─────────────────────────────────────────────────────────┘    ║
╠══════════════════════════════════════════════════════════════════╣
║                            HEAP                                  ║
║  ┌─────────────────────────────────────────────────────────┐    ║
║  │  heap_str points here:                                  │    ║
║  │  ['H','e','l','l','o','\0']  (0x601010)                 │    ║
║  │  (modifiable!)                                          │    ║
║  └─────────────────────────────────────────────────────────┘    ║
╠══════════════════════════════════════════════════════════════════╣
║                         DATA SEGMENT                             ║
║  ┌─────────────────────────────────────────────────────────┐    ║
║  │  global_str[] = ['H','e','l','l','o','\0']  (0x601030)  │    ║
║  │  global_arr[] = ['W','o','r','l','d','\0']  (0x601038)  │    ║
║  │  global_ptr (pointer) = 0x400620           (0x601040)   │    ║
║  │  static_arr[] = ['S','t','a','t','i','c','\0'] (0x601048)│   ║
║  │  (all modifiable!)                                       │    ║
║  └─────────────────────────────────────────────────────────┘    ║
╠══════════════════════════════════════════════════════════════════╣
║                           .rodata                                ║
║  ┌─────────────────────────────────────────────────────────┐    ║
║  │  Address 0x400600: "hello\0"                            │    ║
║  │  Address 0x400606: "world\0"                            │    ║
║  │  Address 0x400610: "one\0"                              │    ║
║  │  Address 0x400615: "two\0"                              │    ║
║  │  Address 0x400620: "global pointer literal\0"           │    ║
║  │  Address 0x400640: "static literal\0"                   │    ║
║  │                                                         │    ║
║  │  ⚠️ READ-ONLY! Modify လုပ်ရင် Segmentation Fault      │    ║
║  └─────────────────────────────────────────────────────────┘    ║
╠══════════════════════════════════════════════════════════════════╣
║                          .text (Code)                            ║
║  ┌─────────────────────────────────────────────────────────┐    ║
║  │  Executable instructions (function code)                │    ║
║  │  main() function, printf(), etc.                        │    ║
║  │  READ-ONLY & EXECUTABLE                                 │    ║
║  └─────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════════════════════════════════════╝
LOW MEMORY ADDRESS (0x400000)
```


##### 1. String Literal (.rodata) - Read-only

```c
// ဒါတွေက .rodata မှာ
char *p1 = "hello";           // "hello" → .rodata
const char *p2 = "world";     // "world" → .rodata
char *arr[] = {"one", "two"}; // "one", "two" → .rodata

// Modify လုပ်ရင် CRASH!
// p1[0] = 'H';  // ❌ Segmentation fault
```


```
HIGH ADDRESS
╔══════════════════════════════════════════════════════════════════╗
║ STACK                                                           ║
║ ┌────────────────────────────────────────────────────────────┐  ║
║ │ p1 (pointer) = 0x400600    (address: 0x7ffd1220)          │  ║
║ │ p2 (pointer) = 0x400606    (address: 0x7ffd1218)          │  ║
║ │ arr[0] = 0x400610          (address: 0x7ffd1210)          │  ║
║ │ arr[1] = 0x400615          (address: 0x7ffd1208)          │  ║
║ └────────────────────────────────────────────────────────────┘  ║
╠══════════════════════════════════════════════════════════════════╣
║ .rodata (READ-ONLY!)                                            ║
║ ┌────────────────────────────────────────────────────────────┐  ║
║ │                                                            │  ║
║ │  Address 0x400600: ┌─────┬─────┬─────┬─────┬─────┬─────┐  │  ║
║ │                    │ 'h' │ 'e' │ 'l' │ 'l' │ 'o' │ '\0'│  │  ║
║ │                    └─────┴─────┴─────┴─────┴─────┴─────┘  │  ║
║ │                                                            │  ║
║ │  Address 0x400606: ┌─────┬─────┬─────┬─────┬─────┬─────┐  │  ║
║ │                    │ 'w' │ 'o' │ 'r' │ 'l' │ 'd' │ '\0'│  │  ║
║ │                    └─────┴─────┴─────┴─────┴─────┴─────┘  │  ║
║ │                                                            │  ║
║ │  Address 0x400610: ┌─────┬─────┬─────┬─────┐              │  ║
║ │                    │ 'o' │ 'n' │ 'e' │ '\0'│              │  ║
║ │                    └─────┴─────┴─────┴─────┘              │  ║
║ │                                                            │  ║
║ │  Address 0x400615: ┌─────┬─────┬─────┬─────┐              │  ║
║ │                    │ 't' │ 'w' │ 'o' │ '\0'│              │  ║
║ │                    └─────┴─────┴─────┴─────┘              │  ║
║ │                                                            │  ║
║ └────────────────────────────────────────────────────────────┘  ║
╚══════════════════════════════════════════════════════════════════╝
LOW ADDRESS
```

---

##### 2. Stack String (NOT in .rodata)

```c
void function() {
    // ဒီ string က STACK ပေါ်မှာ (copy from .rodata)
    char str1[] = "hello";        // Stack (6 bytes)
    
    // ဒါလည်း STACK ပေါ်မှာပဲ
    char str2[10] = "world";      // Stack (10 bytes)
    
    // Modify လုပ်လို့ရတယ်
    str1[0] = 'H';   // ✅ OK - "Hello" ဖြစ်သွားမယ်
    str2[0] = 'W';   // ✅ OK - "World" ဖြစ်သွားမယ်
}
```

##### Memory Layout:
```
Stack:
┌─────────────────────┐
│ str1: h e l l o \0  │ ← Copy from .rodata
│ str2: w o r l d \0  │ ← Copy from .rodata
└─────────────────────┘

.rodata:
┌─────────────────────┐
│ "hello" (original)  │ ← Still there, untouched
│ "world" (original)  │
└─────────────────────┘
```

---

##### 3. Heap String (NOT in .rodata)

```c
#include <stdlib.h>
#include <string.h>

void function() {
    // Heap ပေါ်မှာ string အတွက် နေရာခွဲတယ်
    char *str = malloc(6);  // 6 bytes on HEAP
    strcpy(str, "hello");   // Copy "hello" to heap
    
    // Modify လုပ်လို့ရတယ်
    str[0] = 'H';   // ✅ OK - heap ပေါ်မှာ modify
    
    free(str);
}
```

##### Memory Layout:
```
Heap:
┌─────────────────────┐
│ H e l l o \0        │ ← Modifiable!
└─────────────────────┘

.rodata:
┌─────────────────────┐
│ "hello" (original)  │ ← Still read-only
└─────────────────────┘
```

---

##### 4. Global String Array (NOT in .rodata)

```c
// Global scope - Data Segment
char global_str[] = "hello";     // Data Segment (modifiable!)
char global_arr[10] = "world";   // Data Segment (modifiable!)

int main() {
    global_str[0] = 'H';  // ✅ OK - "Hello" ဖြစ်သွားမယ်
    global_arr[0] = 'W';  // ✅ OK - "World" ဖြစ်သွားမယ်
    return 0;
}
```

##### Memory Layout:
```
Data Segment:
┌─────────────────────┐
│ global_str: H e l l o \0 │ ← Modifiable!
│ global_arr: W o r l d \0 │ ← Modifiable!
└─────────────────────┘

.rodata:
┌─────────────────────┐
│ "hello" (original)  │ ← Still there (unused after copy)
│ "world" (original)  │
└─────────────────────┘
```



```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Global string array - Data Segment (modifiable)
char global_arr[] = "global array string";

// Global pointer to literal - pointer in Data, literal in .rodata
char *global_ptr = "global pointer literal";

void test_strings() {
    // 1. String literal (points to .rodata)
    char *literal_ptr = "hello literal";     // literal → .rodata
    
    // 2. Stack array (copy from .rodata)
    char stack_arr[] = "hello stack";        // entire array → STACK
    
    // 3. Heap string
    char *heap_str = malloc(20);
    strcpy(heap_str, "hello heap");          // string → HEAP
    
    // 4. Static string array (Data Segment)
    static char static_arr[] = "hello static"; // Data Segment
    
    printf("=== WHERE IS EACH STRING STORED? ===\n");
    
    printf("\n1. String literal (via pointer):\n");
    printf("   Pointer address: %p (stack)\n", &literal_ptr);
    printf("   String address:  %p (.rodata)\n", literal_ptr);
    printf("   Modifiable? NO - crash if try\n");
    
    printf("\n2. Stack array:\n");
    printf("   Array address:   %p (STACK)\n", stack_arr);
    printf("   Modifiable? YES (copy from .rodata)\n");
    
    printf("\n3. Heap string:\n");
    printf("   Pointer address: %p (stack)\n", &heap_str);
    printf("   String address:  %p (HEAP)\n", heap_str);
    printf("   Modifiable? YES\n");
    
    printf("\n4. Static array:\n");
    printf("   Array address:   %p (Data Segment)\n", static_arr);
    printf("   Modifiable? YES\n");
    
    printf("\n5. Global array:\n");
    printf("   Array address:   %p (Data Segment)\n", global_arr);
    printf("   Modifiable? YES\n");
    
    printf("\n6. Global pointer:\n");
    printf("   Pointer address: %p (Data Segment)\n", &global_ptr);
    printf("   String address:  %p (.rodata)\n", global_ptr);
    printf("   Modifiable? NO - points to .rodata\n");
    
    free(heap_str);
}

int main() {
    test_strings();
    return 0;
}
```

### Output (x86-64 Linux):
```
=== WHERE IS EACH STRING STORED? ===

1. String literal (via pointer):
   Pointer address: 0x7ffd1234 (stack)
   String address:  0x400600 (.rodata)
   Modifiable? NO - crash if try

2. Stack array:
   Array address:   0x7ffd1220 (STACK)
   Modifiable? YES (copy from .rodata)

3. Heap string:
   Pointer address: 0x7ffd1218 (stack)
   String address:  0x601010 (HEAP)
   Modifiable? YES

4. Static array:
   Array address:   0x601020 (Data Segment)
   Modifiable? YES

5. Global array:
   Array address:   0x601030 (Data Segment)
   Modifiable? YES

6. Global pointer:
   Pointer address: 0x601040 (Data Segment)
   String address:  0x400610 (.rodata)
   Modifiable? NO - points to .rodata
```




----


`String lateral`

String literal ဆိုတာ double quotes `" "` ထဲမှာရေးတဲ့ စာသားဖြစ်ပြီး၊ C compiler က ၎င်းကို read-only memory segment (`.rodata`) မှာ သိမ်းပေးတယ်

```c
char *str = "Hello";  // "Hello" က .rodata မှာ, str pointer က stack မှာ
```

```c
#include <stdio.h>

int main() {
    char *str1 = "AAAA";        // .rodata မှာ
    char *str2 = "AAAA";        // same address! (compiler optimization)
    char arr[] = "AAAA";        // stack ပေါ်မှာ (copy)
    
    printf("str1 points to: %p\n", str1);  // e.g., 0x400600
    printf("str2 points to: %p\n", str2);  // 0x400600 (တူတယ်!)
    printf("arr is at: %p\n", arr);        // 0x7ffd1234 (stack)
    
    return 0;
}
```


```c
High Address
+-------------------------+
|      Stack              |
|   arr[] = "AAAA"        |  ← Modifiable! (copy)
|   (0x7ffd1234)          |
|                         |
|   str1 (pointer)        |  ← 0x7ffd1240 (ဒီမှာ pointer ရှိတယ်)
|   str2 (pointer)        |  ← 0x7ffd1248
+-------------------------+
|      Heap               |
+-------------------------+
|      BSS                |
+-------------------------+
|      Data               |
+-------------------------+
|      .rodata            |
|   "AAAA" (0x400600)     |  ← Read-only! (modify လုပ်ရင် crash)
+-------------------------+
|      .text (code)       |
+-------------------------+
Low Address
```


| ဂုဏ်သတ္တိ                   | အကျိုးသက်ရောက်မှု                                                               |
| --------------------------- | ------------------------------------------------------------------------------- |
| **Read-only**               | Modify လုပ်လို့မရဘူး → classic buffer overflow နဲ့ control hijack မလုပ်နိုင်ဘူး |
| **Fixed address** (No ASLR) | Address predictable → ROP gadgets အတွက် သုံးလို့ရတယ်                            |
| **Shared pooling**          | တူညီတဲ့ literals တွေ address တူ → fingerprinting လုပ်လို့ရတယ်                   |
| **Separate segment**        | Stack overflow က .rodata ကို မထိဘူး → isolation ရှိတယ်                          |
| **Contains strings**        | Format string exploit နဲ့ ဖတ်လို့ရတယ် (information leak)                        |

----

`Scope`  - Variable တစ်ခုကို ဘယ်နေရာကနေ မြင်ရသလဲ (access လုပ်လို့ရသလဲ)

```c
#include <stdio.h>

int global_secret = 100;  // Data segment မှာ

void exploit_me() {
    int global_secret = 200;  // ဒါက local variable (stack) - global ကို ဖုံးထားတယ်
    printf("Local secret: %d\n", global_secret);  // 200
    
    // ❗ ဒီမှာ stack variable ကို overflow လုပ်ရင် 
    // global (data segment) ကို မထိဘူး - different memory regions!
}

int main() {
    exploit_me();
    return 0;
}
```

```
Stack:          global_secret (local) = 200
Data Segment:   global_secret (global) = 100  (သီးခြားနေရာ)
```
---


`const`

Variable ရဲ့တန်ဖိုးကို ပြင်လို့မရဘူး (read-only)

| Declaration                 | Storage           | Can Modify?                                    |
| --------------------------- | ----------------- | ---------------------------------------------- |
| `const int x = 5;` (local)  | **Stack**         | No (compiler error, but possible via overflow) |
| `const int x = 5;` (global) | **Text (rodata)** | No (segfault if modified)                      |
```c
const int GLOBAL_CONST = 100;  // Text segment - read-only

int main() {
    const int local_const = 50;   // Stack - compiler says read-only
    
    // GLOBAL_CONST = 200;  // ❌ Compiler error
    // local_const = 60;    // ❌ Compiler error
    
    // Pwn note: local_const ကို buffer overflow နဲ့ ပြင်နိုင်
    char buf[8];
    gets(buf);  // overflow လုပ်ရင် local_const ပြောင်းသွားနိုင်
}
```


| Feature                           | Global `const`                | Local `const`                |
| --------------------------------- | ----------------------------- | ---------------------------- |
| **Memory Location**               | `.rodata` (read-only segment) | **Stack** (writable memory!) |
| **Modify လုပ်လို့ရလား?**          | ❌ လုံးဝမရဘူး (crash)          | ⚠️ ရတယ် (undefined behavior) |
| **Lifetime**                      | Program စကနေ ပြီးထိ           | Function ထဲမှာပဲ             |
| **ASLR**                          | No ASLR (fixed address)       | Yes ASLR (randomized)        |
| **Overflow နဲ့ ပြောင်းလို့ရလား?** | ❌ မရဘူး (separate segment)    | ✅ ရတယ် (stack မှာရှိတယ်)     |

##### Global `const`:
```c
const int global_const = 100;  // Compiler က .rodata မှာ ထားတယ်
```
- **Compile-time** မှာ တန်ဖိုးသိတယ်
- Program တစ်လျှောက်လုံး လိုအပ်တယ်
- OS က `.rodata` segment ကို **read-only** လုပ်ပေးတယ် (memory protection)
    

##### Local `const`:
```c
void func() {
    const int local_const = 50;  // Compiler က stack မှာ ထားတယ်
}
```
- Function ခေါ်မှသာ တန်ဖိုးသိတယ်
- Function ပြီးရင် မလိုတော့ဘူး
- Stack က **read-write** memory ဖြစ်တယ်

---

 `static`
 Variable ရဲ့ **lifetime** က **program တစ်ခုလုံး** ကြာတယ်
store → **Data** (if initialized) or **BSS** (if zero)
Function ထဲက static variable တွေလည်း Data/BSS မှာ
ဒါမဲ့ scope limit ရှိ (global/local)

```c
void counter() {
    static int count = 0;  // Data/BSS - initialized once
    count++;
    printf("%d\n", count);
}

int main() {
    counter();  // 1
    counter();  // 2
    counter();  // 3  (count က တန်ဖိုးကို သိမ်းထားတယ်)
}
```
- Local variable လို scope ရှိတယ် (function ထဲမှာပဲမြင်ရ)
- ဒါပေမယ့် lifetime က program လုံး
- ပထမဆုံးခေါ်ချိန်မှာ initialize လုပ် (once)

---

