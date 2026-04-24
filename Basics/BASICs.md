

# Compiling

```
 compile
gcc program.c -o program

# အဆင့်ဆင့် ခွဲကြည့်ချင်ရင်
gcc -c program.c          # object file (.o) ထုတ်
gcc program.o -o program  # linking လုပ်

```

```
# 32-bit compile လုပ်ချင်ရင် -m32 ထည့်ရမယ်
gcc -m32 program.c -o program32

# 32-bit with disabled security
gcc -m32 -no-pie -fno-stack-protector program.c -o program32
```

##### Stack Protection

| Option | Effect | Default (gcc) |
|--------|--------|----------------|
| `-fstack-protector` | canary ထည့် (function prologue/epilogue) | OFF for 32-bit, ON for 64-bit |
| `-fstack-protector-strong` | local array/buffer ရှိတဲ့ function အားလုံး canary ထည့် | RECOMMENDED |
| `-fstack-protector-all` | function တိုင်းမှာ canary ထည့် | Overkill |
| `-fno-stack-protector` | canary လုံးဝမထည့် | အန္တရာယ်များ |

```bash
# အကောင်းဆုံး (CTF/pwn အတွက် disable)
gcc -fno-stack-protector program.c -o program

# production အတွက်
gcc -fstack-protector-strong program.c -o program
```

---

##### Position Independent Executable (PIE)

| Option | Effect |
|--------|--------|
| `-fPIE -pie` | ASLR ကိုအသုံးချ - binary load address randomize |
| `-no-pie` | fixed memory address (0x400000) |

```bash
# PIE enabled (ASLR သက်ရောက်)
gcc -fPIE -pie program.c -o program

# PIE disabled (static address - easy to ROP)
gcc -no-pie program.c -o program
```

---

##### RELRO (Relocation Read-Only)

| Option | Effect |
|--------|--------|
| `-Wl,-z,relro` | GOT အပိုင်းကို read-only ပြုလုပ် |
| `-Wl,-z,now` | binding ကို runtime မှာချက်ချင်းလုပ် |
| Full RELRO | `-Wl,-z,relro -Wl,-z,now` |

```bash
gcc -Wl,-z,relro -Wl,-z,now program.c -o program   # Full RELRO
gcc -Wl,-z,norelro program.c -o program            # No RELRO (GOT overwrite ခံနိုင်)
```

---

##### Non-Executable Stack (NX)

| Option | Effect |
|--------|--------|
| `-z noexecstack` | stack မှ code execute မလုပ်ရ |
| `-z execstack` | stack က executable (old school) |

```bash
gcc -z noexecstack program.c -o program   # DEFAULT on modern GCC
gcc -z execstack program.c -o program     # shellcode execution အတွက်
```

##### Fortify Source

| Option | Effect |
|--------|--------|
| `-D_FORTIFY_SOURCE=1` | basic buffer overflow check |
| `-D_FORTIFY_SOURCE=2` | stricter checking (recommended) |
| `-D_FORTIFY_SOURCE=0` | disabled |

```bash
gcc -D_FORTIFY_SOURCE=2 -O1 program.c -o program
```

##### ASLR Influence 
```bash
# ASLR disable (pwn အတွက်)
setarch `uname -m` -R ./program

# ASLR enable default အတိုင်း
./program
```

##### Complete Security Hardened Compilation (Production)

```bash
gcc -o program program.c \
    -fstack-protector-strong \
    -D_FORTIFY_SOURCE=2 \
    -Wl,-z,relro -Wl,-z,now \
    -z noexecstack \
    -fPIE -pie \
    -O2 \
    -Wformat -Wformat-security
```

#####  Complete Disabled Security (For Pwn/CTF Practice)

```bash
gcc -o program program.c \
    -fno-stack-protector \
    -z execstack \
    -no-pie \
    -Wl,-z,norelro \
    -m32 \
    -g
```

**Explanation:**
- `-fno-stack-protector` → no canary
- `-z execstack` → stack executable (shellcode)
- `-no-pie` → fixed address (easy ROP)
- `-Wl,-z,norelro` → GOT writable
- `-m32` → 32-bit (easier to exploit)
- `-g` → debug symbols

---

