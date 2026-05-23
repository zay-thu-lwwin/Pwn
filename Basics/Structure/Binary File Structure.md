


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

```
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
##  `Ropgadget`

### **Install**
```bash
pip install ROPgadget
# ဒါမှမဟုတ်
sudo apt install ropgadget
```

### **Basic Usage**
```bash
# Binary တစ်ခုလုံးက gadgets အားလုံးရှာမယ်
ROPgadget --binary ./r0bob1rd

# Specific gadgets ရှာမယ် (pop rdi)
ROPgadget --binary ./r0bob1rd | grep "pop rdi"

# libc ထဲမှာရှာမယ်
ROPgadget --binary ./glibc/libc.so.6 | grep "pop rdi"

# သတ်မှတ်ထားတဲ့ gadgets ပဲကြည့်မယ်
ROPgadget --binary ./r0bob1rd --only "pop|ret"

# Output ကို file ထဲသိမ်းမယ်
ROPgadget --binary ./r0bob1rd > gadgets.txt
```


```python
from pwn import *

# Binary load
elf = context.binary = ELF('./r0bob1rd')
libc = ELF('./glibc/libc.so.6')

# ROP object ဆောက်မယ်
rop = ROP(elf)

# Specific gadget ရှာမယ်
pop_rdi = rop.find_gadget(['pop rdi', 'ret'])[0]
pop_rsi = rop.find_gadget(['pop rsi', 'ret'])[0]
pop_rdx = rop.find_gadget(['pop rdx', 'ret'])[0]

print(f"pop rdi; ret: {hex(pop_rdi)}")
print(f"pop rsi; ret: {hex(pop_rsi)}")

# ဒါမှမဟုတ် rop object ထဲက direct သုံးမယ်
pop_rdi = rop.rdi.address  # pwntools 4.x+
```


---

#### With rp++
### **Install**
```bash
git clone https://github.com/0vercl0k/rp.git
cd rp
make
sudo make install
```

### **Usage**
```bash
# Binary ကနေ gadgets ရှာမယ်
rp++ -f ./r0bob1rd -r 5

# libc မှာရှာမယ်
rp++ -f ./glibc/libc.so.6 -r 5

# Output ကို json နဲ့ရမယ်
rp++ -f ./r0bob1rd -r 5 --json > gadgets.json
```

---

####  (Manual Way)

```bash
# Disassemble လုပ်ပြီး pattern ရှာမယ်
objdump -d ./r0bob1rd | grep -A 1 "pop.*rdi" | grep ret

# More detailed
objdump -d ./r0bob1rd | awk '/pop/ && /ret/ {print $0}'

# Raw bytes ကိုရှာမယ် (pop rdi = 0x5f, ret = 0xc3)
objdump -d ./r0bob1rd | grep "5f c3"
```

---

### `One-Gadget (to get shell directly)`

```bash
# one_gadget tool install
gem install one_gadget
# ဒါမှမဟုတ်
sudo apt install ruby-one-gadget

# libc ထဲက one-gadget တွေရှာမယ်
one_gadget ./glibc/libc.so.6

# Example output:
# 0xe3afe execve("/bin/sh", r15, r12)
# 0xe3b01 execve("/bin/sh", r15, rdx)
# 0xe3b04 execve("/bin/sh", rsi, rdx)
```

```c
└─$ one_gadget libc.so.6
0xe3afe execve("/bin/sh", r15, r12)
constraints:
  [r15] == NULL || r15 == NULL || r15 is a valid argv
  [r12] == NULL || r12 == NULL || r12 is a valid envp

0xe3b01 execve("/bin/sh", r15, rdx)
constraints:
  [r15] == NULL || r15 == NULL || r15 is a valid argv
  [rdx] == NULL || rdx == NULL || rdx is a valid envp

0xe3b04 execve("/bin/sh", rsi, rdx)
constraints:
  [rsi] == NULL || rsi == NULL || rsi is a valid argv
  [rdx] == NULL || rdx == NULL || rdx is a valid envp

```

##### In pwntools
```python
from pwn import *
libc = ELF('./glibc/libc.so.6')

# One gadget ရှာမယ် (one_gadget tool နဲ့ကြိုရှာထားရမယ်)
one_gadgets = [0xe3afe, 0xe3b01, 0xe3b04]
for og in one_gadgets:
    print(f"One gadget: {hex(libc.address + og)}")
```

---


| Gadget | Purpose | Bytes |
|--------|---------|-------|
| `pop rdi; ret` | First argument | `5f c3` |
| `pop rsi; ret` | Second argument | `5e c3` |
| `pop rdx; ret` | Third argument | `5a c3` |
| `pop rax; ret` | Syscall number | `58 c3` |
| `syscall; ret` | Execute syscall | `0f 05 c3` |
| `leave; ret` | Stack pivot | `c9 c3` |
| `ret` | Stack alignment | `c3` |

---

####  Advanced Tips

### **Find gadgets in stripped binary**
```bash
# Stripped binary မှာလည်း gadgets ရှိတယ်
ROPgadget --binary ./stripped_binary --depth 6 | head -20
```

### **Filter by specific registers**
```bash
# x86 (32-bit) gadgets
ROPgadget --binary ./binary32 | grep "pop eax"

# ARM gadgets
ROPgadget --binary ./arm_binary --arch ARM | grep "pop {r0"
```

### **Save and reuse**
```python
# Save gadgets to file
rop = ROP(elf)
with open('rop_chain.txt', 'w') as f:
    f.write(str(rop.dump()))

# Load later
import pickle
with open('gadgets.pkl', 'wb') as f:
    pickle.dump(rop.gadgets, f)
```


---


#### Binary Gadget Vs `Libc` Gadget (ASLR On, PIE Disabled)

```c
# Binary gadgets → real address = offset (base fixed)
pop_rdi_offset = 0x401234  # This IS the real address!
pop_rdi_real = 0x401234    # No calculation needed

# ✅ Can use directly
payload = p64(0x401234)  # pop rdi; ret
```

```c
# Libc offsets (from libc.so.6)
pop_rdi_libc_offset = 0x2a3e5  # This is just offset!
system_offset = 0x4f4e0
bin_sh_offset = 0x1b75aa

# Runtime: Need to leak libc base first
libc_base = leaked_address - system_offset  # Calculate base
pop_rdi_real = libc_base + pop_rdi_libc_offset  # Now real
system_real = libc_base + system_offset
```

---

#### Be careful of ROP

##### 1. Register Conventions (Calling Convention)

x86-64 (System V AMD64 ABI)
```python
# Function arguments order:
# RDI, RSI, RDX, RCX, R8, R9

# Example: system("/bin/sh")
pop_rdi = 0x401234          # pop rdi; ret
bin_sh = 0x7ffff7f7a123     # address of "/bin/sh"
system = 0x7ffff7e3a1d0     # system() address

rop_chain = p64(pop_rdi) + p64(bin_sh) + p64(system)
# ဒါက call system("/bin/sh") နဲ့အတူတူပဲ
```

x86 (32-bit)
```python
# Arguments on stack (right to left)
# system("/bin/sh") အတွက်:
rop_chain = p32(system) + p32(0xdeadbeef) + p32(bin_sh)
# return address after system() + argument
```

##### 2. Stack Alignment (Very Important!)

```python
# MOVAPS instruction in libc (especially system())
# Requires 16-byte stack alignment

# ❌ Wrong - May crash
payload = p64(pop_rdi) + p64(bin_sh) + p64(system)

# ✅ Correct - Add a ret gadget for alignment
ret = 0x401016  # single ret instruction
payload = p64(ret) + p64(pop_rdi) + p64(bin_sh) + p64(system)

# Why? system() uses MOVAPS that needs rsp % 16 == 0
```

Check alignment in GDB
```bash
gdb ./binary
b *system
run
# At breakpoint: info registers rsp
# rsp should end with 0 (e.g., 0x7fffffffdea0)
```

##### 3. NULL Bytes Problem

```python
# ROP chain ထဲမှာ NULL bytes (\x00) ပါရင်:
# - strcpy() က ဖြတ်ပစ်မယ်
# - scanf() က terminate လုပ်မယ်
# - read() က ရပါတယ် (binary read)

# ❌ Bad - Contains NULL bytes
payload = b'A'*64 + p64(0x0000000000401234)  # Has \x00 bytes

# ✅ Good - No NULL bytes
# Choose gadgets with addresses < 0x1000000
# Or use write_size="short" in fmtstr_payload
payload = fmtstr_payload(8, writes, write_size="short")
```

How to avoid NULL bytes
```python
# 1. Use small addresses (PIE disabled binaries)
# 2. Use write primitives that allow partial writes
# 3. Use encoders (for shellcode)

# Example: Build address from parts
high = 0x4012
low = 0x34
payload = p16(high) + p16(low)  # No NULL bytes


```


 ##### 4. Gadget Availability Checking

```python
from pwn import *

elf = context.binary = ELF('./target')
rop = ROP(elf)

# Check if gadget exists before using
def get_gadget(reg):
    try:
        if reg == 'rdi':
            return rop.find_gadget(['pop rdi', 'ret'])[0]
        elif reg == 'rsi':
            return rop.find_gadget(['pop rsi', 'ret'])[0]
    except:
        print(f"[-] pop {reg}; ret not found!")
        # Alternative: Use different registers or csu_init
        return None

# Alternative if pop rdi missing
if not pop_rdi:
    # Use call primitive through __libc_csu_init
    csu_front = 0x400c00  # pop rbx, rbp, r12, r13, r14, r15; ret
    csu_back = 0x400c20   # mov rdx, r13; mov rsi, r14; mov edi, r15d; call [r12+rbx*8]
```
