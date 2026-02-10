Function စခေါ်တဲ့အခါ မှာ saved `ebp` ပြီးရင် canary value ကိုထည့်ထားတယ်
Function ပြီးသွားရင် canary value ပျက်သွားလားစစ်
Value ပြောင်းသွားရင် program က crash ဖြစ် (Stack Smashing Detected!)
canary value က program run time တစ်ခုမှာ function တိုင်းအတွက်တူတယ်

## Bypass 

	Canary value ကို leak လုပ်နိုင်ရင် 
	stack canary protection ကို bypass လုပ်လို့ရ

	Technique 1
	prinf vuln နဲ့ canary value ကိုဖတ်ယူ
	Buffer overflow လုပ်ရင် မပျက်အောင် အဲ့တန်ဖိုးကိုပြန်ထည့် 