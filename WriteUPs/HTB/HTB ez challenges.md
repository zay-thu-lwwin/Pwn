

#### rObob1rd

```c
wndbg> checksec
File:     /home/Jackfruit/cate/learn/binary/stack/htb/rObob1rd/r0bob1rd
Arch:     amd64
RELRO:      Partial RELRO
Stack:      Canary found
NX:         NX enabled
PIE:        No PIE (0x400000)
RUNPATH:    b'./glibc/'
Stripped:   No

```

GOT overwrite လို့ရတယ်
PIE enable ဆိုတော့ ‌address leak လို့ကောင်းတာပေါ့
ကိုယ်ပိုင် `glibc` နဲ့ run ခိုင်းတယ်
အဆင်ပြေတယ် `glibc` version ရှာစရာမလိုတော့


```c

undefined8 main(void)

{
  ignore_me_init_buffering();
  ignore_me_init_signal();
  banner();
  printRobobirds(robobirdNames);
  operation();
  return 0;
}

```

```c

void operation(void)

{
  long in_FS_OFFSET;
  int local_7c;
  char printf_var [104];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  printf("\nSelect a R0bob1rd > ");
  fflush(stdout);
  __isoc99_scanf(&DAT_00400eb5,&local_7c);
  if ((local_7c < 0) || (9 < local_7c)) {
     printf("\nYou\'ve chosen: %s",robobirdNames + (long)local_7c * 8);
  }
  else {
     printf("\nYou\'ve chosen: %s",*(undefined8 *)(robobirdNames + (long)local_7c * 8));
  }
  getchar();
  puts("\n\nEnter bird\'s little description");
  printf("> ");
  fgets(printf_var,106,stdin);
  puts("Crafting..");
  usleep(2000000);
  start_screen();
  puts("[Description]");
  printf(printf_var);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                         /* WARNING: Subroutine does not return */
     __stack_chk_fail();
  }
  return;
}

```

ဘာမြင်လဲဆို ထုံးစံအတိုင်း format string မြင်တယ် ` fgets(printf_var,106,stdin);` ကလည်း နည်းနည်း 2 byte boundထပ်ကျော်နေတယ်
```c
  puts("\n\nEnter bird\'s little description");
  printf("> ");
  fgets(printf_var,106,stdin);
  puts("Crafting..");
  usleep(2000000);
  start_screen();
  puts("[Description]");
  printf(printf_var);
```


```python
  __isoc99_scanf(&DAT_00400eb5,&local_7c);
  if ((local_7c < 0) || (9 < local_7c)) {
     printf("\nYou\'ve chosen: %s",robobirdNames + (long)local_7c * 8);
  }
  else {
     printf("\nYou\'ve chosen: %s",*(undefined8 *)(robobirdNames + (long)local_7c * 8));
  }
```

အစက ဒီကောင်ကိုသတိမထားမိဘူး 
 first user input (number) မှာတင်  `libc` address  leak ဖို့ ငါတို့ chance ရှိတယ်
 initialized global variable မှာသိမ်းထားတဲ့ array တွေကိုပြပေးတယ်