

### **၁။ Stack ဆိုတာဘာလဲ?**
- Stack ဆိုတာ **memory ထဲက စားပွဲတင်တစ်ခု**လိုမျိုးပါ
- စာအုပ်တွေကို တစ်အုပ်ပေါ်တစ်အုပ် တင်ထားသလိုမျိုးပေါ့
- **နောက်ဆုံးထည့်ထားတဲ့အရာကို အရင်ဆုံးထုတ်ယူရတယ်** (LIFO - Last In, First Out)

### **၂။ `PUSH` နဲ့ `POP` အလုပ်လုပ်ပုံ**

**ဥပမာ - စာအုပ်တင်တဲ့ပုံစံနဲ့ နှိုင်းယှဉ်ကြည့်ရအောင်**

```
စားပွဲပေါ်မှာ စာအုပ်တွေ တင်နေတယ်ဆိုပါစို့
```

**`PUSH` လုပ်တဲ့အခါ (ထည့်တယ်):**
```assembly
PUSH 10    ; စာအုပ် 'A' ကို စားပွဲပေါ်တင်
PUSH 20    ; စာအုပ် 'B' ကို စာအုပ် 'A' ပေါ်မှာတင်
PUSH 30    ; စာအုပ် 'C' ကို စာအုပ် 'B' ပေါ်မှာတင်
```

အခု Stack မှာ ဒီလိုဖြစ်နေပါတယ်:
```
[30]  ← ထိပ်ဆုံးက (နောက်ဆုံးထည့်ထားတဲ့စာအုပ်)
[20]
[10]  ← အောက်ဆုံးက (ပထမဆုံးထည့်ထားတဲ့စာအုပ်)
```

**`POP` လုပ်တဲ့အခါ (ထုတ်ယူတယ်):**

```assembly
POP EAX    ; ထိပ်ဆုံးကစာအုပ် 'C' ကိုယူပြီး EAX ထဲထည့်
```

ဒီလိုဖြစ်သွားပါတယ်:
- **Stack ကနေ** စာအုပ် 'C' (value 30) ကို **ထုတ်ယူလိုက်တယ်**
- **ထုတ်ယူထားတဲ့စာအုပ်ကို** EAX ဆိုတဲ့ **အိမ်** (register) ထဲ **ထည့်လိုက်တယ်**

ဒါကြောင့်:
- **`POP` = Stack ကနေ ထုတ်ယူတာ ✓**
- **ထုတ်ယူထားတဲ့ value ကို register ထဲ ထည့်ပေးတာ ✓**

### **၃။ လက်တွေ့ Code Example**

```assembly
section .data
    ; ဒီမှာ data တွေသတ်မှတ်တယ်

section .text
    global _start

_start:
    ; PUSH တွေလုပ်မယ်
    MOV EAX, 100
    PUSH EAX       ; EAX ထဲက 100 ကို stack ထဲထည့်
    
    MOV EBX, 200
    PUSH EBX       ; EBX ထဲက 200 ကို stack ထဲထည့်
    
    MOV ECX, 300
    PUSH ECX       ; ECX ထဲက 300 ကို stack ထဲထည့်
    
    ; အခု stack မှာ: [300, 200, 100]
    
    ; POP တွေစလုပ်မယ်
    POP EDX        ; Stack ထဲက 300 ကိုထုတ်ယူပြီး EDX ထဲထည့်
                   ; အခု EDX = 300
                   ; Stack မှာ: [200, 100]
    
    POP EAX        ; Stack ထဲက 200 ကိုထုတ်ယူပြီး EAX ထဲထည့်  
                   ; အခု EAX = 200
                   ; Stack မှာ: [100]
    
    POP EBX        ; Stack ထဲက 100 ကိုထုတ်ယူပြီး EBX ထဲထည့်
                   ; အခု EBX = 100
                   ; Stack မှာ: [] (ဘာမှမရှိတော့)
```

### **၄။ ဘာကြောင့် ဒီလိုလုပ်တာလဲ?**

`PUSH` နဲ့ `POP` ကို ဒီအတွက်သုံးပါတယ်:
- **Temporary storage** - ယာယီသိမ်းဆည်းဖို့
- **Function calls** - function တစ်ခုခေါ်တဲ့အခါ parameters တွေပို့ဖို့
- **Register values သိမ်းဖို့** - register တွေမပျောက်အောင် သိမ်းထားဖို့

**ဥပမာ:**
```assembly
PUSH EAX    ; EAX ထဲက value ကို stack မှာယာယီသိမ်းထား
; ... အခြားအလုပ်တွေလုပ် ...
POP EAX     ; Stack ကနေ မူလ value ကိုပြန်ထုတ်ယူပြီး EAX ထဲထည့်
```

### **၅။ အတိုချုပ်**

- **`POP` က Stack ကနေ ထုတ်ယူတာပါ** ✓
- **ထုတ်ယူထားတဲ့ value ကို register တစ်ခုခုထဲ ထည့်ပေးတာပါ** ✓
- **စာအုပ်တွေ တင်ပြီးချတာလိုမျိုးပါ** - စားပွဲပေါ်ကနေ စာအုပ်ကိုယူပြီး လက်ထဲထည့်တာနဲ့တူတူပါပဲ

ဒီလိုနားလည်အောင် စဉ်းစားကြည့်ပါ:
- **Stack = စာအုပ်တင်ထားတဲ့စားပွဲ**
- **`POP` = စားပွဲပေါ်ကနေ စာအုပ်ကိုယူပြီး လက်ထဲထည့်တာ**


----


အရမ်းကောင်းတဲ့အကြံပါ! လွယ်လွယ်ကူကူ example နဲ့ ရှင်းပြပေးပါမယ်။

## **Simple Program Structure**
```c
int another_function() {
    return 42;
}

int main() {
    int result = another_function();
    return 0;
}
```

---

## **Step 1: Main Function စချိန် (Program Start)**

**OS က main function ကို call လုပ်တဲ့အခါ:**
```assembly
; OS calls main function
RIP = 0x400500 (main function address)
RSP = 0x7fffffffe000
RBP = 0x7fffffffe010 (previous function's frame)
```

**Stack မှာ OS က return address ကိုအရင်တင်ထားပြီးသား:**
```
0x7fffffffdff8: [OS return address]  ← RSP points here
0x7fffffffdfe0: [other data...]
```

---

## **Step 2: Main Function Prologue (Stack Frame ဆောက်ခြင်း)**

```assembly
0x400500 <main+0>:  push   rbp       ; Save old RBP
0x400501 <main+1>:  mov    rbp, rsp  ; Create new stack frame
0x400504 <main+4>:  sub    rsp, 0x10 ; Allocate space for local variables
```

**After `PUSH RBP`:**
```
RBP = 0x7fffffffe010 (old value - unchanged)
RSP = 0x7fffffffdff0 (decreased by 8)

Stack:
0x7fffffffdff0: [0x7fffffffe010]    ← RSP points here (saved old RBP)
0x7fffffffdff8: [OS return address]
```

**After `MOV RBP, RSP`:**
```
RBP = 0x7fffffffdff0 (now points to main's frame base)
RSP = 0x7fffffffdff0 (unchanged)
```

**After `SUB RSP, 0x10`:**
```
RBP = 0x7fffffffdff0
RSP = 0x7fffffffdfe0 (decreased by 16 bytes for local variables)

Stack:
0x7fffffffdfe0: [uninitialized]     ← RSP points here (space for local vars)
0x7fffffffdfe8: [uninitialized]
0x7fffffffdff0: [0x7fffffffe010]    ← RBP points here (main's frame base)
0x7fffffffdff8: [OS return address]
```

---

## **Step 3: Main Function က Another Function ကိုခေါ်ခြင်း**

```assembly
0x400508 <main+8>:  call   0x400520 <another_function>
```

**Before CALL:**
```
RIP = 0x400508 (about to call)
RSP = 0x7fffffffdfe0
```

**After CALL (auto PUSH return address):**
```
RIP = 0x400520 (jumped to another_function)
RSP = 0x7fffffffdfd8 (decreased by 8)

Stack:
0x7fffffffdfd8: [0x40050d]          ← RSP points here (return to main+13)
0x7fffffffdfe0: [uninitialized]     
0x7fffffffdfe8: [uninitialized]
0x7fffffffdff0: [0x7fffffffe010]    ← RBP points here (main's frame base)
0x7fffffffdff8: [OS return address]
```

---

## **Step 4: Another Function စချိန်**

```assembly
0x400520 <another_function+0>: push   rbp
0x400521 <another_function+1>: mov    rbp, rsp
```

**After `PUSH RBP` in another_function:**
```
RBP = 0x7fffffffdff0 (main's frame base - unchanged)
RSP = 0x7fffffffdfd0 (decreased by 8)

Stack:
0x7fffffffdfd0: [0x7fffffffdff0]    ← RSP points here (saved main's RBP)
0x7fffffffdfd8: [0x40050d]          (return to main+13)
0x7fffffffdfe0: [uninitialized]     
0x7fffffffdfe8: [uninitialized]
0x7fffffffdff0: [0x7fffffffe010]    ← main's RBP still here
0x7fffffffdff8: [OS return address]
```

**After `MOV RBP, RSP`:**
```
RBP = 0x7fffffffdfd0 (now points to another_function's frame base)
RSP = 0x7fffffffdfd0 (unchanged)
```

---

## **Step 5: Another Function ပြီးချိန် (Return to Main)**

```assembly
0x400525 <another_function+5>: mov    eax, 42    ; return value
0x40052a <another_function+10>: pop    rbp
0x40052b <another_function+11>: ret
```

**After `POP RBP`:**
```
RBP = 0x7fffffffdff0 (restored to main's frame base)
RSP = 0x7fffffffdfd8 (increased by 8)

Stack:
0x7fffffffdfd8: [0x40050d]          ← RSP points here (return address)
0x7fffffffdfe0: [uninitialized]     
0x7fffffffdfe8: [uninitialized]
0x7fffffffdff0: [0x7fffffffe010]    ← RBP points here (back to main's frame)
0x7fffffffdff8: [OS return address]
```

**After `RET` (auto POP to RIP):**
```
RIP = 0x40050d (back to main+13)
RSP = 0x7fffffffdfe0 (increased by 8)

Stack:
0x7fffffffdfe0: [uninitialized]     ← RSP points here
0x7fffffffdfe8: [uninitialized]
0x7fffffffdff0: [0x7fffffffe010]    ← RBP points here
0x7fffffffdff8: [OS return address]
```

---

## **Step 6: Main Function ပြီးချိန်**

```assembly
0x40050d <main+13>: mov    eax, 0    ; return 0
0x400512 <main+18>: leave
0x400513 <main+19>: ret
```

**`LEAVE` instruction = `MOV RSP, RBP` + `POP RBP`**

**After `MOV RSP, RBP`:**
```
RSP = 0x7fffffffdff0 (same as RBP)
RBP = 0x7fffffffdff0 (unchanged)
```

**After `POP RBP` (part of LEAVE):**
```
RBP = 0x7fffffffe010 (restored to original)
RSP = 0x7fffffffdff8 (increased by 8)

Stack:
0x7fffffffdff8: [OS return address]  ← RSP points here
```

**After `RET` in main:**
```
RIP = [OS return address] (back to operating system)
RSP = 0x7ffffffff000 (increased by 8, back to original)
```

---

## **Summary - Stack Frame Lifecycle**

### **Main Function Start:**
```
RBP = old_value
RSP points to: [OS return address]
```

### **Main Function Active:**
```
RBP = main_frame_base (0x7fffffffdff0)
RSP points to: [local variables]
Stack has: [locals][main_frame_base][OS_return]
```

### **Another Function Called:**
```
RBP = another_frame_base (0x7fffffffdfd0) 
RSP points to: [another_frame_base][return_to_main][locals][main_frame_base][OS_return]
```

### **Another Function Returns:**
```
RBP = main_frame_base (0x7fffffffdff0) ← restored!
RSP points to: [locals]
Stack has: [locals][main_frame_base][OS_return] ← clean again!
```

### **Main Function Ends:**
```
RBP = old_value ← restored!
RSP = original_position
Stack clean!
```

**အဓိကအချက်:** ဘယ် function မဆို စချိန်မှာ `PUSH RBP` လုပ်ပြီး၊ ပြီးချိန်မှာ `POP RBP` လုပ်ရမယ်။ ဒါမှ stack က balance ဖြစ်မယ်!

https://chat.deepseek.com/a/chat/s/c483c576-0e48-46f3-a613-43aff487d839
https://chat.deepseek.com/share/m8mp37i7zca30b76d1

---

- **Stack ကိုယ်တိုင်က** high address → low address **ကြီးးတယ်**
    
- **Data within stack က** low address → high address **စီးတယ်**
    
- **Return address overwrite က** buffer ပြည့်ပြီး RBP overwrite ပြီးမှ **ဖြစ်တယ်**

