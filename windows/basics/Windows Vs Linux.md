


##### 1. Binary Format ကွာခြားချက်

- **Linux**: ELF (Executable and Linkable Format) သုံးတယ်။
- **Windows**: PE (Portable Executable) သုံးတယ်။ → ဒါကြောင့် loader, memory layout, section names (.text, .data, .rdata, .pdata) ကွာသွားတယ်



**Binary Format (PE vs ELF) – Memory Mapping ကွာခြားပုံ**

Windows PE ရဲ့ memory layout က Linux ထက် ပိုရှုပ်တယ်။

**Linux ELF** (simple):

text

[ text ] [ data ] [ bss ] [ heap ] → [ mmap region ] ← [ stack ]

PIE ဖွင့်ရင် base address random (e.g., 0x55... စတယ်)

**Windows PE** (complex):

text

[ DOS header ] [ PE header ] [ .text ] [ .rdata ] [ .data ] [ .pdata ] [ .rsrc ] [ .reloc ] 
                                            ↓
                          ImageBase (default 0x400000 for exe, 0x10000000 for dll)

- `.text`: executable code
    
- `.rdata`: read-only data (import tables, const)
    
- `.data`: read-write data
    
- `.pdata`: exception handling info (x64 only) → **SEH အတွက် အရေးကြီး**
    
- `.reloc`: base relocation (ASLR အတွက်)
    

**ကွာခြားချက်**: သင်က ROP လုပ်တဲ့အခါ Linux က `objdump` နဲ့ gadgets ရှာလို့ရတယ်။ Windows က `dumpbin /disasm` သို့ `x64dbg` ရဲ့ "search for gadgets" သုံးရတယ်