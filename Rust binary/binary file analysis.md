

---

## 📚 Part 1: Rust Binary တွေရဲ့ ထူးခြားချက်များ

### 1.1 Rust Symbol Naming Convention

```bash
# Raw symbols (mangled)
_ZN3std2rt10lang_start17h5c4d2e9d1e8f2e1aE

# Demangled (readable)
std::rt::lang_start
```

**Demangling tools:**
```bash
# Install rustfilt
cargo install rustfilt

# Demangle specific symbol
echo "_ZN3std2rt10lang_start17h5c4d2e9d1e8f2e1aE" | rustfilt

# Demangle entire binary
nm tictactoe | rustfilt

# Using objdump with demangling
objdump -d --demangle=rust tictactoe | less
```

### 1.2 Binary Structure Analysis

```bash
# Check section headers (Rust-specific sections)
readelf -S tictactoe | grep -E "rust|debug"

# Common Rust sections:
# .rustc          - Compiler version info
# .debug_rust     - Debug info
# .text.rust      - Rust code section
# .rodata.rust    - Rust constants

# Check compiler version
strings tictactoe | grep -i "rustc version"
# Output: rustc version 1.70.0 (90c541806 2023-05-31)
```

---

## 🔧 Part 2: Static Analysis Techniques

### 2.1 Ghidra Setup for Rust

**Ghidra မှာ Rust binary analyze လုပ်နည်း**

```bash
# 1. Install Ghidra Rust plugin
git clone https://github.com/rust-lang/ghidra-rust.git
# Copy to Ghidra/Extensions

# 2. Load binary, select "Rust" language
# 3. Run analysis with "Rust Demangler" script
```

**Ghidra မှာ လုပ်သင့်တဲ့ steps:**
1. File → Open → Select binary
2. Language: x86:LE:64:default:rust
3. Analysis → One Shot → Rust Demangler
4. Search → Symbol Tree → Look for `tictactoe::*`

### 2.2 Binary Ninja (Better for Rust)

```python
# Binary Ninja Rust analysis script
import binaryninja as bn

def analyze_rust_binary(bv):
    # Find all Rust functions
    for func in bv.functions:
        if "::" in func.name:
            print(f"Rust function: {func.name}")
            
    # Find unsafe blocks
    unsafe_refs = bv.get_symbols_by_name("unsafe")
    
    # Find string literals (Rust stores differently)
    for section in bv.sections:
        if ".rodata" in section:
            data = bv.read(section.start, section.end - section.start)
            strings = extract_strings(data)
```

### 2.3 IDA Pro Tips

```python
# IDA Python for Rust
import idaapi
import idautils

# Demangle all functions
for ea in idautils.Functions():
    name = idaapi.get_func_name(ea)
    demangled = rust_demangle(name)
    if demangled != name:
        idaapi.set_name(ea, demangled, idaapi.SN_NOWARN)

# Find trait implementations
# Pattern: <Type as Trait>::function
for name in idautils.Names():
    if " as " in name[1] and ">::" in name[1]:
        print(f"Trait impl: {name[1]}")
```

---

## 🐛 Part 3: Dynamic Analysis (GDB/pwndbg)

### 3.1 GDB Setup for Rust

```gdb
# ~/.gdbinit
set print demangle on
set demangle-style rust

# pwndbg with Rust support
source ~/pwndbg/gdbinit.py

# Custom Rust pretty printers
python
import gdb
class RustStringPrinter:
    """Print Rust String"""
    def __init__(self, val):
        self.val = val
    def to_string(self):
        return self.val['vec']['buf']['ptr'].string()
    def display_hint(self):
        return 'string'

gdb.printing.add_printer('Rust', '^alloc::string::String$', RustStringPrinter)
end
```

### 3.2 Debugging Rust Functions

```gdb
# Start debugging
gdb ./tictactoe

# Break on Rust main
break 'tictactoe::main'
run

# Inspect Rust strings
p/x *$rdi           # String pointer (Rust uses pointer+length)
x/s $rdi            # String content

# Inspect Vec<T>
p/x *$rsi           # Vec { ptr, len, cap }

# Print backtrace with Rust symbols
backtrace
info frame

# Watch for panics
break rust_begin_unwind
break core::panicking::panic
```

### 3.3 Memory Layout Analysis

```gdb
# Find all Rust allocations
info registers
vmmap               # pwndbg command

# Check stack layout
stack 20
hexdump $rsp 64

# Heap analysis (Rust uses jemalloc by default)
arena               # pwndbg heap commands
```

---

## 🎯 Part 4: Identifying Vulnerabilities

### 4.1 Finding Unsafe Blocks

```bash
# Method 1: Look for unsafe function names
nm tictactoe | grep -i "unsafe"

# Method 2: Search for unsafe patterns in disassembly
objdump -d tictactoe | grep -B5 -A5 "call.*\<(memcpy|memmove|strcpy)\>"

# Method 3: Look for raw pointer operations
objdump -d tictactoe | grep -E "mov.*\[.*\], |mov.*, \[.*\]" | head -20
```

### 4.2 Finding Integer Operations

```bash
# Look for arithmetic with potential overflow
objdump -d tictactoe | grep -E "add|sub|mul|imul" | grep -v "rsp\|rbp"

# Check for unchecked operations
objdump -d tictactoe | grep -B3 "unchecked"
```

### 4.3 Finding Panic Handlers (Information Leak)

```bash
# Find panic strings
strings tictactoe | grep -i "panicked"

# Find panic locations
objdump -d tictactoe | grep -A10 "panic"
```

---

## 🔬 Part 5: Advanced Analysis Techniques

### 5.1 Control Flow Graph Extraction

```python
# Using angr for Rust CFG
import angr

proj = angr.Project('./tictactoe', 
                    auto_load_libs=False,
                    main_opts={'backend': 'blob'})

cfg = proj.analyses.CFGFast()
for func in cfg.functions.values():
    if 'tictactoe' in func.name:
        print(f"{func.name}: {func.addr:#x}")
        for block in func.blocks:
            print(f"  Block: {block.addr:#x}-{block.addr+block.size:#x}")
```

### 5.2 Finding ROP Gadgets (Rust Binary)

```bash
# Use ROPgadget for Rust binary
ROPgadget --binary tictactoe | grep "pop rdi"

# Rust-specific gadget patterns
# 1. ret gadgets for unwinding
# 2. push/pop for calling conventions
# 3. libc gadgets if dynamically linked

# Custom Rust gadget finder
python << EOF
from pwn import *
elf = ELF('./tictactoe')
gadgets = elf.search(asm('pop rdi; ret'))
for g in gadgets:
    print(f"pop rdi; ret at {hex(g)}")
EOF
```

### 5.3 String Extraction (Rust Encoding)

```python
# Rust strings are UTF-8 encoded with length prefix
def extract_rust_strings(data):
    strings = []
    i = 0
    while i < len(data) - 8:
        # Check for length prefix pattern
        length = struct.unpack('<Q', data[i:i+8])[0]
        if 1 <= length <= 1000:
            # Verify valid UTF-8
            try:
                s = data[i+8:i+8+length].decode('utf-8')
                if s.isprintable():
                    strings.append((i, s))
            except:
                pass
        i += 1
    return strings
```

---

## 🛠️ Part 6: Automated Analysis Tools

### 6.1 Custom IDAPython Script

```python
import idaapi
import idautils

class RustAnalyzer(idaapi.plugin_t):
    flags = 0
    comment = "Rust Binary Analyzer"
    
    def init(self):
        self.demangle_all_functions()
        self.find_trait_implementations()
        self.analyze_panic_handlers()
        return idaapi.PLUGIN_KEEP
    
    def demangle_all_functions(self):
        for ea in idautils.Functions():
            name = idaapi.get_func_name(ea)
            if "::" in name and "_ZN" not in name:
                # Already demangled
                continue
            demangled = rust_demangle(name)
            if demangled != name:
                idaapi.set_name(ea, demangled, idaapi.SN_NOWARN)
    
    def find_trait_implementations(self):
        for name in idautils.Names():
            if " as " in name[1] and ">::" in name[1]:
                print(f"[Trait] {name[1]} at {hex(name[0])}")
    
    def analyze_panic_handlers(self):
        for ea in idautils.Functions():
            name = idaapi.get_func_name(ea)
            if "panic" in name.lower():
                print(f"[Panic] {name} at {hex(ea)}")
                # Analyze panic messages
                self.extract_panic_strings(ea)

def rust_demangle(mangled):
    # Call rustfilt or implement demangling
    import subprocess
    result = subprocess.run(['rustfilt'], 
                           input=mangled.encode(),
                           capture_output=True)
    return result.stdout.decode().strip()
```

### 6.2 Frida Hooking for Rust

```javascript
// Rust function hooking with Frida
Interceptor.attach(Module.findExportByName(null, "tictactoe::decrypt_key"), {
    onEnter: function(args) {
        console.log("[*] decrypt_key called");
        console.log("    key_ptr: " + args[0]);
        console.log("    len: " + args[1]);
        
        // Read Rust string (ptr + length)
        var key = Memory.readUtf8String(args[0], args[1].toInt32());
        console.log("    Key: " + key);
    },
    onLeave: function(retval) {
        console.log("[*] decrypt_key returned: " + retval);
    }
});

// Hook allocation functions
Interceptor.attach(Module.findExportByName(null, "__rust_alloc"), {
    onEnter: function(args) {
        console.log(`[Alloc] size=${args[0]}, align=${args[1]}`);
        this.size = args[0];
    },
    onLeave: function(retval) {
        console.log(`[Alloc] returns ${retval} (size=${this.size})`);
    }
});
```

---

## 📊 Part 7: Practical Checklist

### Pre-analysis (5 min)
```bash
# 1. Basic info
file tictactoe
checksec tictactoe
strings tictactoe | head -50

# 2. Rust identification
strings tictactoe | grep -E "rustc|alloc::|core::"
readelf -p .comment tictactoe

# 3. Dependencies
ldd tictactoe
objdump -T tictactoe | head -20
```

### Static Analysis (30 min)
```bash
# 4. Function listing
nm --demangle=rust tictactoe | grep "tictactoe::"
objdump -t --demangle=rust tictactoe | grep "F .text"

# 5. Find interesting functions
objdump -d --demangle=rust tictactoe | grep -A20 "tictactoe::"

# 6. String extraction
strings -n 8 tictactoe | grep -v "^$" | sort -u
```

### Dynamic Analysis (ongoing)
```gdb
# 7. Runtime debugging
gdb ./tictactoe
(gdb) break tictactoe::main
(gdb) run
(gdb) info functions tictactoe::
(gdb) x/100i $rip
```

---

## 🎓 Part 8: Common Patterns in Rust Binaries

### Pattern 1: String Handling
```rust
// Rust stores strings as {ptr, len, cap}
// In memory: [0x00: ptr][0x08: len][0x10: cap]
```

### Pattern 2: Vector Operations
```rust
// Vec<T> is {ptr, len, cap}
// grow_one() appears when capacity exceeded
```

### Pattern 3: Option/Result
```rust
// Option<T>: 0 = None, 1 = Some
// Result<T,E>: 0 = Ok, 1 = Err
```

### Pattern 4: Trait Dispatch (VTable)
```rust
// Dynamic dispatch uses vtables
// Pattern: call qword ptr [rax+offset]
```

---

## 🚀 Quick Reference Commands

```bash
# Demangling
nm tictactoe | rustfilt
objdump -d --demangle=rust tictactoe
gdb -ex "set demangle-style rust" -ex "file tictactoe"

# Analysis
readelf -p .rustc tictactoe          # Compiler info
objdump -s -j .rodata tictactoe      # Constants
strings -t x tictactoe | grep -i "error"  # Error messages

# Patching
echo "patch offset" | xxd -r - tictactoe
objcopy --only-section=.text tictactoe text.bin
```

---

## 💡 Pro Tips

1. **Rust binaries are FAT** - 5-50MB is normal, don't panic
2. **Panic = crash** - Rust panics on errors, may leak info
3. **No NULL pointers** - Rust uses Option<T> instead
4. **Unsafe = vulnerable** - Focus on unsafe blocks
5. **Standard library is static** - But libc is dynamic (hybrid linking)

**မင်းရဲ့ tictactoe binary အတွက် နောက်တစ်ဆင့်:** Ghidra မှာ binary ဖွင့်ပြီး `tictactoe::main` ကို decompile လုပ်ကြည့်မလား။ ဒါမှမဟုတ် gdb နဲ့ live debugging လုပ်ကြည့်ချင်ရင်လည်း ရပါတယ်။ ဘာလေ့လာချင်လဲ။