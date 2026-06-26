

Linked List ဆိုတာ data structure တစ်ခုဖြစ်ပြီး node တွေကို ချိတ်ဆက်ပြီး သိမ်းဆည်းတဲ့ နည်းလမ်းတစ်ခု

Array VS Linked List 

```
Array (ဆက်တိုက် သိမ်းတယ်)
┌────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │  ← memory မှာ ကပ်လျက်ရှိတယ်
└────┴────┴────┴────┴────┘

Linked List (သီးခြား သိမ်းတယ်)
┌───────┐    ┌───────┐    ┌───────┐
│ 10 │ *───→│ 20 │ *───→│ 30 │ *──→ NULL
└───────┘    └───────┘    └───────┘
   ↑            ↑            ↑
  Node 1      Node 2      Node 3
  (Head)                  (Tail)
```

| Array | Linked List |
|---|---|
| ဆက်တိုက် သိမ်းတယ် | သီးခြား သိမ်းတယ် |
| Size ကို ကြိုသတ်မှတ်ရတယ် | Size က dynamic ပြောင်းလို့ရတယ် |
| Random access မြန်တယ် | Sequential access ပဲရတယ် |
| Insert/Delete ခက်တယ် | Insert/Delete လွယ်တယ် |

| info               | Array                                                                            | Linked List                                                                            |
| ------------------ | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| how it take memory | Memory ပေါ်မှာ တစ်ဆက်တည်း (Contiguous) နေရာယူရပါတယ်                              | Memory လွတ်တဲ့နေရာမှာ ခွဲပြီး (Scattered) နေနိုင်ပါတယ်။ Pointer နဲ့ပဲ လှမ်းချိတ်ပါတယ်။ |
| Size               | Fixed Size (ဆောက်ကတည်းက အခန်းအရေအတွက် သတ်မှတ်ရတယ်)                               | Dynamic Size (လိုသလောက် Node အသစ်တွေ အချိန်မရွေး တိုးလို့ရတယ်)                         |
| adding /cutting    | ကြားထဲက Element တစ်ခုကို ဖြတ်ရင် အနောက်ကကောင်တွေအကုန် ရှေ့တိုးပေးရလို့ နှေးပါတယ် | Pointer ချိတ်ဆက်မှုကိုပဲ ပြောင်းပေးရလို့ **အရမ်းမြန်ပါတယ်**                            |
| Search             | Index နံပါတ် သိရင် တိုက်ရိုက်တန်းယူလို့ရလို့ မြန်ပါတယ်                           | အစကနေ တစ်ဆင့်ချင်းစီ လိုက်ရှာရလို့ နှေးပါတယ်                                           |

---

##### Node Structure

Node တစ်ခုမှာ အဓိက အပိုင်း ၂ ပိုင်းပါတယ်

```c
struct Node {
    int data;           // ၁။ သိမ်းချင်တဲ့ data
    struct Node *next;  // ၂။ နောက် node ကို ညွှန်တဲ့ pointer
};
```

```
┌───────────┬───────────┐
│   data    │   next    │
│  (value)  │  (pointer)│
└───────────┴───────────┘
```

---


##### Linked List Types

Linked List မှာ ပုံစံအမျိုးမျိုး ရှိပါတယ် အသုံးအများဆုံး ၃ မျိုးကတော့-

- **Singly Linked List:** မြှားတံက အရှေ့ကနေ အနောက်ကိုပဲ (တစ်ဖက်သတ်) ညွှန်ပါတယ်
    
- **Doubly Linked List:** Node တစ်ခုမှာ Next Pointer အပြင် အရှေ့က Node ကို ပြန်ညွှန်တဲ့ **Previous Pointer** လည်း ပါဝင်ပါတယ်။  ရှေ့တိုးနောက်ဆုတ် သွားလို့ရပါတယ်
    
- **Circular Linked List:** အနောက်ဆုံး Node က Null ဖြစ်မသွားဘဲ အစဆုံး Head Node ကို ပြန်ညွှန်ထားလို့ အဝိုင်းပတ် ဖြစ်နေတဲ့ ပုံစံပါ


##### Linked List Operations Complexity

| Operation | Singly Linked List | Doubly Linked List |
|---|---|---|
| Insert at beginning | O(1) | O(1) |
| Insert at end | O(n) (or O(1) with tail) | O(n) (or O(1) with tail) |
| Delete node | O(n) | O(1) (if node found) |
| Search | O(n) | O(n) |
| Traverse | O(n) | O(n) |


|                       |                                              |
| --------------------- | -------------------------------------------- |
| **Linked List ဆိုတာ** | Node တွေကို ချိတ်ဆက်ထားတဲ့ data structure    |
| **Node ဆိုတာ**        | Data + pointer(s) ပါတဲ့ struct               |
| **Singly vs Doubly**  | Singly = next ပဲရှိတယ်, Doubly = next + prev |
| **Head**              | List ရဲ့ ပထမဆုံး node                        |
| **Tail**              | List ရဲ့ နောက်ဆုံး node (next = NULL)        |
| **Insert**            | ရှေ့/နောက် ထည့်လို့ရတယ်                      |
| **Delete**            | Node ကို ဖယ်ရှားပြီး free လုပ်               |
| **Memory**            | malloc/free ကို သေချာသုံးရမယ်                |

