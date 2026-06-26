
##### Circular Linked List flow

Singly Linked List သို့မဟုတ် Doubly Linked List တွေမှာ နောက်ဆုံး Node ရဲ့ `Next` Pointer ဟာ ဘာမှမရှိတဲ့အတွက် `NULL` ကို ညွှန်ပြလေ့ရှိပါတယ်။
ဒါပေမဲ့ **Circular Linked List** မှာတော့:
- နောက်ဆုံး Node (Tail Node) ရဲ့ `Next` Pointer ဟာ `NULL` ကို မညွှန်ပြတော့ဘဲ **ပထမဆုံး Node (Head Node)** ကို ပြန်ပြီး ချိတ်ဆက်ထားပါတယ်။
- ဒါကြောင့်မို့လို့ List ရဲ့ အဆုံးကိုရောက်သွားရင်လည်း နောက်တစ်ဆင့် ထပ်သွားတဲ့အခါ အစကနေ ပြန်စပြီး ပတ်ချာလည် (Circle ပုံစံ) အလုပ်လုပ်နေမှာ ဖြစ်ပါတယ်။
    

##### Types of Circular Linked List)

Circular Linked List ကို အဓိကအားဖြင့် နှစ်မျိုး ခွဲခြားနိုင်ပါတယ်

 Circular Singly Linked List
Singly Linked List အပေါ်မှာ အခြေခံထားတာ ဖြစ်ပါတယ်။ Node တစ်ခုချင်းစီမှာ Pointer တစ်ခုစီပဲ ပါဝင်ပြီး နောက်ဆုံး Node က ပထမဆုံး Node ကို ပြန်ချိတ်ထားပါတယ်။ (အရှေ့ဘက် တစ်လမ်းမောင်းပဲ သွားလို့ရပါတယ်)။

 Circular Doubly Linked List
Doubly Linked List အပေါ်မှာ အခြေခံထားတာ ဖြစ်ပါတယ်။ Node တစ်ခုချင်းစီမှာ `Prev` ရော `Next` ရော ပါဝင်ပါတယ်။ နောက်ဆုံး Node ရဲ့ `Next` က ပထမဆုံး Node ကို ညွှန်ပြထားသလို၊ ပထမဆုံး Node ရဲ့ `Prev` ကလည်း နောက်ဆုံး Node ကို ပြန်ညွှန်ပြထားပါတယ်။ (အရှေ့၊ အနောက် နှစ်ဖက်လုံး ပတ်ချာလည် သွားလို့ရပါတယ်)။

 အားသာချက်များ (Advantages)

- **ဘယ် Node ကနေမဆို တစ်ပတ်ပတ်လို့ရခြင်း:** သာမန် Linked List မှာ Head ကနေပဲ စလို့ရပေမဲ့ Circular မှာတော့ ကိုယ်ကြိုက်တဲ့ Node ကနေစပြီး တစ်ပတ်ပြည့်အောင် လှည့်ပတ်ကြည့်ရှု (Traverse) လို့ ရပါတယ်။
    
- **Queue Data Structure အတွက် ကောင်းမွန်ခြင်း:** အစနဲ့ အဆုံးကို အမြဲတမ်း ပတ်ချာလည် ချိတ်ထားချင်တဲ့ နေရာတွေ (ဥပမာ- Round Robin Scheduling) အတွက် အလွန်အသုံးဝင်ပါတယ်။
    

 အားနည်းချက်များ (Disadvantages)

- **Infinite Loop ဖြစ်နိုင်ခြင်း:** ကုဒ်ရေးတဲ့အခါ သေချာမထိန်းရင် (ဥပမာ- `while(current != NULL)` လို့ ရေးမိရင်) `NULL` ဆိုတာ မရှိတဲ့အတွက် ကုဒ်က ရပ်မသွားဘဲ တစ်သက်လုံး ပတ်ချာလည်ပြီး Loop ပတ်နေတတ်ပါတယ်။
    
- **ကုဒ်ရေးရတာ ပိုမိုနက်နဲခြင်း:** Node အသစ်ထည့်တာ၊ ဖျက်တာတွေလုပ်တဲ့အခါ နောက်ဆုံး Node ရဲ့ Pointer ကိုပါ အမြဲတမ်း လိုက်ပြင်ပေးနေရလို့ သာမန် Linked List ထက် ပိုသတိထားရပါတယ်။
    

##### Real-world Applications)

- **Operating System Scheduling (Round Robin):** OS တစ်ခုကနေ CPU ပေါ်မှာ Process တွေကို တစ်ခုပြီးတစ်ခု အချိန်ပိုင်းစီ ခွဲဝေပေးတဲ့အခါ (ဥပမာ- Process A ကို 2ms၊ ပြီးရင် Process B ကို 2ms၊ ပြီးရင် Process C၊ ပြီးရင် Process A ကို ပြန်စ) စတဲ့နေရာမျိုးမှာ သုံးပါတယ်။
- **Multiplayer Games:** ကစားသမား ၄ ယောက် အလှည့်ကျ ဒါန တိုက်ခိုက်ရတဲ့ ဂိမ်းတွေမှာ (ဥပမာ- Player 1 -> Player 2 -> Player 3 -> Player 4 -> Player 1) ဆိုပြီး အလှည့်ကျ စနစ်အတွက် အသုံးပြုပါတယ်။
- **PC ရဲ့ Alt + Tab စနစ်:** Windows မှာ Alt + Tab နှိပ်ပြီး ဖွင့်ထားတဲ့ App တွေကို လှည့်ပတ်ရွေးချယ်တဲ့ နေရာမှာလည်း အဆုံးထိရောက်သွားရင် အစကနေ ပြန်ပတ်တဲ့ စနစ်မျိုးအတွက် အသုံးပြုပါတယ်။



အောက်ပါ code ကတော့ Circular Linked List ထဲကို data အသစ်ထည့်သွင်းတဲ့ `insertEnd` function ဖြစ်ပါတယ်။ (List ရဲ့ အဆုံးမှာ ထည့်မှာဖြစ်ပေမဲ့ သူက Circular ဖြစ်လို့ `head` ကို ပြန်ညွှန်ပေးရမှာပါ)


```c
#include <stdio.h>
#include <stdlib.h>

struct Node {
    int data;
    struct Node* next;
};

// Circular List ရဲ့ အဆုံးမှာ Node အသစ်ထည့်တဲ့ function
void insertEnd(struct Node** head_ref, int data) {
    // 1. Node အသစ်အတွက် memory ခွဲဝေမယ်
    struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));
    newNode->data = data;
    
    // 2. တကယ်လို့ List က အလွတ်ကြီး ဖြစ်နေရင်
    if (*head_ref == NULL) {
        *head_ref = newNode;
        newNode->next = *head_ref; // သူ့ကိုယ်သူ ပြန်ပတ်ပြီး ချိတ်ထားမယ်
        return;
    }

    // 3. List ထဲမှာ Node တွေရှိနေရင် နောက်ဆုံး Node ရောက်အောင် ရှာမယ်
    struct Node* temp = *head_ref;
    while (temp->next != *head_ref) { // NULL အစား *head_ref ကို စစ်တာ သတိပြုပါ
        temp = temp->next;
    }

    // 4. Pointer ချိတ်ဆက်မှုတွေကို ပြင်ဆင်မယ်
    temp->next = newNode;   // ယခင်နောက်ဆုံး Node က Node အသစ်ကို ညွှန်မယ်
    newNode->next = *head_ref; // Node အသစ်က Head Node ကို ပြန်ညွှန်ပြီး Circle လုပ်မယ်
}
```

##### Decompiled C (Low-Level Perspective)

အပေါ်က C code ကို compile လုပ်ပြီး binary အဖြစ် ပြောင်းလိုက်တဲ့အခါ `struct Node` ရဲ့ memory structure က ရိုးရှင်းသွားပါတယ်။ 64-bit system မှာ-

- `int data;` -> Offset `+0` (4 bytes နေရာယူပေမဲ့ 8-byte alignment ကြောင့် နောက် pointer အတွက် 4 bytes gap ချန်တတ်ပါတယ်)
    
- `struct Node* next;` -> Offset `+8` (8 bytes နေရာယူတဲ့ pointer)
    

ဒါကြောင့် Decompiler (ဥပမာ- Ghidra သို့မဟုတ် IDA Pro) ကနေ ပြန်ကြည့်ရင် structure နာမည်တွေ မရှိတော့ဘဲ Address Offset တွေနဲ့ပဲ အောက်ပါအတိုင်း မြင်ရမှာ ဖြစ်ပါတယ်

```c
// Decompiler မှ ထွက်လာနိုင်မယ့် ပုံစံ (Ghidra Style Pseudo-C)
void decompile_insertEnd(longlong *head_ref, int data) {
    longlong *newNode;
    longlong *temp;

    // 1. malloc ဖြင့် 16 bytes ခွဲဝေခြင်း (Data 8 + Next 8)
    newNode = (longlong *)malloc(0x10); 
    if (newNode != (longlong *)0x0) {
        *(int *)newNode = data; // Offset +0 မှာ data ထည့်တယ်
    }

    // 2. *head_ref == NULL စစ်ခြင်း
    if (*head_ref == 0x0) {
        *head_ref = (longlong)newNode;
        // newNode->next = *head_ref
        *(longlong *)(newNode + 1) = (longlong)newNode; // pointer arithmetic (+1 က +8 bytes ကို ပြတယ်)
    } 
    else {
        // 3. Loop ပတ်ပြီး နောက်ဆုံး Node ရှာခြင်း
        temp = (longlong *)*head_ref;
        
        // while (temp->next != *head_ref)
        // temp + 1 က next pointer ရှိရာ offset +8 ကို လှမ်းကြည့်တာ ဖြစ်ပါတယ်
        while (*(longlong *)(temp + 1) != *head_ref) {
            temp = (longlong *)*(longlong *)(temp + 1);
        }

        // 4. Pointer ပြန်ပြင်ခြင်း
        *(longlong *)(temp + 1) = (longlong)newNode;   // temp->next = newNode
        *(longlong *)(newNode + 1) = *head_ref;        // newNode->next = *head_ref
    }
    return;
}
```


1. **Loop ရဲ့ ရပ်တန့်မှု စည်းကမ်း:** သာမန် Linked List ရဲ့ decompiled code တွေမှာ `while (temp != 0)` ဆိုပြီး memory address သုည (NULL) ဖြစ်မဖြစ်ပဲ စစ်လေ့ရှိပါတယ်။ ဒါပေမဲ့ Circular ဖြစ်တဲ့အတွက် စက်ကုဒ် (Assembly/Decompiled C) မှာပါ `*head_ref` ရဲ့ variable တန်ဖိုးနဲ့ တိုက်တိုက်စစ်နေတာကို ထင်ရှားစွာ မြင်တွေ့ရမှာ ဖြစ်ပါတယ်။
    
2. **Memory Offset:** `*(longlong *)(newNode + 1)` ဆိုတာ `newNode->next` ကို ဆိုလိုခြင်း ဖြစ်ပြီး၊ ကွန်ပျူတာသည် မြားပြထားတဲ့ pointer လမ်းကြောင်းတွေအတိုင်း မသိပါဘူး၊ Address တစ်ခုရဲ့ ၈ ဘိုက်မြောက် နေရာကို တန်ဖိုးအသစ်တွေ အစားထိုးပြီး Circular ဖြစ်အောင် လုပ်သွားတာ ဖြစ်ပါတယ်။