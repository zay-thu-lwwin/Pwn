
## Bad Characters ကို Program က ဘာလုပ်ပစ်သလဲ?

ရှာတွေ့ထားတဲ့ bad characters တွေကို program က အောက်ပါအတိုင်း ဖျက်ဆီးပစ်တယ်-

| Bad Character | Program က ဘာလုပ်သလဲ |
|---|---|
| `\x00` (Null) | String terminator အဖြစ် သတ်မှတ်ပြီး ဒီနေရာမှာ ရပ်လိုက်တယ်။ နောက် bytes တွေ ဆက်မဖတ်တော့ဘူး။ |
| `\x09` (Tab) | ပြောင်းလဲသွားတယ်။ များသောအားဖြင့် space (သို့) null ဖြစ်သွားတယ်။ |
| `\x0a` (Line Feed) | Input delimiter အဖြစ် သတ်မှတ်ပြီး လိုင်းဆုံးတယ်လို့ ထင်သွားတယ်။ |
| `\x20` (Space) | Argument separator အဖြစ် သတ်မှတ်တယ်။ Command-line argument တွေခွဲတဲ့အခါ ဖြတ်တောက်ခံရတယ်။ |

### ဘာကြောင့် ဒီလိုဖြစ်တာလဲ?

အဓိက အကြောင်းရင်းက **program က input ကို ဘယ်လို handle လုပ်သလဲ** ဆိုတာပေါ် မူတည်တယ်။

1. **String Functions** (`strcpy`, `strlen`, `strcat`) - `\x00` မှာ ရပ်သွားတယ်
2. **Input Parsing** - `\x0a` (enter) က input အဆုံးလို့ ထင်တယ်
3. **Command-line Parsing** - Space (`\x20`) က argument ခွဲတဲ့နေရာမှာ delimiter အဖြစ် သုံးတယ်
4. **Encoding/Decoding** - တချို့ characters တွေကို escape လုပ်တာ (သို့) ပြောင်းလဲပစ်တယ်

---

## Program တစ်ခုအတွက် Bad Characters က တူလား?

**မတူပါဘူး။** Program တစ်ခုချင်းစီအတွက် bad characters တွေ ကွဲပြားနိုင်တယ်။

ဥပမာ-

| Program အမျိုးအစား | ဖြစ်နိုင်ချေရှိတဲ့ Bad Characters |
|---|---|
| Web server (HTTP) | `\x00`, `\x0a`, `\x0d`, `\x20`, `\x25` (%) |
| FTP server | `\x00`, `\x0a`, `\x0d` |
| Database | `\x00`, `\x27` ('), `\x22` ("), `\x3b` (;) |
| Binary protocol | bad characters နည်းနည်းပဲရှိတတ်တယ် |
| memcpy သုံးတဲ့ program | bad characters လုံးဝမရှိတာမျိုး ဖြစ်နိုင်တယ် |

---

## Program အားလုံးအတွက် ဖြစ်နိုင်ချေရှိတဲ့ Bad Characters တွေက အတူတူဘဲလား?

**မဟုတ်ဘူး။** ဒါပေမယ့် သတိထားစရာ အချက်တွေ ရှိတယ်။

### အမြဲလိုလို Bad ဖြစ်တတ်တဲ့ Characters (Common Bad Characters)

| Character | ဘာကြောင့် |
|---|---|
| `\x00` | C string terminator - အားလုံးနီးပါးမှာ bad |
| `\x0a` | Line feed - input delimiter |
| `\x0d` | Carriage return - delimiter အဖြစ် သုံးများတယ် |

### Program အမျိုးအစားပေါ်မူတည်တဲ့ Bad Characters

| Character | ဘယ်အခါမှာ Bad ဖြစ်တတ်သလဲ |
|---|---|
| `\x20` (space) | Command-line arguments, HTTP URLs, shell commands |
| `\x09` (tab) | Text parsing, configuration files |
| `\x2f` (/) | File paths, URLs |
| `\x5c` (\) | Windows paths |
| `\x27` (') | SQL queries, shell commands |
| `\x22` (") | SQL, JSON, CSV |
| `\x3b` (;) | SQL, shell commands separator |
| `\x26` (&) | URL parameters, shell background |

---

## အကျဉ်းချုပ်

| မေးခွန်း | အဖြေ |
|---|---|
| Program က bad character ကို ဘာလုပ်သလဲ? | ဖျက်ပစ်တယ်၊ ပြောင်းလဲပစ်တယ်၊ ဒါမှမဟုတ် ရပ်ပစ်လိုက်တယ် |
| Program တစ်ခုအတွက် bad character က တူလား? | **မတူဘူး** - target ပေါ်မူတည်တယ် |
| Program အားလုံးအတွက် bad character တွေ အတူတူလား? | **မတူဘူး** - ဒါပေမယ့် `\x00`, `\x0a`, `\x0d` တို့က များသောအားဖြင့် bad ဖြစ်တတ်တယ် |

**အရေးကြီးဆုံးက** - ကိုယ်တိုင် စမ်းသပ်ပြီး target program အတွက် bad characters တွေကို ရှာရမယ်။ ဘယ်သူ့ပြောကိုမှ အလွယ်တကူ မယုံဘူး။ လက်တွေ့ run ကြည့်ပြီးမှ သတ်မှတ်ရမယ်။