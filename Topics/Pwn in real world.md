
ဒီနေ့ခေတ် ကမ္ဘာမှာ buffer overflow ဖြစ်တာတွေ သိပ်မတွေ့ရတော့ဘူး။ ဘာလို့လဲဆိုတော့ ခေတ်ပေါ် compiler တွေမှာ memory ကာကွယ်ရေး အစီအစဉ်တွေ ထည့်သွင်းပြီးသားဖြစ်လို့ memory ပျက်စီးစေတဲ့ bug တွေ မတော်တဆဖြစ်ဖို့ ခက်ခဲသွားတယ်။ ဒါပေမယ့် C လိုမျိုး language တွေ ဘယ်တော့မှ ပျောက်ကွယ်သွားမှာမဟုတ်ဘူး။ အထူးသဖြင့် embedded software နဲ့ IOT တွေမှာ အဓိကသုံးနေကြတုန်းပဲ။

ကျွန်တော်ကြိုက်တဲ့ မကြာသေးခင်က buffer overflow တစ်ခုကတော့ CVE-2021-3156 ဖြစ်တယ်။ ဒါက sudo ထဲက Heap-Based Buffer Overflow ဖြစ်တယ်။

ဒီတိုက်ခိုက်မှုတွေက binary တွေထဲမှာပဲ ရှိတာမဟုတ်ဘူး။ web application တွေ၊ အထူးသဖြင့် စိတ်ကြိုက် webserver တွေသုံးထားတဲ့ embedded device တွေမှာလည်း buffer overflow တွေ အများကြီးဖြစ်တယ်။ ဥပမာကောင်းတစ်ခုက HP iLO Management device တွေမှာ ဖြစ်ခဲ့တဲ့ CVE-2017-12542 ပါ။ HTTP Header parameter တစ်ခုထဲကို စာလုံး ၂၉ လုံးပဲ ပို့လိုက်ရုံနဲ့ login ကို ကျော်ပြီး buffer overflow ဖြစ်သွားတယ်။ ဒီဥပမာကို ကျွန်တော်ကြိုက်တာက payload လိုတောင်မလိုဘူး။ system က error ရောက်သွားတာနဲ့ "failed open" ဖြစ်သွားလို့ပါ။

ခြုံပြောရရင် buffer overflow ဖြစ်တာက ပရိုဂရမ်ကုဒ် မှားနေလို့ပါ။ CPU က data အများကြီးကို မှန်မှန်မကိုင်တွယ်နိုင်တဲ့အခါ CPU ရဲ့ processing ကို ခြယ်လှယ်နိုင်သွားတယ်။ ဥပမာ reserved memory buffer ဒါမှမဟုတ် stack ထဲကို data အရမ်းများများရေးလိုက်ရင် register အချို့ကို ထပ်ရေးမိသွားပြီး code ကို execute လုပ်ခွင့်ရသွားနိုင်တယ်။

Buffer overflow က ပရိုဂရမ်ကို crash ဖြစ်စေတာ၊ data တွေပျက်စီးစေတာ၊ ဒါမှမဟုတ် program runtime ထဲက data structure တွေကို ထိခိုက်စေနိုင်တယ်။ နောက်ဆုံးအချက်က program ရဲ့ return address ကို လိုသလိုပြောင်းပြီး attacker က သူပို့လိုက်တဲ့ machine code တွေကို ခွင့်ပြုချက်နဲ့ execute လုပ်နိုင်စေတယ်။ ဒီ code က များသောအားဖြင့် system ကို ကိုယ်လိုသလို access ရအောင် လုပ်ဖို့ပါ။ ဒီလို buffer overflow တွေက server တွေမှာဖြစ်တတ်ပြီး internet worm တွေကလည်း client software တွေကို exploit လုပ်ကြတယ်။

Unix စနစ်တွေမှာ လူကြိုက်များတဲ့ target က root access ပါ။ ဒါက system ကို အပြည့်အဝခွင့်ပြုချက်ပေးတယ်။ ဒါပေမယ့် လူအများနားလွဲတတ်တာက standard user privilege "ပဲ" ရတဲ့ buffer overflow က အန္တရာယ်မရှိဘူးလို့ ထင်ကြတယ်။ တကယ်တမ်းက user privilege ရပြီးသားဆိုရင် root access ရဖို့က ပိုလွယ်သွားတယ်။

Buffer overflow တွေက programming ပေါ့ဆမှုအပြင် Von-Neumann architecture ကို အခြေခံတဲ့ ကွန်ပျူတာစနစ်တွေကြောင့် အဓိကဖြစ်နိုင်တယ်။

Buffer overflow ဖြစ်ရခြင်း အကြီးမားဆုံးအကြောင်းရင်းက memory buffer ဒါမှမဟုတ် stack ရဲ့ ကန့်သတ်ချက်တွေကို အလိုအလျောက် စောင့်ကြည့်မပေးတဲ့ programming language တွေကို သုံးတာပါ။ ဥပမာ C နဲ့ C++ တို့က performance ကို အဓိကထားတဲ့အတွက် စောင့်ကြည့်ဖို့ မလိုအပ်ဘူးလို့ သတ်မှတ်ထားတယ်။

ဒါကြောင့် developer တွေက ဒီလို memory နေရာတွေကို ကိုယ်တိုင် code ထဲမှာ သတ်မှတ်ပေးရတယ်။ ဒါက vulnerability ကို အဆမတန်များစေတယ်။ ဒီနေရာတွေကို testing အတွက်ဖြစ်ဖြစ်၊ ပေါ့ဆလို့ဖြစ်ဖြစ် မသတ်မှတ်ဘဲထားတတ်ကြတယ်။ testing အတွက်သုံးထားရင်တောင် development ပြီးခါနီးမှာ မေ့ကျန်ခဲ့တတ်ကြတယ်။

ဒါပေမယ့် application တိုင်းလိုလို buffer overflow ဖြစ်နိုင်ခြေ ရှိချင်မှရှိမယ်။ ဥပမာ standalone Java application ကတော့ Java က memory ကို ဘယ်လိုစီမံခန့်ခွဲသလဲဆိုတဲ့ သဘောအရ တခြား language တွေထက် buffer overflow ဖြစ်နိုင်ခြေ အနည်းဆုံးပါ။ Java က "garbage collection" technique ကို သုံးပြီး memory ကို စီမံခန့်ခွဲတယ်။ ဒါက buffer overflow ဖြစ်တာကို ကာကွယ်ပေးတယ်။