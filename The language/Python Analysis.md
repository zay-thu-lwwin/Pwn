
### **`__class__` attribute**

`__class__` သည် Python object တစ်ခုရဲ့ **class ကိုယ်တိုင်ကို** ရည်ညွှန်းတဲ့ attribute တစ်ခုဖြစ်ပါတယ်။ ရိုးရိုးရှင်းရှင်းပြောရရင် object တစ်ခုက ဘယ် class ကနေ ဆင်းသက်လာတယ်ဆိုတာကို ပြောပြတဲ့အရာပါ။

```
# ဘယ် object မဆို __class__ မှာ သူ့ class ရှိတယ်
x = "hello"
print(x.__class__)  # <class 'str'>
```

```
print(student1.__class__)
# Output: <class '__main__.Student'>

# Class ရဲ့ name ကိုပဲ ယူချင်ရင်
print(student1.__class__.__name__)
# Output: Student
```

```
# Class ရဲ့ attribute တွေကို ကြည့်ချင်ရင်
print("Class name:", my_car.__class__.__name__)
print("Class attributes:", my_car.__class__.__dict__)
```

## `type()` Function ဆိုတာ ဘာလဲ?


`type()` သည် Python built-in function တစ်ခုဖြစ်ပြီး **object တစ်ခုရဲ့ data type (သို့) class ကို** ပြန်ပြောပေးတဲ့ function ဖြစ်ပါတယ်။

**ကွာခြားချက်:**

- `type()` က built-in function တစ်ခု
    
- `__class__` က object ရဲ့ attribute တစ်ခု

```
# အရိုးရှင်းဆုံး ဥပမာများ
print(type(5))           # Output: <class 'int'>
print(type(3.14))        # Output: <class 'float'>
print(type("Hello"))     # Output: <class 'str'>
print(type(True))        # Output: <class 'bool'>
print(type([1, 2, 3]))   # Output: <class 'list'>
print(type({"name": "John"}))  # Output: <class 'dict'>
```


## `__mro__` ဆိုတာ ဘာလဲ?


`__mro__` သည် Python မှာ **Multiple Inheritance** (မိဘ class များစွာကနေ အမွေဆက်ခံခြင်း) ရှိတဲ့အခါ method တွေကို ဘယ်အစဉ်အတိုင်း ရှာဖွေရမယ်ဆိုတာကို သတ်မှတ်ပေးတဲ့ order ဖြစ်ပါတယ်။

```
class A:
    def method(self):
        return "A class method"

class B(A):
    def method(self):
        return "B class method"

class C(A):
    def method(self):
        return "C class method"

class D(B, C):
    pass

# MRO ကို ကြည့်ကြည့်ရအောင်
print(D.__mro__)
# Output: (<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)
```

## `__subclasses__()` ဆိုတာ ဘာလဲ?

`__subclasses__()` သည် class တစ်ခုကနေ **ဘယ် subclass တွေ ဆင်းသက်လာလဲ** ဆိုတာကို ပြန်ပြောပေးတဲ့ method ဖြစ်ပါတယ်။

## **အခြေခံအလုပ်လုပ်ပုံ**

- Class တစ်ခုကို inherit လုပ်ထားတဲ့ subclass တွေအားလုံးကို list အဖြစ် return ပြန်ပေးတယ်
    
- Parent class ကနေ ခေါ်သုံးရတယ်

## globals ဆိုတာဘာလဲ?

`__globals__` သည် Python function objects တွေမှာရှိတဲ့ attribute တစ်ခုဖြစ်ပြီး function ရဲ့ global namespace ကို reference လုပ်ပေးပါတယ်။


---

### `Input() in python 2.7`


	Python 2.7 မှာ input() function က အန္တရာယ်ရှိပါတယ်။
	Python 3 မှာ input() က string ကိုပဲ return ပြန်ပေးပါတယ်
	 Python 2.7 မှာ input() က user ရိုက်ထည့်တဲ့ code ကို evaluate လုပ်ပါတယ် (eval လိုမျိုး)
	
---

1. **`elf.symbols['read']`** က:
    
    - **PIE disabled** ဆိုရင် → **absolute address** ပြန်ပေး (ဒီ challenge မှာ)
        
    - **PIE enabled** ဆိုရင် → **offset from base** ပြန်ပေး
        
---
