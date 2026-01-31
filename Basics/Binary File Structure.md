


```
┌─────────────────────────┐
│ ELF Header              │ ← Metadata about the file
├─────────────────────────┤
│ Program Headers         │ ← How to load into memory
├─────────────────────────┤
│ Sections                │ ← Actual code and data
│ ├── .text               │ ← Your compiled code
│ ├── .data               │ ← Initialized global variables
│ ├── .bss                │ ← Uninitialized global variables
│ ├── .rodata             │ ← Read-only data (strings, constants)
│ ├── .plt                │ ← Procedure Linkage Table
│ ├── .got/.got.plt       │ ← Global Offset Table
│ └── ... other sections  │
├─────────────────────────┤
│ Section Headers         │ ← Information about sections
├─────────────────────────┤
│ String Table            │ ← "Names" of sections/functions
├─────────────────────────┤
│ Symbol Table            │ ← Function/variable addresses
└─────────────────────────┘
```


```
BINARY FILE:
┌─────────────────┐
│     .text       │ ← Code
│     .rodata     │ ← Read-only data
│     .data       │ ← Initialized data (1337 stored here)
│     .bss        │ ← Only size information, no content!
└─────────────────┘

MEMORY AT RUNTIME:
┌─────────────────┐
│     .text       │
│     .rodata     │
│     .data       │ ← 1337 loaded here
│     .bss        │ ← OS allocates memory and fills with zeros
└─────────────────┘
```
	BSS ကို OS က runtime မှာ zero memory allocate လုပ်ပေးတာ၊ binary file ထဲမှာ content မသိမ်းဘူး (disk space ချွေတာဖို့)။
	to find bss address
	readelf -S vuln | grep -A1 -B1 "\.bss"
	or with elf.bss() in python script


	သိထားရမယ့် အချက်တွေ 
```
1. ELF Header - ဘာလဲဆိုတာပြော
2. Code (.text) - မင်း code အားလုံး
3. Data (.data, .bss) - Variables
4. PLT/GOT - Dynamic linking အတွက်
5. Symbol Table - Function names
6. String Table - Strings
7. Relocations - Address fixes
```

```
# Static vs Dynamic:
- Static: Libc functions in binary
- Dynamic: Libc functions in separate library

# Important addresses:
- Entry point: Program စတဲ့နေရာ
- main(): Main function
- win(): Target function (CTF)
- system(), execve(): Shell ရဖို့

# Exploitation အတွက်:
- .text: ROP gadgets
- .plt: Function calls  
- .got.plt: Libc leaks
- .data/.bss: Writable memory
```

---

##  `readelf`
	file structere တွေကို ကြည်ဖို့ readelf ကိုသုံးနိုင်
	ELf header တွေ sectionတွေ symbols တွေကိုကြည့်နိုင်
	
	elf header 
	 binary ရဲ့ metadata တွေကိုပြ
	Elf header တွေကြည့်ရန်
```
$ readelf -h ./vuln
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
...
```

	Section header
	section တွေရဲ့ meta data တွေကိုကြည့်ရန်
	section headerတွေကိုကြည့်ရန်
```
$ readelf -S ./vuln
There are 30 section headers, starting at offset 0x3a18:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [13] .text             PROGBITS         0000000000400c8c  00000c8c  ← Code
       0000000000000b62  0000000000000000  AX       0     0     16
  [14] .fini             PROGBITS         00000000004017f0  000017f0
       0000000000000009  0000000000000000  AX       0     0     4
....
```

	အရေးပါတဲ့ section header တွေကို ကြည့်ရန်
```
# Check specific sections for PWN
$ readelf -S ./vuln | grep -E "(\.got|\.plt|\.text|\.data|\.bss)"

  [13] .text             PROGBITS         0000000000400c8c  00000c8c  ← Code section
  [20] .plt              PROGBITS         0000000000401020  00001020  ← PLT
  [21] .plt.got          PROGBITS         0000000000402000  00002000  ← PLT for GOT
  [22] .got              PROGBITS         00000000006bc000  000bc000  ← GOT
  [23] .data             PROGBITS         00000000006bc4a0  000bc4a0  ← Writable data
  [24] .bss              NOBITS           00000000006bc4b0  000bc4b0  ← Writable (uninit)
```



	program headerက
	binary ကို memory ထဲဘယ်လို့ load မလဲဆိုတာပြပါတယ်
	Program headerတွေကြည့်ရန်
```
$ readelf -l ./vuln

Elf file type is EXEC (Executable file)
Entry point 0x400c8c
There are 9 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000400040 0x0000000000400040
                 0x00000000000001f8 0x00000000000001f8  R      0x8
  INTERP         0x0000000000000238 0x0000000000400238 0x0000000000400238
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x0000000000002038 0x0000000000002038  R E    0x200000  ← CODE (RX)
  LOAD           0x0000000000002e10 0x0000000000602e10 0x0000000000602e10
                 0x0000000000000238 0x0000000000000238  RW     0x200000  ← DATA 
                 .......  
```


	specific section dataတွေကို hex dump နဲ့ကြည့်ရန်
```
$ readelf -x .text ./vuln  # .text section ထဲက bytes တွေကို hex အနေနဲ့ကြည့်

Hex dump of section '.text':
  0x00400c8c 554889e5 4883ec10 897dfc48 8975f048 UH..H....}.H.u.H
  0x00400c9c 8b054e23 2b00be00 00000048 89c7e871 ..N#+......H...q
```

	symbol table ကြည့်ရန်
```
$ readelf -s ./vuln | head -20
Symbol table '.symtab' contains 100 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 000000000040038c     0 SECTION LOCAL  DEFAULT    1 
    50: 0000000000400c8c   101 FUNC    GLOBAL DEFAULT   13 main
    51: 0000000000400bd0   112 FUNC    GLOBAL DEFAULT   13 win
```



---

##  `Objdump`

	Disassemble code အလွယ်ကြည်ဖို့ objdump ကိုသုံးလို့ရ
	Machine code ကနေ assembly ကိုပြောင်းပြီးကြည့်တဲ့သဘော
	assembly instruction တွေ hex dump (raw byte)တွေကိုကြည့်လို့ရ
	Section info - Binary structure ကြည့်လို့ရ
	Symbol table - Functions/variables ကြည့်လို့ရ


	program code တွေကိုကြည့်ရန်
```
# All code disassembly
$ objdump -d ./vuln -M intel | head -30

0000000000400c8c <main>:
  400c8c:       55                      push   rbp
  400c8d:       48 89 e5                mov    rbp,rsp
  400c90:       48 83 ec 10             sub    rsp,0x10
  400c94:       89 7d fc                mov    DWORD PTR [rbp-0x4],edi
  400c97:       48 89 75 f0             mov    QWORD PTR [rbp-0x10],rsi
  400c9b:       48 8b 05 4e 23 2b 00    mov    rax,QWORD PTR [rip+0x2b234e]  # 6b2ff0 <stdout@@GLIBC_2.2.5>
  400ca2:       be 00 00 00 00          mov    esi,0x0
  400ca7:       48 89 c7                mov    rdi,rax
  400caa:       e8 71 fd ff ff          call   400a20 <setvbuf@plt>
```

	specific function ကိုကြည့်ရန်
```
# main() function only
$ objdump -d ./vuln -M intel --start-address=0x400c8c --stop-address=0x400d00

# win() function ရှာပြီးကြည့်မယ်
$ objdump -t ./vuln | grep win
0000000000400bd0 g     F .text  0000000000000070 win

$ objdump -d ./vuln -M intel --start-address=0x400bd0 --stop-address=0x400c40
```

	plt section ကိုကြည့်ရန်
```
$ objdump -d -j .plt ./vuln -M intel

Disassembly of section .plt:

0000000000400a20 <setvbuf@plt>:
  400a20:       ff 25 f2 25 2b 00       jmp    QWORD PTR [rip+0x2b25f2]  # 6b3018
  400a26:       68 00 00 00 00          push   0x0
  400a2b:       e9 e0 ff ff ff          jmp    400a10 <.plt>

0000000000400a30 <system@plt>:
  400a30:       ff 25 ea 25 2b 00       jmp    QWORD PTR [rip+0x2b25ea]  # 6b3020
```


	section headerတွေကိုကြည့်ရန်
```
$ objdump -h ./vuln

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .interp       0000001c  0000000000400238  0000000000400238  00000238  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 13 .text         00000b62  0000000000400c8c  0000000000400c8c  00000c8c  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 15 .rodata       000006d0  0000000000401800  0000000000401800  00001800  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 23 .data         00000010  00000000006bc4a0  00000000006bc4a0  000bc4a0  2**3
                  CONTENTS, ALLOC, LOAD, DATA
```

	symbols table anaylsis 
	symbols တွေကိုကြည့်ရန် (functions, variables)
```
$ objdump -t ./vuln | head -20

SYMBOL TABLE:
0000000000400c8c g     F .text  0000000000000065 main
0000000000400bd0 g     F .text  0000000000000070 win
0000000000400b40 g     F .text  000000000000008c do_stuff
0000000000400b10 g     F .text  000000000000002c get_random
0000000000400b00 g     F .text  000000000000000a increment
```

	Dynamic symbols တွေကိုကြည့်ရန်
```
$ objdump -T ./vuln | head -10

DYNAMIC SYMBOL TABLE:
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 setvbuf
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 system
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 rand
```

```
objdump [options] <file>
objdump -f program        # Display file header info
objdump -h program        # Show section headers (-headers)
objdump -d program        # Disassemble executable sections
objdump -D program        # Disassemble ALL sections
objdump -d -M intel program  # Intel syntax (default is AT&T)
objdump -t program        # Display symbol table
objdump -T program        # Dynamic symbol table
objdump -s program.so     # Shared library symbols
objdump -s -j .text program  # Show raw contents of .text section
objdump -s -j .data program  # Show .data section

objdump -T /lib/x86_64-linux-gnu/libc.so.6 | grep printf
```





----

##  `ldd`

	ldd (List Dynamic Dependencies) သည် executable file သို့မဟုတ် shared library တစ်ခု မည်သည့် shared libraries များကို လိုအပ်သည်ကို ပြသပေးသော tool
	ldd သည် binary ကို execute လုပ်ပြီးစစ်ဆေး

```
ldd [options] <executable-or-library>

```

---



---

##  `Ropgadget`

---

