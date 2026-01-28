


bits -> bytes -> how many bytes ??
1 byte -> 8 bit
32 bit system  -> 4 byte system 
64 bit system -> 8 bytes sysetem
32-bit vs 64-bit က **POINTER size** ကွာတယ်
32-bit vs 64-bit က **ADDRESS range** ကွာတယ်


little endian 


---
## Variable naming system

	decomplierတွေက variable name ကိုပေးချင်တိုင်းပေးတာမဟုတ်ပါဘူး အရင်ဆုံး prefix လေးတွေကြည့်ရအောင်

| Prefix     | Meaning                          | Example               |
| ---------- | -------------------------------- | --------------------- |
| `local_`   | Stack variable                   | `local_c`, `local_10` |
| `param_`   | Function parameter               | `param_1`, `param_2`  |
| `uVar`     | Undefined variable               | `uVar1`, `uVar2`      |
| `iVar`     | Integer variable                 | `iVar1`, `iVar2`      |
| `bVar`     | Boolean variable                 | `bVar4`               |
| `pcVar`    | Pointer to char                  | `pcVar5`              |
| `piVar`    | Pointer to int                   | `piVar6`              |
| `ppcVar`   | Pointer to pointer to char       | `ppcVar7`             |
| `DAT_`     | global variable                  | `DAT_00402000`        |
| `FUN_xxxx` | function name                    | `FUN_00401000`        |
| `LAB_xxxx` | Code label Jump/call destination | `LAB_00401050`        |
	local_ - prefix နဲ့ဆို stack variable ပါ သူ့နောက်ကကောင်က hex အနေနဲ့လာပြီး stack frame (base pointer) ကနေ ဘယ်လောက်ဝေးလဲဆိုတာပြပါတယ်
	param_ - ကထုံးစံအတိုင်း paramterတွေပေါ့
	ပုံမှန် variable အတွက်က Var ပေါ့ သူရှေ့က data type ပေါ်မူတည်ပြီး integer ဆို i , undefined ဆို  i အစရှိသဖြင့်ပေါ့..
	DATA_  - ကတော့ global variable ပေါ့ 
	 &DATA_ - ဆိုရင် global variable ရဲ့ memory address

```
// Memory layout:
// Address    Value
// 0x00602140: 0x1000
// 0x00602148: 0x2000
// 0x00602150: 0x3000

DAT_00602140 = 0x1000      // value at address
&DAT_00602140 = 0x00602140 // address itself
```

## Data Type

	 Data type declare လုပ်တဲ့အခါလည်း မြင်နေကြမဟုတ်တာရှိပါတယ်
	 uint/ulong - ဒီကောင်တွေမှာက u က unsigned ကိုပြောတာပါ အနုတ်ကိန်းမပါတာကို ပြောတာပါ
		        unsigned int ဆို အနုတ်ကိန်းမပါတဲ့ integer
	undefined - decompiler က မသိတဲ့ datatype ဆို undefined သတ်မှတ်ပေးပါတယ် 8bytes
				data ဆို undefined8 4 bytes data ဆို undefined4 ဆိုပြီး သတ်မှတ်
	void * -     genereic pointer အနေနဲ့သတ်မှတ်
				Universal pointer type လို့လည်းခေါ်
			    pointer တစ်ခုကို  void * ptr ဆိုပြီး ကြေညာထားရင် ptr ကို ဘယ် 
				data typeရဲ့ memory address နေနေ store လုပ်လို့ရ
				(void ကို variable data type အဖြစ်သုံးလို့မရ pointer အတွက်သာ။ pointer
				arithemic မလုပ်နိုင်)
	size_t - unsigned interger
	__gid_t - data type တစ်မျိုး (group ID သိမ်းဖို့)

			
	


\0
	strings တွေအဆုံးမှာ null terminator အမြဲပါ
	`strlen` ဆို \0 အတွက်ပါထည့်မရေ
	`sizeof()` ဆို \0 အတွက်ပါ count
	`sizeof()` က strings size သတ်မှတ်ထားရင် အဲ size ဘဲပြ not actual string length
	
	sizeof(ptr)    // pointer variable ရဲ့ size (usually 8 bytes)
	sizeof(*ptr)   // pointer က ညွှန်ပြနေတဲ့ data type ရဲ့ size


### Stack Canary

```c
long in_FS_OFFSET;          // FS segment register offset
stack_canary = *(long *)(in_FS_OFFSET + 0x28);
```
1. Function စဝင်တိုင်း stack ရဲ့ အစမှာ random value တစ်ခုထားတယ် (canary)
    
2. Function ပြီးတိုင်း ဒီ value မပြောင်းလဲသေးရင် check လုပ်တယ်
    
3. Buffer overflow ဖြစ်ရင် canary value ပျက်သွားမယ်
    
4. ပျက်သွားရင် program က crash ဖြစ်မယ်

### 0xffffffff (32bit) က -1 နဲ့ ညီတာလဲ

-1 ကို signed integer  negative အနေနဲ့သုံးပြီး unsigned integer မှာကြတော့ negative ကို 0xffffffff အနေနဲ့သုံးပါတယ် ဘာလို့လဲ ?? signed မှာ -1ဆိုတာ 0 မတိုခင်ကောင်ပါ counter မရေတွက်ခင် ။ counter တစ်ခုမှာ limit ဆိုတာရှိပါတယ် limit ကျော်သွားရင် အစက ပြန်စပါတယ်  ဆိုတော့ 0xffffffff ပြီးရင် 0x0ကပြန်စမှာဆိုတော့ ဒီကောင်က -1 လိုသဘောတရားနဲ့အတူတူဘဲ

###  user_input_long & 0xffffffff


`user_input_long & 0xffffffff` က ဘာလုပ်ပေးသလဲဆိုရင် -

`user_input_long` ဆိုတဲ့ variable ထဲက **မူရင်းတန်ဖိုး** ကို `0xffffffff` နဲ့ **bitwise AND** operation လုပ်ပါတယ်။

`0xffffffff` ဆိုတာ hexadecimal ဖြစ်ပြီး 32-bit (4 bytes) ရှိတဲ့ binary value တစ်ခုပါ။

- Decimal: `4294967295`
    
- Binary: `11111111111111111111111111111111` (1 ဆို 32 လုံး)
    

**အဓိကရလဒ် ၂ မျိုး ရနိုင်တယ်:**

1. **`user_input_long` က positive ဖြစ်ရင် / သေးရင်**
    
    - တန်ဖိုး ပြောင်းမသွားပါဘူး။
        
    - Example: `5 & 0xffffffff = 5`
        
2. **`user_input_long` က negative ဖြစ်ရင် / 32-bit ထက်ကြီးရင်**
    
    - 32-bit range အတွင်း **wrap around** လုပ်ပေးတာ (modulo လိုမျိုး)
        
    - Negative numbers တွေကို unsigned 32-bit ပုံစံပြောင်းပေးတယ်
        
    - Example in Python:  
        `-1 & 0xffffffff = 4294967295`  
        `-5 & 0xffffffff = 4294967291`
        

**အသုံးဝင်ပုံ:**

- 32-bit system တွေမှာ integer values တွေကို limit ချဖို့
    
- Memory addresses, hash calculations, networking protocols တွေမှာသုံးတယ်
    
- Negative numbers တွေကို unsigned အနေနဲ့ ကိုင်တွယ်ဖို့


---

### Array Access
```
ပုံမှန် array access: array[index]
Memory calculation: array_address + (index * element_size)
```



```c
void *local_80_pointer;
local_80_pointer = *(void **)(&DAT_00602140 + (user_input_long & 0xffffffff) * 8);
```


	(user_input_long & 0xffffffff)ကနေ 32 bit ဖြတ်လိုက်တယ် တကယ်တမ်းက အဲ‌လောက်အထိ
	အများကြီး input မလာဘူး ဥပမာ 3 ဆို 3*8 = 24 bytes
	&DAT_00602140 ဒီ global variable addressကို ယူ 24 bytes ပေါင်း
	ဉပမာ 0x00602158 ရ ။
	(void **) ဒီကောင်က type casting ** ဆိုတော့ pointer ကို to pointer ဖြစ်အောင်လုပ် 
	0x00602158မှာ pointer တစ်ခုstore ထားပါတယ်ပေါ့
	ရှေ့ဆုံးက *က 0x00602158မှာ ရှိတဲ့ pointer ကိုယူလိုက်ပြီး local_80_pointer မှာ store လုပ် 


```c
sVar2 = fread(local_80_pointer, 1, local_88, stdin);
```
- `local_80_pointer` - data သိမ်းမယ့် memory address
    
- `1` - item 1 ခုရဲ့ size = 1 byte (ဆိုလိုတာ byte-by-byte ဖတ်မယ်)
    
- `local_88` - ဘယ်နှစ် bytes ဖတ်မလဲ
    
- `stdin` - standard input (keyboard/input) ကနေဖတ်မယ်

### Pointer Note

	pointer တစ်ခုကို declare လုပ်မယ်ဆို int *ptr = ...  ဆိုပြီးကြေညာ မူရင်း variable value ကို ယူချင်ရင် declare လုပ်ထားတဲ့အတိုင်း *ptr ပြန်သုံး
	pointer to pointer လည်းအဲအတိုင်းဘဲ **ptr , *ptr ဆို variable ရဲ့ memory address
### Pointer Arithmetic 

	pointer တစ်ခုကို 1 add or 1 subtract လိုက်ရင်  အဲ pointerမှာ store လုပ်ထားတဲ့ memory address ကို အဲ pointer ရဲ့ data type size ကိုပေါင်းတာနုတ်တာနဲ့အတူတူဘဲ
```c
So if both pointers start at memory address 1000:

- int* → p + 1 would move to address 1004
- char* → p + 1 would move to address 1001
```

### Pointer to Pointer(**)
	pointer က variable တစ်ခုရဲ့ memoryaddress ကို store လုပ်တယ် 
	အဲpointer ရဲ့ memory address ကို store လုပ်ချင်ရင် ** သုံးပြီး ptr တစ်ခုခေါ်ပြီးလုပ်လို့ရ

```c
int myNum = 10;       // normal variable  
int *ptr = &myNum;    // pointer to int  
int **pptr = &ptr;    // pointer to pointer
```


---
### Built in Functions

	fget(variableToStore, size_to_accept_including_null_terminator, stdin)


	atoll() -  ascii to long long ပြောင်းပေး
```c
char str[] = "123456789012";
long num = atol(str);      // 32-bit integer
long long big_num = atoll(str); // 64-bit integer
```
	ato , a - ascii , to is just to, next is what you want tochange ?? i for integer, l for long ....



## `memmem()`

 
```c
char data[] = "hello this is my badge number";
void *result = memmem(data, strlen(data), "badge", 5);
// result != NULL ဖြစ်မယ်၊ ဘာလို့လဲဆို "badge" ပါတယ်။
```
	data ဆိုတဲ့ string ထဲမှာ badge ဆိုတာကိုရှာတာ


## `strcspn()`
	ပထမ string ထဲတွင်၊ ဒုတိယ string ထဲရှိ character များ မတွေ့မချင်း ရေတွက်သည်
```c
char str[] = "Hello World";
size_t result = strcspn(str, " ");
// result = 5 ("Hello" ထဲတွင် space မတွေ့သေးပါ)
```


### `scanf()` က whitespace (space, tab, newline) တွေ့တာနဲ့ input reading ရပ်သွားတယ်

how to bypass ??
```
cat<flag.txt
cat$(ls)
```


### `fflush(stdout)`
	
	Output buffer ကိုရှင်းပြီး ချက်ချင်းပြစေ
	User က output အားလုံးကို သေချာမြင်ရအောင်၊ interaction ကောင်းအောင်

### `signal(0xb, sigsegv_handler);`

	Segmentation fault signal ဖြစ်ရင် ဘာလုပ်မလဲဆိုတာ ကြိုတင်သတ်မှတ်ထားခြင်း
	0xb = Signal number 11, ဒါက SIGSEGV (Segmentation Violation) signal
	 sigsegv_handler = အဲဒီ signal ဖြစ်ရင် run မယ့် function နာမည်

### `setresgid(local_1c, local_1c, local_1c`

	Program ရဲ့ Group ID သုံးခုလုံးကို တူအောင်ပြောင်းတာ
	local_1c = getegid() ကနေ ရလာတဲ့ effective group ID
	setresgid() က real GID, effective GID, saved set-GID ဆိုတဲ့ group ID သုံးမျိုးလုံးကို ပြောင်းပေးတယ်
    ဒီမှာ သုံးခုလုံးကို local_1cတူတူပြောင်းထား
```
# ဥပမာ program က setgid bit နဲ့ run ထားရင်
$ ls -l program
-rwsr-sr-x 1 root ctf_team 12345 Jan 1 12:00 program
# ^ s bit ရှိတယ်၊ ဒါကြောင့် program က root အုပ်စု permission နဲ့ run နိုင်တယ်

# ဒါပေမယ့် setresgid() ခေါ်ပြီး group ID ကို ပြန်ချိန်လိုက်တဲ့အတွက်
# နောက်ပိုင်း operations တွေက ပုံမှန် user permission နဲ့ပဲ run မယ်
# ဒါက privilege escalation attacks ကို ကာကွယ်ဖို့ပါ
```


### `setvbuf((FILE *)stdout,(char *)0x0,2,0);`

	 Output buffering mode ကိုချိန်ညှိတယ်
	- stdout = standard output (စခရင်ကိုရေးမယ့် stream)
	- (char *)0x0 = NULL pointer (no buffer သုံးဘူး)
	- 2 = _IONBF (no buffering) mode
	- ဆိုလိုတာ: Output တွေကို buffer မထည့်ဘဲ ချက်ချင်းပြခိုင်းတာ
	- ဥပမာ: printf("Hello"); ဆိုရင် "Hello"က ချက်ချင်းပေါ်မယ် (စောင့်မနေဘူး)


---

### Memory Management 

pointer တွေဘဲသုံးလို့ရ variable သုံးလို့မရ

Memory allocation မှာ function 2 မျိုးရှိ

	 int *ptr1 = malloc(size);     - parameter 1ခုထဲပါ ကိုယ်ယူမယ့် size
		                        - malloc()မှာက initialization မရှိဘူး allocation အစ
		                            မှာ gabage တွေချည်း
		                            
	 int *ptr2 = calloc(_amount_, _size_);   - parameter 2ခုပါ ကိုယ်ယူမယ့် size ရယ်၊ အဲ
	                                       လို size ဘယ်နှစ်ကောင်ယူမလဲဆိုတာရယ်
	                                       - calloc()မှာက initialization ရှိ 0 တွေနဲ့
	                                        လိုက်ဖြည့်

Accessing Memory 
```c
int *ptr;  
ptr = calloc(4, sizeof(*ptr));  
  
// Write to the memory  
*ptr = 2;   //Same like ptr[0]
ptr[1] = 4;  
ptr[2] = 6;
```


Re allocate -> Reallocate

	 memory size မလောက်ရင် size တိုး တဲ့ကောင် 
	 int *ptr2 = realloc(ptr1, size);
	 မူလmemory မှာ size တိုးလို့ရရင် မူလမှာဘဲ တိုးပေး အရင် data ဆက်ရှိနေ တိုးလို့မရရင် different addressမှာသွားလုပ်
	  လုပ်လို့မရရင် ptr2 is set to NUll

Deallocate (free) Memory

```c
int *ptr;  
ptr = malloc(sizeof(*ptr));  
  
free(ptr); 
ptr = NULL;
```
	ptrကို NULL မထားရင် use after free vuln ဖြစ်နိုင် 


Struct with Pointer


```c
struct Car *ptr = (struct Car*) malloc(sizeof(struct Car));
```
We use `->` to access members through the pointer


---
