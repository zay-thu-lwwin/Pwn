

##### PIC(Position Independent Code) , GOT (Global offset table), PLT( Procedure Linkage Table)


```

Higher Addresses
┌─────────────────┐
│     Stack       │ ← Local variables, function frames
├─────────────────┤
│       ↓         │
│                 │
├─────────────────┤
│      Heap       │ ← malloc(), dynamic memory
├─────────────────┤
│       ↓         │
│                 │
├─────────────────┤
│     .bss        │ ← Uninitialized global variables
├─────────────────┤
│    .data        │ ← Initialized global variables  
├─────────────────┤
│     .got        │ ← GOT (Global Offset Table) ★
├─────────────────┤
│    .plt         │ ← PLT (Procedure Linkage Table) ★
├─────────────────┤
│    .text        │ ← Code (PIC code here) ★
└─────────────────┘
Lower Addresses
```



---


### PLT & GOT 

#### PLT (Procedure Linkage Table)

- Code section ထဲမှာရှိတဲ့  stub functions တွေဖြစ်တယ်
- ပထမဆုံးခေါ်တဲ့အခါ GOT ကနေ real address ကိုသွားယူ
- နောက်ပိုင်းခေါ်ရင် GOT ထဲက real address ကို တန်းသွားခေါ်
- Example: `puts()` ဆိုတဲ့ function ကို call လိုက်ရင် တကယ်တမ်းက `puts@plt` ကိုခေါ်တာ


#### GOT (Global Offset Table)
- တကယ့် function addresses တွေသိမ်းထားတဲ့ address book
- Memory ထဲက libc functions တွေရဲ့တကယ့်လိပ်စာတွေပါ
-  Data section ထဲမှာရှိတဲ့ address table ဖြစ်တယ်
- dynamic linker က real function address တွေကို ဖြည့်ပေးတယ်
- Runtime မှာ address randomization (ASLR) ရှိရင် ကွဲပြားနိုင်တယ်



##### GOT (Global Offset Table)  Structure

GOT ကအပိုင်းနှစ်ပိုင်း ရှိတယ်

 1. GOT (non-PLT) - `.got` section
- သိမ်းဆည်းတဲ့အကြောင်းအရာ: Global variables, static variables, constants
- အသုံးပြုပုံ: Program က သူ့ရဲ့ **data** ကို ရှာဖွေဖို့


 2. GOT (PLT) - `.got.plt` section
- သိမ်းဆည်းတဲ့အကြောင်းအရာ: **Function pointers** (puts, printf, etc.)
- အသုံးပြုပုံ: Program က dynamic linker ကနေ function address တွေကို ရှာဖွေဖို့


GOT (PLT) က PLT stub တွေနဲ့ တိုက်ရိုက် ချိတ်ဆက်ထားတယ် (jump table အတွက်)
GOT (non-PLT) က သီးခြား data သိုလှောင်ရုံ။ PLT နဲ့ ဘာမှ မဆိုင်ဘူး

```bash
pwndbg> got -r

# GOT (non-PLT) - .got section
[0x601ff0] __libc_start_main@GLIBC_2.2.5 -> 0x7ffff7c58f90  ← real libc address
[0x601ff8] __gmon_start__ -> 0

# GOT (PLT) - .got.plt section  
[0x602018] _exit@GLIBC_2.2.5 -> 0x400766
[0x602020] puts@GLIBC_2.2.5 -> 0x400776
[0x602028] __stack_chk_fail@GLIBC_2.4 -> 0x400786
...
```

| Feature | GOT (non-PLT) | GOT (PLT) |
|---------|---------------|------------|
| Section name | `.got` | `.got.plt` |
| သိမ်းတဲ့အရာ | Real libc addresses (resolved) | PLT stub addresses (not resolved) |
| ဥပမာ | `__libc_start_main` | `puts`, `printf`, `__stack_chk_fail` |
| Permission (Partial RELRO) | **Read-only** | **Writable** |

##### GOT (non-plt example)


```c
// global variable တစ်ခု
int global_counter = 10;

int main() {
    // ဒီ variable ကို သုံးတဲ့အခါ GOT (non-PLT) ကိုသုံးတယ်
    global_counter = 20;  // GOT (non-PLT) ကနေ address ယူတယ်
}
```

**Assembly မှာ**:
```assembly
mov eax, DWORD PTR [global_counter@GOT]  ; GOT (non-PLT) ကိုသုံးတယ်
mov DWORD PTR [eax], 20
```




#### PLT VS .got.plt

|              | **PLT** (Procedure Linkage Table)      | **.got.plt** (GOT for PLT)                |
| ------------ | -------------------------------------- | ----------------------------------------- |
|              | Code section (executable instructions) | Data section (address storage)            |
| အလုပ်လုပ်ပုံ | Function ကို ခေါ်တဲ့ code stub         | Function address တွေ သိမ်းထားတဲ့ table    |
| Permission   | **RX** (Read + Execute)                | **RW** (Read + Write) - Partial RELRO မှာ |
| သိမ်းတာ      | Instructions (jmp, push, etc.)         | Addresses (function pointers)             |


---

##### Visual Diagram

```
Program Memory Layout:

┌─────────────────────────────────────────────────────┐
│ 0x400000 - 0x400750: (Other code)                  │
├─────────────────────────────────────────────────────┤
│ 0x400750 - 0x400820: PLT (Procedure Linkage Table) │
│                                                     │
│   [0x400760] _exit@plt:                            │
│       jmp [0x602018]  ──────────────┐              │
│       push 0                        │              │
│       jmp 0x400750                   │              │
│                                     │              │
│   [0x400770] puts@plt:              │              │
│       jmp [0x602020]  ──────────────┼──┐           │
│       push 1                        │  │           │
│       jmp 0x400750                   │  │           │
│                                     │  │           │
│   [0x400780] __stack_chk_fail@plt:  │  │           │
│       jmp [0x602028]  ──────────────┼──┼──┐        │
│       push 2                        │  │  │        │
│       jmp 0x400750                   │  │  │        │
│                                     │  │  │        │
│ **RX (Read + Execute)**             │  │  │        │
├─────────────────────────────────────┼──┼──┼────────┤
│ 0x602000 - 0x602fff: .got.plt       │  │  │        │
│                                     │  │  │        │
│   [0x602018] ───────────────────────┘  │  │        │
│   [0x602020] ──────────────────────────┘  │        │
│   [0x602028] ─────────────────────────────┘        │
│                                                     │
│ **RW (Read + Write)**                              │
└─────────────────────────────────────────────────────┘
```


##### Step by Step

1. first time when we call functions
```
puts@plt ကိုခေါ်
↓
GOT ထဲကြည့် - လိပ်စာမရှိသေး
↓
ld.so (dynamic linker) ကိုခေါ်
↓
တကယ့် puts လိပ်စာရှာ
↓
GOT ထဲမှာသိမ်း
↓
puts ကိုသွား

----------------------------------------

- PLT က GOT ကိုသွား → dynamic linker ကို jump
    
- Linker က libc ထဲက real puts address ကို ရှာတယ် (ဥပမာ: `0xf7e5a190`)
    
- အဲဒီ address ကို GOT ထဲမှာ ပြန်သိမ်းတယ်
    
- real puts ကို execute လုပ်တယ်
  
```

2. next times
```
puts@plt ကိုခေါ်
↓
GOT ထဲကြည့် - လိပ်စာရှိနေပြီ
↓
တန်းသွား
```


| Component    | ဘာလဲ                      | သိမ်းတာ                                      | Permission      |
| ------------ | ------------------------- | -------------------------------------------- | --------------- |
| **PLT**      | Code section (executable) | `jmp [GOT]` လို instructions                 | **RX**          |
| **.got.plt** | Data section              | Function addresses (သို့) PLT stub addresses | RELRO ပေါ်မူတည် |
| **.got**     | Data section              | Global variables, real libc addresses        | RELRO ပေါ်မူတည် |

#### how to use in binary Exploitation 

##### 1.  Direct use of PLT 
- Example: `system` function PLT ရှိရင်
- ကိုယ်တိုင်လိပ်စာရှာစရာမလိုဘူး
- PLT address ကိုပဲ jump လုပ်ရုံ

##### 2. Leak form GOT
- GOT က binary ထဲမှာပါတယ်
- PIE disable ဖြစ်ရင် GOT address သိတယ်
- Arbitrary read ရှိရင် GOT ထဲကလိပ်စာတွေဖတ်လို့ရတယ်
- **ဒါဆို libc base address ရမယ်**
- **ASLR ကျော်ပြီ**

#####  Exploitation Scenario
```
1. GOT ထဲက puts address ကိုဖတ်
2. Libc base address တွက်ချက် (puts_offset သိထားရင်)
3. System function address တွက်ချက်
4. System("/bin/sh") ကိုခေါ်
5. Shell ရမယ်
```


PLT = ကြားခံ function calls  
GOT = တကယ့် function addresses သိမ်းတဲ့နေရာ  
ASLR Bypass = GOT ကနေ addresses leak လုပ် → libc base တွက် → လိုချင်တဲ့ function address ရှာ  


```
pwndbg> plt
Section .plt 0x401020 - 0x4010a0:
No symbols found in section .plt
Section .plt.sec 0x4010a0 - 0x401110:
0x4010a0: strcpy@plt
0x4010b0: puts@plt
0x4010c0: system@plt
0x4010d0: printf@plt
0x4010e0: strcspn@plt
0x4010f0: fgets@plt
0x401100: fflush@plt
pwndbg> 


```

##### before calling puts
```

pwndbg> disas 0x4010b0
Dump of assembler code for function puts@plt:
   0x00000000004010b0 <+0>:     endbr64
   0x00000000004010b4 <+4>:     bnd jmp QWORD PTR [rip+0x2f65]        # 0x404020 <puts@got.plt>
   0x00000000004010bb <+11>:    nop    DWORD PTR [rax+rax*1+0x0]
End of assembler dump.
pwndbg> x 0x404020
0x404020 <puts@got.plt>:        0x00401040
pwndbg> 

```


```
pwndbg> got
Filtering out read-only entries (display them with -r or --show-readonly)

State of the GOT of /home/Jackfruit/cate/learn/binary/stack/picoCTF/inputInjection/vuln:
GOT protection: Partial RELRO | Found 7 GOT entries passing the filter
[0x404018] strcpy@GLIBC_2.2.5 -> 0x401030 ◂— endbr64 
[0x404020] puts@GLIBC_2.2.5 -> 0x401040 ◂— endbr64 
[0x404028] system@GLIBC_2.2.5 -> 0x401050 ◂— endbr64 
[0x404030] printf@GLIBC_2.2.5 -> 0x401060 ◂— endbr64 
[0x404038] strcspn@GLIBC_2.2.5 -> 0x401070 ◂— endbr64 
[0x404040] fgets@GLIBC_2.2.5 -> 0x401080 ◂— endbr64 
[0x404048] fflush@GLIBC_2.2.5 -> 0x401090 ◂— endbr64 


```

##### after calling puts
```
pwndbg> disas 0x4010b0
Dump of assembler code for function puts@plt:
   0x00000000004010b0 <+0>:     endbr64
   0x00000000004010b4 <+4>:     bnd jmp QWORD PTR [rip+0x2f65]        # 0x404020 <puts@got.plt>
   0x00000000004010bb <+11>:    nop    DWORD PTR [rax+rax*1+0x0]
End of assembler dump.
pwndbg> x 0x404020
0x404020 <puts@got.plt>:        0xf7c80e60


pwndbg>  x/2wx 0x404020 
0x404020 <puts@got.plt>:        0xf7c80e60      0x00007fff
pwndbg> 


```

```
pwndbg> got
Filtering out read-only entries (display them with -r or --show-readonly)

State of the GOT of /home/Jackfruit/cate/learn/binary/stack/picoCTF/inputInjection/vuln:
GOT protection: Partial RELRO | Found 7 GOT entries passing the filter
[0x404018] strcpy@GLIBC_2.2.5 -> 0x401030 ◂— endbr64 
[0x404020] puts@GLIBC_2.2.5 -> 0x7ffff7c80e60 (puts) ◂— push r14
[0x404028] system@GLIBC_2.2.5 -> 0x401050 ◂— endbr64 
[0x404030] printf@GLIBC_2.2.5 -> 0x401060 ◂— endbr64 
[0x404038] strcspn@GLIBC_2.2.5 -> 0x401070 ◂— endbr64 
[0x404040] fgets@GLIBC_2.2.5 -> 0x401080 ◂— endbr64 
[0x404048] fflush@GLIBC_2.2.5 -> 0x401090 ◂— endbr64 
pwndbg> 

```






#### RELRO (`RELocation` Read-Only)

RELRO က binary security mechanism တစ်ခုဖြစ်ပြီး GOT (Global Offset Table) ကို ကာကွယ်ပေးတဲ့ နည်းလမ်း


##### NO RELRO

```bash
checksec : RELRO: No RELRO
```

GOT အားလုံး **writeable** (read-write)
Lazy binding **yes** (normal)
Lazy Binding ဆိုတာ **dynamic linker** က shared library function တွေရဲ့  တကယ့် address ကို  အခေါ်ခံရမှ ရှာဖွေပေးတဲ့ စနစ်
`.got` နဲ့ `.got.plt` အကုန် writeable
Format string bug ရှိရင် GOT ကို တန်း overwrite လုပ်လို့ရတယ်
Buffer overflow ရှိရင် GOT ကို write လုပ်လို့ရတယ် (RW ဖြစ်လို့)


```bash
pwndbg> checksec
RELRO: No RELRO

pwndbg> x/wx 0x8049fdc      # puts မခေါ်ခင်
0x8049fdc: 0x80484c6         # PLT resolver stub

pwndbg> ni                  # puts ခေါ်လိုက်
pwndbg> x/wx 0x8049fdc      
0x8049fdc: 0xf7de1bc0        # Real address (updated)

pwndbg> vmmap 0x8049fdc
0x8049fdc rw-p ...           # Writeable
```


##### PARTIAL RELRO

```bash
checksec: RELRO: Partial RELRO
```

`.got` section → **read-only**  (non-PLT GOT)
`.got.plt` section → **writeable** (အရေးကြီးတဲ့ GOT entries တွေ)  (lazy binding လုပ်ဖို့) (PLT GOT) 
Lazy binding **yes** (ပုံမှန်)
`.dynamic` section ကို read-only လုပ်တယ်
Function pointers (puts, `printf`, etc.) ကို ပြောင်းလို့ရတယ်

```bash
pwndbg> checksec
RELRO: Partial RELRO

# .got.plt က writeable ဖြစ်နေသေးတယ်
pwndbg> vmmap .got.plt
0x8049fdc rw-p ...           # Writeable

# .got က read-only
pwndbg> vmmap .got
0x8049fd0 r--p ...           # Read-only

# Lazy binding အလုပ်လုပ်တယ်
pwndbg> x/wx 0x8049fdc      # puts မခေါ်ခင်
0x8049fdc: 0x80484c6

pwndbg> ni                  # puts ခေါ်ပြီး
pwndbg> x/wx 0x8049fdc
0x8049fdc: 0xf7de1bc0        # Update ဖြစ်တယ် (RW ဖြစ်လို့)
```


##### FULL RELRO

```bash
checksec : RELRO: Full RELRO
```

`.got` → **read-only** (`mprotect` သုံးပြီး)
`.got.plt` → **read-only** (အရေးကြီးဆုံး)
Lazy binding **no** (disabled) Lazy binding လုံးဝမရှိတော့ (runtime မှာ resolver မခေါ်ရတော့)
Program စတင်ကတည်းက (main မရောက်ခင်မှာပဲ) dynamic linker က GOT အားလုံးကို real address တွေနဲ့ တစ်ခါတည်း ဖြည့်ပေးတယ်

```bash
pwndbg> checksec
RELRO: Full RELRO

pwndbg> x/wx 0x8049fdc      # puts မခေါ်ခင်
0x8049fdc: 0xf7de1bc0        # Real address (အစကတည်းက!)

pwndbg> ni                  # puts ခေါ်ပြီး
pwndbg> x/wx 0x8049fdc
0x8049fdc: 0xf7de1bc0        # အတူတူပဲ (update မဖြစ်)

pwndbg> vmmap 0x8049fdc
0x8049fdc r--p ...           # Read-only! (မပြောင်းနိုင်)
```




##### No PIPE

```
# Local
elf.got['printf'] = 0x8049fc8

# Remote (same binary)
elf.got['printf'] = 0x8049fc8  # ✅ တူတူဘဲ
```


##### With PIE

```
# Local
elf.got['printf'] = 0x561234567010  (random base)

# Remote  
elf.got['printf'] = 0x7fabcdef0010  (different random base)
```


```
End of assembler dump.
pwndbg> disas 0x8048470
Dump of assembler code for function printf@plt:
   0x08048470 <+0>:     jmp    DWORD PTR ds:0x8049fc8
   0x08048476 <+6>:     push   0x0
   0x0804847b <+11>:    jmp    0x8048460
End of assembler dump.
pwndbg> x 0x8049fc8
0x8049fc8 <printf@got.plt>:     0xf7d8a2d0
pwndbg> 

```


|                | `elf.plt['printf']`                | `elf.got['printf']`                    |
| -------------- | ---------------------------------- | -------------------------------------- |
| ဘာလဲ           | PLT stub ရဲ့ လိပ်စာ                | GOT entry ရဲ့ လိပ်စာ                   |
| တန်ဖိုး        | `0x8048470`                        | `0x8049fc8`                            |
| အဲဒီမှာဘာရှိလဲ | `jmp [0x8049fc8]` (code)           | `0xf7d8a2d0` (real address data)       |
| ဘာအတွက်သုံးလဲ  | Function ကို **ခေါ်ဖို့** (call)   | GOT ထဲက **value ကို leak** ဖို့        |
| ဘယ်ကရလဲ        | ELF binary က code section (`.plt`) | ELF binary က data section (`.got.plt`) |
| ပြောင်းလဲလား   | No (PIE မပါရင်)                    | No (PIE မပါရင်)                        |



```
Address: 0x8048470 (printf@PLT)  ← elf.plt['printf'] က ဒီကိုညွှန်း
┌─────────────────────────────┐
│ 0x8048470: jmp [0x8049fc8]  │
│ 0x8048476: push 0x0         │
│ 0x804847b: jmp 0x8048460    │
└─────────────────────────────┘
              │
              │ (jump to address stored at 0x8049fc8)
              ▼
Address: 0x8049fc8 (printf@GOT) ← elf.got['printf'] က ဒီကိုညွှန်း
┌─────────────────────────────┐
│ 0x8049fc8: 0xf7d8a2d0       │ 
└─────────────────────────────┘
              │
              │ (contains real address)
              ▼
Address: 0xf7d8a2d0 (real printf in libc)
┌─────────────────────────────┐
│ 0xf7d8a2d0: push ebp        │ ← ဒါက real printf function
│ 0xf7d8a2d1: mov ebp, esp    │
│ ...                         │
└─────────────────────────────┘
```




---


credit - Ko Kaung Min Myat
Resource : https://can-ozkan.medium.com/got-vs-plt-in-binary-analysis-888770f9cc5a

.got section (global offset table) ကို program ထဲက global variables အတွက်သုံးတယ်။
.got.plt subsection က libc ရဲ့ function address ကို သိမ်းပေးဖို့သုံးတယ်။

.plt (procedure linked table) section
Program မှာ printf လိုမျိုး libc ရဲ့ functions တွေ run မယ်ဆို plt ကို အရင်သွားရတယ်။
plt က ဂိတ်ပေါက်လို အလုပ်လုပ်ပေးတယ်။
plt က got.plt ကို သွားစစ်တယ်။
-> got.plt မှာ printf address မရှိရင် dl_runtime_resolve ကို run ပြီး address ရှာရတယ်။ ပြီးရင် address ကို got.plt မှာ တစ်ခါထည်း မှတ်ထားတယ်။
-> got.plt မှာ printf address ရှိရင် printf function ကိုခေါ်လိုက်တယ်။

---
Resource : https://gist.github.com/ricardo2197/8c7f6f5b8950ed6771c1cd3a116f7e62

Dl_runtime_resolve ( dl_resolve )
_dl_resolve က ld.so libc နဲ့ ချိန်ဆက်ပြီး address တွေကို ရှာပေးပါတယ်။
အဲ့လိုရှာဖို့  .dynamic section ကို အားကိုးရပါတယ်။
.dynamic section ထဲမှာဆို dl_resolve အတွက် လိုအပ်တဲ့ information တွေသိမ်းပေးထားတယ်။
⦁ JMPREL (jump relocation) က .rel.plt (PLT relocation table) ရဲ့ address ကို jump ပေးပါတယ်။
⦁ jmprel က dl_resolve ကို သက်ဆိုင်ရာ .rel.plt ထဲက offset တွေ info တွေ ပြသပေးပြီး ရှာရလွယ်ကူအောင်လုပ်ပေးပါတယ်။
⦁ STRTAB (string table) က symbol name  (like printf, read, libc.so.6) တွေကို သိမ်ပေးထားပါတယ်။
⦁ SYMTAB (symbol table) symbol နဲ့ သတ်ဆိုင်တဲ့ information တွေ သိမ်းပေးတယ်။
⦁ SYMTAB ထဲက symbol name မှာ strings (like printf) ကို မသိမ်းပဲ STRTAB ထဲက offset ( printf နေရာ) ကို သိမ်းပါတယ်။
⦁ dl_resolve က JMPREL နဲ့ loaded libraries တွေ ပါတဲ့ rel.plt ထဲမှာ SYMTAB ကို သုံးပြီး လိုချင်တဲ့ symbol တွေလိုက်ရှာပါတယ်။ 
⦁ တွေ့ရင်  symbol address ကို .got.plt မှာသိမ်းပေးပါတယ်။



