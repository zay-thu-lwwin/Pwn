
ASLR (Address Space Layout Randomization) ကို Kali Linux မှာ ဖွင့်/ပိတ် နည်းများကို ရှင်းပြပါမယ်။

## ASLR Status ကြည့်နည်း
```bash
# လက်ရှိ ASLR status ကြည့်ရန်
cat /proc/sys/kernel/randomize_va_space
```
**တန်ဖိုးများ:**
- `0` = ASLR ပိတ်ထား
- `1` = Conservative Randomization (stack, vdso, heap)
- `2` = Full Randomization (default)

## ASLR ပိတ်နည်း (Temporary)

```bash
# Temporary disable (reboot လုပ်ရင် ပြန်ဖွင့်သွား)
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

## `ASLR ဖွင့်နည်း (Temporary)

```bash
# Default setting ပြန်ဖွင့်
echo 2 | sudo tee /proc/sys/kernel/randomize_va_space
```

## Permanent ပြင်နည်း

```bash
# sysctl configuration file ကို edit လုပ်ပါ
sudo nano /etc/sysctl.conf
```

**အောက်က line ကို ထည့်/ပြင်ပါ:**
```
# ASLR ပိတ်ရန်
kernel.randomize_va_space = 0

# ASLR ဖွင့်ရန် (default)
kernel.randomize_va_space = 2
```

**ပြင်ပြီးရင်:**  
```bash
# sysctl settings များ reload လုပ်ပါ
sudo sysctl -p
```

## Process-specific ASLR Control
```bash
# Single process အတွက် ASLR ပိတ်ရန်
setarch $(uname -m) -R /path/to/program

# Example:
setarch x86_64 -R ./vulnerable_program
```

## Check ASLR status နောက်တစ်နည်း
```bash
# Multiple methods
sudo sysctl kernel.randomize_va_space
```

**သတိပြုရန်:**  
ASLR ကို ပိတ်ထားခြင်းက security ကို ကျဆင်းစေ။ လေ့လာချင်တဲ့ program ကို debugging လုပ်ဖို့ပဲ ပိတ်သင့်။

---
