

```c
# Compile and check sections
$ gcc -o program program.c
$ size program
   text    data     bss     dec     hex filename
   2048     512     256    2816     b00 program

$ readelf -S program | grep -E "data|bss|text"
  [14] .text             PROGBITS      00400000 000000 002000 00  AX  0   0 16
  [15] .rodata           PROGBITS      00402000 002000 000200 00   A  0   0 8
  [16] .data             PROGBITS      00403000 002200 000200 00  WA  0   0 8
  [17] .bss              NOBITS        00403200 002400 000100 00  WA  0   0 8
```

```
High addresses
+------------------+
|      Stack       |  ← const_local (read-only by compiler)
|                  |  ← normal_local
+------------------+
|        ↓         |
+------------------+
|       Heap       |
+------------------+
|       BSS        |  ← static_global (uninit part)
|                  |  ← static_local
+------------------+
|      Data        |  ← normal_global
|                  |  ← static_global (init part) /static local 
+------------------+
|   Text (rodata)  |  ← const_global
|                  |  ← static_const_global/static_const_local
|                  |  ← string literals
+------------------+
Low addresses
```




```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// ============================================
// GLOBAL SCOPE (File scope)
// ============================================

// ---- Basic types ----
int g_int;                          // BSS (uninitialized) → value: 0
int g_int_init = 100;               // DATA (initialized) → value: 100
float g_float = 3.14;               // DATA → value: 3.14
double g_double = 2.71828;          // DATA → value: 2.71828
char g_char = 'A';                  // DATA → value: 'A'

// ---- Const types ----
const int G_CONST = 999;            // TEXT (rodata) → value: 999 (read-only)
const float G_CONST_FLOAT = 1.618;  // TEXT (rodata)

// ---- Static types (file scope) ----
static int static_global;           // BSS (uninit) → value: 0, visible only this file
static int static_global_init = 50; // DATA, visible only this file
static const int static_const_global = 77; // TEXT (rodata), file scope

// ---- Arrays (global) ----
int g_arr[5];                       // BSS → 5 zeros
int g_arr_init[3] = {1, 2, 3};      // DATA → values: 1,2,3
char g_str_arr[] = "global";        // DATA → 'g','l','o','b','a','l','\0'

// ---- Pointers (global) ----
int *g_ptr;                         // BSS (NULL pointer)
int *g_ptr_init = &g_int_init;      // DATA (stores address of g_int_init)
char *g_str_ptr = "hello";          // DATA (pointer) + TEXT (string literal)
const char *g_const_ptr = "world";  // DATA (pointer) + TEXT
char *const g_ptr_const = g_str_arr; // DATA (const pointer) → points to g_str_arr

// ---- String literals (global) ----
// "hello" → TEXT segment
// "world" → TEXT segment

// ============================================
// STRUCTURES
// ============================================
struct Point {
    int x;
    int y;
};

struct Student {
    char name[20];
    int age;
    float gpa;
};

struct Point g_point = {10, 20};     // DATA segment
static struct Student g_student;     // BSS (zeros)

// ============================================
// FUNCTION DECLARATIONS
// ============================================
void demonstrate_stack();
void demonstrate_heap();

// ============================================
// MAIN FUNCTION
// ============================================
int main() {
    printf("========== MEMORY LAYOUT DEMO ==========\n");
    printf("Text (code) segment: %p\n", (void*)main);
    printf("String literal 'hello': %p\n", (void*)"hello");
    printf("\n");
    
    // Call other functions
    demonstrate_stack();
    demonstrate_heap();
    
    // Show global addresses
    printf("\n========== GLOBAL VARIABLES ==========\n");
    printf("g_int (BSS): %p\n", &g_int);
    printf("g_int_init (DATA): %p\n", &g_int_init);
    printf("G_CONST (TEXT/rodata): %p\n", &G_CONST);
    printf("static_global (BSS, file scope): %p\n", &static_global);
    printf("g_arr (BSS): %p\n", g_arr);
    printf("g_str_ptr (DATA, points to TEXT): %p -> %p\n", &g_str_ptr, g_str_ptr);
    
    return 0;
}

// ============================================
// STACK DEMONSTRATION
// ============================================
void demonstrate_stack() {
    printf("\n========== STACK VARIABLES ==========\n");
    
    // ---- Local basic types ----
    int local_int = 42;                     // Stack
    float local_float = 9.99f;              // Stack
    char local_char = 'Z';                  // Stack
    
    // ---- Local const ----
    const int local_const = 123;            // Stack (compiler-enforced read-only)
    
    // ---- Local static (not on stack!) ----
    static int local_static = 555;          // DATA/BSS (persistent)
    static const int local_static_const = 888; // TEXT/rodata
    
    // ---- Local arrays ----
    int local_arr[4] = {10, 20, 30, 40};    // Stack (array content)
    char local_str[] = "stack string";      // Stack (copy of string literal)
    
    // ---- Local pointers ----
    int *local_ptr = &local_int;            // Stack (pointer variable)
    char *local_str_ptr = "literal";        // Stack (pointer) + TEXT (string)
    const char *local_const_ptr = "readonly"; // Stack pointer + TEXT
    
    // ---- Array of pointers ----
    char *str_arr[3] = {"one", "two", "three"}; // Stack array of pointers
                                                // Each pointer → TEXT segment
    
    // ---- Structure on stack ----
    struct Point stack_point = {5, 15};     // Stack (entire struct)
    struct Student stack_student = {"Alice", 20, 3.8}; // Stack
    
    // Print addresses
    printf("local_int: %p\n", &local_int);
    printf("local_arr: %p\n", local_arr);
    printf("local_str[]: %p (content copy)\n", local_str);
    printf("local_str_ptr (pointer): %p, points to: %p (TEXT)\n", &local_str_ptr, local_str_ptr);
    printf("stack_point: %p\n", &stack_point);
    printf("local_static (DATA/BSS, not stack): %p\n", &local_static);
    
    // Demonstrate pointer to stack
    int *stack_ptr = &local_int;            // Points to stack
    printf("stack_ptr value (address of local_int): %p\n", (void*)stack_ptr);
}

// ============================================
// HEAP DEMONSTRATION
// ============================================
void demonstrate_heap() {
    printf("\n========== HEAP VARIABLES ==========\n");
    
    // ---- Basic types on heap ----
    int *heap_int = (int*)malloc(sizeof(int));      // Heap
    *heap_int = 1000;                                // Value on heap
    float *heap_float = (float*)malloc(sizeof(float));
    *heap_float = 3.14159f;
    
    // ---- Arrays on heap ----
    int *heap_arr = (int*)malloc(5 * sizeof(int));   // Heap array
    for(int i = 0; i < 5; i++) heap_arr[i] = i * 10;
    
    // ---- String on heap ----
    char *heap_str = (char*)malloc(20);
    strcpy(heap_str, "heap string");
    
    // ---- Structure on heap ----
    struct Point *heap_point = (struct Point*)malloc(sizeof(struct Point));
    heap_point->x = 100;
    heap_point->y = 200;
    
    // ---- Array of structures on heap ----
    struct Student *heap_students = (struct Student*)malloc(2 * sizeof(struct Student));
    strcpy(heap_students[0].name, "Bob");
    heap_students[0].age = 22;
    heap_students[0].gpa = 3.5;
    
    // ---- Pointer to pointer on heap ----
    int **heap_ptr_ptr = (int**)malloc(sizeof(int*));
    *heap_ptr_ptr = heap_int;  // Points to heap_int
    
    // Print addresses
    printf("heap_int (pointer on stack): %p, points to heap: %p\n", &heap_int, (void*)heap_int);
    printf("heap_int value: %d\n", *heap_int);
    printf("heap_arr (stack pointer): %p, points to heap: %p\n", &heap_arr, heap_arr);
    printf("heap_str (stack pointer): %p, points to heap: %p\n", &heap_str, heap_str);
    printf("heap_point (stack pointer): %p, points to heap: %p\n", &heap_point, heap_point);
    printf("heap_point->x value: %d (on heap)\n", heap_point->x);
    
    // Free memory
    free(heap_int);
    free(heap_float);
    free(heap_arr);
    free(heap_str);
    free(heap_point);
    free(heap_students);
    free(heap_ptr_ptr);
}
```

```
HIGH MEMORY ADDRESSES (0x7fffffffffff)
┌─────────────────────────────────────────────────────────────┐
│                         STACK                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ main() stack frame:                                 │   │
│  │   - local_int = 42                                  │   │
│  │   - local_arr[4] = [10,20,30,40]                   │   │
│  │   - local_str[] = 's','t','a','c','k'...          │   │
│  │   - local_ptr (address of local_int)               │   │
│  │   - stack_point {x=5, y=15}                        │   │
│  │   - stack_student                                   │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │ demonstrate_stack() stack frame:                   │   │
│  │   - local_int, local_float, local_char             │   │
│  │   - local_arr[], local_str[]                       │   │
│  │   - local_ptr, str_arr[3] (pointers to TEXT)       │   │
│  └─────────────────────────────────────────────────────┘   │
│                         ↓                                   │
│                         ↑                                   │
├─────────────────────────────────────────────────────────────┤
│                         HEAP                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ malloc(4) → int value 1000                         │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │ malloc(20) → [0,10,20,30,40] (array)               │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │ malloc(20) → 'h','e','a','p',' ','s','t','r'...    │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │ malloc(8) → Point {x=100, y=200}                   │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │ malloc(48) → Student[2] (Bob, age22, gpa3.5...)    │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                      BSS (Block Started by Symbol)          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ g_int = 0 (uninit global)                          │   │
│  │ static_global = 0 (file-scope static)              │   │
│  │ g_arr[5] = {0,0,0,0,0}                             │   │
│  │ g_ptr = NULL                                       │   │
│  │ g_student (struct, all zeros)                      │   │
│  │ local_static = 555 (function static, persists)     │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                     DATA (Initialized)                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ g_int_init = 100                                   │   │
│  │ g_float = 3.14                                     │   │
│  │ g_double = 2.71828                                 │   │
│  │ g_char = 'A'                                       │   │
│  │ static_global_init = 50                            │   │
│  │ g_arr_init[3] = {1,2,3}                            │   │
│  │ g_str_arr[] = 'g','l','o','b','a','l','\0'        │   │
│  │ g_ptr_init = &g_int_init (address)                 │   │
│  │ g_str_ptr = &"hello" (address of TEXT)             │   │
│  │ g_const_ptr = &"world"                             │   │
│  │ g_ptr_const = &g_str_arr (const pointer)           │   │
│  │ g_point = {x=10, y=20}                             │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│              TEXT (Code + Read-Only Data)                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Program code (machine instructions)                │   │
│  │ main() function code                               │   │
│  │ demonstrate_stack() code                          │   │
│  │ demonstrate_heap() code                           │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │ rodata (Read-Only Data):                           │   │
│  │   G_CONST = 999                                    │   │
│  │   G_CONST_FLOAT = 1.618                            │   │
│  │   static_const_global = 77                         │   │
│  │   local_static_const = 888                         │   │
│  │   "hello" (string literal)                         │   │
│  │   "world" (string literal)                         │   │
│  │   "literal"                                        │   │
│  │   "readonly"                                       │   │
│  │   "one", "two", "three"                            │   │
│  └─────────────────────────────────────────────────────┘   │
LOW MEMORY ADDRESSES (0x00400000)
```


### **STACK မှာသိမ်းတဲ့ Data တွေ**

|Data Type|Example|Value|Location|
|---|---|---|---|
|Local int|`int x = 5;`|5|Stack|
|Local float|`float f = 3.14;`|3.14|Stack|
|Local array|`int arr[3] = {1,2,3};`|1,2,3 (content)|Stack|
|Local char array|`char str[] = "hi";`|'h','i','\0' (copy)|Stack|
|Local pointer|`int *p = &x;`|address of x|Stack|
|Local const|`const int c = 10;`|10 (compiler read-only)|Stack|
|Struct (local)|`struct Point p = {1,2};`|x=1, y=2 (content)|Stack|
|Function parameters|`void func(int a, int b)`|a, b values|Stack|
|Return address|(hidden)|return address|Stack|
|Frame pointer|(hidden)|base pointer|Stack|

### **HEAP မှာသိမ်းတဲ့ Data တွေ**

|Data Type|Example|Value|Location|
|---|---|---|---|
|Dynamic int|`int *p = malloc(4);`|value set by user|Heap|
|Dynamic array|`int *arr = malloc(20);`|array content|Heap|
|Dynamic string|`char *s = malloc(10);`|string content|Heap|
|Dynamic struct|`struct Point *p = malloc(8);`|x, y values|Heap|
|Dynamic nested|`int **pp = malloc(8);`|pointer to heap|Heap|

### **DATA/BSS မှာသိမ်းတဲ့ Data တွေ**

|Data Type|Example|Segment|Value|
|---|---|---|---|
|Global (uninit)|`int g;`|BSS|0|
|Global (init)|`int g = 5;`|DATA|5|
|Static local (uninit)|`static int s;`|BSS|0|
|Static local (init)|`static int s = 5;`|DATA|5|
|Static global (uninit)|`static int sg;`|BSS|0|
|Static global (init)|`static int sg = 5;`|DATA|5|
|Global array (init)|`int arr[] = {1,2};`|DATA|1,2|
|Global pointer|`int *p = &g;`|DATA|address of g|
|Global struct|`struct Point p = {1,2};`|DATA|x=1, y=2|

### **TEXT (rodata) မှာသိမ်းတဲ့ Data တွေ**

| Data Type          | Example                    | Value                    | Notes        |
| ------------------ | -------------------------- | ------------------------ | ------------ |
| String literal     | `"hello"`                  | 'h','e','l','l','o','\0' | Read-only    |
| Global const       | `const int c = 5;`         | 5                        | Read-only    |
| Static const       | `static const int sc = 5;` | 5                        | Read-only    |
| Local static const | function's `static const`  | value                    | Read-only    |
| Code               | function bodies            | machine instructions     | Execute-only |


##### Example 1: String vs Char Array vs Pointer

```c
// GLOBAL
char *s1 = "hello";     // s1 pointer in DATA, "hello" in TEXT
char s2[] = "hello";    // s2 array in DATA (copy)

int main() {
    // LOCAL
    char *s3 = "hello";     // s3 pointer on STACK, "hello" in TEXT
    char s4[] = "hello";    // s4 array on STACK (copy)
    
    // Values and addresses
    // s1 = 0x400600 (points to TEXT)
    // s2 = 0x400700 (array in DATA: 'h','e','l','l','o','\0')
    // s3 = 0x400600 (same TEXT as s1)
    // s4 = 0x7fff1234 (stack copy)
}
```

##### Example 2: All Pointer Combinations

```c
int normal_var = 100;           // Which segment? → depends (stack or data)
int *p1 = &normal_var;          // p1 stores address of normal_var

const int *p2 = &normal_var;    // p2 is mutable, *p2 read-only
int *const p3 = &normal_var;    // p3 is const, *p3 mutable
const int *const p4 = &normal_var; // both const

// Memory:
// p1 (stack/data) ──→ normal_var (stack/data)
// p2 (stack/data) ──→ normal_var (but treated as read-only)
// p3 (stack/data, fixed address) ──→ normal_var
// p4 (stack/data, fixed) ──→ normal_var (read-only view)
```


##### Example 3: Static Inside Function

```c
void counter() {
    static int count = 0;  // Stored in BSS (initially 0)
    count++;                // Value persists between calls
    printf("%d\n", count);
}

int main() {
    counter();  // count becomes 1 (BSS location: 0x600800)
    counter();  // same BSS location, now 2
    counter();  // now 3
    // Address of count is fixed, not on stack!
}
```

| Attack Target             | Memory Segment | Can Overwrite?         | Typical Exploit                |
| ------------------------- | -------------- | ---------------------- | ------------------------------ |
| Return address            | Stack          | ✅ Yes                  | Buffer overflow → ROP          |
| Local array               | Stack          | ✅ Yes                  | Stack overflow                 |
| Local pointer             | Stack          | ✅ Yes                  | Overwrite to arbitrary address |
| Function pointer (local)  | Stack          | ✅ Yes                  | Control flow hijack            |
| Function pointer (global) | Data           | ✅ Yes                  | Same                           |
| GOT entry                 | Data           | ✅ Yes                  | GOT overwrite                  |
| vtable pointer            | Heap/Data      | ✅ Yes                  | vtable hijacking               |
| Heap metadata             | Heap           | ✅ Yes                  | Heap feng shui, unlink         |
| String literal            | Text           | ❌ No (segfault)        | Not useful                     |
| Global const              | Text           | ❌ No                   | Not useful                     |
| Code section              | Text           | ❌ No (mprotect needed) | Can use as ROP gadgets         |

```
┌─────────────────────────────────────────────────────────────┐
│  "I see this in code"        →  "Stored in"                │
├─────────────────────────────────────────────────────────────┤
│  int x;  (inside function)   →  STACK                      │
│  int x = 5; (inside func)    →  STACK                      │
│  static int x; (inside func) →  BSS/DATA (persistent)      │
│  static int x=5; (inside)    →  DATA (persistent)          │
│  int global;                 →  BSS                        │
│  int global=5;               →  DATA                       │
│  static int global;          →  BSS (file scope)           │
│  const int x=5; (global)     →  TEXT (rodata)              │
│  const int x=5; (local)      →  STACK (read-only by comp)  │
│  "string literal"            →  TEXT (rodata)              │
│  char arr[]="str"; (local)   →  STACK (copy of literal)    │
│  char arr[]="str"; (global)  →  DATA (copy)                │
│  char *p = "str"; (local)    →  p:STACK, "str":TEXT        │
│  char *p = "str"; (global)   →  p:DATA, "str":TEXT         │
│  malloc(100)                 →  HEAP                       │
│  struct on stack             →  STACK (entire struct)      │
│  struct on heap              →  HEAP                       │
│  struct global               →  DATA/BSS                   │
└─────────────────────────────────────────────────────────────┘
```