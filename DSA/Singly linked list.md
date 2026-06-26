

##### Singly Linked List Structure 

Singly Linked List မှာ Node တစ်ခုချင်းစီကို pointer တစ်ခုတည်း နဲ့ပဲ ချိတ်ဆက်ထားပါတယ်။အဲဒီ Pointer ကို `next` pointer လို့ ခေါ်ပါတယ်

Node တစ်ခုအတွင်းမှာ အပိုင်း ၂ ပိုင်းပဲ ပါဝင်တယ်-
1. **Data Field:** တကယ့် Data တန်ဖိုး (ဥပမာ - ကိန်းဂဏန်း `10`, `20` သို့မဟုတ် စာသား) ကို သိမ်းပါတယ်။
2. **Next Pointer Field:** နောက်ထပ်လာမယ့် Node ရဲ့ Memory Address (လိပ်စာ) ကိုပဲ သိမ်းပါတယ်။ သူ့အရှေ့က Node က ဘယ်ကောင်လဲဆိုတာကို သူ ပြန်မသိနိုင်ပါဘူး (တစ်ဖက်သတ်လမ်းပြင် ဖြစ်လို့ပါ)။
    

 Singly Linked List တစ်ခုလုံး ချိတ်ဆက်ပုံ

ဥပမာအားဖြင့် ကျွန်တော်တို့မှာ `10`, `20`, `30` ဆိုတဲ့ Data ၃ ခု ရှိတယ်ဆိုပါစို့။ Memory ပေါ်မှာ သူတို့ ဒီလို ချိတ်ဆက်သွားပါလိမ့်မယ်-
- **Head :** ပထမဆုံး Node (Data: 10) ကို ညွှန်ပြနေတဲ့ Pointer ဖြစ်ပါတယ်။ List တစ်ခုလုံးကို ရှာဖွေချင်ရင် Head ကနေပဲ စရပါတယ်
- **Middle :** ဒုတိယ Node (Data: 20) ရဲ့ `next` က တတိယ Node (Data: 30) ကို ညွှန်ပါတယ်
- **Tail / Null :** တတိယ Node (Data: 30) ဟာ နောက်ဆုံး Node ဖြစ်တဲ့အတွက် သူ့ရဲ့ `next` နေရာမှာ ဘယ်လိပ်စာမှမရှိဘဲ **`NULL`** ဖြစ်နေမှာ ဖြစ်ပါတယ်
    

 ##### Singly Linked List Operation

Singly Linked List ထဲကို Data အသစ်ထည့်တာ၊ ဖြတ်တာတွေက Array ထက် ပိုပြီး လိုက်လျောညီထွေရှိတယ်

 Data Insertion
- **အစမှာ dataထည့်ခြင်း (Insert at Head):** Node အသစ်တစ်ခု ဆောက်လိုက်မယ် အဲဒီ Node ရဲ့ `next` ကို လက်ရှိ Head ဆီ ညွှန်ခိုင်းလိုက်ပြီး၊ Node အသစ်ကို Head အဖြစ် ပြောင်းလိုက်ရုံပါပဲ
- **အလယ်မှာ ထည့်ခြင်း (Insert in Middle):** ဥပမာ- 1 နဲ့ 2ကြားထဲ ထည့်ချင်ရင်၊ Node 1 ရဲ့ မြှားတံကို Node အသစ်ဆီ ညွှန်ခိုင်းပြီး၊ Node အသစ်ရဲ့ မြှားတံကို Node 2 ဆီ ညွှန်ခိုင်းလိုက်တာ ဖြစ်ပါတယ်။ (Array လို အနောက်ကကောင်တွေကို နေရာရွှေ့ပေးစရာ မလိုပါဘူး)
    

 Data Deletion
- ဥပမာ- အလယ်က Node တစ်ခုကို ဖြတ်ချင်ရင် သူ့အရှေ့က Node ရဲ့ `next` မြှားတံကို ပြုတ်သွားမယ့် Node ရဲ့ အနောက်က Node ဆီကို ကျော်ပြီး ချိတ်ပေးလိုက်ရုံပါပဲ။ ပြီးရင်တော့ ဖြတ်လိုက်တဲ့ Node ကို memory ပေါ်ကနေ ဖျက်ထုတ် (free) လိုက်ပါတယ်။
    


**အားသာချက်:**

- Memory space ကို ချွေတာနိုင်ပါတယ် (Doubly Linked List လိုမျိုး အရှေ့ပြန်ညွှန်တဲ့ `prev` pointer မလိုတဲ့အတွက် pointer တစ်ခုစာ memory သက်သာပါတယ်)
- Data အသစ်ထည့်ခြင်း၊ ဖြတ်ခြင်း လုပ်ငန်းစဉ်တွေဟာ ရိုးရှင်းပြီး မြန်ဆန်ပါတယ်
    

**အားနည်းချက်:**

- **နောက်ပြန်ဆုတ်လို့ မရခြင်း (No Backward Traversal):** အနောက် Node ကနေ အရှေ့ Node ဆီ ပြန်သွားလို့ မရပါဘူး လမ်းအစ Head ကနေပဲ အမြဲတမ်း အစပြန်အဆုံး ရှာရပါတယ်
- တစ်ခုခုကြောင့် ကြားထဲက Node တစ်ခုရဲ့ Pointer ပျက်စီးသွားရင် အနောက်က Node အကုန်လုံးနဲ့ အဆက်အသွယ် ပြတ်တောက်သွားနိုင်ပါတယ်
    



C language မှာ Singly Linked List ရဲ့ Node တစ်ခုကို `struct` သုံးပြီး တည်ဆောက်ပါတယ် ဒီကုဒ်ကတော့ Node အသစ်တစ်ခုဆောက်ပြီး List ရဲ့ ထိပ်ဆုံးမှာ သွားထည့်ပေးတဲ့ (Insert at Head) လုပ်ဆောင်ချက် ဖြစ်ပါတယ်

```c
#include <stdio.h>
#include <stdlib.h>

struct Node {
    int data;
    struct Node* next;
};

// ပြင်ဆင်ချက်- struct Node* ဖြစ်သွားပါပြီ (Pointer သုံးထားသည်)
struct Node* insert_at_head(struct Node* head, int new_data) {
    
    // ပြင်ဆင်ချက်- struct Node* နှင့် (struct Node*) ပြောင်းလဲထားသည်
    struct Node* new_node = (struct Node*)malloc(sizeof(struct Node));
    
    if (new_node == NULL) return head; // Memory မလောက်ပါက လက်ရှိ head ကိုပဲ ပြန်ပေးရန်
    
    new_node->data = new_data;
    new_node->next = head; // Node အသစ်က လက်ရှိ head ကို ထောက်ပြသည်

    return new_node; // Node အသစ်သည် head အသစ် ဖြစ်သွားသည်
}

// စမ်းသပ်ရန် Main Function
int main() {
    struct Node* head = NULL;
    head = insert_at_head(head, 10);
    head = insert_at_head(head, 20);
    
    // ထွက်လာမည့်ရလဒ်ကို စစ်ဆေးခြင်း
    struct Node* temp = head;
    while(temp != NULL) {
        printf("%d -> ", temp->data);
        temp = temp->next;
    }
    printf("NULL\n");
    return 0;
}
```



 Decompiled C Code (`Ghidra` ကနေ မြင်ရမယ့်ပုံစံ)

အပေါ်ကကုဒ်ကို Compile လုပ်ပြီး စက်နားလည်တဲ့ Binary အဖြစ် ပြောင်းလိုက်တဲ့အခါ `struct Node` ဆိုတဲ့ နာမည်တွေ၊ `->data`, `->next` ဆိုတဲ့ စာသားတွေ အကုန်ပျောက်သွားပါတယ်။ Compiler က ဒါတွေကို **Memory Offset (အကွာအဝေး)** တွေအဖြစ်ပဲ မြင်တော့တာပါ။

`Ghidra` ကနေ အဲဒီ Binary ကို ပြန် Decompile လုပ်ရင် အခုလို ပုံစံမျိုး တွေ့ရပါလိမ့်မယ်-


```c
longlong FUN_00101180(longlong head_address, int new_data)

{
  longlong *new_node;

  // 1. malloc(16) ဟု ပြောင်းသွားခြင်း (int 4 bytes + pointer 8 bytes = 12 bytes ဖြစ်သော်လည်း 
  // Memory Alignment ကြောင့် 8 ရဲ့ ဆတိုးဖြစ်သော 16 bytes နေရာယူသွားခြင်း ဖြစ်သည်)
  new_node = (longlong *)malloc(16);
  
  if (new_node != (longlong *)0x0) {
    
    // 2. new_node->data = new_data; အစား အခန်း [0] နေရာတွင် data ထည့်ခြင်း
    // (int)new_node အဖြစ် typecast လုပ်ပြီး သတ်မှတ်လေ့ရှိသည်
    *(int *)new_node = new_data;
    
    // 3. new_node->next = head; အစား offset 8 byte အကွာ (ဒုတိယအခန်း) တွင် head address ကို ထည့်ခြင်း
    // 64-bit system တွင် pointer သည် 8 bytes ရှိသဖြင့် new_node[1] ဟု ပြုလုပ်ခြင်း ဖြစ်သည်
    new_node[1] = head_address;
  }
  
  // 4. Node အသစ်၏ address ကို return ပြန်ခြင်း
  return (longlong)new_node;
}

```

```c
longlong FUN_00101180(longlong head_address, int new_data)
{
    longlong *new_node;

    // 1. Memory 16 bytes တောင်းခြင်း (Data 4 bytes + Alignment 4 bytes + Pointer 8 bytes)
    new_node = (longlong *)malloc(16);
    
    if (new_node != (longlong *)0x0) {
        // 2. Offset +0 နေရာတွင် 4-byte integer (data) ထည့်ခြင်း
        *(int *)new_node = new_data;

        // 3. Offset +8 နေရာတွင် 8-byte head address (pointer) ကို ထည့်ခြင်း
        // Decompiler များသည် offset ကို တိကျစေရန် ဤကဲ့သို့ ရေးလေ့ရှိသည်
        *(longlong *)((longlong)new_node + 8) = head_address;
    }
    
    // 4. ဖန်တီးလိုက်သော Node ၏ Address ကို ပေးပို့ခြင်း
    return (longlong)new_node;
}
```

 အဓိက ကွဲပြားသွားတဲ့ အချက်များကို နှိုင်းယှဉ်ချက်

Ghidra code တွေကို ဖတ်တဲ့အခါ ပိုနားလည်သွားအောင် အဓိက ပြောင်းလဲသွားတဲ့ အချက် ၃ ချက်ကို ရှင်းပြပေးပါမယ်-

 `struct` နေရာတွင် Array သို့မဟုတ် Pointer Pointer များ ဖြစ်သွားခြင်း

ကျွန်တော်တို့ ရေးခဲ့တဲ့ `struct Node` ဆိုတာ မရှိတော့ပါဘူး။ Ghidra က `longlong *` (သို့မဟုတ် `undefined8 *`) လို့ပဲ ပြပါလိမ့်မယ်။

`->data` နှင့် `->next` နေရာတွင် Offset များ ဖြစ်သွားခြင်း

- **`new_node->data`** သည် `*(int *)new_node` ဖြစ်သွားပါတယ်။ ဆိုလိုတာက memory ရဲ့ အစကနဦး (Offset 0) မှာ ရှိတဲ့ 4-byte နေရာမှာ ကိန်းဂဏန်း သွားသိမ်းတာပါ။
    
- **`new_node->next`** သည် `new_node[1]` ဖြစ်သွားပါတယ်။ `longlong` pointer ဖြစ်တဲ့အတွက် `[1]` ဆိုတာ အစကနေ 8 bytes ကျော်တဲ့နေရာ (Offset 8) မှာ နောက် Node ရဲ့ လိပ်စာကို သွားသိမ်းတာ ဖြစ်ပါတယ်။
    
 `sizeof()` နေရာတွင် ကိန်းဂဏန်း တိုက်ရိုက်ပေါ်လာခြင်း

`sizeof(struct Node)` လို့ ရေးခဲ့တာကို Compiler က တွက်ချက်ပြီး `malloc(16)` လို့ ကိန်းဂဏန်း အသေ တိုက်ရိုက် ထည့်ပေးလိုက်တာ ဖြစ်ပါတယ်။

