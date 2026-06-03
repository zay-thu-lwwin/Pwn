
|အဆင့်|ဘာဖြစ်လဲ|
|---|---|
|1|GDB attach လုပ်တယ်၊ breakpoint နှစ်ခုထားတယ်|
|2|`continue` ကြောင့် program စပြီး run တယ်|
|3|Program က main() ကိုရောက်တယ် → **breakpoint #1 ကိုထိတယ်**|
|4|**Program ရပ်သွားတယ်** (pause)|
|5|GDB prompt ပေါ်လာတယ် `(gdb)`|
|6|သင်က `info reg`, `x/20gx $rsp` စတာတွေ ရိုက်စစ်လို့ရ|
|7|`continue` (GDB command) ရိုက်မှ program ဆက်သွားမယ်|
|8|Program က `0x401234` ကိုရောက်ရင် → **breakpoint #2 ထပ်ရပ်မယ်**|
|9|ထပ်ပြီး debug လုပ်လို့ရ|


Terminal နှစ်ခုလုံးမှာ Program က `main` မှာ မရပ်သွားရတဲ့ အဓိက အကြောင်းရင်းက **Race Condition** (အပြိုင်အဆိုင် ဖြစ်သွားခြင်း) ကြောင့်ပါ။

### ဘာကြောင့်ဖြစ်တာလဲ?

မင်းရဲ့ Python script ထဲမှာ `conn = process("./valley")` လို့ ရေးလိုက်တာနဲ့ Program က စတင် Run နေပါပြီ။ `gdb.attach(conn)` က နောက်မှ လိုက်လာတာပါ။

1. **Program က မြန်မြန် Run သွားတယ်:** Python script က GDB ကို attach လုပ်ဖို့ ပြင်ဆင်နေတုန်းမှာပဲ Binary program က `main` function ကို ဖြတ်ကျော်ပြီး `fgets` (သို့မဟုတ် input စောင့်တဲ့နေရာ) ကို ရောက်သွားနှင့်တာပါ။
    
2. **Breakpoints အလုပ်မလုပ်တော့ခြင်း:** Program က `main` ကို ကျော်သွားပြီးမှ GDB က ဝင်လာတာ ဖြစ်တဲ့အတွက် `b *main` ဆိုတဲ့ breakpoint က ဘယ်တော့မှ မိတော့မှာ မဟုတ်ပါဘူး (Program က main ထဲကို နောက်တစ်ခါ ပြန်မဝင်တော့လို့ပါ)။
    
3. **Syscall မှာ ပိတ်မိနေခြင်း:** မင်းတွေ့ရတဲ့ `__internal_syscall_cancel` ဆိုတာ GDB က ဝင်ကြည့်လိုက်တဲ့အချိန်မှာ Program က input ကို စောင့်နေတဲ့ (Sleep ဖြစ်နေတဲ့) အခြေအနေပါ။
    

---

### ဖြေရှင်းနည်း (၂) မျိုး

#### နည်းလမ်း (၁) - `gdb.debug` ကို သုံးပါ (အကောင်းဆုံးနည်းလမ်း)

`gdb.debug` ကို သုံးရင် Pwntools က Program ကို စကတည်းက GDB အောက်မှာ တင်ပေးမှာဖြစ်လို့ `main` မှာ ရပ်ဖို့ သေချာပါတယ်။

Python

```
from pwn import *

context.binary = elf = ELF("./valley")
context.terminal = ['tmux', 'splitw', '-h'] # ဒါမှမဟုတ် မင်းသုံးနေတဲ့ terminal type

# process နေရာမှာ gdb.debug ကို အစားထိုးပါ
conn = gdb.debug("./valley", gdbscript='''
    break main
    continue
''')lo

# ကျန်တဲ့ code တွေ ဆက်ရေးပါ...
```

#### နည်းလမ်း (၂) - `pause()` သုံးပြီး အချိန်ဆွဲပါ

`gdb.attach` ကို သုံးချင်သေးတယ်ဆိုရင် GDB ကောင်းကောင်း attach ဖြစ်သွားအောင် ခေတ္တ စောင့်ခိုင်းတဲ့ နည်းလမ်းပါ။

Python

```
conn = process("./valley")

gdb.attach(conn, '''
    break main
    continue
''')

pause() # ဒီနေရာမှာ script က ခဏရပ်နေပါမယ်။ GDB window ပွင့်လာပြီး breakpoint ချပြီးမှ keyboard က key တစ်ခုခု နှိပ်လိုက်ပါ။

conn.sendline(b'%20$lx')
# ...
```

### အကြံပြုချက်

မင်းရဲ့ လက်ရှိ GDB output ထဲမှာ `__internal_syscall_cancel` ရောက်နေပြီဆိုရင် Program က input ကို စောင့်နေတာ ဖြစ်တဲ့အတွက်:

1. GDB terminal ထဲမှာ **`c`** (continue) ကို အရင်နှိပ်ပါ။
    
2. အဲ့ဒီနောက်မှ Python script run နေတဲ့ terminal ထဲမှာ `Enter` သို့မဟုတ် input ပို့တာမျိုး လုပ်ပါ။
    

ဒါပေမဲ့ exploit ရေးတဲ့အခါ အဆင်အပြေဆုံးကတော့ **နည်းလမ်း (၁)** ဖြစ်တဲ့ `gdb.debug` ကို သုံးတာပါပဲ။ သူက automatic breakpoint တွေ အကုန်လုံးကို program မစခင် ကြိုတင်စီမံပေးပါတယ်။