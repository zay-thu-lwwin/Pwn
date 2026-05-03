


```
User က ./program ကို run
↓
Kernel က process တစ်ခု create
↓
ld.so (dynamic linker) က libc တင်သုံး  
↓
libc ရဲ့ __libc_start_main() က canary generate
↓
main() function ကိုခေါ်
```

```
# Kernel က entropy sources တွေကနေ random data ထုတ်
# - Mouse movements
# - Keyboard timings  
# - Disk I/O timings
# - Interrupt timings

# /dev/urandom ထဲမှာ သိမ်းထား
$ cat /proc/sys/kernel/random/entropy_avail
2564  # Available random bits
```

```
# TLS (Thread Local Storage) setup
# fs:0x28 မှာ canary သိမ်း

mov    rax, QWORD PTR fs:0x28  # canary ကိုယူ
mov    QWORD PTR [rbp-0x8], rax  # stack ပေါ်တင်
```

Canary တန်ဖိုးက random value တစ်ခုဖြစ်ပြီး program start လုပ်တည်က generate လုပ်ထားတာ (ဥပမာ - global variable `__stack_chk_guard` ထဲမှာ)
Function စခေါ်တဲ့အခါ မှာ saved `ebp` ပြီးရင် canary value ကိုထည့်ထားတယ်
Function ပြီးသွားရင် canary value ပျက်သွားလား stack ပေါ်ကနေ ပြန်ဖတ်ပြီး global variable(original canary (global/thread-local)) နဲ့ နှိုင်းယှဉ်တယ်
Value ပြောင်းသွားရင် program က crash ဖြစ် (Stack Smashing Detected!)
canary value က program run time တစ်ခုမှာ ` function တိုင်းအတွက်တူတယ်`
Canary က buffer overflow ဖြစ်နိုင်တဲ့ function တွေမှာပဲ compiler က auto ထည့်ပေး




### Bypass 

	Canary value ကို leak လုပ်နိုင်ရင် 
	stack canary protection ကို bypass လုပ်လို့ရ

	Technique 1
	prinf vuln နဲ့ canary value ကိုဖတ်ယူ
	Buffer overflow လုပ်ရင် မပျက်အောင် အဲ့တန်ဖိုးကိုပြန်ထည့် 


---

-3423


**Terminal canary (glibc 2.23+ မှာ):**

- Fork-based server တွေမှာ (ဥပမာ - web server child processes) child တိုင်း canary ပြောင်းသွားနိုင်တယ်။
    
- ဒါပေမဲ့ parent process က fork လုပ်တဲ့အချိန်မှာ canary value ကို copy လုပ်လိုက်တာမို့ child process တွေအတွင်းမှာ တူနေသေးတယ်။