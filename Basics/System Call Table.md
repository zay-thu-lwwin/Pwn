

	System Call (syscall) ဆိုတာ  User Program ကနေ Linux Kernel တိုက်ရိုက်ခိုင်းတဲ့
	CPU instruction တစ်ခုဖြစ်ပါတယ် 
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

    ebx - 1st argument
    ecx - 2nd argument  
    edx - 3rd argument
    esi - 4th argument
    edi - 5th argument
    ebp - 6th argument
    
```
    
	 64-bit Linux အတွက်
 ```
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
