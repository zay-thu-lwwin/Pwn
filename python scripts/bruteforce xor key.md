
```python
#!/usr/bin/env python3

# ၁။ မူရင်း string ကို byte အနေနဲ့ သတ်မှတ်ပါ (8 bytes ပြည့်အောင် အနောက်မှာ null သို့မဟုတ် space အုပ်ပေးရပါတယ်)
# /bin/sh က 7 bytes ပဲရှိလို့ အရှေ့မှာ 0x00 တစ်လုံး ပါနေတတ်ပါတယ်။ 
# ဒါကြောင့် အဆင်ပြေအောင် '//bin/sh' (8 bytes) လို့ အသုံးများပါတယ်။
original_string = b"//bin/sh" 

# ၂။ အစ်ကို့ challenge ရဲ့ Blacklist (Bad Bytes) တွေကို ဒီထဲမှာ ထည့်ပါ
# ဥပမာ - 0x00 (Null), 0x0a (Newline) နဲ့ အစ်ကို့ရဲ့ local_78 ထဲက ကောင်အချို့
blacklist = [0x00, 0x0a, 0x3b, 0x54, 0x62, 0x69, 0x6e, 0x73, 0x68]

print("[*] Scanning for valid 1-byte repeating XOR keys...")
print("-" * 50)

# ၃။ 1-byte key (0x01 မှ 0xFF) အထိကို Bruteforce ပတ်ပါမယ်
# (0x2a2a2a2a2a2a2a2a ဆိုတာ 0x2a ကို 8 ကြိမ် ထပ်ထားတာ ဖြစ်လို့ 1-byte bruteforce နဲ့ မိပါတယ်)
found_any = False

for k in range(1, 256):
    # ကာကွယ်ရေး - Key ကိုယ်တိုင်က blacklist ထဲမှာ ပါနေရင် ကျော်မယ်
    if k in blacklist:
        continue
        
    # ရွေးချယ်ထားတဲ့ key နဲ့ original string ကို byte တစ်လုံးချင်းစီ လိုက် XOR လုပ်မယ်
    xored_output = bytearray()
    bad_byte_found = False
    
    for b in original_string:
        xored_byte = b ^ k
        # ရလာတဲ့ ရလဒ် byte က blacklist ထဲမှာ ပါနေရင် ဒီ key ကို ပယ်မယ်
        if xored_byte in blacklist:
            bad_byte_found = True
            break
        xored_output.append(xored_byte)
        
    # အကယ်၍ Key ထဲမှာရော၊ ရလဒ်ထဲမှာရော bad byte မပါဘူးဆိုရင်... ဒါ ကျွန်တော်တို့ လိုချင်တဲ့ Key ပဲ!
    if not bad_byte_found:
        found_any = True
        # 8-byte key ကြီးအဖြစ် ပြောင်းလဲ သတ်မှတ်ပြတာပါ (e.g., 0x2a2a2a2a2a2a2a2a)
        full_key_hex = f"0x{bytes([k]*8).hex()}"
        full_result_hex = f"0x{bytes(xored_output[::-1]).hex()}" # Little-endian ပုံစံ push ဖို့ အနောက်ကနေ ပြန်လှန်ပြတာပါ
        
        print(f"[+] FOUND VALID KEY: 0x{k:02x} (ASCII: {chr(k)!r})")
        print(f"    -> Full 8-Byte Key   : {full_key_hex}")
        print(f"    -> XORed Value (RAX) : {full_result_hex}\n")

if not found_any:
    print("[-] No single-byte repeating key found. You might need a multi-byte key pattern!")
```