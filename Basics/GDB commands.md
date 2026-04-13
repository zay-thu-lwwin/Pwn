
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
info proc mappings

x/[COUNT][FORMAT][SIZE]

py exec("import gdb; gdb.execute('ni'); gdb.execute('telescope 20'); gdb.execute('x/30wx 0x804d1a0 - 16')")

command
define hook-stop





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
