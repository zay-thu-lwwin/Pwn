## Input Injection 2
#input_injection


```
└─$ pwn checksec vuln
[*] '/home/Jackfruit/cate/learn/binary/stack/picoCTF/inputInjection2/vuln'
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
  void *pvVar1;
  char *__command;
  
  pvVar1 = malloc(0x1c);
  __command = (char *)malloc(0x1c);
  printf("username at %p\n",pvVar1);
  fflush(stdout);
  printf("shell at %p\n",__command);
  fflush(stdout);
  builtin_strncpy(__command,"/bin/pwd",9);
  printf("Enter username: ");
  fflush(stdout);
  FUN_004010c0(&DAT_00402032,pvVar1);
  printf("Hello, %s. Your shell is %s.\n",pvVar1,__command);
  system(__command);
  fflush(stdout);
  return 0;
}


```

	heap ပေါ်မှာ နေရာနှစ်ခုယူတယ် တစ်ခုကို username ထည့်မယ် 
	တစ်ခုကို  run မယ် user name ကို heap overflow ပြီး နောက်တစ်ခုပေါ်overflowမယ်  FUN_004010c0(&DAT_00402032,pvVar1);မှာ scanf ကိုသုံးထားတယ် space တွေထည့်လို့မရ ဆိုတော့ cat<flag.txt or cat$(ls) နဲ့ bypassမယ်