
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


#### PLT & GOT 

##### PLT (Procedure Linkage Table)

- Code section ထဲမှာရှိတဲ့  stub functions တွေဖြစ်တယ်
- ပထမဆုံးခေါ်တဲ့အခါ GOT ကနေ real address ကိုသွားယူ
- နောက်ပိုင်းခေါ်ရင် GOT ထဲက real address ကို တန်းသွားခေါ်
- Example: `puts()` ဆိုတဲ့ function ကို call လိုက်ရင် တကယ်တမ်းက `puts@plt` ကိုခေါ်တာ


##### GOT (Global Offset Table)
- တကယ့် function addresses တွေသိမ်းထားတဲ့ address book
- Memory ထဲက libc functions တွေရဲ့တကယ့်လိပ်စာတွေပါ
-  Data section ထဲမှာရှိတဲ့ address table ဖြစ်တယ်
- dynamic linker က real function address တွေကို ဖြည့်ပေးတယ်
- Runtime မှာ address randomization (ASLR) ရှိရင် ကွဲပြားနိုင်တယ်

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
Section .plt 0x8048460 - 0x8048510:
0x8048470: printf@plt
0x8048480: gets@plt
0x8048490: fgets@plt
0x80484a0: __stack_chk_fail@plt
0x80484b0: getegid@plt
0x80484c0: puts@plt
0x80484d0: __libc_start_main@plt
0x80484e0: atol@plt
0x80484f0: setvbuf@plt
0x8048500: setresgid@plt
Section .plt.got 0x8048510 - 0x8048518:
0x8048510: __gmon_start__@plt
pwndbg> 

```

```
pwndbg> x/5i 0x80484c0
   0x80484c0 <puts@plt>:    jmp    DWORD PTR ds:0x804a018
   0x80484c6 <puts@plt+6>:  push   0x30
   0x80484cb <puts@plt+11>: jmp    0x8048440
```

```
got -r
State of the GOT of /home/Jackfruit/cate/learn/binary/stack/picoCTF/guessingGame2/vuln:
GOT protection: Full RELRO | Found 14 GOT entries passing the filter
[0x8049fc8] printf@GLIBC_2.0 -> 0xf7dbf2d0 (printf) ◂— call __x86.get_pc_thunk.ax
[0x8049fcc] gets@GLIBC_2.0 -> 0xf7de0f90 (gets) ◂— push ebp
[0x8049fd0] fgets@GLIBC_2.0 -> 0xf7ddfa80 (fgets) ◂— push ebp
[0x8049fd4] __stack_chk_fail@GLIBC_2.4 -> 0xf7e9b1f0 (__stack_chk_fail) ◂— call __x86.get_pc_thunk.ax
[0x8049fd8] getegid@GLIBC_2.0 -> 0xf7e51820 (getegid) ◂— mov eax, 0xca
[0x8049fdc] puts@GLIBC_2.0 -> 0xf7de1bc0 (puts) ◂— push ebp
[0x8049fe0] __libc_start_main@GLIBC_2.0 -> 0xf7d89f00 (__libc_start_main) ◂— push ebp
[0x8049fe4] atol@GLIBC_2.0 -> 0xf7da2a20 (atol) ◂— sub esp, 0x10
[0x8049fe8] setvbuf@GLIBC_2.0 -> 0xf7de2460 (setvbuf) ◂— push ebp
[0x8049fec] setresgid@GLIBC_2.0 -> 0xf7e6d3b0 (setresgid) ◂— push esi
[0x8049ff0] __gmon_start__ -> 0
[0x8049ff4] stdin@GLIBC_2.0 -> 0xf7f98de0 (stdin) —▸ 0xf7f985c0 (_IO_2_1_stdin_) ◂— 0xfbad2088
[0x8049ff8] rand@GLIBC_2.0 -> 0xf7da62a0 (rand) ◂— jmp random
[0x8049ffc] stdout@GLIBC_2.0 -> 0xf7f98ddc (stdout) —▸ 0xf7f98d40 (_IO_2_1_stdout_) ◂— 0xfbad2087
pwndbg> 

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
