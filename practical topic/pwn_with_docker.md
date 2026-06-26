
 Why do we use Docker? (Docker ကို ဘာလို့ သုံးရတာလဲ?

`Pwnable` ပုစ္ဆာတွေမှာ Docker ကို သုံးရတဲ့ အဓိကအကြောင်းရင်းက ပုစ္ဆာထုတ်တဲ့သူရဲ့ Server အခြေအနေအတိုင်း 100% တထပ်တည်းကျတဲ့ Sandbox environment (Exact Environment Match) ကို local ထဲမှာတည်ဆောက်ချင်လို့ဖြစ်တယ်
တကယ့် Remote Server ပေါ်မှာ Program က ဘယ်လို Memory ချလဲ၊ Network ကနေ Input/Output တွေကို ဘယ်လိုလက်ခံလဲဆိုတာကို local Kali Linux ပေါ်မှာ ပုံတူကူးယူပြီး Exploit ရေးရတာ သေချာစေဖို့အတွက် သုံးတာဖြစ်တယ်



Can we use ONLY `patchelf`? (`patchelf` တစ်ခုတည်းနဲ့တင် မရဘူးလား?)

ရပါတယ် တကယ်တော့ `Pwn Target `တော်တော်များများမှာ `patchelf` (သို့မဟုတ် `pwninit`) ကို သုံးပြီး challenge ကပေးတဲ့ `libc.so.6` နဲ့ `ld.so` ကို binary ထဲ ဇွတ်လမ်းကြောင်းလွှဲ (patch) လိုက်ရင် Local ရော Remote ရော Libc Function Offset ချင်း အတူတူပဲ ဖြစ်သွားမှာမို့လို့ တွက်ချက်ရတာ အဆင်ပြေတယ်


ဒါဆို ဘာလို့ Docker လိုနေသေးလဲ? (The Differences)

`patchelf` က Binary ထဲက `Libc` ချိတ်ဆက်မှုကိုပဲ ပြောင်းပေးနိုင်တာပါ 
ဒါပေမဲ့ OS ရဲ့ Memory Stack နေရာချပုံနဲ့ Sandbox Jail Environmental Factor တွေကို မပြောင်းလဲနိုင်ပါဘူး

1. **Stack Alignment ပြဿနာ (`MOVAPS` Crash):** local Kali Linux Kernel က Binary ကို Process memory ထဲ တင်ပေးတဲ့အချိန် Stack Address တွေ နေရာချပုံက Server ပေါ်က Ubuntu နေရာချပုံနဲ့ မတူနိုင်ပါဘူး ဒါကြောင့် `patchelf` လုပ်ထားတဲ့ binary က local စက်ပေါ်မှာ run ရင် Shell ရပေမယ့်၊ Remote Server ကို ပစ်တဲ့အခါကျမှ Stack Alignment က 8 bytes လွဲနေလို့ Crash ဖြစ်သွားတတ်ပါတယ်
    
2. **Network/Jail Environment:** တကယ့် Server ပေါ်မှာ Binary က Network Port ကနေ input ယူဖို့ `pwn.red/jail` လို Sandbox ခံထားရတာပါ Docker က အဲ့ဒီ Network အလုပ်လုပ်ပုံနဲ့ ခံထားတဲ့ Jail စနစ်ပါ Remote နဲ့ တထပ်တည်းတူအောင် လုပ်ပေးနိုင်ပါတယ်
    

##### Pros and Cons: Docker vs `Patchelf`


| **Tool**             | **Pros (အားသာချက်)**                                                                                                                                                      | **Cons (အားနည်းချက်)**                                                                                                                                                                    |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Docker Container** | • Remote Server နဲ့ **100% တထပ်တည်းကျတယ်**<br><br>  <br><br>• Stack Alignment လွဲပြီး Crash တဲ့ပြဿနာ မရှိဘူး။<br><br>  <br><br>• Network Connection အလုပ်လုပ်ပုံ တူညီတယ်။ | • Setup လုပ်ရတာ နည်းနည်းလေး ပိုလက်ဝင်တယ်<br><br>  <br><br>• Docker ထဲက process ကို Native `gdb` နဲ့ တိုက်ရိုက်လှမ်းကြည့်ရတာ ခက်ခဲတတ်တယ် (Remote Debug setup လိုအပ်)                       |
| **Patchelf**         | • **အရမ်းမြန်တယ်၊ ပေါ့ပါးတယ်**<br><br>  <br><br>• မင်းရဲ့ Native Kali ပေါ်မှာတင် `gdb ./patched_binay` ဆိုပြီး စိတ်ကြိုက် လျင်လျင်မြန်မြန် Debug လုပ်လို့ရတယ်။            | • Stack Alignment ကံဆိုးရင် လွဲတတ်တယ်။<br><br>  <br><br>• တချို့ Ubuntu ဗားရှင်းအဟောင်းက `libc` တွေကို Modern Kali Kernel ပေါ်မှာ ဇွတ် run ရင် `SegFault` ပြပြီး ပွင့်တောင်ပွင့်မလာတတ်ဘူး |

---

##### How to use the `Dockerfile` from the challenge?

####################

##### 1. Make flag

Docker ထဲမှာ Program အလုပ်လုပ်တဲ့အခါ `flag.txt` မရှိရင် Error တက်တတ်လို့ လက်ရှိ folder ထဲမှာ အရင်ဆောက်

```bash
echo "boroCTF{this_is_a_local_fake_flag}" > flag.txt
```

##### 2. Build docker image in `pwd`

`Dockerfile` ရှိတဲ့ `pwd` မှာ ရိုက် (အနောက်က `.` လေး ပါအောင်ရိုက်ပါ)


```bash
docker build -t pwn_challenge .
# pwn_challenge is image name as you wish
```

_(ဒါဆိုရင် Docker က လိုအပ်တဲ့ Ubuntu OS နဲ့ ဖိုင်တွေကို Blueprint အတိုင်း စုစည်းထုပ်ပိုးလိုက်ပါပြီ)_



##### 3. Run docker container

CTF ပုစ္ဆာတွေမှာ ပါတဲ့ Sandbox Jail တွေ ကောင်းကောင်းအလုပ်လုပ်နိုင်ဖို့ `--privileged` flag ထည့်ပြီး Run ပေးရပါမယ်

```bash
docker run -d -p 1337:1337 --privileged --name my_pwn_container pwn_challenge
# my_pwn_container is container name as you wish
```
_(အခုဆိုရင်  localထဲမှာတင် Port `1337` မှာ CTF Mini-Server ကြီး အောင်မြင်စွာ ပွင့်သွားပြီ)_



##### 4. Test it

Python Exploit Script ထဲမှာ တကယ့် Remote ကို exploit မလုပ်ခင်  Local Docker ကို အရင်စမ်းလို့ ရပြီ

```python
from pwn import *
io = remote("127.0.0.1", 1337) # Local Docker ကို ချိတ်ခြင်း
# io = remote("challenge.remote.server", 34069) # Remote Server ကို ချိတ်ခြင်း
```

#### Standard Workflow
 
ဒါဆို ငါတို့ ဘယ်လမ်းကြောင်းကို အရင်သွားရမလဲ?

1. **First Step:** Binary ကို `patchelf` အရင်လုပ်၊ ပြီးရင် Kali ပေါ်မှာတင် `gdb` နဲ့ အမြန်ဆုံး Overflow ဖြစ်မယ့် Offset (စာလုံးအရေအတွက်) ကို အရင်ရှာပါ။ (Offset က ဘယ် OS မှာမဆို အတူတူပဲမို့လို့ ကာလီပေါ်မှာ ရှာတာ ပိုမြန်ပါတယ်)
2. **Second step:** Offset ရပြီဆိုမှ Payload ကို အပြီးသတ်ရေးပြီး **Local Docker Container** ကို run ပြီး `remote("127.0.0.1", 1337)` နဲ့ exploit လုပ် အဲ့ဒီမှာ Shell ရပြီဆိုရင် တကယ့် Remote Server ကို exploit ဖို့ 100% သေချာပြီ

----


`docker run` ပြီးသွားရင် terminal ထဲက binary က remote က binary လို ဖြစ်သွားပြီလား?

Native Terminal (Kali) ပေါ်မှာ ရှိနေတဲ့ မူရင်း binary ကတော့ ပြောင်းလဲမသွားပေမယ့်၊ Docker Container အတွင်းထဲ ရောက်သွားတဲ့ binary ကတော့ Remote Server က အခြေအနေအတိုင်း (ပုစ္ဆာပေးထားတဲ့ `libc`, `ld` လမ်းကြောင်းတွေ၊ `redpwn jail` Sandbox တွေနဲ့) တစ်ထပ်တည်း အလုပ်လုပ်နေပါပြီ
 `docker run -d -p 1337:1337 --privileged --name my_container pwn_challenge` လို့ ရိုက်လိုက်တာနဲ့ Local Port `1337` ပေါ်မှာ တကယ့် Remote Server တိုင်းကျတဲ့ Environment တစ်ခု နောက်ကွယ်မှာ အသက်ဝင်ပြီး Run နေပြီ


#### (Next Steps)

Container Run နေပြီဆိုရင် 

##### check container status
```bash
docker ps
```

ဒီ Command ကို ရိုက်လို့ `STATUS` နေရာမှာ `Up ...` ဆိုပြီး ပြနေရင်  Lab Server  Live ဖြစ်နေပြီ

##### - Test  Local Network Connection က

Terminal အသစ်တစ်ခုဖွင့်ပြီး `nc` (Netcat) နဲ့ လှမ်းချိတ်ကြည့်

```bash
nc localhost 1337
```


---

#### How to Debug with docker ?


###########

#### first way:install `pwngdb` in docker

ဒါက အရှင်းဆုံးနဲ့ အလုပ်အဖြစ်ဆုံး လမ်း
Container ထဲကို `docker exec` နဲ့ ဝင်ပြီး `pwndbg` ကို install

##### connect container via sh

```bash
docker exec -it my_pwn_container /bin/sh
```

##### install packages

Container ထဲမှာ `pwndbg` ကို ဒေါင်းဖို့အတွက် `git` နဲ့ `curl` တွေ လိုအပ်ပါတယ်။ (Ubuntu container မို့လို့ `apt` သုံးရပါမယ်)

```bash
apt update && apt install -y git curl python3-pip
```

##### clone pwngdb and make setup

`pwndbg` ရဲ့ တရားဝင် GitHub ကနေ down ပြီး Container ထဲမှာ Auto-install လုပ်ခိုင်း

```bash
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
```

_(ဒီအဆင့်မှာ Setup ပြီးအောင် ခဏစောင့်ပေးရပါမယ်)_


##### find PID in container and attach with pwngdb

Setup ပြီးသွားရင် ပုံမှန်အတိုင်း `ps aux` နဲ့ ပုစ္ဆာရဲ့ Process ID (`PID`) ကို ရှာ
(ဥပမာ- `PID` က `42` ဆိုပါစို့)။ ပြီးရင် အခုလို လှမ်းချိတ်လိုက်

Container ရဲ့ Shell ထဲမှာပဲ အခုလို ရိုက်ထည့်

```bash
ps aux
```

ဒါဆိုရင် Container ထဲမှာ Run နေတဲ့ Program စာရင်းတွေ ကျလာလိမ့်မယ်။ ဥပမာအားဖြင့် အောက်ပါအတိုင်း တွေ့ရတတ်ပါတယ်:


```
USER       PID   %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1    0.0  0.0   4292   752 ?        Ss   12:00   0:00 /usr/bin/socat ...
root        42    0.0  0.0  16832  2104 ?        S    12:01   0:00 ./dinner_party
```

```
gdb -p 42
```

ဒါဆိုရင် Registers တန်ဖိုးတွေ၊ Stack view တွေနဲ့ `pwngdb`  interface Docker Container ထဲမှာတင် ပွင့်လာပါလိမ့်မယ်


----

#### second way :install gdbserver and attach with local

တကယ်လို့ Container ထဲမှာ Tools တွေ အများကြီး လျှောက်မသွင်းချင်ဘူး
Native Kali Linux ပေါ်မှာ ရှိပြီးသား `pwndbg` ကြီးနဲ့ပဲ Network ကနေတစ်ဆင့် လှမ်းကြည့်ချင်တယ်ဆိုရင် Remote Debugging လုပ်ရမယ်

#1 Docker Run တဲ့ Command မှာ Debug Port (ဥပမာ 1234) ထပ်ဖွင့်ပေးပါ

မအစောက run ခဲ့တဲ့ container ကို ဖျက်ပြီး `gdbserver` အတွက် port `1234` ထပ်တိုးပြီး ပြန် run ပေးရ

```bash
docker rm -f my_pwn_container
docker run -d -p 1337:1337 -p 1234:1234 --privileged --name my_pwn_container pwn_challenge
```

#2 Container ထဲဝင်ပြီး `gdbserver` ကို သွင်း

```bash
docker exec -it my_pwn_container /bin/sh
apt update && apt install -y gdbserver
```

#2 `gdbserver` နဲ့ ပုစ္ဆာ binary ရဲ့ PID ကို ချိတ်ပြီး Port 1234 မှာလွှင့်

`ps aux` နဲ့ ပုစ္ဆာရဲ့ PID (ဥပမာ `42`) ကို အရင်ရှာပြီး အခုလို run

```bash
gdbserver 0.0.0.0:1234 --attach 42
```

_(ဒါဆိုရင် Container ထဲက binary ကြီး Freeze ဖြစ်သွားပြီး kaliဘက်က လှမ်းချိတ်မှာကို စောင့်နေပါလိမ့်မယ်)_


#3 Kali Linux (Native Terminal) ဘက်ကနေ လှမ်းချိတ်

 kaliမှာ ပုံမှန် terminal တစ်ခုဖွင့်၊ `gdb` ကို ရိုက်ဖွင့်ပြီး `pwndbg` ထဲရောက်ရင် အောက်ပါ command နဲ့ လှမ်းချိတ်လိုက်

```
pwndbg> target remote localhost:1234
```

----


