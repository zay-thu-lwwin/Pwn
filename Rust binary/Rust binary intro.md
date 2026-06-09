
 Rust  challenge က သိပ်ခက်တယ် ဘာလို့လဲဆိုတော့ Rust က memory safety ကို compile time မှာကို သေချာကာကွယ်ပေးလို့ ဒါပေမယ့် vulnerability  တွေကတော့ ရှိနိုင်တယ်
Rust က borrow checker ကြောင့် classic memory corruption bugs တွေ သဘာဝအတိုင်း မရှိသလောက်


### How to identify a Rust binary
#### Function Naming Convention

Rust က **mangled name** (decorated name) ကို သူ့ဟာသူထူးခြားတဲ့ပုံစံနဲ့ သုံးတယ်

```C
Rust :   alloc::raw_vec::RawVec<T,A>::grow_one
C++ :    _ZNK3std6string6String5capacityEv
C :      printf, malloc, strcpy
```
- `::` (double colon) တွေအများကြီးပါတယ် (namespace/module separator)
- `<T,A>` လို generic parameters တွေပါတယ်
- `alloc::` (Memory allocation လုပ်တဲ့အပိုင်း)
- `core::` (Rust ရဲ့ internal အခြေခံ core libraries)
- `std::` (Rust ရဲ့ Standard Library)

#### Standard Library Functions
```
alloc::raw_vec::RawVec<T,A>::grow_one     # Rust's vector
core::num::<impl u32>::to_be_bytes        # Integer conversion
core::str::iter::SplitWhitespace          # String handling
alloc::string::String::push_str           # String operations
regex_automata::...                        # Regex crate
digest::core_api::wrapper::CoreWrapper    # Cryptography crate
```

keywords:

- `alloc::` - memory allocation
- `core::` - core language features
- `std::` - standard library
- `::{{closure}}` - anonymous function (closure)
- `vtable.shim` - trait object vtable

---
#### Hybrid Linking 

Rust compiler (`rustc`) ဟာ binary တစ်ခုကို တည်ဆောက်တဲ့အခါ Linking လုပ်ငန်းစဉ်ကို နေရာနှစ်ပိုင်း ခွဲခြားပြီး ကိုင်တွယ်ပါတယ်-

#####  Static Linking for Rust Packages

 `i fun` ထဲမှာ မြင်ရတတ်တဲ့ `regex`, `digest` (SHA256) စတဲ့ External Rust Crates (third-party libraries) တွေနဲ့ Rust ရဲ့ ကိုယ်ပိုင် Standard Library (`std`) တွေကိုတော့ Rust က အပြင်မှာ လုံးဝမချန်ထားခဲ့ဘူး အဲဒီ code တွေ အကုန်လုံးကို Statically Linked လုပ်ပြီးbinary ထဲကို တစ်ခါတည်း သွတ်သွင်းပစ်လိုက်တယ်
  ဒါကြောင့်မို့လို့ `ls -al` စစ်လိုက်တဲ့အခါ စာသား binary တစ်ခုသက်သက် ဖြစ်ပေမဲ့ file size က **5.4 MB (`5468704` bytes)** တောင် ဖြစ်နေတာပါ
   (C နဲ့သာ ရေးရင် 20 KB လောက်ပဲ ရှိမှာပါ)။
    

##### Dynamic Linking for Operating System's Low-level APIs  

Rust က ဘယ်လောက်ပဲ ကောင်းကောင်း၊  Rust က `syscall` wrapper ကို မရေးဘူး
Rust က Linux kernel နဲ့ တိုက်ရိုက် `syscall` မလုပ်ဘူး အဲဒီအစား `C library (libc) `ရဲ့ wrapper functions တွေကို ခေါ်သုံးတယ် ဥပမာ- screenပေါ် စာသားလှမ်းပြဖို့ `printf/write` သုံးတာ၊ network ချိတ်ဖို့ `socket` သုံးတာ၊ core thread တွေ စီမံခန့်ခွဲတာ စတဲ့ **Low-level OS Functions** တွေအတွက် Linux ရဲ့ အခြေခံအကျဆုံးဖြစ်တဲ့ **`libc.so.6` (C Library)** နဲ့ **`libgcc_s.so.1`** တို့ကို လှမ်းကိုင်ရတယ်
အဲဒီ OS Level Libraries တွေကိုတော့ Rust က binary ထဲ ထည့်မချုပ်ဘူး 
Operating System ရဲ့ လုံခြုံရေးနဲ့ memory ချွေတာရေးအတွက် OS ရဲ့ standard libraries တွေကိုပဲ **Dynamic Linking** အနေနဲ့ လှမ်းချိတ်ပြီး သုံးဖို့ ချန်ထားခဲ့တာ ဖြစ်တယ်

```rust
// Rust code ထဲမှာ
std::fs::File::open("file.txt")

// အောက်ခြေမှာ libc က open() syscall wrapper ကိုခေါ်တယ်
libc::open("file.txt", flags, mode)
```

```bash
# 1. Check all dynamic dependencies
objdump -p tictactoe | grep NEEDED
# Output:
#   NEEDED   libgcc_s.so.1
#   NEEDED   libc.so.6

# 2. Check which functions are dynamically linked
objdump -T tictactoe | head -20
# Output: puts, malloc, free, memcpy, etc.

# 3. Check Rust static code size
size tictactoe
# text section က 5MB+ ဆိုတာ static code ပါ
```

```bash
──(Jackfruit㉿kali)-[~/…/stack/htb/TicTacToed/pwn_tictactoed]
└─$ file tictactoe 
tictactoe: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 4.4.0, BuildID[sha1]=6f3bdd1a7989d5ba761bc461c285bf98da6ca4e5, not stripped
                                                                                                                                       
┌──(Jackfruit㉿kali)-[~/…/stack/htb/TicTacToed/pwn_tictactoed]
└─$ ls -al tictactoe 
-rwxr-xr-x 1 Jackfruit kali 5468704 Mar  7  2025 tictactoe
                                                                                                                                       
┌──(Jackfruit㉿kali)-[~/…/stack/htb/TicTacToed/pwn_tictactoed]
└─$ ldd tictactoe   
        linux-vdso.so.1 (0x00007fc53c7dd000)
        libgcc_s.so.1 => /usr/lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fc53c78f000)
        libc.so.6 => /usr/lib/x86_64-linux-gnu/libc.so.6 (0x00007fc53be00000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fc53c7df000)

```

```
┌────────────────────────────────────────────────────────┐
│  tictactoe Binary File (5.4 MB)                        │
│                                                        │
│  ├── [Custom Code]   -> tictactoe::main()              │
│  ├── [Rust Standard] -> alloc::, core::                 │  <-- Rust အပိုင်းတွေကို
│  └── [Third-Party]   -> regex_automata::, digest::      │      အပိုင် Static သွင်းထားတယ်
└───────────────────────────┬────────────────────────────┘
                            │
                            │ (Dynamic Link)
                            ▼
               ┌────────────────────────┐
               │ Linux System Libraries │
               │ ├── libc.so.6          │  <-- OS အပိုင်းကိုတော့
               │ └── libgcc_s.so.1      │      စက်ထဲကကောင်ကိုပဲ လှမ်းသုံးတယ်
               └────────────────────────┘
```

---
#### Custom Logic

Rust မှာ ကိုယ်တိုင်ရေးတဲ့ code တွေ (Custom Logic) က ကိုယ်ပေးထားတဲ့ project နာမည်အောက်မှာပဲ ရှိနေတတ်တယ်။ ဥပမာ- binary နာမည်က `tictactoe` ဖြစ်တဲ့အတွက် အရှေ့မှာ **`tictactoe::...`** လို့ စတဲ့ကောင်မှန်သမျှဟာ ဒီ challenge ရေးတဲ့လူ ကိုယ်တိုင် လက်နဲ့ ထိုင်ရေးထားတဲ့ custom functions တွေ ဖြစ်တယ်

- `tictactoe::detect_debugger` (သူတို့ကိုယ်တိုင် ရေးထားတဲ့ debugger ဖမ်းတဲ့စနစ်)
- `tictactoe::validate_access_code` (သူတို့ကိုယ်တိုင် ဉာဏ်ဆင်ထားတဲ့ access code စစ်တဲ့စနစ်)

ကျန်တဲ့ `alloc::`, `regex_automata::` စတာတွေက Rust ရဲ့ compiler က ကြားထဲကနေ အလိုအလျောက် ထည့်ပေးလိုက်တဲ့ Library တွေမို့လို့ Reverse Engineering လုပ်ရင် လုံးဝ (လုံးဝ) ထိုင်ဖတ်နေစရာမလိုတဲ့ အမှိုက် (Boilerplate code) တွေ ဖြစ်တယ်

---


#### Vuln Focus on 


| Language | Memory Safety   | Buffer Overflow | Use After Free | Double Free |
| -------- | --------------- | --------------- | -------------- | ----------- |
| C/C++    | no              | happens         | happens        | happens     |
| **Rust** | safe by default | rare            | rare           | rare        |

#####  Unsafe Rust (main vuln)
Rust က `unsafe` block ထဲမှာ C လို pointer manipulation လုပ်လို့ရတယ်။
```rust
unsafe {
    let ptr = heap_address as *mut u8;
    *ptr = 0xFF;  // buffer overflow ဖြစ်နိုင်တယ်
}
```

ဘယ်လိုရှာမလဲ `unsafe` keyword ကို binary ထဲမှာ ရှာပါ
```bash
strings binary | grep -i unsafe
objdump -d binary | grep -i unsafe
```

##### Integer Overflows

Rust က debug mode မှာ overflow စစ်ပေမယ့် **release mode** မှာ wrapping လုပ်တယ်။
```rust
let x: u8 = 255;
let y = x + 1;  // release mode မှာ 0 ဖြစ်သွားမယ်
```

##### Panics & Unwinding
Rust က error ဖြစ်ရင် `panic!` macro နဲ့ program ပျက်တယ် Unwind လုပ်ချိန်မှာ အားနည်းချက်ရှိနိုင်တယ်။
```bash
# panic handler တွေကို ရှာပါ
nm binary | grep panic
```

##### Logic Bugs (common)
	- Type confusion (enums နဲ့)
	- Integer underflow/overflow
	- Race conditions (threads သုံးရင်)
	- Cryptographic implementation bugs

##### Format String Vulnerabilities
Rust က `println!` macro သုံးတယ်၊ 
ဒါပေမယ့် format string vulnerability က မရှိသလောက်
(compile time မှာစစ်တယ်)


---

