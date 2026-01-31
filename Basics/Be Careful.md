
### Strings and null byte (\x00)

string မှာ null byte ပါတယ် string ဆိုတာ character တွေလေးတွေစုစည်းထားတဲ့ကောင်
string length ကိုသတ်မှတ်ရင် null byte အတွက်ပါထည့်ပေးရ
`strlen`, `strcpy`, `strcat`, `printf("%s")` အားလုံးဟာ null byte ကိုရှာပြီးမှ အလုပ်လုပ်

```
char greeting[6] = {'H', 'e', 'l', 'l', 'o', '\0'};
```

