
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


```

pwndbg> set disable-randomization off
pwndbg> break main
pwndbg> run

info funcions
info registers
info frame 
si (go into function call)
ni (skip function call)
set $rip
set $rax
disas main
continue
break *main+59
info break (i b)
delete break 1
del 1
run
info registers eax (register)
x/4xb $rbp-0x4 (memory)
x/s $rbp-0x30
x/s 0x003d3
x/10gx $rsp
pattern create
pattern offset 0x0a66a6a6a6a6a66 
x $ebp - 0xc
x/10x
(gdb) info symbol 0xf7e36d70
# printf in section .text of /lib/i386-linux-gnu/libc.so.6
telescope 20 (tel 20)
vmmap libc
info proc all
info proc mappings



py exec("import gdb; gdb.execute('ni'); gdb.execute('telescope 20'); gdb.execute('x/30wx 0x804d1a0 - 16')")

command
define hook-stop

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
(gdb) info registers           # All registers
(gdb) x/10gx $rsp             # Examine stack as 8-byte values  
(gdb) x/20bx $rsp             # Examine stack as 1-byte values
(gdb) info frame              # Current stack frame info
(gdb) backtrace               # Function call stack
(gdb) stepi                   # Single instruction step
(gdb) nexti                   # Step over function calls
(gdb) print $rsp              # Print RSP value
(gdb) print (void*)buffer     # Print buffer address
 p num (print num variable)


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