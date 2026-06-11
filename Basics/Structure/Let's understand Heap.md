

## Glibc
	Glibc (GNU C Library) က Linux operating system အတွက် standard C library အဓိကတစ်ခုဖြစ်။ 

```
Glibc က 4 အဓိက လုပ်ဆောင်ချက်တွေလုပ်ပေးတယ်:

1. System Calls Wrapper
   └── User code → Glibc → Kernel system calls
   └── Example: printf() → write() system call

2. Standard C Library Implementation  
   └── ISO C Standard အကုန်လုံးပါ
   └── printf(), malloc(), strcpy(), fopen(), etc.

3. POSIX Compliance (Unix Standard)
   └── fork(), exec(), pthread_create(), socket()
   └── မတူတဲ့ Unix systems တွေမှာ တူညီတဲ့ interface

4. Memory Management (Heap Allocator)
   └── malloc(), free(), calloc(), realloc()
   └── ptmalloc2 algorithm သုံးထားတယ်
```
	check glibc version with
	 ldd --version




	challenge တွေဖြေတဲ့အခါမှာ တစ်ခါတစ်လေ libc version တူပေမဲ့ offset တွေကွဲနေတတ်တယ် ဆိုတော့ be careful . ဘာလို့ မတူလဲဆို

1. **Different Build Configurations**  
    `libc` ကို compile တဲ့အခါ configuration options တွေ (ဥပမာ: `--enable-debug`, `--with-float`, threading model) ပေါ်မူတည်ပြီး symbol offsets တွေပြောင်းသွားနိုင်တယ်။
    
2. **Patch Level ကွာခြားမှု**  
   ` glibc 2.23` ထဲမှာတောင် security patches, bug fixes တွေအမျိုးမျိုးရှိတယ်။ အဲ့တာတွေကလည်း offsets တွေကိုပြောင်းစေနိုင်တယ်။
    
3. **Architecture/Platform Differences**  
     `libc` နှစ်ခုကို မတူတဲ့ system ကနေ download လုပ်ထားတာ (သို့) compile လုပ်ထားတာ ဖြစ်နိုင်တယ်။






---

### what is Heap ??


Heap ဆိုတာက data တွေ သိမ်းဆည်းမယ့် နေရာ တစ်ခုဖြစ်ပါတယ်။  အခြေခံအားဖြင့်နားလည်ရမယ်ဆိုရင် malloc() နဲ့ data တွေအတွက်နေရာယူပြီး free() နဲ့ dataတွေကိုရှင်းလင်းပေးပါတယ်။ 
malloc() အပြင် `calloc , realloc and mmemalign` တွေကို အသုံးပြုလို့ရတယ်။ Heap ကို chunk တွေနဲ့တည်ဆောက်ထားပါတယ်။ 



##### Memory Chunk and Chunk allocation

ဆိုတော့ ငါတို့က malloc ကို သုံးပြီး 10 byte နေရာယူမယ်ဆိုပါစို့  ဒါမဲ့ heap manager က 10 Byte ထက်ပိုတဲ့ region ကို ရှာတယ် ဘာလို့ဆို allocation related metadata ကို store လုပ်ဖို့အတွက် heap ရှေ့နေရာပိုယူတယ်
- 64-bit မှာ `prev_size` က $8\text{ bytes}$၊ `size` က $8\text{ bytes}$ စုစုပေါင်း **$16\text{ bytes}$** ပိုယူပါတယ်
- 32-bit မှာ `prev_size` က $4\text{ bytes}$၊ `size` က $4\text{ bytes}$ စုစုပေါင်း **$8\text{ bytes}$** ပဲ ပိုယူပါတယ်
metadata မှာက chunk size ရယ် AMP info ရယ်  prev_sizeရယ်ပါပါတယ်

 architecture alignment system 
PU တွေက Memory ကို ဖတ်တဲ့အခါ `8 bytes` သို့မဟုတ် `16 bytes` လိုင်းအလိုက် တန်းစီဖတ်ရတာ ပိုမြန်ပါတယ် ဒါကြောင့် Heap Manager က Chunk အရွယ်အစားတွေကို စိတ်ကြိုက် ဖြတ်ချခွင့်မရှိပါဘူး
-  64-bit မှာ Chunk Size စုစုပေါင်းဟာ အမြဲတမ်း 16-byte multiple ဖြစ်ရပါမယ်(ဥပမာ- 32, 48, 64, 80, 96, ...)
- 32-bit မှာ Chunk Size စုစုပေါင်းဟာ8-byte multiple ဖြစ်ရပါမယ် (ဥပမာ- 16, 24, 32, 40, 48, ...)

နောက်တစ်ခု က minimum size
Chunk တစ်ခုက Free ဖြစ်သွားတဲ့အခါ (အပေါ်မှာ ငါတို့ဆွေးနွေးခဲ့တဲ့) Bins တွေထဲမှာ Linked List အနေနဲ့ ချိတ်ဖို့အတွက် `fd` pointer တွေ၊ `bk` pointer တွေ ထည့်ဖို့ နေရာလိုပါတယ် ဒါကြောင့် Chunk တစ်ခုဟာ ဘယ်လောက်ပဲ သေးသေး အောက်ပါ အနည်းဆုံး Size ထက် မငယ်ရဘူး လို့ ကန့်သတ်ထားပါတယ်
- 64-bit မှာ Minimum Size `32 bytes` ($0x20$)
- 32-bit မှာ Minimum Size `16 bytes` ($0x10$)

 examples for 64 bit system
 
 `malloc(1)` 
1. သင်္ချာနည်းအရ ပေါင်းခြင်း: $1\text{ byte (User)} + 16\text{ bytes (Header)} = 17\text{ bytes}$
2. Alignment ညှိခြင်း:  16-byte multiple ဖြစ်အောင် ညှိလိုက်ရင် `32 bytes` ဖြစ်သွားပါတယ်။
3. **Total Chunk Size:** **`32 bytes`** ($0x20$) ဖြစ်သွားပါတယ်။ _(ဒီနေရာမှာ User သုံးဖို့ တကယ့် Payload space က $16\text{ bytes}$ တောင် ရနေမှာပါ)_
    

 `malloc(16)` 
1. သင်္ချာနည်းအရ ပေါင်းခြင်း: $16\text{ bytes (User)} + 16\text{ bytes (Header)} = 32\text{ bytes}$
2. Alignment ညှိခြင်း: `32` ဟာ 16-byte multiple  ကွက်တိ ဖြစ်နေပါတယ်။
3. **Total Chunk Size:** **`32 bytes`** ($0x20$) အတိအကျ ရပါတယ်။ _(ဒါကြောင့် `malloc(1)` တောင်းတောင်း `malloc(16)` တောင်းတောင်း RAM ပေါ်မှာ နေရာယူတာ အတူတူပဲလို့ ပြောတာပါ)_
    

 `malloc(17)`
1. သင်္ချာနည်းအရ ပေါင်းခြင်း: $17\text{ bytes (User)} + 16\text{ bytes (Header)} = 33\text{ bytes}$
2. Alignment ညှိခြင်း: `33`  16-byte multiple `32` ထက် ကျော်သွားပါပြီ။ ဒါကြောင့် နောက်ထပ် မှတ်တိုင်ဖြစ်တဲ့ `48` ဘက်ကို တိုးပေးရပါတယ်
3. **Total Chunk Size:** **`48 bytes`** ($0x30$) ဖြစ်သွားပါတယ်

‌allocate လုပ်ပြီးသွားရင်  user data ရှိတဲ့ memory address ကို return ပြန်တယ် (metadata ကစတာမဟုတ်ဘူး) ပုံမှန် အားဖြင့် `eax` registerထဲမှာ ထည့်တယ်




![](chunk-allocated-CS.png)

	chunk တစ်ခုမှာ အဲ chunk မတိုင်ခင် chunk ကို previous chunk လို့ခေါ်တယ်  previous chunk က free ဖြစ်နေရင် current chunk free လုပ်လိုက်တဲ့အခါ chunk နှစ်ခု ပေါင်းလို့ရတယ် (coalescing) ဆိုတော့ previous chunk freeဆိုရင် current chunk ရဲ့ prev_sizeမှာ previous chunk size ထည့်ထားတယ် free မဟုတ် ဘူးဆိုရင် prve size နေရာကို user data အဖြစ်သုံးတယ်


Malloc ကဘယ်လိုနေရာယူလဲဆိုရင်

- previously freeလုပ်ထားတဲ့ chunckတွေကို bin ထဲမှာလိုက်ရှာတယ် ကိုက် ညီတဲ့ chunk တွေ့ရင် ယူတယ် မတွေ့ရင်
- top heapမှာ နေရာအလွတ်ကျန်သေးရင် new chunk create လုပ်တယ် နေရာလွတ် မကျန်တော့ရင်
- heap manager က kernel ကို heap အစွန်းမှာ memory ထည့်ပေးဖို့တောင်းတယ် ထပ်တောင်းလို့မရတော့ရင်
- malloc return Null





## Arena 

	Arena ဆိုတာ ptmalloc2 memory allocator မှာ heap memory ကို စီမံခန့်ခွဲဖို့အတွက် သီးသန့်နေရာတစ်ခု ဖြစ်တယ်  Multi-threading Performance အတွက်ဖြစ်တယ် malloc() function ကိုသုံးဖို့ဆိုရင် thread တစ်ခုချင်းစီတိုင်းက lock ယူထားတဲ့ threadကို စောင့်နေရတယ်။ ဆိုတော့ အဲလိုမစောင့်ရအောင် thread တစ်ခုချင်းစီတိုင်းကို Arena သုံးပြီး imporve performance ဖြစ်အောင်လုပ်တာ မတူညီတဲ့ Arenas သုံးလို့ lock contention နည်းတယ် ။ Arena တစ်ခုစီမှာသူ့ရဲ့ ကိုယ်ပိုင်

- **Heap memory region**
- **Bins `(tcache, fastbin, smallbin, unsortedbin, largebin)`**
- **Lock (mutex)**

	ပါဝင်တယ် 
	program စတာနဲ့စပြီး create လုပ်တဲ့ heap ကို main arena လို့ခေါ်တယ် Single-threaded program တွေအတွက် ဒီကောင်အဆင်ပြေတယ် sbrk()/brk() သုံးပြီး heap တိုးတယ်

##### Thread Arena

	Thread အသစ်တစ်ခုစီအတွက် ဖန်တီးထားတာကို Thread Arena (Non-main Arena)/Secondary Arena လို့ခေါ်တယ်` mmap()` သုံးပြီး anonymous mapping လုပ်တယ် Shared Library Region တွေကြားထဲမှာ နေရာသွားယူတယ် Heap memory တွေက မဆက်စပ်တဲ့နေရာမှာရှိတယ် (secondary arena က heap နဲ့ တစ်ဆက်တည်းမဟုတ်ပါဘူး shared library region လိုနေရာမှာ memory အသစ် တစ်ခု create လုပ်သလိုဖြစ်ပါတယ် memory ပေါ်မှာ ကွက်ကြားကွက်ကြား  (Non-contiguous) ရှိတဲ့ သဘောမျိူးပါဘဲ) အဲလို ယူတဲ့ကောင်ကို Sub-heap  လို့လည်းခေါ်တယ်
	Thread အသစ်တစ်ခုစီအတွက် new arena ဖန်တီးနိုင်တယ် (large allocation တွေအတွက် main arena ကလည်း` mmap() `သုံးပြီးနေရာယူပါတယ် `malloc()` ကနေ ပမာဏအကြီးကြီး ဥပမာ- 128 KB ထက်ကြီးတာမျိုး) တောင်းရင် Main Arena လည်း `mmap()` သုံးပြီး သီးသန့်နေရာသွားယူတတ်  ဒါမဲ့ `sbrk()/brk() `လို တစ်ဆက်တည်း မဟုတ်တော့ )

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

void* thread_function(void* arg) {
    int thread_num = *((int*)arg);
    printf("Thread %d: Starting and allocating memory...\n", thread_num);
    
    // Thread တစ်ခုချင်းစီက memory ပြိုင်တူ တောင်းကြမယ်
    void* ptr = malloc(100); 
    
    // ဒီနေရာမှာ ptmalloc က Lock မငြိအောင် Thread Arena တွေ ခွဲပေးပါလိမ့်မယ်
    
    printf("Thread %d: Allocated at address: %p\n", thread_num, ptr);
    
    sleep(2); // memory ကို ခဏကိုင်ထားမယ်
    free(ptr);
    return NULL;
}

int main() {
    pthread_t threads[3];
    int thread_ids[3] = {1, 2, 3};

    // Thread ၃ ခုကို တစ်ပြိုင်နက်တည်း (concurrently) Run လိုက်မယ်
    for (int i = 0; i < 3; i++) {
        pthread_create(&threads[i], NULL, thread_function, &thread_ids[i]);
    }

    // Thread တွေ အလုပ်ပြီးတဲ့အထိ စောင့်မယ်
    for (int i = 0; i < 3; i++) {
        pthread_join(threads[i], NULL);
    }

    return 0;
}
```

- Main Thread စပွင့်ချိန် OS က Program ကို စ run တဲ့အခါ Main Thread ပေါ်လာပြီး Main Arena (Main Heap) ကို `brk()` နဲ့ စတင် တည်ဆောက်ပါတယ်
- Thread 1, 2, 3 တွေ အလုပ်လုပ်ချိန်
    - Thread 1 က `malloc(100)` ကို အရင်ဆုံး လှမ်းခေါ်လိုက်တယ်။ သူက Main Arena ရဲ့ Lock ကို သွားစစ်တယ်။ အားနေရင် Main Arena ထဲကပဲ Chunk တစ်ခု ဖြတ်ပေးလိုက်မယ်
    - Thread 2 ကလည်း တစ်ပြိုင်တည်းမှာ `malloc(100)` ထပ်ခေါ်တယ်။ ဒါပေမဲ့ Main Arena က Thread 1 ကြောင့် Lock ဖြစ်နေပြီ။  Lock Contention(lock ပြိုင်လုတာ) ဖြစ်နေပြီ အဲဒီအခါ Thread 2 က စောင့်မနေတော့ဘဲ Thread Arena 1 (Non-main Arena) အသစ်တစ်ခုကို `mmap()` သုံးပြီး ဆောက်လိုက်တယ်။ အဲဒီ Arena 1 ထဲက Sub-heap ထဲကနေ Chunk တစ်ခု ယူလိုက်ပါတယ်
    - Thread 3 ကလည်း ထပ်ခေါ်တဲ့အခါ Main Arena ရော၊ Thread Arena 1 ရော Lock မိနေရင် သူ့အတွက် **Thread Arena 2** ကို ထပ်ဆောက်ပြီး အဲဒီထဲက Sub-heap ကနေ Chunk ထုတ်ပေးလိုက်ပါတယ်


Glibc 2.26 နောက်ပိုင်းမှာ `tcache` ပါလာတဲ့အတွက် Thread တွေဟာ တော်ရုံတန်ရုံ `malloc` / `free` ကို ကိုယ်ပိုင် Tcache ထဲမှာတင် Lock လုံးဝမလိုဘဲ အလုပ်လုပ်နိုင်ပါတယ်။ ဒါကြောင့် Thread Arena အသစ် ဆောက်တာကို မြင်ချင်ရင် **Tcache ထက်ကြီးတဲ့ size (ဥပမာ- Chunk size > 0x410)** ကို တောင်းပြီး တစ်ပြိုင်နက်တည်း လုသုံးခိုင်းမှ Thread Arena Trigger ဖြစ်မှာပါ
#####






![](heap-arenas-CS.png)


#### What is Thread

Thread ဆိုတာ Operating System က CPU ပေါ်မှာ Task တစ်ခုကို အလုပ်လုပ်ခိုင်းဖို့အတွက် ပေးလိုက်တဲ့ Lightweight Process / Unit of Execution

##### Process Vs Thread

- Process :  Binary တစ်ခုကို Run လိုက်တဲ့အခါ OS က သူ့အတွက် သီးသန့် Virtual Memory Space (Address Space) ကြီးတစ်ခုလုံးကို သီးသန့် ချပေးလိုက်တာ ဖြစ်ပါတယ် အဲဒီ Process ထဲမှာ `Code`, `Data`, `Heap` နဲ့ `Shared Libraries` တွေ အကုန်ပါဝင်ပြီး တခြား Process တွေက လာရောက် ကျူးကျော်ဖတ်ရှုလို့ မရပါဘူး (Isolated ဖြစ်ပါတယ်)
    
- Thread : Thread ဆိုတာကတော့ အဲဒီ Process ကြီးရဲ့ အတွင်းထဲမှာ တကယ့် CPU ကုဒ်တွေကို လိုက်ပြီး Run ပေးရတဲ့ ကောင်တွေ ဖြစ်ပါတယ် ပိုပြီး ထူးခြားတာက Process တစ်ခုထဲမှာ Thread ပေါင်းများစွာ (Multi-threading) ရှိနေနိုင်ပါတယ်


##### Shared vs Unique

Multi-threaded program တစ်ခု (ဥပမာ- Glibc `ptmalloc` သုံးထားတဲ့ web server တစ်ခု) အလုပ်လုပ်တဲ့အခါ Thread အားလုံးဟာ Process ရဲ့ The Process Address Space ကြီးကို အတူတူ မျှဝေသုံးစွဲကြပေမဲ့၊ သူတို့အချင်းချင်း အလုပ်ရှုပ်မကုန်အောင် Thread-Local Infrastructure (ကိုယ်ပိုင် ပစ္စည်းအချို့) ကို သီးသန့် ကိုင်ဆောင်ထားကြပါတယ်

```
       ┌────────────────────────────────────────────────────────┐
       │             PROCESS MEMORY ADDRESS SPACE               │
       │  ┌──────────────────────┐   ┌───────────────────────┐  │
       │  │      Code Segment    │   │      Heap Segment     │  │
       │  └──────────────────────┘   └───────────────────────┘  │
       │             ▲                           ▲              │
       │             │ (Shared)                  │ (Shared)     │
       └─────────────┼───────────────────────────┼──────────────┘
                     │                           │
          ┌──────────┴──────────┐     ┌──────────┴──────────┐
          │      THREAD 1       │     │      THREAD 2       │
          ├─────────────────────┤     ├─────────────────────┤
          │  Unique Stack       │     │  Unique Stack       │
          │  Unique CPU Regs    │     │  Unique CPU Regs    │
          │  TLS (Tcache, etc)  │     │  TLS (Tcache, etc)  │
          └─────────────────────┘     └─────────────────────┘
```


Shared Area)

- The Code (0x400000 ~ Text Segment): Run မယ့် machine code (Assembly Instructions) တွေဟာ အတူတူပဲ ဖြစ်လို့ Thread အားလုံးက အဲဒီ Code တွေကို လှမ်း run နိုင်တယ်
- The Global Data (`.data` / `.bss`): Global Variable တွေ မှန်သမျှကို Thread အားလုံးက မျှဝေဖတ်ရှု/ပြင်ဆင်နိုင်တယ် (ဒီနေရာမှာ Race Condition ဆိုတဲ့ vulnerability စတင်ဖြစ်ပေါ်တတ်ပါတယ်)
- The Heap: ဒါက အရေးကြီးပါတယ် Main Arena Heap Memory ကို Thread အားလုံးက Share လုပ်ပြီး သုံးကြတာ ဖြစ်ပါတယ်
    

 (Private/Unique Area):

- Unique Stack (`rsp` / Stack Pointer): Thread တစ်ခုစီမှာ ကိုယ်ပိုင် Stack Segment တစ်ခုစီ သီးသန့်ရှိပါတယ် ဒါကြောင့် Thread A က function တစ်ခု ခေါ်နေချိန်မှာ Thread B ရဲ့ local variable တွေနဲ့ သွားပြီး ရောထွေးကုန်ခြင်း မရှိတာ ဖြစ်ပါတယ်
- Unique CPU Registers (rip, `rax`, `etc`): CPU ပေါ်မှာ ၎င်း Thread လက်ရှိ ဘယ် assembly line ကို ရောက်နေလဲဆိုတဲ့ Instruction Pointer (rip) နဲ့ တွက်ချက်နေတဲ့ Data (Registers) တွေက Thread တစ်ခုစီအတွက် သီးသန့် ဖြစ်ပါတယ်
- TLS (Thread Local Storage): ဒါက  **`Tcache`** ရှိတဲ့ နေရာ ဖြစ်ပါတယ် Thread တစ်ခုစီမှာ တခြားကောင်တွေ လာနှောင့်ယှက်လို့မရတဲ့ TLS ဆိုတဲ့ data ထားသိုမယ့်နေရာလေး ပါဝင်ပြီး ၎င်းထဲမှာ `tcache_perthread_struct` ကို သိမ်းဆည်းထားတာ ဖြစ်ပါတယ်

##### Thread's TLS

Thread တစ်ခုက `malloc(0x20)` တောင်းလိုက်တဲ့ အခြေအနေကို low-level အဆင့်ဆင့် ကြည့်ရအောင်

1. `Tcache` ကို အရင်စစ်မယ် (No Lock) Thread က သူ့ရဲ့ ကိုယ်ပိုင် TLS ထဲက `Tcache` ထဲမှာ `0x20` size ရှိတဲ့ Free Chunk ရှိမရှိ အရင်ကြည့်ပါတယ်။ (ဒီနေရာမှာ ကိုယ်ပိုင်အိတ်ကပ်ထဲ ရှာတာဖြစ်လို့ တခြား Thread တွေနဲ့ လုစရာမလိုဘူး၊ Arena Lock ချစရာမလိုဘူး၊ အလွန်မြန်ပါတယ်)။
2. `Tcache` မှာ မရှိရင် Arena ဆီသွားမယ် (Lock Required) အကယ်၍ `Tcache` ထဲမှာ အလွတ်မရှိရင် (သို့မဟုတ် မင်းပြောသလို Chunk Size က `> 0x410` ဖြစ်လို့ `Tcache` က လက်မခံရင်) အဲဒီ Thread က Global ကွင်းပြင်ဖြစ်တဲ့ Arena ဆီကို ထွက်လာရပါတော့တယ်
3. အဲဒီအချိန်မှာ ဘယ် Arena ဆီ သွားမလဲ?
    - သူက Main Arena ဆီကို အရင်သွားပြီး Lock ယူဖို့ ကြိုးစားပါတယ်
    - အကယ်၍ Main Arena က အားနေရင် Main Arena ဆီကနေ Memory Chunk ကို ဖြတ်ယူပြီး Thread ဆီ ပေးလိုက်ပါတယ်။ (ပြီးရင် နောက်တစ်ခါ သုံးရလွယ်အောင် အဲဒီ chunk တွေကို Thread ရဲ့ ကိုယ်ပိုင် `Tcache` ထဲမှာပါ အပိုထည့်သိမ်းပေးလိုက်ပါသေးတယ်)
    - အကယ်၍ Main Arena ကို တခြားကောင်က Lock ချထားလို့ (Lock Contention ဖြစ်လို့) သုံးမရရင် `ptmalloc` က အဲဒီ Thread အတွက် Non-main Arena တစ်ခုကို အသစ်ဆောက်ပေးပြီး ၎င်း Arena အသစ်ဆီကနေပဲ Memory ဖြတ်ပေးလိုက်တာ ဖြစ်ပါတယ်


#### Allocating  From  Freed chunks (from bins)

free chunk တွေကို bins တွေထဲထည့်ထားတယ် malloc ယူတဲ့အခါ ဒီကောင်တွေကိုပြန်သုံးတယ်
free chunk မှာ user data တွေကို `fd` နဲ့ `bk` နဲ့ အစားထိုးလိုက်တယ်
`Singly linked list (tcache, fastbin) `တွေမှာဆိုရင် `fd` (Forward Pointer) တစ်ခုတည်းကိုပဲ အစားထိုးပြီး `bk` (Backward Pointer) ကို အသုံးမပြုပါဘူး (မလိုအပ်လို့ပါ) `Doubly linked list (unsorted, small, large) `တွေကျမှသာ `fd` ရော `bk` ရော နှစ်ခုလုံးကို အစားထိုး အသုံးပြုတာ ဖြစ်တယ်
bin 5 မျိုးရှိတယ် 


#### Bins

bin ဆိုတာက free လုပ်လိုက်တဲ့ chunkတွေကို စနစ်တကျထာတဲ့ container တစ်ခုလိုဘဲ (free လုပ်လိုက်ရင် binထဲထည့်မယ် နောက်ထပ် storage တစ်ခုယူမယ်ဆိုရင် binထဲကနေ ကိုက်တဲ့ကောင်ကိုရှာမယ်)
ဥပမာ - သံပုရာသီး အလုံး 1000 ရှိတယ်ထားပါတော့ အကုန်လုံးကို ပုံးကြီးတစ်ပုံးထဲစုထည့်လိုက်ရင် size မညီမျှတော့ လိုချင်တဲ့ size မရဘူး
size အလိုက် ပုံးတွေထဲ ခွဲထည့်လိုက်ရင် လိုချင်တဲ့ size ကိုတန်းယူလို့ရတယ် bins တွေရဲ့သဘောက အဲလိုဘဲ
ရိုးရိုးရှင်းရှင်း ပြောရရင် `ptmalloc` allocator ထဲမှာ `bin` ဆိုတဲ့ array ကြီးတစ်ခု ရှိပါတယ်
ဒီ Array ရဲ့ အကွက်တစ်ခုစီ (Index တစ်ခုစီ) ကို Bin Number လို့ ခေါ်တာပါ `ptmalloc` က Size အလိုက် Bin နံပါတ်တွေကို တိတိကျကျ သတ်မှတ်ပေးလိုက်ပါတယ်
Free ဖြစ်သွားတဲ့ Chunk တွေကို  အဲဒီ ဘင်နံပါတ် အကွက်တစ်ခုစီကနေ Linked List ပုံစံနဲ့ လှမ်းပြီး ခေါင်းစဉ်တပ် ချိတ်ဆက်ထားပါတယ်

- **Bin 2:** $32\text{ bytes}$ ရှိတဲ့ အမှိုက်တွေကိုပဲ သိမ်းမယ်
- **Bin 3:** $40\text{ bytes}$ ရှိတဲ့ အမှိုက်တွေကိုပဲ သိမ်းမယ်
- **Bin 4:** $48\text{ bytes}$ ရှိတဲ့ အမှိုက်တွေကိုပဲ သိမ်းမယ်

Bin Numbers တွေ ရှိနေရတာဟာ `malloc()` ခေါ်တဲ့အခါ ကိုယ်လိုချင်တဲ့ Size ရှိတဲ့ Free Chunk ကို Memory ထဲမှာ မျက်စိမှိတ် လိုက်မရှာရဘဲ ဘယ်နေရာမှာ ရှိလဲဆိုတာ ချက်ချင်း တန်းသိစေဖို့ (Instant Lookup ရဖို့) အတွက် ဖြစ်ပါတယ်

bins စုစုပေါင်း 126 bins ရှိတယ်
`ptmalloc` ရဲ့ အတွင်းပိုင်း Source Code ထဲမှာ ဒီ Bin တွေကို သိမ်းဖို့အတွက် တည်ဆောက်ထားတဲ့ Array ရဲ့ အကျယ်အဝန်း (Size) က တကယ်တော့ `127` ကွက် ရှိတာ ဖြစ်ပါတယ်
Size က `127` ဆိုရင် သူ့ရဲ့ Index (နံပါတ်) တွေက `0` ကနေ `126` အထိ ရှိတယ်
ဒါမဲ့ index 0 ကို မသုံးဘူး တွက်ရလွယ်အောင် Bin နံပါတ်ကို `1` ကနေပဲ စချင်လို့ `0` ကို ကျော်ခဲ့တာ ဖြစ်ပါတယ်
ဆိုတော့ bin index က 1 ကနေစပြီး index 126 အထိ စုစုပေါင်း 126 binsရှိပါတယ်

| Array Index            | Bin Classification    | Count   |
| ---------------------- | --------------------- | ------- |
| **0**                  | **Unused** (not used) | 1       |
| **1**                  | **Unsorted Bin**      | 1       |
| **2 - 63**             | **Small Bins**        | 62      |
| **64 - 126**           | **Large Bins**        | 63      |
| **Total Regular Bins** |                       | **126** |

`Tcache` နဲ့ `Fastbin` ကတော့ သူတို့ရဲ့ Performance ပိုမြန်စေဖို့အတွက် ဒီ ပင်မ Bin Array ထဲမှာ မနေဘဲ သီးသန့် Array အသေးလေးတွေနဲ့ ခွဲနေကြတာ ဖြစ်ပါတယ်

| Bin Type | count | where locates                  |
| -------- | ----- | ------------------------------ |
| Fastbins | 10    | သီးခြား array (fastbinsY[0-9]) |
| Tcache   | 64    | Per-thread (arena အပြင်ဘက်)    |
Tcache ကတော့ အပေါ်က Arena တွေထဲမှာ ရှိမနေပါဘူး။ Thread တစ်ခုချင်းစီရဲ့ သီးသန့် Area ဖြစ်တဲ့ **TLS (Thread Local Storage)** ထဲမှာ တည်ရှိတာ ဖြစ်ပါတယ်
Arena တွေဆိုတာ Program တစ်ခုလုံး (Process-wide) မှာရှိတဲ့ Global Heap Memory ကွင်းပြင်ကြီးတွေ ဖြစ်ပါတယ် သူတို့က Thread တစ်ခုစီရဲ့ အတွင်းထဲမှာ ရှိတာမဟုတ်ပါဘူး Thread အားလုံး လှမ်းယူလို့ရတဲ့ နေရာမှာ ရှိနေတာပါ



	 Memory တွေ "Free" လုပ်လိုက်တဲ့အခါ ဘာဖြစ်သွားလဲ?

 C code ထဲမှာ `free(ptr)` လို့ ခေါ်လိုက်တာနဲ့ အဲဒီ memory တုံး (Chunk) တွေက အောက်ကလမ်းကြောင်းအတိုင်း Bins တွေထဲကို စီးဆင်းသွားတယ်
1. `Tcache` Bin (first) : အရွယ်အစားက သေးတယ် ($< 1016\text{ bytes}$)၊ ပြီးတော့ အဲဒီ Size အတွက် `Tcache` bin ထဲမှာ နေရာလွတ်သေးတယ် (7 ခုမပြည့်သေးဘူး) ဆိုရင် `Tcache` ထဲကို အရင်ဆုံး ပစ်ထည့်လိုက်ပါတယ်။ (သူက Lock မလိုလို့ အမြန်ဆုံး အလုပ်လုပ်တဲ့ နေရာပါ)

2. Unsorted Bin (second):  အကယ်၍ Size က ကြီးနေလို့ပဲဖြစ်ဖြစ်၊ သို့မဟုတ် `Tcache` ထဲမှာ 7 ခု ပြည့်သွားလို့ပဲဖြစ်ဖြစ်... Tcache ထဲ ထည့်လို့မရရင် `ptmalloc` က Size အလိုက် လိုက်မခွဲခြားတော့ဘဲ Unsorted Bin ဆိုတဲ့ temporary bin ထဲကို အကုန်လုံး စုပြုံပစ်ထည့်လိုက်ပါတယ်
    
3. Fast Bin (bat-man): တကယ်လို့ Size က အရမ်းသေးပြီး ($< 128\text{ bytes}$) အနီးအနားက chunk တွေနဲ့ သွားမပေါင်းစေချင်ရင်တော့ Unsorted bin ထဲမသွားဘဲ Fast bin ထဲကို သွားပါတယ်။



Free လုပ်လိုက်တဲ့အချိန်မှာ Small Bin နဲ့ Large Bin ထဲကို တိုက်ရိုက် လုံးဝမဝင်ပါဘူး unsorted bin ထဲကိုတန်းကောက်ထည့်လိုက်ပါတယ်
နောက်တစ်ခါ `malloc()` ခေါ်ပြီး Unsorted Bin ထဲကို လိုက်ရှင်းတဲ့အဆင့် (Sorting Phase) ကျမှသာ 
Chunk တွေကို လက်ခံရရှိတာ ဖြစ်ပါတယ်
Sorting phaseက ဘာလဲဆိုရင်
Code ထဲမှာ `malloc(40)` (64-bit မှာ Alignment အရ `0x30` သို့မဟုတ် `48 bytes` chunk လိုချင်လို့) လှမ်းခေါ်လိုက်ပြီ ဆိုပါစို့။ Tcache ရော Fastbin မှာရော မရှိတော့ဘူးဆိုရင် `ptmalloc` က Unsorted Bin ကြီးဆီကို ရောက်လာပါတယ်
Unsorted bin ထဲမှာ အရွယ်အစားမျိုးစုံရှိတဲ့ Chunk တွေ ရောပြွမ်းပြီး ချိတ်ဆက်နေပါတယ်။ `ptmalloc` က List ရဲ့ အနောက်ဆုံး (Tail) မှာ ရှိတဲ့ Chunk ကို အရင်ဆုံး ဆွဲထုတ်ပြီး သူ့ရဲ့ Size ကို ကြည့်ပါတယ်
- ဥပမာ- ဆွဲထုတ်လိုက်တဲ့ ပထမဆုံး chunk ကြီးက $2000\text{ bytes}$ ကြီး ဖြစ်နေတယ် ဆိုပါစို့
`tmalloc` က မေးခွန်းထုတ်ပါတယ်။ "ငါ အခု လိုချင်တာက $48\text{ bytes}$ တုံးလေး၊ အခု လက်ထဲရောက်လာတာက $2000\text{ bytes}$ ကြီး... ကိုက်ညီမှု ရှိလား?"
မကိုက်ညီပါဘူး (Size က ကွာခြားလွန်းနေလို့ ဖြတ်သုံးဖို့လည်း အဆင်မပြေသေးဘူး ဆိုပါစို့)။
ကိုယ့် `malloc` တောင်းတဲ့ size နဲ့ မကိုက်ညီတဲ့အတွက် ဒီ $2000\text{ bytes}$ Chunk ကြီးကို Unsorted bin ထဲမှာ ဆက်ထားလို့ မဖြစ်တော့ပါဘူး (ရှုပ်နေမှာစိုးလို့) အဲဒီအခါကျမှ `ptmalloc` က သူ့ရဲ့ Size ကို သေချာတွက်ချက်ပြီး-
 " $2000\text{ bytes}$ ဆိုတော့ $1024$ ထက် ကြီးတယ် ဒါဆိုရင် မင်းက Large Bin ထဲ သွားရမယ် $2000\text{ bytes}$ နဲ့ ဆီလျော်မယ့် Large Bin နံပါတ် (ဥပမာ Bin 64) ထဲကို သွားပြီး ကြီးစဉ်ငယ်လိုက် နေရာဝင်ယူလိုက်တော့" ဆိုပြီး ဒီအဆင့်ကျမှ Large Bin ထဲကို Push လုပ်ပြီး ထည့်လိုက်တာ ဖြစ်ပါတယ်
`ptmalloc` က အားမလျှော့ဘဲ Unsorted bin ထဲက နောက်ထပ် chunk တစ်တုံးကို ထပ်ဆွဲထုတ်ပါတယ်
- ဒီတစ်ခါ ဆွဲထုတ်လိုက်တဲ့ကောင်က $32\text{ bytes}$ ဖြစ်နေတယ် ဆိုပါစို့
- ငါလိုချင်တာက $48\text{ bytes}$ မို့လို့ ထပ်ပြီး မကိုက်ညီပြန်ပါဘူး
- အဲဒီအခါ Size ကို ထပ်စစ်ပါတယ်။ "$32\text{ bytes}$ ဆိုတော့ $1016$ ထက် သေးတယ်။ ဒါဆို မင်းက Small Bin အုပ်စုဝင်ပဲ။ $32\text{ bytes}$ သီးသန့်သိမ်းတဲ့ **Small Bin 2** ထဲကို သွားချိတ်လိုက်တော့" ဆိုပြီး ဒီအချိန်ကျမှ Small Bin ထဲ ရောက်သွားပြန်ပါတယ်
လမ်းကြုံလို့ Unsorted bin ထဲက Chunk တွေကို လိုက်ရှင်းပေးရင်းနဲ့...
မသက်ဆိုင်တဲ့ Chunk တွေကို Small Bin တွေ၊ Large Bin တွေထဲကို စနစ်တကျ ခွဲထုတ် (Sort) ပေးသွားတာ ဖြစ်ပါတယ်




	`malloc()` နဲ့ Memory ပြန်တောင်းတဲ့အခါ ဘယ်လိုရှာလဲ?

 `malloc(size)` လို့ ပြန်တောင်းတဲ့အခါ `ptmalloc`က အောက်ကအတိုင်း အစဉ်လိုက် လိုက်ရှာတယ်

	- `Tcache` Bin ကို အရင်ကြည့်မယ်
သူ့မှာ Lock သုံးစရာမလိုလို့ အမြန်ဆုံး ဂိတ်ဖြစ်ပါတယ် 
ကိုယ်တောင်းတဲ့ Size နဲ့ ကွက်တိတူတဲ့ Chunk ရှိလား ကြည့်ပြီး ရှိရင် ချက်ချင်းထုတ်ပေးလိုက်ပါတယ်

	 - Fast Bin ကို လှမ်းကြည့်မယ်
တောင်းတဲ့ size က သေးရင် Fast bin ထဲမှာ ရှိနေမလား လှမ်းရှာပါတယ်
ရှိရင် ယူသုံးပါတယ်

	 - Unsorted Bin  ကို ရှင်းလင်းရေးလုပ်မယ်
အထက်က ဂိတ်တွေမှာ မရှိရင် `ptmalloc` က Unsorted Bin ထဲက Chunk တွေကို တစ်ခုချင်းစီ ဆွဲထုတ်ပြီး စစ်ဆေးပါတယ်
- ဆွဲထုတ်လိုက်တဲ့ Chunk Size က ကိုယ်တောင်းတဲ့ Size နဲ့ ကွက်တိတူရင် (သို့) ဖြတ်သုံးလို့ရရင် -> အဲဒီကောင်ကို `malloc` အတွက် ထုတ်ပေးလိုက်ပါတယ်
- ကိုယ်တောင်းတဲ့ size နဲ့ မကိုက်ညီရင် -> အဲဒီ Chunk ကို သူ့ရဲ့ အရွယ်အစားအလိုက် Small Bin သို့မဟုတ် Large Bin ထဲကို သေချာ စနစ်တကျ ခွဲခြားပြီး ရွှေ့ပြောင်း (Sort) ပစ်လိုက်ပါတယ်
 Small Bin နဲ့ Large Bin ထဲကို Chunk တွေ ရောက်သွားတာဟာ `free()` လုပ်လို့ ရောက်သွားတာ မဟုတ်ပါဘူး `malloc()` ခေါ်တဲ့အချိန် Unsorted bin ထဲကနေ လျှောက်ရှင်းရင်းနဲ့မှ ရောက်သွားတာ ဖြစ်ပါတယ်

	 - Small Bin သို့မဟုတ် Large Bin ထဲကနေ ယူမယ်
Unsorted bin ကို အကုန်ရှင်းလို့မှ မတွေ့သေးရင် အခုနက ရောက်သွားတဲ့ Small bin (သို့) Large bin တွေထဲကနေ ကိုယ်နဲ့ကိုက်ညီမယ့် Size ကို လိုက်ရှာပြီး ဖြတ်ယူပါတယ်

	- Top Chunk (last)
ဘယ် Bin ထဲမှာမှ သုံးလို့ရတဲ့ Free chunk မရှိတော့ရင်တော့ Heap ရဲ့ အပေါ်ဆုံးမှာ ရှိတဲ့ Top Chunk (ပင်မ memory နယ်မြေကြီး) ထဲကနေ ဖြတ်ပြီး အသစ်ထုတ်ပေးပါတယ် Top Chunk မှာပါ ကုန်နေရင်တော့ OS ဆီကနေ `brk` သို့မဟုတ် `mmap` နဲ့ memory အသစ် ထပ်တောင်းပါတယ်


နောက်တစ်ခု မေးခွန်းထုတ်စရာက 
binsတွေကို size အလိုက်ဘယ်လိုခွဲတာလည်းဆိုတာလည်းသိပြီ
သူ့ size နဲ့ ညီတဲ့ bin index တစ်ခု ထဲကို chunk ဘယ်လောက် ထိထည့်နိုင်လဲပေါ့
အဖြေက `tcache` bin ကလွဲပြီးကျန်တာ unlimited ပါ
`tcache` bin index တစ်ခုကို chunk 7 ခုထိဘဲဆန့်ပါတယ်
7ခုကျော်ရင် fast binထဲဆန့်မလားကြည့်
fast binထဲမဆန့်ရင် unsorted bin ထဲ ဂျောင်းလိုက်ပေါ့


| Bin Type         | Order (FIFO/LIFO)             | Direction (Singly/Doubly) | Circular? | Sorted?                | Notes                                 |
| ---------------- | ----------------------------- | ------------------------- | --------- | ---------------------- | ------------------------------------- |
| **Tcache**       | **LIFO** (Last In, First Out) | **Singly** (fd only)      | ❌ No      | ❌ No                   | Per-thread, max 7 chunks, **fastest** |
| **Fastbin**      | **LIFO**                      | **Singly** (fd only)      | ❌ No      | ❌ No                   | No coalescing, fixed sizes            |
| **Unsorted Bin** | **FIFO**                      | **Doubly** (fd & bk)      | ✅ **Yes** | ❌ No                   | Cache layer, temporary storage        |
| **Small Bin**    | **FIFO**                      | **Doubly** (fd & bk)      | ✅ **Yes** | ❌ No (fixed size)      | Fixed size per bin                    |
| **Large Bin**    | **FIFO**                      | **Doubly** (fd & bk)      | ✅ **Yes** | ✅ **Yes** (descending) | Best-fit search, ranges               |

----


##### 1. `Tcache` bin (Thread local cache) Singly Linked LIFO


	Tcache သည် multi-threaded programs များတွင် speed အမြန်ဆုံးရရှိရန် TLS (Thread Local Storage) ထဲတွင် thread တစ်ခုချင်းစီအတွက် သီးသန့်ဆောက်ထားသော structure 
	  malloc က ဒီကောင် ကို ပထမဆုံး check လုပ်တယ် multi-threaded program တွေမှာ heap lock contention ကိုလျှော့ချဖို့ ဒီဇိုင်းလုပ်ထားတာဖြစ်တယ်   glibc 2.26 (2017) (Modern Malloc) ကစပြီးထည့်သွင်းခဲ့တယ်
	  tcache bin တွေက main arena ထဲမှာမရှိဘူး၊ thread ရဲ့ local storage ထဲမှာရှိတယ်
	  Singly Linked LIFO ကိုသုံးတယ် binပေါင်း 64 bin ပါတယ် Bin တစ်ခုစီက chunk size range တစ်ခုကို ကိုယ်စားပြုပြီး 
	  pointer က လတ်တလော free ဖြစ်ထားသော အပေါ်ဆုံး chunk ဆီကို တန်းညွှန်နေပြီး အဲဒီ chunk ၏ user data နေရာပေါ်တွင် nex` (သို့မဟုတ် fd) pointer တစ်ခုသာ ရှိ (bk မသုံး အဲကြောင့်မြန်)
	  maximum chunk  7 ခု ထိဘဲရ အရင် bin တွေဖြစ်တဲ့ fastbin, smallbin, unsortedbin, largebin တွေဟာ main arena ထဲမှာရှိပြီး lock လိုအပ်တယ်
	  Security ပိုင်းမှာတော့ Tcache က Lock မရှိသလို၊ Security Checks အလွန်နည်းလို့ Heap Exploitation (ဥပမာ - Tcache Poisoning, Double Free) လုပ်ရတာ ပိုတောင် လွယ်ကူသွားစေတယ်။ (Glibc versionအသစ်တွေကျမှ Safe Linking စတဲ့ security တွေ ထပ်ထည့်လာတာ ဖြစ်တယ်)


 📌 64-bit Specification
- **Bin Count:** 64 ခု
- **Size Range:** $32\text{ bytes}$ မှ $1016\text{ bytes}$ အထိ (တစ်ဆင့်တိုးလျှင် **$16\text{ bytes}$** စီ တိုးသည်)
- **Index Formula:** $\text{Index} = (\text{Size} - 32) / 16$
- **Capacity:** Index တစ်ခုစီတွင် အများဆုံး **၇ တုံး (7 Chunks)** သာ ဆံ့သည်။
    
 📌 32-bit Specification
- **Bin Count:** 64 ခု
- **Size Range:** **$16\text{ bytes}$** မှ **$520\text{ bytes}$** အထိ (တစ်ဆင့်တိုးလျှင် **$8\text{ bytes}$** စီ တိုးသည်)
- **Index Formula:** $\text{Index} = (\text{Size} - 16) / 8$
- **Capacity:** အများဆုံး **၇ တုံး (7 Chunks)** သာ ဆံ့သည်။
    


```
when call malloc(100)
Step 1: Tcache ကိုစစ်တယ်
        ├─ ရှိရင် → ချက်ချင်းယူတယ် (lock မလို) → ပြီးတယ်
        └─ မရှိရင် → Step 2

Step 2: Arena ကို သွားတယ် (Main Arena or Thread Arena)
        ├─ Lock ယူတယ်
        ├─ Fastbin → Smallbin → Unsortedbin → Largebin စစ်တယ်
        ├─ ရှိရင် → ယူတယ် → Lock ပြန်လွှတ်တယ်
        └─ မရှိရင် → mmap or sbrk




when call free(ptr)
Step 1: size က < 0x410 လား?
        ├─ မဟုတ်ရင် → arena ကိုသွား (lock ယူ) → coalesce → bin ထဲထည့်
        └─ ဟုတ်ရင် → Step 2

Step 2: Tcache bin ပြည့်နေလား? (max 7)
        ├─ မပြည့်သေးရင် → Tcache ထဲထည့်တယ် (lock မလို) → ပြီးတယ်
        └─ ပြည့်နေရင် → arena ကိုသွား (lock ယူ) → arena bin ထဲထည့်



example for 2 thread

// Thread 1
void* p1 = malloc(100);  // Tcache ထဲမရှိ → Arena သွား (lock) → ယူတယ်
free(p1);                 // Tcache ထဲထည့် (lock မလို)

void* p2 = malloc(100);  // Tcache ထဲရှိတယ် → ချက်ချင်းယူတယ် (lock မလို) ⚡မြန်တယ်

// Thread 2 (တစ်ချိန်တည်း)
void* q1 = malloc(100);  // Thread 2 ရဲ့ Tcache ထဲမရှိ → Arena သွား (lock စောင့်)
                         // ဒါပေမယ့် Thread 1 က lock မကိုင်ထားဘူး (tcache သုံးလို့)
                         // → ဒါကြောင့် lock contention နည်းတယ်
```

	  
![](threads.png)



##### 2. Fast bin

![](bins-fast.png)

	အနီးအနားမှ free chunks များနှင့် ချက်ချင်းသွား၍ မပေါင်းစပ်စေရန် (Deferred Coalescing) ကာကွယ်ပေးထားသော အမြန်သုံး bin စနစ်
	 multi thread မဟုတ် or tcache binထဲမရှိရင် fast binကို ကြည့်
	ချက်ချင်းပြန်သုံးလို့ရအောင်လုပ်တာဆိုတော့ fast bin က အမှန်တကယ် free လုပ်ထားတဲ့ကောင်မဟုတ်ဘူး (coalescing မလုပ်ဘဲ ချက်ချင်းပြန်သုံးလို့ရအောင် သိမ်းထားတာ) 
	previous chunk နဲ့ coalesce မလုပ်ဘူး 
	The `PREV_INUSE` Bit Illusion: Fastbin ထဲကို chunk တစ်ခု ရောက်သွားတဲ့အခါ အဲ chunk ရဲ့ အနောက်ကကပ်လျက်ရှိတဲ့ chunk ရဲ့ `PREV_INUSE` (Previous chunk is allocated) bit ကို 1 (True) ဆိုပြီးဘဲဆက်ထားတယ် မပြောင်းလဲ။ အဲကြောင် ဘေးက chunks တွေက အဲ fastbin ထဲကကောင်ကို free ဖြစ်နေမှန်း မသိကြဘဲ `free()` လုပ်တဲ့အခါ auto-coalesce (အလိုအလျောက် ပေါင်းစပ်ပြီး chunk အကြီးကြီးဖြစ်သွားတာ) မဖြစ် 
	fast bin က singly link list (LIFO) နည်းသုံးတယ်
	 ဆိုတော့ chunk တွေထည့်ပြီး ပြန်ထုတ် ရင် အပေါ်ဆုံးကကောင်က စယူတယ် (တစ်ဖက်ပိတ်)
	 ဆိုတော့ fd ဘဲသုံးတယ်
	 fast bin 10ခုရှိတယ် fixed size (same as small bin) တွေဖြစ်တယ်




 📌 64-bit Specification

- **Bin Count:** 10 ခု (`fastbinsY` array)
- **Size Range:** $32\text{ bytes}$ မှ **$128\text{ bytes}$** ($0x80$) အထိ (16-byte alignment ဖြစ်၍ `32, 48, 64, ..., 128` )
- **Capacity:** **အကန့်အသတ်မရှိ (Unlimited)** ဝင်ဆံ့နိုင်

`glibc` မှာ fast bins array က `bin[0]` ကနေ `bin[9]` အထိ (10 bins) နေရာအပြည့် ယူထားပေးနိုင်ပေမဲ့၊ standard အနေနဲ့ binary တစ်ခု run တဲ့အခါ **80 bytes (0x50) ဒါမှမဟုတ် 128 bytes (0x80)** အထိကိုပဲ fast bin အဖြစ် သုံးမယ်ဆိုပြီး global maximum size (`global_max_fast`) တစ်ခု ကန့်သတ်ထားလေ့ရှိပါတယ်
runtime မှာ allocation size **128 bytes** ကျော်သွားရင် ၎င်းတို့ကို fastbin ထဲမထည့်တော့ဘဲ Unsorted bin/Normal bin တွေထဲကိုပဲ ပို့


 📌 32-bit Specification

- **Bin Count:** 10 ခု
- **Size Range:** $16\text{ bytes}$ မှ **$80\text{ bytes}$** ($0x50$) အထိ (8-byte alignment ဖြစ်၍ `16, 24, 32, ..., 80` )
- **Capacity:** **အကန့်အသတ်မရှိ (Unlimited)** ဝင်ဆံ့နိုင်
    

```
10 Fast bins total:
Bin 0: 16-byte chunks
Bin 1: 24-byte chunks
Bin 2: 32-byte chunks
Bin 3: 40-byte chunks  
Bin 4: 48-byte chunks
Bin 5: 56-byte chunks
Bin 6: 64-byte chunks
Bin 7: 72-byte chunks
Bin 8: 80-byte chunks
Bin 9: 88-byte chunks

88 bytes အထက်ဆိုရင် fast bin မဟုတ်တော့ အခြားbinရှာ
```


##### 3. Small Bin 
![](bins-small.png)

	အရွယ်အစား တိကျသေချာတဲ့ Chunk တွေကို စနစ်တကျ ခွဲခြားသိမ်းဆည်းတဲ့ နေရာဖြစ်ပါတယ်။
	number of bins = 62 and fixed sized ဖြစ်တယ် doubly linked list (Fist in Fisrt Out) အရင်ဝင်တဲ့ကောင် အရင်ထွက်တယ် ဆိုတော့ fd bk နှစ်ခုလုံးသုံးတယ် Coalescing လုပ်တယ်


 📌 64-bit Specification
- Bin Count: 62 ခု (Index 2 မှ 63 အထိ)
- Size Range: $32\text{ bytes}$ မှ **$1016\text{ bytes}$** အထိ (16-byte စီ တိုးသွား)
- Capacity: Unlimited)


 📌 32-bit Specification
- Bin Count: 62 ခု (Index 2 မှ 63 အထိ)
- Size Range: $16\text{ bytes}$ မှ **$504\text{ bytes}$** အထိ (8-byte စီ တိုးသွား)
- Capacity: Unlimited
    

---


##### 4. Unsorted Bin /Circular Doubly Linked List (FIFO) 


![](bins-unsorted.png)

	free() လုပ်သမျှ တန်ဖိုးတွေ အကုန်လာစုဝေးရာ ယာယီအမှိုက်ပုံကြီး ဖြစ်ပါတယ် Architecture နှစ်ခုလုံးမှာ သူက 1 ခုတည်းပဲ ရှိပါတယ်
	ptmalloc2 ရဲ့ optimizing cache layer 
	1 bin ထဲဘဲရှိတယ် free chunk တွေကို neighbour chunkတွေနဲ့ coalesces လုပ်တယ်(free() ခေါ်တဲ့အခါ coalescing လုပ်ပြီးမှ အဲဒီ chunk ကို unsorted bin ထဲထည့်တာ) temporary အနေနဲ့ဘဲသုံးတယ် လိုအပ်တဲ့ size နဲ့ ကိုက်ညီရင် split လုပ်ပြီး ယူတယ် မကိုက်ရင် ဒီ chunk ကို small/large bin ထဲ ပြန်ထည့်တယ်။ circular ပုံစံဖြစ်တယ် doubly linked list ဖြစ်တယ်
	 no sorting ဆိုတော့ ရှာရမြန်တယ်  Small bins လို Fixed size order Large bins လို Sorted by size မဟုတ်
	  use both fd and bk


 📌 64-bit & 32-bit Specification

- **Bin Count:** 1 (ပင်မ Bin array ရဲ့ **Index 1** နေရာ)
- **Size Range:** အကန့်အသတ်မရှိ (Omnivorous) 32B ကောင်ရော၊ 5000B ကောင်ရော အရွယ်အစားစုံ ရောပြွမ်း ဝင်ဆံ့နိုင်
- **Linkage Geometry:** Circular Doubly Linked List (FIFO) စနစ် ဖြစ်၍ `fd` ရော `bk` ပါ သုံး
- **Capacity:** အကန့်အသတ်မရှိ (Unlimited) ဝင်ဆံ့နိုင်
    

---


##### 5. large bin

	အရွယ်အစား အလွန်ကြီးမားတဲ့ Allocation တွေအတွက် Range အလိုက် ခွဲခြားသိမ်းဆည်းပြီး ကြီးစဉ်ငယ်လိုက် စီထားတဲ့ စနစ်
	512 bytes (1024 bytes on 64-bit) small binရဲ့အမြင့်ဆုံး ထက်ကျော်ရင် large bin ထဲသိမ်း
	Large bin တွေက fixed range မဟုတ်ဘူး
	bin 63 ခုရှိ ဒီမှာ binတစ်ခုစီမှာက fixed size မဟုတ်တော့ဘူး range နဲ့သွားတယ်
	 bin တစ်ခုချင်းစီမှာ chunk တွေကို ကြီးစဉ်ငယ်လိုက် (Sorted by size (descending))
	 ဆိုလိုတာက Bin ရဲ့ ခေါင်းဆောင် (`head`) ရဲ့ အနီးဆုံးမှာ အကြီးဆုံး chunk (Largest chunk) ရှိနေပြီး၊ အနောက်ကို ရောက်လေလေ သေးသွားလေလေ
	 


 📌 64-bit Specification

- Bin Count: 63 ခု (Index 64 မှ 126 အထိ)
- Size Range: **$1024\text{ bytes}$ ($1\text{ KB}$)** မှစ၍ အထက် infinity အထိ
- Range Steps (Granularity) စစချင်း Bin ၃၂ ခုက $64\text{ bytes}$ စီ ကျယ်ပြီး၊ နောက်ပိုင်းတွင် $512\text{B}, 4\text{KB}, 32\text{KB}$ စသဖြင့် အဆင့်ဆင့် အဆမတန် ကျယ်ပြန့်သွား
- *Capacity: Unlimited
    

 📌 32-bit Specification

- Bin Count: 63 ခု (Index 64 မှ 126 အထိ)
- Size Range: **$512\text{ bytes}$** မှစ၍ အထက် infinity အထိ
- Range Steps (Granularity): စစချင်း Bin ၃၂ ခုက **$32\text{ bytes}$** စီသာ ကျယ် (ဥပမာ- Bin 64 က `512-543 bytes` အထိ)၊ ထို့နောက်မှ $256\text{B}, 2\text{KB}, 16\text{KB}$ စသဖြင့် အဆင့်ဆင့် ကျယ်သွား
- Capacity: Unlimited
    

| Bin Index Range | Number of Bins | Granularity (Width per Bin) | Chunk Size Range                  | Notes                                     |
| --------------- | -------------- | --------------------------- | --------------------------------- | ----------------------------------------- |
| **64 ~ 95**     | **32 bins**    | **64 bytes**                | **1,024 bytes ~ 3,072 bytes**     | 1024 = 0x400, 3072 = 0xC00                |
| **96 ~ 111**    | **16 bins**    | **512 bytes**               | **3,072 bytes ~ 11,264 bytes**    | 11264 = 0x2C00                            |
| **112 ~ 119**   | **8 bins**     | **4,096 bytes (4KB)**       | **11,264 bytes ~ 44,032 bytes**   | 44032 = 0xAC00                            |
| **120 ~ 123**   | **4 bins**     | **32,768 bytes (32KB)**     | **44,032 bytes ~ 175,104 bytes**  | 175104 = 0x2AC00                          |
| **124 ~ 125**   | **2 bins**     | **262,144 bytes (256KB)**   | **175,104 bytes ~ 699,392 bytes** | 699392 = 0xAAC00                          |
| **126**         | **1 bin**      | Remaining                   | **699,392 bytes and above**       | This bin holds all extremely large chunks |

Granularity (Width per Bin) = bin တစ်ခုနဲ့တစ်ခုဘယ်လောက်ကွာလဲပြတာ (step)

| Bin Index Range | Number of Bins | Granularity (Width per Bin) | Chunk Size Range                 | Notes                                             |
| --------------- | -------------- | --------------------------- | -------------------------------- | ------------------------------------------------- |
| **64 ~ 95**     | **32 bins**    | **32 bytes**                | **512 bytes ~ 1,536 bytes**      | 32-bit မှာ အနိမ့်ဆုံး Large Bin က 512 bytes ကစသည် |
| **96 ~ 111**    | **16 bins**    | **256 bytes**               | **1,536 bytes ~ 5,632 bytes**    |                                                   |
| **112 ~ 119**   | **8 bins**     | **2,048 bytes (2KB)**       | **5,632 bytes ~ 22,016 bytes**   |                                                   |
| **120 ~ 123**   | **4 bins**     | **16,384 bytes (16KB)**     | **22,016 bytes ~ 87,552 bytes**  |                                                   |
| **124 ~ 125**   | **2 bins**     | **131,072 bytes (128KB)**   | **87,552 bytes ~ 349,696 bytes** |                                                   |
| **126**         | **1 bin**      | Remaining                   | **349,696 bytes and above**      |                                                   |



![](bins-large.png)


##### Note

- **Coalescing လုပ်တဲ့အချိန်** = free() ခေါ်တဲ့အခါ (တန်းချင်းယှဉ်ရင် ပေါင်းတယ်)
- **Small/Unsorted/Large bin** ကိုယ်တိုင် coalescing မလုပ်ဘူး
- free() က coalescing လုပ်ပြီးမှ **သက်ဆိုင်ရာ bin** ထဲထည့်တာ
- Fast bin ကလွဲလို့ **ကျန်တဲ့ bin အားလုံးက coalesced chunks တွေကိုပဲ သိမ်းတယ်**


---

# Common Vulnerabilities

```
Pwngdb Commands to see heap flow:

heap
bins
vis
arena
arenas Glibc က Arena ဘယ်နှခု ဆောက်ထားလဲဆိုတာစစ်
f4
ptype user
dq &user
p user


# Pwndbg heap commands
heap               # Heap overview
arena              # Arena information
chunk $rax         # Inspect chunk at address
bins               # Show all bins (fast, unsorted, small, large)
fastbins           # Fastbins only
unsortedbin        # Unsorted bin
smallbin           # Small bins
largebin           # Large bins
tcachebins         # Tcache bins (glibc 2.26+)
tracemalloc        # malloc hook tracing

# Example heap debugging
gdb-pwndbg$ heap
gdb-pwndbg$ bins
gdb-pwndbg$ chunk 0x555555559000
```


### Double free

free နှစ်ခါသုံးမိတာပါ memory တစ်ခုကို allocate ယူပြီး နှစ်ခါ free လုပ်မယ် ပြီး ရင် allocateနှစ်ခါ ယူရင် အစောက memory ‌addressကိုနှစ်ခါပေးမိသွားတယ် ဆိုတော့ memory addressတူ pointer နှစ်ခုရတယ် pointer တစ်ခုကို ဖျက်ပီး နောက် pointer ကို data ထည့်လို့ရတယ် free chunk ကို edit လုပ်လို့ရတဲ့သဘောဖြစ်သွားမယ်။ 
You can see example : https://guyinatuxedo.github.io/27-edit_free_chunk/double_free_explanation/index.html

### Heap Consolidation

Chunk 3 ခု allocate ယူမယ်  first chunk ကို free and pointer ကို ရှင်း ။ တတိယchunkကို prev_in use ဖြုတ် prev in size ( chunk 1+ chunk2) ထည့်   ပြီးရင် free လိုက်။ chunk2 ကနေရင်းထိုင်ရင်း free သလို ဖြစ်သွား( not actually free) ။ နောက် ထပ် chunk နှစ်ခုထပ်ယူ (chunk 4 and chunk5) chunk 2 pointer နဲ့ chunk 5 pointer တူနေ။ chunk 2 ကို free လုပ် pointer ရှင်း but chunk5 pointer ကနေ data edit လုပ်လို့ရ။
You can see example : https://guyinatuxedo.github.io/27-edit_free_chunk/heap_consolidation_explanation/index.html


### Use After Free

chunk ကို free လုပ်ခဲ့တယ် but  ptr ကို မရှင်းခဲ့ဘူး ဆိုတော့ ptr ကို သုံးပြီး free chunk မှာ data ထည့်လို့ရတယ်
you can see example : https://guyinatuxedo.github.io/27-edit_free_chunk/uaf_explanation/index.html


### Unlink() Exploit

unlink() က ဘာလုပ်တာလဲဆို memory allocate လုပ်တဲ့အခါ bin ထဲက free chunk ကို ထုတ်ပေးတာ။ ဥပမာအနေနဲ့ ပြောရရင် free chunk သုံးခုရှိတယ် အလယ်ကကောင်ကို ထုတ်မယ်ဆိုရင် ထုတ်မယ့်ကောင်ရဲ့ fd နဲ့ bkကို ကျန်တဲ့နှစ်ကောင်ကိုချိတ်ဆက်ပေးမယ်။ ဆိုတော့ အရင်ဆုံး အဲကျန်တဲ့နှစ်ကောင် ရဲ့  (previous chunk ရဲ့  fd နဲ့  next chunk ရဲ့ bk ) pointerတွေက ငါတို့ထုတ်မယ့်ကောင်ကို point လုပ်နေလားဆိုတာစစ်တယ်
ဒီနေရာမှာ သတိထားရမှာက fd နဲ့ bk က other chunk ကို ရည်ညွှန်းတဲ့ အခါ metadata စတဲ့ memory address ကိုဘဲ pointလုပ်တယ် user data မဟုတ်ဘူး ဒီအချက်ကိုမသိတော့ နည်းနည်း လည်သွားတယ် :)) 
ဒီ example မှာက chunk1  memory address (user data address) ကို target global variable  ထဲထည့်မယ်
ပြီးရင်  chunk 1 ကို free chunk ဖြစ်အောင် ပြင်မယ်  user data အစကို chunk အစ ဖြစ်အောင်လုပ်မယ်။      
ဆိုတော့ target address က  chunk အစကို point လုပ်သလိုဖြစ်သွားမယ် fd နဲ့ bk point လုပ်သလိုပေါ့။ chunk 1ဘက်ပြန်လှည့်ကြည့်ရအောင် chunk1 ကို  user data အစဖြစ်အောင် ပြင်တယ် အဲမှာ prev size ထားမယ် 0တွေပေါ့ နောက်ထပ် 8 byte ဆင်းပြီး chunk size ကိုယ့်ဘာသာထည့်မယ် 0xa ပေါ့ ငါတို့က 0xa ကို malloc ကနေတောင်းခဲ့တယ် ဆိုတော့ ဒီအတိုင်းကောက်ထည့်လို့အဆင်ပြေတယ် fd နဲ့ bk ကိုထည့်မယ် bkက အစောက target ‌address ရဲ့ အပေါ် 0x10 ( 16 bytes) အကွာကို ရည်ညွှန်းမယ် အဲကနေ fd တွက်ရင် target address ရောက်အောင်လို့ ဒါမှ target chunk ရဲ့ fd က chunk ကို point လုပ်နေမယ်  chunk 1 ရဲ့ fd ကိုလည်း ဒီtrick သုံးပြီးလုပ်မယ် 

chunk1 ကို bin ထဲက free chunk မှန်းသိအောင် chunk 2 ရဲ့  prev  size ကို chunk 1 size ထည့်မယ်။
chunk2 ကို free လုပ်လိုက်ရင် free chunk1 နဲ့ chunk2 ပေါင်းဖို့ unlink() function ကို trigger လုပ်မယ်
ပုံမှန် binထဲကထုတ်မယ်ဆိုရင်    (binထဲကထုတ်မယ့် chunk က chunkA ဆိုပါစို့) chunk Aကို bin ထဲကထုတ်မယ်ဆိုရင် chunkA ရဲ့ fd ညွှန်းနေတဲ့ FD chunk ကိုအရင်သွားကြည့်ပြီး  FD chunk ရဲ့ bk (chunk Aကိုညွှန်းနေတာ) ကို  chunkA ရဲ့ bkနဲ့ overwrite, ပြီးရင်  chunk Aကို bin ထဲကထုတ်မယ်ဆိုရင် chunkA ရဲ့ bk ညွှန်းနေတဲ့ BK chunk ကိုအရင်သွားကြည့်ပြီး  BK chunk ရဲ့ fd (chunk Aကိုညွှန်းနေတာ) ကို  chunkA ရဲ့ fdနဲ့ overwrite ဆို့တော့ BK process က နောက်မှလာတယ်
ဆိုတော့ အဲ global variable မှာနောက်ဆုံးကျန်ရှိမယ့် value က BK chunk fd ကို ကို overwrite မယ့် global variable အထက် 3ဆ(0x18)အကွာမှာ ရှိနေမယ့် address ဘဲဖြစ်ပါတယ် () global variable - 0x18)


![](unlink-1.jpg)

example: https://guyinatuxedo.github.io/30-unlink/unlink_explanation/index.html




### Heap Grooming

3 chunk allocate ယူမယ် free မယ် binထဲ ရောက်သွားမယ် 3 chunk ထည့်ယူမယ် နောက်ဆုံး free တဲ့ memory address က ရှေ့ဆုံးမှာ ရနေမယ် ဆိုတော့ fast binဖြစ်ဖို့များတယ်'
example: https://guyinatuxedo.github.io/26-heap_grooming/explanation_heap_grooming/index.html


---

## NOTES


_note :program တစ်ခုမှာ chunk တစ်ခုရဲ့ metadata ကို ပြင်ချင်ရင်  previous chunk ကို overflow ပြီးမှ  ပြင်လို့ရ code နဲ့ဆိုရင်တော့ chunk_pointer[-1] ( chunk size)  chunk_pointer[-2] (previous_size) ဆိုပြီး accessလုပ်ပြီးပြင်လို့ရ


# Challenges


### Heap2

```
──(Jackfruit㉿kali)-[~/…/learn/binary/stack/liveOverFLow]
└─$ file heap2_32 
heap2_32: ELF 32-bit LSB executable, Intel i386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=ae1eca6c4c9c285f6d3443901009deb1d345a62c, for GNU/Linux 3.2.0, not stripped
                                                                                                                                                             
┌──(Jackfruit㉿kali)-[~/…/learn/binary/stack/liveOverFLow]
└─$ pwn checksec heap2_32 
[*] '/home/Jackfruit/cate/learn/binary/stack/liveOverFLow/heap2_32'
    Arch:       i386-32-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX unknown - GNU_STACK missing
    PIE:        No PIE (0x8048000)
    Stack:      Executable
    RWX:        Has RWX segments
    Stripped:   No
                                                                                
```

```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

struct auth {
  char name[32];
  int auth;
};

struct auth *auth;
char *service;

int main(int argc, char **argv)
{
  char line[128];

  while(1) {
    printf("[ auth = %p, service = %p ]\n", auth, service);

    if(fgets(line, sizeof(line), stdin) == NULL) break;
    
    if(strncmp(line, "auth ", 5) == 0) {
      auth = malloc(sizeof(auth));
      memset(auth, 0, sizeof(auth));
      if(strlen(line + 5) < 31) {
        strcpy(auth->name, line + 5);
      }
    }
    if(strncmp(line, "reset", 5) == 0) {
      free(auth);#####1
    }
    if(strncmp(line, "service", 6) == 0) {
      service = strdup(line + 7);#####3
    }
    if(strncmp(line, "login", 5) == 0) {
      if(auth->auth) #####2{
        printf("you have logged in already!\n");
      } else {
        printf("please enter your password\n");
      }
    }
  }
}
```


	#####1 နေရာမှာ free သုံးတယ် ပီးတော့ pointer ကိုမရှင်းခဲ့ဘဲနဲ့ #####2 auth->auth အရင် pointer ဟောင်းကိုဘဲပြန်သုံးပြီး checkလုပ်တယ် -> syntax နဲ့ pointer ကနေ တစ်ဆင့် struct member ကို access လုပ်လို့ရတယ် (*ptr).member နဲ့အတူတူဘဲ 
	#####3 strdup ကို memory အသစ်ယူပြီး copy ကူးထည့်ပေးတယ် ဆိုတော့ free လုပ်ပီး service ယူရင် auth heap address ကိုဘဲပြန်သုံးမယ် service ကို struct ထဲက auth ရောက်မယ့် offset ထိရောက်အောက်ယူပြီး loginဝင် auth->auth check passsed!


---
## Unlink Exploit:
## stkof

```
└─$ pwn checksec stkof
[*] '/home/Jackfruit/cate/learn/binary/heap/nightmare/modules/30-unlink/hitcon14_stkof/stkof'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x3fe000)
                                     
```

main ကို တစ်ချက်ကြည့်လိုက်ရင် menu 4 ခုတွေ့မယ်
1 - allocate chunk
2 - scan data
3 -  free function
4 -  print data 

ဆိုတော့ သူကတော့ menu prompt ပေးမထားဘူး
main function ဖတ်ရင်တော့တွေ့ရလိမ့်မယ်

	Allocate function
```c
  fgets(local_78,0x10,stdin);
  __size = atoll(local_78);
  ptr = malloc(__size);
  if (ptr == (void *)0x0) {
     uVar1 = 0xffffffff;
  }
  else {
     DAT_00602100 = DAT_00602100 + 1;
     *(void **)(&DAT_00602140 + (long)(int)DAT_00602100 * 8) = ptr;
     printf("%d\n",(ulong)DAT_00602100);
     uVar1 = 0;
  }
```

	ဒီနေရာမှာ user ပေးလာတဲ့ input ကို ဘဲ တစ်ခါတည်း size လုပ်ပြီးတန်းယူတာ check မလုပ်ဘူး 
	DAT_00602100 ဒီကောင်က index ပေးတာ allocate ယူတိုင်း index တစ်ခုတိုးတိုးပေး
	(void **)(&DAT_00602140 + (long)(int)DAT_00602100 * 8) = ptr; ဒီ code က
	ဘာလုပ်လဲဆို heap ထဲက user data address pointer ကို global variable ထဲမှာ သိမ်း
	ပြီးရင် index ထုတ်ပြ printf("%d\n",(ulong)DAT_00602100);


	Scan function
```c
  fgets(user_input,0x10,stdin);
  index = atol(user_input);
  if ((uint)index < 0x100001) {
     if (*(long *)(&DAT_00602140 + (index & 0xffffffff) * 8) == 0) {
        return_address = 0xffffffff;
     }
```

	စစချင်း  user index တောင်းမယ်
	ငါတို့အစောက စခဲ့တဲ့ DAT_00602100 ကို index တွက်ပြီး မင်းတောင်းတဲ့ index မှာ heap address ရှိမရှိစစ်မယ်

```c
     else {
        fgets(user_input,0x10,stdin);
        size = atoll(user_input);
        data_address_on_heap = *(void **)(&DAT_00602140 + (index & 0xffffffff) * 8);
        while( true ) {
           user_data_from_heap = fread(data_address_on_heap,1,size,stdin);
           user_data_from_heap(int) = (int)user_data_from_heap;
           if (user_data_from_heap(int) < 1) break;
           data_address_on_heap = (void *)((long)data_address_on_heap + (long)user_data_from_heap(int))
           ;
           size = size - (long)user_data_from_heap(int);
        }
        if (size == 0) {
           return_address = 0;
        }
```
	heap address ရှိရင် size တောင်းမယ်
	ပြီးရင် အစောက index သုံးပြီး heap addressယူမယ်

```c
while( true ) {
    // 1. stdin ကနေ data ဖတ်တယ်
    user_data_from_heap = fread(data_address_on_heap, 1, size, stdin);
    
    // 2. ဘယ်လောက် bytes ဖတ်လဲဆိုတာ integer အဖြစ်ပြောင်း
    user_data_from_heap(int) = (int)user_data_from_heap;
    
    // 3. ဖတ်လို့မရတော့ရင် break
    if (user_data_from_heap(int) < 1) break;
    
    // 4. pointer ကို ရွှေ့ (ဖတ်ပြီးသား data ရဲ့ နောက်တစ်နေရာကို)
    data_address_on_heap = (void *)((long)data_address_on_heap + (long)user_data_from_heap(int));
    
    // 5. ကျန်သေးတဲ့ size ကို update
    size = size - (long)user_data_from_heap(int);
}
```



---

# Fastbin attack :
## 0ctf baby heap

အချို့ heap exploit တွေက နောက်ပိုင်း version တွေမှာအလုပ်မလုပ်တော့ဘူး
ဆိုတော့  default glibc version မသုံးအောင် program ရဲ့  elf setting ကို ပြင်မယ်


```
patchelf --set-interpreter ~/glibc/glibc_2.23/ld-2.23.so 0ctfbabyheap
patchelf --set-rpath ~/glibc/glibc_2.23/ 0ctfbabyheap
```

ဒီ program မှာ မစခင် သိထား ဖို့က

fast bin တွေက 
	- **32-bit systems** မှာ16 bytes ကနေ 80 bytes အထိ (default malloc chunk alignment နဲ့)
	- 64-bit systems မှာ 32 bytes ကနေ 160 bytes အထိ ဆို fast bin ထဲဝင်တယ်
	- chunk တစ်ခု fast bin တွေ ဝင်သွားရင်  next chunk က လက်ရှိ free chunk in fast bin ကို` prev in use `လို့ သဘောထားထားတယ် (ဘာလို့ဆို fast binတွေကို ချက်ချင်းပြန်သုံးနိုင်အောင်လုပ်ထား)
fast bin attack တွေက
	same chunk ကို မှ pointer နှစ်ခု ထောက်နိုင်အောင်လုပ်ပီး တစ်ခု က allocate နောက်တစ်ခုက free ထား ဆိုတော့ bin ထဲကကောင်ကို data ထည့်နိုင်အောင်လုပ်
   
unsorted bin တွေမှာ
	 free chunk တစ်ခုထဲဆို အဲ free chunk က `main_arena +88` ကို `fd` ရော `bk` ရော pointလုပ်တယ်

malloc ခေါ်တဲ့အခါ
	malloc က `__malloc_hook` null ဖြစ်မဖြစ် check တယ် null မဟုတ်ဘဲ hook set ထားရင် အဲ addressကို ခေါ်


ဆိုတော့ general exploit က  unsorted bin ထဲက address ကို leak , `libc` base address တွက်, shell execution address ကို တွက် `__malloc_hook` မှာထား malloc နောက်ထပ်ခေါ်ရင် shell ရ

ဒီမှာ ပုံသေ သတ်မှတ် ထားရမယ့် allocation technique တစ်ခုရှိတယ်

```

allocation part 1

[1] 0xf0 (240)
[2] 0x70 (112)

နဲ့

allocation part 2
[1] 0x10 (16)
[2] 0x60 (96)
[3] 0x60 (96)
[4] 0x60 (96)

မှာက first allocation part 1 က chunk2 pointer နဲ့ ‌allocation chunk4 pointer တို့က အတူတူ ဘဲ ဒီ technique ကို သုံးပြီး fast bin အတွက် pointer နှစ်ခုဖြစ်အောင်လုပ်လို့ရတယ်

```

ဒီနေရာမှာ program က index နဲ့မမှားစေနဲ့ သဘောတရားနားလည်အောင် chunk 1,2,3 ... ရေးထားတာ
ဆိုတော့ 
 
```
allocation part 1
[1] 0xf0 (240)
[2] 0x70 (112)
[4] 0xf0 (240)
[5] 0x30 (48)
```
	ဆိုပီး chunk 4ခုယူမယ်
	1 နဲ့ 2 ကို free လုပ်လိုက်မယ် 
	1 က unsorted binထဲရောက်သွားမယ် 2 က fast bin ထဲရောက်
	နောက်ထပ် chunk တစ်ခုထပ်ယူမယ် 0x78 (120) ယူမယ် free လုပ်ထားတဲ့ 2 ကို ပေးလိမ့်မယ် ဘာလို့ 8 ယူလဲဆို chunk တွေရဲ့နောက်ဆုံး၎င်းမှာ next chunk အတွက် prev size( prevous chunk free ပါကထည့်ပေးဖို့) ထည်ဖို့ 8 bytes ပေးတာရှိတယ်next chunk က prev size မထည့်ရင်တော့ current chunk အပိုင်ဘဲအဲနေရာက user data ထည့်လို့ရ ။ အဲကြောင့် 8 byte ပိုယူလို့ရ -> fast bin ထဲကကောင်ကိုရ။
	ဒီ program မှာ fill လုပ်မယ်ဆို fill မယ် size တောင်းတယ် ငါတို့ fill ချင်သလောက်ရ ဆိုတော့ 8 byte ပိုယူမယ် (128) ဘာလို့ဆို next chunk ရဲ့ meta data ကိုပြင်ချင်လို့ ဘယ်လိုပြင်ချင်လဲဆို လက်ရှိ  chunk no  2 ကို free chunk လို့မြင်အောင် chunk no 1 နဲ့ ပေါင်းထားတယ်လို့မြင်အောင်ပြင်မယ် prev size ထည့် prev in use last bit 1ကို 0 ထား 
	လုပ်ပီးရင် chunk no 3ကို free လုပ်လိုက်ရင် အပေါ်ကကောင်တွေနဲ့ ပေါင်း ပြီး unsorted bin ထဲရောက်
	chunk 2 pointer က access လုပ်လို့ရ program မှာ dump function ပါတော့ data ကိုကြည့်လို့ရ , ဒီကောင်က unsorted binထဲရောက်နေတာဆိုတော့ data အနေနဲ့ fd နဲ့ bk ကို ကြည့်လို့ရ , main_arena +88 address leak လို့ရ chunk 2 pointer က အလယ်ရောက်နေတာဆိုတော့ အဆင်မပြေသေး, ဆိုတော့ chunk 1 ကိုသုံးအောင် malloc(240) ထပ်ယူ unsorted bin က split ပေး ။ ဆိုတော့ chunk 2 pointer အထပ်ရောက် address leak လို့ရ

Calculation
	ရလာတာက `[main_arena + 88(0x58)] address`
	 `[main_arena + 88(0x58)] address` -0x58 ဆို main_arena address ရ

```
Libc ELF File Structure:
┌─────────────────────────┐
│ .text section           │ ← Code (functions တွေရဲ့ machine code)
│   ├── malloc() code     │
│   ├── free() code       │
│   └── system() code     │
├─────────────────────────┤
│ .data section           │ ← Initialized global variables
├─────────────────────────┤
│ .bss section            │ ← UNinitialized global variables
│   ├── __malloc_hook     │ ← အရေးကြီးပါ!
│   ├── __free_hook       │
│   └── main_arena        │
├─────────────────────────┤
│ .rodata section         │ ← Read-only data (strings)
│   └── "/bin/sh" string  │
└─────────────────────────┘
```

	`__malloc_hook` ကို main_arena ထက် 0x10 bytes ပို နီး 
	ဆိုတော့  `[main_arena + 88(0x58)] address` -0x68 ဆို __malloc_hook address ရ
		ဆိုတော့  `[main_arena + 88(0x58)] address` -0x68 - (__malloc_hook offset from libc base)address ဆို libc base addressရ


```
┌──(Jackfruit㉿kali)-[~/glibc/glibc_2.23]
└─$ one_gadget libc-2.23.so 
0x3f6be execve("/bin/sh", rsp+0x30, environ)
constraints:
  address rsp+0x40 is writable
  rax == NULL || {rax, "-c", r12, NULL} is a valid argv

0x3f712 execve("/bin/sh", rsp+0x30, environ)
constraints:
  [rsp+0x30] == NULL || {[rsp+0x30], [rsp+0x38], [rsp+0x40], [rsp+0x48], ...} is a valid argv

0xd6701 execve("/bin/sh", rsp+0x50, environ)
constraints:
  [rsp+0x50] == NULL || {[rsp+0x50], [rsp+0x58], [rsp+0x60], [rsp+0x68], ...} is a valid argv

```
	one_gadget ကိုသုံးပြီး shell execute မယ့် address ရဲ့ offset ရ, libc base addressနဲ့ပေါင်းပြီး  shell execute မယ့် address ရ

#### allocation part 2

	ဆိုတော့ အပေါ်က ပြထားတဲ့အတိုင်း allocation part2အတိုင်း allocate ယူမယ်
	[4] 0x60 (96) ဒီကကောင်နဲ့ အပေါ်က poninter နဲ့က တူတူဘဲ 


![](0ctf-1.png)


![](0ctf-2.png)

	0x555555e01100 ဘဲ သွားကျနေတယ် part 1က  pointer က free မလုပ်လိုက်ဘူးဆိုတော့သုံးလို့ရယ်
	ဒီနေရာမှာ သူ့ index ကို သုံးလို့ရသေးတယ်ပေါ့ nightmare မှာ index 0 , part 2 ကလည်း index ထပ်ပေး (index 5)
	
![](0ctf-3.png)


	fast bin ထဲမှာလည်း ဘာမှ မရှိသေး ဆိုတော့ fast bin attack လုပ်ဖို့ fast binထဲထည့်ရမယ် index0 , index5 ptr နှစ်ခု လုံးကို ထည့်ရမယ် ပြီးရင် index တစ်ခုကို allocate ယူ ဆိုတော့ ptr တစ်ခုက allocate , ptr တစ်ခုက free , ‌allocat ထားတဲ့ ptr ကို သုံးပြီး data ထည့် , free လုပ်ထားတဲ့ ptr ကို data ထည့်သလိုဖြစ်သွား ,cuz both pointers have the same memory address, 
	ဆိုတော့ fast bin ထဲကို ptrတွေထည့်မယ် ဒါမဲ့ bin ထဲမှာ နှစ်ခုလုံး တစ်ဆက်တည်းဖြစ်လို့မရဘူး ကြားခံထားပြီးထည့်မယ် index 4ကိုသုံးမယ်

	ဆိုတော့ 
```
free(5)
free(4)
free(0)
```
![](0ctf-4.png)


![](0ctf-5.png)


	ပြီးရင် allocate 0x60(96) နှစ်ခါယူမယ်  fast  binထဲကကောင်တွေနဲ့ size ကိုက်တော့ binထဲကကောင်တွေနဲ့ဘဲသုံးမယ် binထဲကနှစ်ကောင်ယူမယ် fast bin က fast in last out ဆိုတော့ နောက်ဆုံးထည့်ကောင်ကစယူမယ် 0 ယူ ပြီးရင် 4 ယူ ဆိုတော့ ဒီနေရာမှ fast bin exploit စလို့ရပီ same ptr , ptr တစ်ကောင်က binထဲမှာ တစ်ကောင်က allocate, allocate ဖြစ်ထားတဲ့ ptrကောင်ကို သုံးပြီး data ထည့်မယ် (binထဲကကောင်ဆိုerror တက်မယ် binထဲကကောင်ပါဆို မှ data ထည့်လို့မှမရတာ) 
	ဒီနေရာမှာ ဘာထည့်မှာလဲဆို fd ဖြစ်မယ့်ကောင်ကိုထည့်မယ် fast binတွေက fdတွေဘဲသုံး 
	fd တွေက binထဲကကောင်တွေကို ရည်ညွှန်းနေတာဆိုတော့ fd ကို memory address တစ်ခုခု ထားလိုက်ရင် အဲ address က binထဲကကောင်ဖြစ်သွားမှာ  နောက်ထပ် allocate ထည့်ယူရင် အဲ address ကိုယူမယ်ဆိုတော့ အဲ address ကို data ထည့်လို့ရသွားမယ် ငါတို့က __maloc_hook ကို overwrite ချင်တာဆိုတော့ __malloc_hook မတိုင်ခင်က address ကို fdနေရာထားလိုက်ရင် __malloc_hook ကိုoverwite လို့ရသွားမယ်မလား
		ဆိုတော့ __malloc_hook - x023 (35) address ကို fd နေရာထားမယ် allocate ယူ,binထဲမှာ __malloc_hook - x023 (35) address ကျန်ခဲ့ အဲaddress က chunk အစ, ဘာလို့ဆိုfd တွေက chunk အစကို ဘဲ point လုပ်လို့, allocate ထပ်ယူရင် အဲ address ရ, data ထည့်လို့ရ chnksize 35 -metadata (16) = 19 (data ထည့်စတဲ့နေရာကနေ __malloc_hook addressရဲ့အကွာအဝေး)
		ဆိုတော့ 19 + __malloc_hook ကို overwirte မယ် 8 bytes address(shell execute address) = 27 ထည့်မယ် fill funtion ကိုchoose လိုက်ပီးထည့်လိုက်မယ်ဆို overwite ပြီးသားဖြစ်
		နောက်ထပ် chunk allocate တစ်ခုထပ်ယူ , mallocက __malloc_hookကို check လိုက်တဲ့အချိန်မှာ BOoooom! shell ရ ။




---


## Auir

```
# Binary ရဲ့ interpreter (linker) ကို ပြောင်းမယ်
patchelf --set-interpreter ./ld-2.31.so ./challenge

# Binary ရဲ့ libc ကို ပြောင်းမယ်
patchelf --replace-needed libc.so.6 ./libc-2.31.so ./challenge

# Run မယ်
./challenge
```

**Error က ပြောနေတာက: "မင်း [libc-2.23.so](https://libc-2.23.so) ထဲမှာ ကျွန်တော်လိုအပ်တဲ့ features တွေ မပါဘူး"**

## **Error ကို ရှင်းပြချက်:**

text

./auir: ./libc-2.23.so: version `GLIBC_2.38' not found (required by /lib/x86_64-linux-gnu/libstdc++.so.6)

- `libstdc++.so.6` (C++ standard library) က `GLIBC_2.38` version symbol ကို လိုအပ်တယ်
    
- ဒါပေမယ့် မင်း `libc-2.23.so` ထဲမှာ ဒီ symbol မရှိဘူး
    
- ဘာလို့လဲဆိုတော့ glibc 2.23 က 2016 ကဖြစ်ပြီး၊ GLIBC_2.38 က 2023 မှထွက်တာ
    

## **ဘာကြောင့်ဖြစ်ရတာလဲ:**

မင်း system မှာ **modern libraries** တွေ install လုပ်ထားတယ်:

1. `libstdc++.so.6` - C++ library (new, needs glibc 2.38)
    
2. `libm.so.6` - Math library (new, needs glibc 2.36)
    
3. `libgcc_s.so.1` - GCC runtime (new, needs glibc 2.35)
    

ဒီ libraries တွေက compile လုပ်တုန်းက **new glibc features** တွေသုံးထားတယ်။ မင်း `LD_PRELOAD` နဲ့ old libc (2.23) ကို load လုပ်လိုက်တော့၊ ဒီ new features တွေကို မတွေ့တော့ဘူး။

---


# 0ctf 2016 - Zerostorage


	ဒီမှာ Merge ဆိုတာပါလာတယ် Insert function chunk ယူတဲ့အခါ 0x80 ထက်ငယ်ရင် 0x80 ဘဲ user
	data ယူတယ် အဲတော့ ငါတို့ free လုပ်လိုက်ရင် unsorted binထဲရောက်ရောက်သွားမယ် fast bin
	attack မရအောင်လုပ်ထားတယ်ပေါ့ ငါတို့ bypass လုပ်ရမယ် ဘယ်လိုလဲဆို porgram ရဲ့ 0x80အတွင်းဘဲ fast bin ထဲကို ထည့်မယ်ဆိုတဲ့ logic ကိုပြင်ရမယ် logic ဆိုတာက code နဲ့ရေးထားတာပေါ့ 0x80 နဲ့ check လုပ်တာကို ပြင်ရမယ် 0x80 ကို global_max_fast ဆိုတဲ့ variable ဘဲထည့်ထားတယ် ဒီကောင်က mallo() implementation ထဲမှာ  fastbin size limit ကို သတ်မှတ်ပေးတယ် global_max_fast ရှိတယ် အဲကောင် က libc data section ထဲမှာရှိတယ်။ 
	 ပြီးတော့ Delete function မှာ double free, use after
	free protection ပါတယ် ဆိုတော့ ခါတိုင်းလို free chunk ယောင်ယောင် allocated chunk ယောင်ယောင်နဲ့ chunk ကိုပြင်လို့မရတော့ဘူး ဒီမှာ Bug ရှိတာက Merge function , ဘာလုပ်တာလဲဆို from နဲ့
	to ရှိတယ် from chunk ထဲက data ကို to နဲ့သွားပေါင်းပြီး index တစ်ခုပေး from ထဲကကောင်ကို
	unsorted bin ထဲထည့်တယ်
	so think about it.... how to expoit??

	ဒါက delete funtion ထဲက ဘယ်လို protection လုပ်ထားလဲဆိုတာ
	 1. Used Flag ကို 0 ပြန်လုပ်ခြင်း
```c
	(&flag_1_or_not)[lVar3 * 6] = 0;  // ဒီ line က UAF ကာကွယ်ဖို့```
```
	Free လုပ်ပြီးတာနဲ့ used flag ကို 0 ပြန်လုပ်လိုက်တယ်။  
	ဒါကြောင့် View(), Update(), Merge() function တွေကို ခေါ်ရင်
```c
if ((&flag_1_or_not)[index * 6] == 1)  // condition fail ဖြစ်မယ်```
```
	Result: Freed chunk ကို access လုပ်လို့မရတော့ဘူး။

	2. XORed Pointer ကို 0 ပြန်လုပ်ခြင်း
```c
	(&DAT_00303070)[lVar3 * 3] = 0;  // pointer ကို 0 လုပ်တယ်
```
	ဒီလိုလုပ်လိုက်တာကြောင့်:
	- View() → `0 ^ GLOBAL_KEY = GLOBAL_KEY` → invalid address
	- Update() → အလားတူ invalid address
	- Merge() → အလားတူ



	ဆိုတော့ စလိုက်ရအောင်
	 အရင်ဆုံး insert ကို 0x20 နှစ်ခါယူလိုက်တယ် index 0 နဲ့ index 1 ပေးတယ် merge သုံးပြီး from 0 နဲ့ to 0 ပေးလိုက်တယ်  ဆိုတော့ from 0 အနေနဲ့က index 0 က unsorted bin ထဲရောက်သွားပြီး to 0 အနေနဲ့လည်း index0 နေရာ ကို index 2 အနေနဲ့ပေးလိုက်တယ် ဆိုတော့ unsorted bin ထဲကကောင်ကို data access လုပ်လို့ရ data edit လုပ်လို့ရသလိုဖြစ်သွားမယ် view နဲ့ index2ကိုကြည့်လိုက်ရင် fd နဲ့ bk နေရာက main_arena+88 address ကိုရ ထုံးစံအတိုင်း libc တွက်။ ပြီးရင် ငါတို့ overwrite မယ့်ကောင် ရဲ့ offset ရှာပြီဂ address ကိုပါတွက်။ 
	 ဆိုတော့ နောက်တစ်ဆင့်မစခင် fd နဲ့ bk တွေ ဘယ်လိုအလုပ်လုပ်သွားလဲဆိုတာမြင်ဖို့လိုတယ် တကယ် theory ပိုင်းမဟုတ်ဘဲ မျက်လုံးထဲမြင်မှ နောက်တစ်ဆင့် expoit ကို ချက်ချင်းတန်းအဖြေရမှာ
![](zero-3.jpg)



ပုံမှန် bin ထဲက free chunk တစ်ခုစီမှာ သူ့ဆီကထွက်သွားတဲ့မြှားနှစ်ခု ရှိတယ် အပေါ်တစ်ခု အောက်တစ်ခုပေါ့ (ရှေ့နောက်လည်းပြောလို့ရ) သူဆီဝင်တဲ့ မျှားနှစ်ခုလည်း ရှိတယ် အဝင်နှစ်ခု အထွက်နှစ်ခုပေါ့။  ဆိုတော့ပုံမှာအပြဝိုင်းပြထားတဲ့ ကောင်ကို binထဲက ထုတ်မယ်ဆိုရင်  အဲchunk ရဲ့ အဝင်မျှားကို direction တူရာ အထွက်မျှားနဲ့ overwrite လုပ်ပါတယ် ဆိုတော့ ထုတ် လုပ်မယ့်ကောင်ရဲ့ fd ကို value က သူ့အောက်က chunk ရဲ့ fd value ကို overwrite လုပ်ပါတယ် ဆိုတော့ ထုတ် လုပ်မယ့်ကောင်ရဲ့ bk ကို value က သူ့အထက်က chunk ရဲ့ bk value ကို
overwrite လုပ်ပါတယ် 


ဆိုတော့ unsorted binထဲက free chunk တစ်ခုထဲ အခြေအနေ ကိုကြည့်ရအောင်

unsorted ထဲ bin ကိုယ်တိုင်ကလည်း chunk တစ်ခုလိုပါဘဲ unsorted bin က circular doubly linked list ပုံစံနဲ့ရှိလို့ပါ unsorted bin ‌address က main_arena  နဲ့ offset 0x58 အကွာမှာရှိတာပါ အပေါ်က leak ဒါကလည်း unsorted bin address ပါ
![](zero-4.jpg)



ဆိုတော့ unsorted binထဲ free chunk တစ်ခုရောက်ရင် chunk နှစ်ခုရသလိုဖြစ်ပြီး တစ်ခုကိုတစ်ခု pointer ညွှန်းနေကြတာပါ 

![[zero-5.jpg]]

pic 1နဲ့ pic2 က အတူတူပါ ပိုပြီးရှင်းအောင် မြင်ရတဲ့ ပုံစံပါ 
chunk တစ်ခု ကို free လုပ်ရင် ထုံးစံအတိုင်း မျှားတွေ overwrite ပါမယ် unsorted binနဲ့  free chunk က အချင်းချင်း fd နဲ့ bk နှစ်ခုလုံး point လုပ်နေတာဆိုတော့
unsorted bin ရဲ့ fd နဲ့ bk က binထဲကထုတ်မယ့်  chunk ရဲ့ fd နဲ့ bk တွေ value တွေနဲ့ overwrite ခံရပါမယ်
ဒီနေရာမှာက unsorted binက သူ့ address သူ fd နဲ့ bkမှာရသွားပါတယ်
ဆိုတော့   binထဲကထုတ်မယ့် chunk  fd နဲ့ bk ကို အခြား valueတွေထားပြီး free လုပ်ရင် ဘယ်လိုဖြစ်မလဲ???

![](zero-6.jpg)


unsorted binရဲ့ fd , bk  value တွေက ငါတို့ချိန်းထားတဲ့ fd bk value တွေ ရသွားမှာပါ
ဒီနေရမှာက ငါတို့ရည်ရွယ်ချက်က global_max_fast ကို large number နဲ့ overwrite မှာဆိုတော့ ဒီကောင်ကို chunk တစ်ခုရဲ့ user data အစ (fd) လိုနေရမှာထားပြီး ဒီကောင်ရဲ့  global_max_fast-0x10 (chunk အစ) ‌address ကို unsorted binရဲ့ bkနေရာဖြစ်အောင်လုပ်လိုက်ရင်  global_max_fast(fd in fake chunk) က unsorted bin value ဖြစ်သွား
ဒီကောင်ကို unsorted binရဲ့ fd လိုနေရမှာ ထားလည်းရတယ် အဲကြ  chunk အစက global_max_fast-0x18 ဖြစ်သွား။ ဘာလို့ဆို global_max_fast ကို fake chunk ရဲ့ fdနေရာရောက်အောင်လို့
![[zero-7.jpg]]

ပုံက unsorted bin ရဲ့ fd နေရာထားတဲ့ပုံစံ
ဆိုတော့ နှစ်ခုလုံး test စမ်းကြည့်တော့ ပုံမှာပြထားတဲ့အတိုင်းဘဲအလုပ်ဖြစ်တယ်
ဘာလို့လဲဆိုပြီး ရှာလိုက်တော့
malloc က forward link ကိုမပြင်ဘူး backward link ကိုဘဲပြင်တယ်ပြောတယ်
unlink()မှာတုန်းက (binထဲကထုတ်မယ့် chunk က chunkA ဆိုပါစို့) chunk Aကို bin ထဲကထုတ်မယ်ဆိုရင် chunkA ရဲ့ fd ညွှန်းနေတဲ့ FD chunk ကိုအရင်သွားကြည့်ပြီး  FD chunk ရဲ့ bk (chunk Aကိုညွှန်းနေတာ) ကို  chunkA ရဲ့ bkနဲ့ overwrite, ပြီးရင်  chunk Aကို bin ထဲကထုတ်မယ်ဆိုရင် chunkA ရဲ့ bk ညွှန်းနေတဲ့ BK chunk ကိုအရင်သွားကြည့်ပြီး  BK chunk ရဲ့ fd (chunk Aကိုညွှန်းနေတာ) ကို  chunkA ရဲ့ fdနဲ့ overwrite ဆို့တော့ BK process က နောက်မှလာတယ်
ဒါမဲ့ ဒီနေရာမှာ ဘာလို့ automatically ဖြစ်သွားတာလဲ security check မရှိဘူးလား??????

	ဆိုတော့ exploit အဆင့်တွေပြန်စရအောင်။
	index 2 ကို data ဖြည့်ပြီး bkနေရာ global_max_fast - 0x10 address ဖြစ်အောင်လုပ်
	 နောက်ထပ် chunk ယူရင် binထဲကကောင်ကိုယူ unsorted bin ရဲ့ fd က  global_max_fast - 0x10 address ဖြစ်  global_max_fast ရဲ့ value က 0x80 ကနေ unsorted bin ရဲ့ addressဖြစ်
	 unsorted bin က corrupt ဖြစ်သွား




