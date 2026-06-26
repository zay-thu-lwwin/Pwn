

#### pwn105

wrap around ကိုအသုံးချတဲ့ကောင်
`uint` ကနေ int ကိုပြောင်းပြီး
`uint` ရဲ့ limit က 4,294,967,295 အထိဘဲ
int မှာ  (+) limit က +2,147,483,647 ထိ , (-) ကတော့ အနောက်ကနေစရေတယ်
ဆိုတော့  +2,147,483,647  ထပ်ကျော်တာကိုပေးလိုက်ရင် int ပြောင်းရင် - ဖြစ်မှာဘဲ
(-)နှစ်ခုပေါင်းလည်း (-) ဖြစ်မှဆိုတော့ 2000000000    2000000000 ပေးလိုက်ရင်ရပြီ

```c

void main(void)

{
  long in_FS_OFFSET;
  uint local_1c;
  uint local_18;
  uint local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  setup();
  banner();
  puts("-------=[ BAD INTEGERS ]=-------");
  puts("|-< Enter two numbers to add >-|\n");
  printf("]>> ");
  __isoc99_scanf(&DAT_0010216f,&local_1c);
  printf("]>> ");
  __isoc99_scanf(&DAT_0010216f,&local_18);
  local_14 = local_18 + local_1c;
  if (((int)local_1c < 0) || ((int)local_18 < 0)) {
    printf("\n[o.O] Hmmm... that was a Good try!\n",(ulong)local_1c,(ulong)local_18,(ulong)local_14)
    ;
  }
  else if ((int)local_14 < 0) {
    printf("\n[*] C: %d",(ulong)local_14);
    puts("\n[*] Popped Shell\n[*] Switching to interactive mode");
    system("/bin/sh");
  }
  else {
    printf("\n[*] ADDING %d + %d",(ulong)local_1c,(ulong)local_18);
    printf("\n[*] RESULT: %d\n",(ulong)local_14);
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}


```

