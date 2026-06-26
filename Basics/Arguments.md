
## 32-bit (x86) Program 
	32-bit mode မှာ stack ကိုအဓိကသုံးပြီး argument တွေကို pass လုပ်ပါတယ်။

	စည်းမျဉ်း (cdecl calling convention အရ)
	1. Argument တွေကို ညာဘက်ကနေ ဘယ်ဘက် စီပြီး stack ထဲ push လုပ်တယ်။
	2. Function ခေါ်ပြီးရင် caller က stack ကို cleanup လုပ်ရတယ်။


`func(1, 2, 3);` ကို assembly မှာ:
```assembly
push 3      ; ညာဆုံး argument ကို အရင်ထည့်
push 2
push 1      ; ဘယ်ဆုံး argument
call func
add esp, 12 ; stack cleanup (argument 3ခု * 4 bytes = 12)
```


	အခြေခံအားဖြင့် eax, ecx, edx တို့က return value နဲ့ temporary အတွက်သာသုံးပြီး၊ argument တွေအတွက် stack ကိုပဲသုံးတယ်။

`Example`

```assembly
push 30      ; arg3
push 20      ; arg2
push 10      ; arg1
call func    ; return address auto push ဖြစ်မယ်
```

```stack
0xbfffeff0    0x08048456   return address 
0xbfffeff4     10          arg1
0xbfffeff8     20          arg2
0xbfffeffc     30          arg3
0xbffff000     ???
```

---


## 64-bit (x86-64)
	64-bit mode မှာ register တွေကို အဓိက သုံးပါတယ်။ (Windows နဲ့ Linux/macOS မှာနည်းနည်းကွာတယ်၊ Linux/macOS ကို ဒီမှာပြောမယ်)
	Linux/macOS (System V ABI) စည်းမျဉ်း
	1. Integer/Pointer argument ၆ခုအထိကို register တွေနဲ့ pass လုပ်တယ်:
	   - rdi, rsi, rdx, rcx, r8, r9 
	1. ၆ခုထက်ကျော်ရင် ကျန်တဲ့ argument တွေကို stack ထဲထည့်ရတယ်။
	2. Floating point argument တွေအတွက် xmm0 - xmm7 register တွေကိုသုံးတယ်။

 
`func(1, 2, 3, 4, 5, 6, 7);` ဆိုရင်:
```assembly
mov rdi, 1   ; arg1
mov rsi, 2   ; arg2
mov rdx, 3   ; arg3
mov rcx, 4   ; arg4
mov r8, 5    ; arg5
mov r9, 6    ; arg6
push 7       ; 7th arg → stack
call func
add rsp, 8   ; stack cleanup for the pushed argument
```

`Another Example`

```
mov edi, 10    ; arg1
mov esi, 20    ; arg2  
mov edx, 30    ; arg3
mov ecx, 40    ; arg4
mov r8d, 50    ; arg5
mov r9d, 60    ; arg6
push 70        ; arg7 (stack မှာ)
call func
```

```
0x7fffffffdfd8: 70     ; arg7 (64-bit မှာ push က 8 bytes ယူတယ်)
0x7fffffffdfe0: ???    
0x7fffffffe000: ???    <-- အရင် RSP
```

	Windows x64 မှာဆို  
	rcx, rdx, r8, r9 ထိ ၄ခုပဲ register နဲ့ pass ၊ ကျန်တာက stack။

---


