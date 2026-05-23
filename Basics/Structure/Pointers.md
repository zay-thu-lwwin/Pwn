


Pointer ဆိုတာ memory address ကို သိမ်းတဲ့ variable တစ်ခု

variable အားလုံး ကို address လို့မြင်
ဒါမဲ့ အဲ address ကို name(နာမည်)တွေပေးကြတယ်
ပုံမှန် nameနဲ့ ခေါ်ရင်အဲ address မှာသိမ်းတဲ့ value ရလာမယ်
ဒါမဲ့ & + name ပေါင်း‌ခေါ်ရင်  အဲဒီ address ရလာမယ်
အဲဒီ address ကိုသိမ်းချင်ရင် * ကို သုံးပြီး
တူညီတဲ့ data type ရယ် နောက်ထပ် name တစ်ခုကို တွဲပြီးသုံးလို့ရတယ်
memory address တစ်ခုသိမ်းထားတဲ့ အဲကောင်ကို pointer လို့ခေါ်တယ်

##### Dereference Operator(`*`)
```

int* p = &x  (* data type of pointer)
*p ---> original value (*&x = x)  (ဒီလို original value ယူတာကို dereference လုပ်တာလို့ခေါ်တယ်)
x[] = {3, 4}
*x --> first value (3) x[0]
*(x+1) --> second valud

array copy
int* p = x;
p ---> first value address x[0]
*p --> first value
p++ (*(p+1)) 
p ---> second value address
*p ---> second valule

```
**name of an array**, is actually a **pointer** to the **first element**

##### Address-of Operator &
```

scanf("%d", &x) we scan the address of integer
&x ---> memory address
prinf("%p", &x)
```


```
ptr->member က (*ptr).member
ptr->age = (*ptr).age
```

##### Pointer to Pointer(`**`)
	pointer က variable တစ်ခုရဲ့ memoryaddress ကို store လုပ်တယ် 
	အဲpointer ရဲ့ memory address ကို store လုပ်ချင်ရင် ** သုံးပြီး ptr တစ်ခုခေါ်ပြီးလုပ်လို့ရ

```c
int myNum = 10;       // normal variable  
int *ptr = &myNum;    // pointer to int  
int **pptr = &ptr;    // pointer to pointer
```


|Declaration|Storage (Pointer Itself)|Points To|Example Value|
|---|---|---|---|
|`int *p;` (global)|**BSS**|Unknown (garbage/NULL)|0x0|
|`int *p = &g;` (global)|**DATA**|Global variable (DATA)|0x400600|
|`int *p = &l;` (global, local var)|**DATA**|Local (STACK) - dangerous!|0x7fff1234|
|`int *p;` (local)|**STACK**|Garbage|0xdeadbeef|
|`int *p = &l;` (local)|**STACK**|Local (STACK)|0x7fff1230|
|`int *p = &g;` (local)|**STACK**|Global (DATA)|0x400600|
|`static int *p;` (local)|**BSS**|NULL|0x0|
|`static int *p = &g;` (local)|**DATA**|Global (DATA)|0x400600|
|`char *p = "hi";` (global)|**DATA**|String literal (TEXT)|0x400500|
|`char *p = "hi";` (local)|**STACK**|String literal (TEXT)|0x400500|
|`int **p;` (global)|**BSS**|Pointer to pointer|0x0|
|`int **p = &ptr;` (global)|**DATA**|Another pointer (DATA)|0x400608|




> [!NOTE]
> pointer တစ်ခုကို declare လုပ်မယ်ဆို int *ptr = ...  ဆိုပြီးကြေညာ မူရင်း variable value ကို ယူချင်ရင် declare လုပ်ထားတဲ့အတိုင်း *ptr ပြန်သုံး
> pointer to pointer လည်းအဲအတိုင်းဘဲ **ptr , *ptr ဆို variable ရဲ့ memory address



---



#### Pointer Arithmetic 
	Pointer arithmetic ဆိုတာ pointer variable ကို သင်္ချာတွက်ချက်တာ (ပေါင်းခြင်း၊ နှုတ်ခြင်း) ဖြစ်ပြီး memory address တွေကို ရွှေ့ပြောင်းတာ ဖြစ်ပါတယ်
	pointer တစ်ခုကို 1 add or 1 subtract လိုက်ရင်  အဲ pointerမှာ store လုပ်ထားတဲ့ memory address ကို 
	အဲ pointer ရဲ့ data type size နဲ့ပေါင်းတာ နုတ်တာ နဲ့အတူတူဘဲ
```c
So if both pointers start at memory address 1000:

- int* → p + 1 would move to address 1004
- char* → p + 1 would move to address 1001
```


```c
#include <stdio.h>

int main() {
    char *c_ptr = (char*)0x1000;
    int *i_ptr = (int*)0x1000;
    double *d_ptr = (double*)0x1000;
    
    printf("char* + 1 = %p\n", c_ptr + 1);    // 0x1001 (+1 byte)
    printf("int* + 1 = %p\n", i_ptr + 1);     // 0x1004 (+4 bytes)
    printf("double* + 1 = %p\n", d_ptr + 1);  // 0x1008 (+8 bytes)
    
    return 0;
}
```

##### Pointer Subtraction

```c
int arr[5] = {10, 20, 30, 40, 50};
int *p = &arr[0];
int *q = &arr[4];

int diff = q - p;  // 4 (not 16 bytes!)
printf("Difference in elements: %d\n", diff);  // 4
printf("Difference in bytes: %ld\n", (char*)q - (char*)p);  // 16
```

Why? ဘာလို့ 16 bytes ကွာပေမယ့် 4 ပြန်တာလဲ? → Pointer subtraction က element count ကိုပြန်တယ်။

---


#### Array Indexing


Array indexing ဆိုတာ array ထဲက element တစ်ခုခုကို access လုပ်တဲ့နည်း** ဖြစ်ပြီး ၎င်းက pointer arithmetic ရဲ့ syntactic sugar

```c
arr[i]  ==  *(arr + i)
&arr[i] ==  arr + i
```
`[]`မပါဘဲ  `arr(array name)` ဒီကောင်ချည်းသက်သက်ခေါ်တာက array ရဲ့ memory address ကိုခေါ်တာနဲ့တူတူဘဲ (& ကိုသုံးပြီး)
ဆိုတော့ `&arr[i] ==  arr + i` ကအဲလိုဖြစ်လာတယ် index ရွှေ့ချင်ရင် `arr`  ကို  index ပေါင်းပေးရုံဘဲ


##### 1D Array
```c
int arr[5] = {100, 200, 300, 400, 500};

// These are EQUIVALENT:
arr[0] == *(arr + 0) == 100
arr[1] == *(arr + 1) == 200
arr[2] == *(arr + 2) == 300
arr[3] == *(arr + 3) == 400
arr[4] == *(arr + 4) == 500

// Addresses:
&arr[0] == arr + 0
&arr[1] == arr + 1
&arr[2] == arr + 2
```

```c
int arr[5] = {10, 20, 30, 40, 50};
int *p = arr;  // p points to arr[0]

// Now p can be used like array:
p[0] == 10
p[1] == 20
p[2] == 30
p[3] == 40
p[4] == 50

// And pointer arithmetic:
*(p + 2) == p[2] == 30
```

```c
arr (base address: 0x1000)
┌──────┬──────┬──────┬──────┬──────┐
│ 100  │ 200  │ 300  │ 400  │ 500  │
└──────┴──────┴──────┴──────┴──────┘
0x1000 0x1004 0x1008 0x100c 0x1010

arr[0] = *(arr + 0) = *(0x1000) = 100
arr[1] = *(arr + 1) = *(0x1004) = 200
arr[2] = *(arr + 2) = *(0x1008) = 300
```

##### 2D Array

```c
int matrix[3][4] = {
    {1, 2, 3, 4},
    {5, 6, 7, 8},
    {9, 10, 11, 12}
};
```

```c
matrix (base: 0x1000)
Row 0: [1][2][3][4]  (0x1000 - 0x100f)
Row 1: [5][6][7][8]  (0x1010 - 0x101f)
Row 2: [9][10][11][12] (0x1020 - 0x102f)
```

```c
// These are EQUIVALENT:
matrix[row][col] == *(*(matrix + row) + col)

// Example: matrix[1][2] = 7
matrix[1][2] == *(*(matrix + 1) + 2)
           == *(matrix[1] + 2)
           == 7

// Step by step:
matrix + 1     → points to row 1 (0x1010)
*(matrix + 1)  → row 1 array itself (also 0x1010)
*(matrix + 1) + 2 → address of element at col 2 (0x1018)
*(*(matrix + 1) + 2) → value at that address (7)
```

```c
matrix (0x1000)
    ↓
┌─────────────────────┐
│ Row 0: 0x1000       │────→ [1][2][3][4]
├─────────────────────┤
│ Row 1: 0x1010       │────→ [5][6][7][8]
├─────────────────────┤
│ Row 2: 0x1020       │────→ [9][10][11][12]
└─────────────────────┘

matrix[1][2]:
1. matrix + 1 = 0x1010 (row 1 address)
2. *(0x1010) = row 1 array (still 0x1010)
3. + 2 = 0x1018 (address of element 7)
4. *(0x1018) = 7
```

| Operation  | What it does               | Example (int*, 4-byte) |
| ---------- | -------------------------- | ---------------------- |
| `p + n`    | Move forward n elements    | `p+1` → +4 bytes       |
| `p - n`    | Move backward n elements   | `p-1` → -4 bytes       |
| `p++`      | Move to next element       | `p++` → +4 bytes       |
| `p--`      | Move to previous element   | `p--` → -4 bytes       |
| `q - p`    | Number of elements between | `q-p` = 3 (not 12)     |
| `arr[i]`   | Element at index i         | `arr[2]` = 30          |
| `*(arr+i)` | Same as arr[i]             | `*(arr+2)` = 30        |
| `p[i]`     | Element at index i from p  | `p[2]` = 30            |
| `*(p+i)`   | Same as p[i]               | `*(p+2)` = 30          |

---



---

#### pointer with heap

memset()
Memory block တစ်ခုကို specific value နဲ့ ဖြည့်ပေးတယ်

strdup()
String တစ်ခုကို duplicate လုပ်ပြီး memory အသစ် allocate လုပ်ပေးတယ်

```
service = strdup(line + 7);
1. `line + 7` မှာရှိတဲ့ string ကို ယူမယ်
    
2. အဲ့ string အတွက် memory အသစ် malloc() နဲ့ ယူမယ်
    
3. String ကို copy ကူးမယ်
    
4. Pointer ကို return ပြန်မယ်
```


---



