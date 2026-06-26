
##### Doubly Linked List (Structure)

Single Linked List မှာ Node တစ်ခုက သူ့နောက်က Node ကိုပဲ သိပေမဲ့ Doubly Linked List မှာတော့ Node တစ်ခုမှာ အစိတ်အပိုင်း **၃ ခု** ပါဝင်ပါတယ်

1. **Prev (Previous Pointer):** သူ့အရှေ့က Node ရဲ့ memory address ကို ညွှန်ပြပါတယ်။
2. **Data:** ကိုယ်သိမ်းဆည်းချင်တဲ့ တန်ဖိုး (Value) ဖြစ်ပါတယ်။
3. **Next (Next Pointer):** သူ့အနောက်က Node ရဲ့ memory address ကို ညွှန်ပြပါတယ်။
    

- **Head (အစဆုံး Node):** သူ့ရဲ့ `Prev` pointer က အမြဲတမ်း `NULL` ဖြစ်နေပါလိမ့်မယ် (ဘာလို့လဲဆိုတော့ သူ့အရှေ့မှာ ဘာမှမရှိလို့ပါ)။
- **Tail (အဆုံးစွန် Node):** သူ့ရဲ့ `Next` pointer က `NULL` ဖြစ်နေပါလိမ့်မယ် (သူ့နောက်မှာ ဘာမှမရှိတော့လို့ပါ)။
    

##### singly linked link vs doubly linked list

| info          | Single Linked List                               | Doubly Linked List                                         |
| ------------- | ------------------------------------------------ | ---------------------------------------------------------- |
| no of pointer | Pointer ၁ ခုပဲပါမယ် (`Next`)                     | Pointer ၂ ခုပါမယ် (`Prev` နှင့် `Next`)                    |
| Traversal     | အရှေ့ကနေ အနောက်ပဲ သွားလို့ရတယ် (Forward)         | အရှေ့ရော အနောက်ပါ အပြန်အလှန် သွားလို့ရတယ် (Bi-directional) |
| Memory usage  | Memory စားတာ နည်းတယ်                             | Pointer ၂ ခု သိမ်းရလို့ Memory ပိုစားတယ်                   |
| how to delete | ပိုရှုပ်ထွေးတယ် (အရှေ့ Node ကို လိုက်ရှာနေရလို့) | ပိုလွယ်ကူမြန်ဆန်တယ်                                        |



 အားသာချက်များ (Advantages)

- **ရှေ့နောက် အပြန်အလှန်သွားနိုင်ခြင်း:** List ထဲမှာ ရှေ့ကိုရော နောက်ကိုရော လွတ်လပ်စွာ ရွှေ့လျားနိုင်ပါတယ်။
- **ဖျက်ရတာ လွယ်ကူခြင်း (Easier Deletion):** Node တစ်ခုကို ဖျက်ချင်ရင် သူ့ရဲ့ `Prev` pointer ရှိပြီးသားဖြစ်လို့ Single Linked List လိုမျိုး အရှေ့က Node ကို လိုက်ရှာစရာမလိုဘဲ တန်းဖျက်နိုင်ပါတယ်။
    

 အားနည်းချက်များ (Disadvantages)

- **Memory ပိုကုန်ခြင်း:** Node တိုင်းမှာ Pointer နှစ်ခုစီ သိမ်းရတဲ့အတွက် Memory ပိုနေရာယူပါတယ်။
- **Code ရေးရတာ ပိုရှုပ်ထွေးခြင်း:** Node အသစ်တစ်ခု ထည့်တာ (Insertion) ဒါမှမဟုတ် ဖျက်တာ (Deletion) လုပ်တဲ့အခါ `Next` ရော `Prev` pointer တွေကိုပါ သေချာချိတ်ဆက်ပေးရလို့ Code ရေးရတာ ပိုသတိထားရပါတယ်။
    

##### Real-world Applications

- **Web Browsers:** Browser တွေမှာ Forward (ရှေ့သို့) နှင့် Back (နောက်သို့) ခလုတ်တွေ နှိပ်ပြီး ဖွင့်ခဲ့တဲ့ Website တွေကို ပြန်ကြည့်တဲ့နေရာမှာ သုံးပါတယ်
- **Music Player Apps:** သီချင်းစာရင်း (Playlist) ထဲမှာ Next Song (နောက်သီချင်း) သို့မဟုတ် Previous Song (ရှေ့သီချင်း) ပြောင်းလဲနားထောင်တဲ့ စနစ်တွေမှာ သုံးပါတယ်
- **Undo and Redo functionality:** Photoshop သို့မဟုတ် Word document တွေမှာ Ctrl+Z (Undo) နှင့် Ctrl+Y (Redo) လုပ်တဲ့ လုပ်ဆောင်ချက်တွေမှာ အသုံးပြုပါတယ်



```c
#include <stdio.h>
#include <stdlib.h>

// Step 1: Structure Definition
struct Node {
    int data;
    struct Node* prev;
    struct Node* next;
};

// Step 2: Create Node Function
struct Node* createNode(int data) {
    struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));
    newNode->data = data;
    newNode->prev = NULL;
    newNode->next = NULL;
    return newNode;
}

// Step 3: Insert at Head Function
struct Node* insertAtHead(struct Node* head, int data) {
    struct Node* newNode = createNode(data);
    if (head != NULL) {
        newNode->next = head;
        head->prev = newNode;
    }
    head = newNode;
    return head;
}

// Step 4: Display Function
void printList(struct Node* head) {
    struct Node* temp = head;
    struct Node* last = NULL;

    printf("Forward Traversal:  ");
    while (temp != NULL) {
        printf("%d <-> ", temp->data);
        last = temp;
        temp = temp->next;
    }
    printf("NULL\n");

    printf("Backward Traversal: ");
    while (last != NULL) {
        printf("%d <-> ", last->data);
        last = last->prev;
    }
    printf("NULL\n");
}

int main() {
    struct Node* head = NULL; // စစချင်း List အလွတ်

    // Data များ တစ်ဆင့်ချင်း ထည့်ခြင်း
    head = insertAtHead(head, 30); // 30
    head = insertAtHead(head, 20); // 20 <-> 30
    head = insertAtHead(head, 10); // 10 <-> 20 <-> 30

    // List ကို ထုတ်ပြခြင်း
    printList(head);

    // Memory ပြန်လွှတ်ပေးခြင်း (Free Memory)
    struct Node* temp;
    while (head != NULL) {
        temp = head;
        head = head->next;
        free(temp);
    }

    return 0;
}
```


```c
void printList(struct Node* head) {
    struct Node* temp = head;
    struct Node* last = NULL;

    // အရှေ့ကနေ အနောက်သို့ သွားခြင်း (Forward Traversal)
    printf("Forward Traversal: ");
    while (temp != NULL) {
        printf("%d -> ", temp->data);
        last = temp;     // နောက်ဆုံး Node ကို မှတ်ထားရန် (Backward အတွက်သုံးမည်)
        temp = temp->next;
    }
    printf("NULL\n");

    // အနောက်ကနေ အရှေ့သို့ ပြန်လာခြင်း (Backward Traversal)
    printf("Backward Traversal: ");
    while (last != NULL) {
        printf("%d -> ", last->data);
        last = last->prev; // prev pointer သုံးပြီး နောက်ပြန်ဆုတ်ခြင်း
    }
    printf("NULL\n");
}
```


##### Decompiled C (Low-Level / Reverse Engineered Perspective)

C code ကို compiler ကနေ compile လုပ်ပြီး binary file (ဥပမာ- `.exe` သို့မဟုတ် `.elf`) အဖြစ် ပြောင်းလိုက်တဲ့အခါ `Struct` ဆိုတဲ့ concept က ပျောက်သွားပါတယ်။ Memory ထဲမှာတော့ သူက **Offset (နေရာအကွာအဝေး)** အနေနဲ့ပဲ ရှိပါတော့တယ်။

အပေါ်က `struct Node` ကို 64-bit architecture မှာ တွက်ကြည့်ရင်
- `int data;` -> 4 bytes နေရာယူတယ် (ဒါပေမဲ့ Alignment ကြောင့် 8 bytes ဖြစ်သွားတတ်ပါတယ်)
- `struct Node* prev;` -> Pointer ဖြစ်လို့ 8 bytes နေရာယူတယ်။
- `struct Node* next;` -> Pointer ဖြစ်လို့ 8 bytes နေရာယူတယ်။
    

ဒါကြောင့် Decompiler (ဥပမာ- IDA Pro, Ghidra) တွေကနေ binary ကို ပြန်ကြည့်ရင် Struct နာမည်တွေ မသိတော့ဘဲ **Memory Base Address + Offset** အနေနဲ့ပဲ မြင်ရပါတော့တယ်


```c
// Decompiler က မြင်ရမယ့် ပုံစံ (Ghidra/IDA Style Pseudo-code)
void decompile_deleteNode(longlong *head_ref, longlong *delNode) {
    longlong next_node;
    longlong prev_node;

    // NULL Check ကို အရင်လုပ်တယ်
    if ((head_ref != (longlong *)0x0) && (delNode != (longlong *)0x0)) {
        
        // *head_ref == delNode ကို စစ်တာ
        if (*head_ref == (longlong)delNode) {
            // delNode->next ရဲ့ memory offset က +16 ဖြစ်နေတတ်ပါတယ် (Data 8 + Prev 8 ကြောင့်)
            *head_ref = *(longlong *)(delNode + 2); // pointer arithmetic မှာ +2 က 16 bytes ကို ပြတယ်
        }

        // delNode->next ကို ယူတယ်
        next_node = *(longlong *)(delNode + 2); 
        if (next_node != 0x0) {
            // next_node->prev = delNode->prev;
            // next_node ရဲ့ +8 byte နေရာ (prev pointer) ထဲကို delNode ရဲ့ +8 byte နေရာကတန်ဖိုး ထည့်တယ်
            prev_node = *(longlong *)(delNode + 1);
            *(longlong *)(next_node + 1) = prev_node;
        }

        // delNode->prev ကို ယူတယ်
        prev_node = *(longlong *)(delNode + 1);
        if (prev_node != 0x0) {
            // prev_node->next = delNode->next;
            // prev_node ရဲ့ +16 byte နေရာ (next pointer) ထဲကို delNode ရဲ့ +16 byte နေရာကတန်ဖိုး ထည့်တယ်
            next_node = *(longlong *)(delNode + 2);
            *(longlong *)(prev_node + 2) = next_node;
        }

        // heap memory free လုပ်ခြင်း
        free(delNode);
    }
    return;
}
```

##### Key Takeaways
 
High-level C မှာ `delNode->next->prev` လို့ လှပစွာ ရေးလို့ရပေမဲ့၊
Decompiled C စက်ထဲမှာတော့ `*(longlong *)(*(longlong *)(delNode + 16) + 8)` ဆိုပြီး Pointer pointer ချင်း ထပ်ဆင့်ပြီး Address တွေ တွက်ချက်သွားတာ ဖြစ်ပါတယ်။ Offset `+0` က data၊ `+8` က `prev` pointer၊ `+16` က next pointer ဆိုပြီး internal memory ထဲမှာ တန်းစီတည်ရှိနေတာကို တွေ့ရမှာ ဖြစ်ပါတယ်
    

