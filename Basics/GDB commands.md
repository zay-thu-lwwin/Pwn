
```
x/[COUNT][FORMAT][SIZE] address
များသောအားဖြင် size က formatထက်အရင်လာ

 Format Letters (display format)


- x - hexadecimal   
- d - decimal  
- u - unsigned decimal  
- o - octal  
- t - binary  
- c - character   
- s - string  
- i - instructions 
  
 Size Letters

- b - byte (1 byte) 
- h - halfword (2 bytes) 
- w - word (4 bytes) 
- g - giant (8 bytes)
  
```

##### How count works

Count က ဘယ်နှစ်ကြိမ် (ယူနစ်ဘယ်နှစ်ခု) ပြမလဲ ဆိုတာကို ဆုံးဖြတ်

##### 64-bit (g size = 8 bytes)
```
# count = 2 (2 ခုပြမယ်)
(gdb) x/2gx 0x1000
0x1000: 0x0807060504030201  0x100f0e0d0c0b0a09
        ^^^^^^^^^^^^^^^^    ^^^^^^^^^^^^^^^^
        (1st 8 bytes)       (2nd 8 bytes)

# count = 3 (3 ခုပြမယ်)
(gdb) x/3gx 0x1000
0x1000: 0x0807060504030201  0x100f0e0d0c0b0a09  0x????????????????
        (1st)               (2nd)               (3rd - နောက် 8 bytes)
```

##### 32-bit (w size = 4 bytes)
```
# count = 4 (4 ခုပြမယ် - တစ်ခုက 4 bytes)
(gdb) x/4wx 0x1000
0x1000: 0x04030201  0x08070605  0x0c0b0a09  0x100f0e0d

# count = 2 (2 ခုပြမယ်)
(gdb) x/2wx 0x1000
0x1000: 0x04030201  0x08070605
```

##### Total Bytes = count × size

- `x/10gx` → 10 × 8 = 80 bytes ပြမယ်
- `x/20bx` → 20 × 1 = 20 bytes ပြမယ်
- `x/5wx` → 5 × 4 = 20 bytes ပြမယ်


`Just x`

x ချည်းသက်သက်က သုံးခဲ့ဖူးတဲ့ instruction ကိုမှတ်ထားပြီးပြန်သုံးတဲ့သဘော (countတော့အကျုံးမဝင်)
```
pwndbg> x/wx 0x8049fc8
0x8049fc8 <printf@got.plt>:     0xf7dbf2d0
pwndbg> x/i 0x8049fc8
   0x8049fc8 <printf@got.plt>:  shl    dl,1
pwndbg> x 0x8049fc8
   0x8049fc8 <printf@got.plt>:  shl    dl,1
pwndbg> x/wx 0x8049fc8
0x8049fc8 <printf@got.plt>:     0xf7dbf2d0
pwndbg> x 0x8049fc8
0x8049fc8 <printf@got.plt>:     0xf7dbf2d0
pwndbg> 

```

`i` က **instruction** ကိုပြဖို့ဖြစ်ပြီး၊ သူက size မလို
```
# Current instruction pointer (RIP) မှာ 5 instructions ပြမယ်
(gdb) x/5i $rip
=> 0x4005b0 <main>:     push   %rbp          # 1 byte instruction
   0x4005b1 <main+1>:   mov    %rsp,%rbp     # 3 bytes
   0x4005b4 <main+4>:   sub    $0x10,%rsp    # 4 bytes
   0x4005b8 <main+8>:   movl   $0xa,-0x4(%rbp) # 6 bytes
   0x4005bf <main+15>:  mov    $0x0,%eax     # 5 bytes
```
အပေါ်မှာ မြင်ရတဲ့အတိုင်း instruction တွေက အရွယ်အစားမတူကြဘူး (1, 3, 4, 6, 5 bytes) GDB က သူ့ဘာသာသိ

```

set disable-randomization off

checksec
r <arg1> <arg2>

restart   #restart program (pwndbg)

p/x $rbp-0x70  # stack address ကိုကြည့်

info file          # Show program sections, entry point
info functions     # List all functions
info variables     # List all variables
info registers     # Show all register values
i r eax ebx ecx     # Specific registers
info args          # Show function arguments
info locals        # Show local variables
info frame         # Show current stack frame
info sharedlibrary # Show loaded libraries
backtrace (bt)             # Function call stack


vmmap                # Memory mapping (check protections)
search "/bin/sh"     # Search string in memory
search 0xdeadbeef    # Search value
procinfo             # Process info (PID, path, etc.)
elfheader            # ELF section headers
checksec             # Security mitigations
aslr                 # Check/modify ASLR


si (go into function call)
ni (skip function call)
continue (c)
run

until *0xaddr        Run until address
finish               Run until function returns
jump *0x401234       Jump to address


set $rip
set $rax 
disas main
continue (c)
run

break *main+59
info break (i b)
delete break 1
delete 1            # Delete breakpoint 1
del 1
disable 2           # Disable breakpoint 2
enable 2            # Enable breakpoint 2
clear main          # Clear breakpoint at main

context             # Show full context (pwndbg)
context code        # Show only code
nearpc 20           # Show 20 instructions near PC

x/4xb $rbp-0x4 (memory)
x/s $rbp-0x30
x/s 0x003d3
x/10gx $rsp

pattern create
pattern offset 0x0a66a6a6a6a6a66 
cyclic 100
cyclic -l 0x0a66a6a6a6a6a66

x $ebp - 0xc
x/10x
(gdb) info symbol 0xf7e36d70 # printf in section .text of /lib/i386-linux-gnu/libc.so.6
(gdb) x/10gx $rsp             # Examine stack as 8-byte values  
(gdb) x/20bx $rsp             # Examine stack as 1-byte values


got                 # Show all GOT entries (pwndbg)
plt                 # Show all PLT stubs (pwndbg)
x/10i 0x80484c0     # Examine PLT stub
x/wx 0x804a018      # Examine GOT entry
shell objdump -R ./vuln  # Show relocations


vmmap libc
info proc all
info proc mappings

print $rsp                      # Print RSP value
print (void*)buffer     # Print buffer address
p num (print num variable)


define hook-stop

telescope 20 (tel 20)

python
import gdb
def show_stack(event):
    gdb.execute("tel 50")
gdb.events.stop.connect(show_stack)


pwndbg> python gdb.events.stop.connect(lambda event: gdb.execute("tel 50"))


set telescope-skip-repeating-val off
echo "set telescope-skip-repeating-val off" >> ~/.gdbinit


python -c "import sys; sys.stdout.buffer.write(b'\x13\xc4\x61\x56\xea\x56 %p %p %p %p %p %p')" | ./valley



```

```
Variable

# Variable declaration
int num = 123;

# View as different types
pwndbg> p num
$1 = 123

# View as hex (4 bytes, little endian)
pwndbg> x/4bx &num
0x7fffffffdb00: 0x7b 0x00 0x00 0x00

# View as 1 byte (LSB)
pwndbg> x/1bx &num
0x7fffffffdb00: 0x7b

# View as 2 bytes (short)
pwndbg> x/2hx &num
0x7fffffffdb00: 0x007b

# View as 4 bytes (int)
pwndbg> x/4wx &num
0x7fffffffdb00: 0x0000007b

# View as decimal
pwndbg> x/1dw &num
0x7fffffffdb00: 123

```
```