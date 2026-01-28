

*
```

int* p = &x  (* data type of pointer)
*p ---> original value (*&x = x)
x[] = {3, 4}
*x --> first value (3) x[0]
*(x+1) --> second valud

array copy
int* p = x;
p ---> first value address x[0]
*p --> first value
p++ (*(p+1)) 
p ---> second value address
*p ---> second valule

```
**name of an array**, is actually a **pointer** to the **first element**
&
```

scanf("%d", &x) we scan the address of integer
&x ---> memory address
prinf("%p", &x)
```


```
ptr->member က (*ptr).member
ptr->age = (*ptr).age
```

memset()
Memory block တစ်ခုကို specific value နဲ့ ဖြည့်ပေးတယ်

strdup()
String တစ်ခုကို duplicate လုပ်ပြီး memory အသစ် allocate လုပ်ပေးတယ်

```
service = strdup(line + 7);
1. `line + 7` မှာရှိတဲ့ string ကို ယူမယ်
    
2. အဲ့ string အတွက် memory အသစ် malloc() နဲ့ ယူမယ်
    
3. String ကို copy ကူးမယ်
    
4. Pointer ကို return ပြန်မယ်
```