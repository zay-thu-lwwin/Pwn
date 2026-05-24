
Kernel Exploits
Browser Exploits
Zero Day exploit
software နဲ့ OS တွေမှာရှိတဲ့ binary vulnerabilities
To play in Pwn2Own
companyကြီးတွေရဲ့ infrastructure တွေကို တကယ့် hacker အဆင့်မြင့်တွေအတိုင်း binary အဆင့်ကနေ ထိုးဖောက်ပြီး security အခြေအနေကို စမ်းသပ်ပေးရ

Blockchain & Smart Contract Auditing (Web3 Security)
ဒီဘက်ခေတ် Web3 နဲ့ Crypto လောကမှာ Smart Contract တွေဟာ code အမှားတစ်ခုတည်းနဲ့တင် ဒေါ်လာသန်းပေါင်းများစွာ ဆုံးရှုံးသွားနိုင်ပါတယ်။ Web3 မှာ သုံးတဲ့ Virtual Machines တွေရဲ့ compiler binary တွေ၊ core protocol memory တွေကို စစ်ဆေးပေးရတဲ့ **Web3 Auditor** တွေဟာ standard web developers တွေထက် ဝင်ငွေ အဆမတန် ပိုမိုရရှိကြပါတယ်

**CVE (Common Vulnerabilities and Expositions) ရအောင်ယူပါ:** Public software တစ်ခုခုမှာ bug အသစ်ရှာတွေ့ပြီး သက်ဆိုင်ရာအဖွဲ့အစည်းက အသိအမှတ်ပြုရင် CVE ID ထုတ်ပေးပါတယ်။ CV (ကိုယ်ရေးအကျဉ်း) မှာ CVE ID ပါလာပြီဆိုရင် အလုပ်အကိုင်နဲ့ ဝင်ငွေလမ်းကြောင်းဟာ အလိုလို ပွင့်လာ

---

Real-world software တွေ (ဥပမာ - Adobe Reader, Windows Kernel, Chrome Browser) ထဲက ဘယ်သူမှမသိသေးတဲ့ **Zero-Day** binary vulnerabilities တွေကို ရှာဖွေဖို့ဆိုတာ လက်နဲ့ လိုက်နှိပ်ကြည့်ရုံ၊ စမ်းကြည့်ရုံနဲ့ မရနိုင်
အဆင့်မြင့် Security Researcher တွေ သုံးတဲ့ စနစ်တကျ အလုပ်လုပ်ပုံ နည်းလမ်းနဲ့ မှရ


#### Real World How to achieve

အဓိကအားဖြင့် နည်းလမ်း ၃ ခုကို သုံးပြီး အားနည်းချက်တွေကို ရှာဖွေကြပါတယ်:

##### Fuzzing (Fuzz Testing)  (most common)

Real-world binary ကြီးတွေကို လူကထိုင်ပြီး code တစ်ကြောင်းချင်း လိုက်ဖတ်ဖို့ မလွယ်ကူပါဘူး။ ဒါကြောင့် **Fuzzing** နည်းလမ်းကို သုံးပါတယ်။ Fuzzing ဆိုတာ ပရိုဂရမ်တစ်ခုထဲကို ပုံမှန်မဟုတ်တဲ့၊ ပျက်စီးနေတဲ့ data (Malformed inputs) သန်းပေါင်းများစွာကို automation tools တွေသုံးပြီး အတင်းထည့်သွင်း (Inject) တာ ဖြစ်ပါတယ်။

အထက်ပါ ပုံမှာ မြင်ရတဲ့အတိုင်း ပရိုဂရမ်ကို target ထားပြီး input တွေ အဆက်မပြတ် ထည့်သွင်းမောင်းနှင်ပါတယ်။ အဲဒီလို ထည့်လိုက်တဲ့အခါ program က မခံနိုင်ဘဲ **Crash (ရပ်ဆိုင်း)** သွားရင် အဲဒီနေရာမှာ Memory Leak သို့မဟုတ် Vulnerability ရှိနေပြီလို့ သတ်မှတ်ပြီး ပညာရှင်တွေက ၎င်း Crash ဖြစ်သွားတဲ့နေရာကို အသေးစိတ် ခွဲခြမ်းစိပ်ဖြာ (Analysis) ကြပါတယ်။ တကယ့် 0-day အများစုဟာ ဒီလို **Fuzzing** လုပ်ရင်းနဲ့ ထွက်လာတာ ဖြစ်ပါတယ်။

##### Source Code Review (White-box Testing)

အကယ်၍ ကိုယ်စမ်းသပ်မယ့် software က open-source ဖြစ်နေရင် (ဥပမာ - Linux Kernel သို့မဟုတ် VLC Media Player) ၎င်းရဲ့ C/C++ source code တွေကို လုံုံခြုံရေးအမြင်နဲ့ လိုက်လံဖတ်ရှုပြီး developer တွေ မှားရေးထားတဲ့ နေရာတွေကို ရှာဖွေတာ ဖြစ်ပါတယ်။

##### Reverse Engineering (Black-box Testing)

Closed-source software တွေ (ဥပမာ - Windows OS အစိတ်အပိုင်းများ) ဆိုရင်တော့ source code မရှိတဲ့အတွက် binary file ကြီးကို **Ghidra** သို့မဟုတ် **IDA Pro** ထဲ ထည့်ပြီး Assembly code အဖြစ် ပြန်ပြောင်းကာ memory handle လုပ်တဲ့ logic တွေကို စေ့စေ့စပ်စပ် လေ့လာရှာဖွေရပါတယ်။



----

#### How to learn Roadmap

##### Phase 1: CTF (Beginner)

တကယ့် software ကြီးတွေကို တန်းစမ်းရင် စိတ်ပျက်သွားပါလိမ့်မယ်။ အရင်ဆုံး အားနည်းချက် တမင်ထည့်ထားတဲ့ binary ပဟေဠိလေးတွေကို CTF ကစားရင်း အကျွမ်းတဝင်ဖြစ်အောင် လုပ်ပါ။
- **လေ့လာရန် Platform များ:**
    - **pwn.college:** ဒါဟာ ကမ္ဘာပေါ်မှာ Binary Exploitation အတွက် အကောင်းဆုံး Free Course ဖြစ်ပါတယ်။ ဗီဒီယိုတွေရော၊ လက်တွေ့စမ်းသပ်ရမယ့် lab တွေပါ စနစ်တကျ ရှိပါတယ်။
    - **Nightmare:** CTF binary challenges တွေကို တစ်ဆင့်ချင်း ဘယ်လိုတွေး၊ ဘယ်လိုပုန်းရမလဲဆိုတာ သင်ပေးတဲ့ open-source book/course ဖြစ်ပါတယ်။
- **ရရှိမယ့် အရည်အချင်း:** Buffer Overflow အခြေခံများ၊ Stack Canary ကျော်ဖြတ်နည်း၊ ASLR/NX ကဲ့သို့ security mitigations တွေကို ကျော်ဖြတ်ပြီး exploit code ရေးတတ်လာခြင်း။
    
##### Phase 2: learning Real-World 0-Day Tools (Intermediate)

ဒီအဆင့်မှာ Automation တိုက်ခိုက်မှုတွေ လုပ်ဖို့အတွက် Fuzzing tools တွေကို သုံးတတ်အောင် သင်ယူရမယ်
- **လေ့လာရန် Tools များ:**
    - **AFL++ (American Fuzzy Lop):** ကမ္ဘာပေါ်မှာ အသုံးအများဆုံး ကောင်းမွန်တဲ့ fuzzer တစ်ခု ဖြစ်ပါတယ်။ Binary တစ်ခုကို AFL++ သုံးပြီး ဘယ်လို fuzz ရမလဲဆိုတာ သေချာလေ့လာပါ။
    - **LibFuzzer / Honggfuzz:** C/C++ code တွေကို fuzz လုပ်ဖို့ သုံးတဲ့ အဆင့်မြင့် tools များ။
- **လေ့လာရန် လမ်းကြောင်း:** GitHub ပေါ်မှာရှိတဲ့ တကယ့် researcher တွေရဲ့ 0-day write-ups တွေကို ရှာဖတ်ပါ။ သူတို့ bug တွေ့သွားတဲ့အခါ ဘယ်လိုပုံစံနဲ့ တွေ့တာလဲ၊ ဘယ်လို fuzzing လုပ်ခဲ့လဲဆိုတဲ့ နည်းဗျူဟာ (Strategy) ကို အတုယူပါ။
    

##### Phase 3: Testing Public CVE  (Advanced)

လူသိများပြီးသား (ဟောင်းနေတဲ့) အားနည်းချက်တွေဖြစ်တဲ့ **CVE (Common Vulnerabilities and Exposures)** တွေကို လေ့လာပြီး real-world application အဟောင်းတွေပေါ်မှာ ကိုယ်တိုင် exploit ပြန်ရေးကြည့်ပါ။
- ဥပမာ - လွန်ခဲ့တဲ့ ၂ နှစ်က bug ရှိခဲ့တဲ့ Adobe Reader version အဟောင်းတစ်ခုကို ဒေါင်းလုဒ်ချပါ။ အဲဒီတုန်းက သူများတွေ ရေးထားတဲ့ exploit logic ကို ဖတ်ပြီး ကိုယ့်စက်ထဲမှာ run အောင် ပြန်လုပ်ကြည့်ပါ။
- ဒီအဆင့်ကို ကျွမ်းကျင်သွားရင် လက်ရှိသုံးနေတဲ့ application အသစ်တွေပေါ်မှာ ကိုယ်ပိုင် Fuzzing setup တွေဆောက်ပြီး Zero-Day ရှာဖွေတဲ့ လုပ်ငန်းစဉ်ကို စတင်နိုင်ပြီ ဖြစ်ပါတယ်။
    

> 💡 **အကြံပြုချက်:** လေ့လာတဲ့အခါ ဇွဲရှိဖို့ အလွန်လိုပါတယ်။ Pwn ပညာရပ်ဟာ တစ်ခါတရံ ရက်ပေါင်းများစွာ၊ သီတင်းပတ်ပေါင်းများစွာ ဘာ bug မှ ရှာမတွေ့ဘဲ စိတ်ပျက်စရာ ကောင်းတတ်ပါတယ်။ ဒါပေမဲ့ bug တစ်ခု မိသွားပြီဆိုရင်တော့ ထိုက်တန်တဲ့ ပီတိနဲ့ အကျိုးအမြတ်ကို ပေးစွမ်းနိုင်တဲ့ ပညာရပ် ဖြစ်ပါတယ်။