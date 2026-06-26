
### Input Injection 1

#input_injection

```
└─$ pwn checksec vuln                              
[*] '/home/Jackfruit/cate/learn/binary/stack/picoCTF/inputInjection/vuln'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No

```


```c

undefined8 main(void)

{
   size_t sVar1;
   char local_d8 [208];
   
   puts("What is your name?");
   FUN_00401100(stdout);
   fgets(local_d8,200,stdin);
   sVar1 = strcspn(local_d8,"\n");
   local_d8[sVar1] = '\0';
   fun(local_d8,"uname");
   return 0;
}
```


```c

void fun(char *param_1,char *param_2)

{
   char local_1c [10];
   char local_12 [10];
   
   strcpy(local_12,param_2);
   strcpy(local_1c,param_1);
   printf("Goodbye, %s!\n",local_1c);
   FUN_00401100(stdout);
   system(local_12);
   return;
}

```

	user input ကြ အများကြီးယူပြီး (can buffer overflow) fun ထဲမှာကြ 10 byte ဘဲရှိတဲ့ local_1cထဲ copy ကူးထားတယ် system() ခေါ်ထားတယ် variable အတွက် default ဆို uname ပါပေါ့ ဆိုတော့ 10bytes + "cat flag.txt\n" 