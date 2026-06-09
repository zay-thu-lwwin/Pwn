
#### Tic Tac Toe

```c
└─$ file tictactoe 
tictactoe: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 4.4.0, BuildID[sha1]=6f3bdd1a7989d5ba761bc461c285bf98da6ca4e5, not stripped
                                                                  
```

```c

If you want Pwndbg to clear screen on each command (but still save previous output in history) use set context-clear-screen on
pwndbg> checksec
File:     /home/Jackfruit/cate/learn/binary/stack/htb/TicTacToed/pwn_tictactoed/tictactoe
Arch:     amd64
RELRO:      Full RELRO
Stack:      No canary found
NX:         NX enabled
PIE:        PIE enabled
Stripped:   No
pwndbg> 

```

buffer overflow တစ်ခုဘဲရအောင်လုပ်ထားတယ်

```c
pwndbg> i fun
All defined functions:

Non-debugging symbols:
0x000000000012b790  _start
0x000000000012b890  alloc::raw_vec::RawVec<T,A>::grow_one
0x000000000012b900  alloc::raw_vec::RawVec<T,A>::grow_one
0x000000000012b970  std::thread::local::LocalKey<T>::with
0x000000000012b9b0  std::thread::local::LocalKey<T>::with
0x000000000012b9f0  std::thread::local::LocalKey<T>::try_with
0x000000000012bac0  std::thread::local::LocalKey<T>::try_with
0x000000000012bb90  <D as digest::digest::Digest>::new
0x000000000012bbb0  <D as digest::digest::Digest>::update
0x000000000012bc10  <D as digest::digest::Digest>::finalize
0x000000000012bc30  digest::FixedOutput::finalize_fixed
0x000000000012bd60  <digest::core_api::wrapper::CoreWrapper<T> as digest::Update>::update
0x000000000012bd80  <digest::core_api::wrapper::CoreWrapper<T> as digest::Update>::update::{{closure}}
0x000000000012bd90  <digest::core_api::wrapper::CoreWrapper<T> as digest::FixedOutput>::finalize_into
0x000000000012bdd0  <digest::core_api::wrapper::CoreWrapper<T> as core::default::Default>::default
0x000000000012be60  alloc::slice::hack::into_vec
0x000000000012bec0  alloc::string::String::as_mut_str
0x000000000012bf00  alloc::string::String::len
0x000000000012bf10  alloc::string::String::new
0x000000000012bf50  alloc::string::String::push
0x000000000012bff0  alloc::string::String::as_str
0x000000000012c030  alloc::string::String::push_str
0x000000000012c050  <alloc::string::String as core::ops::deref::Deref>::deref
0x000000000012c060  <alloc::string::String as core::ops::deref::DerefMut>::deref_mut
0x000000000012c070  <alloc::string::String as core::cmp::PartialEq<&str>>::eq
0x000000000012c0f0  <alloc::string::String as core::cmp::PartialEq<&str>>::ne
0x000000000012c170  regex_automata::util::pool::inner::Pool<T,F>::guard_owned
0x000000000012c1a0  regex_automata::util::pool::inner::Pool<T,F>::guard_stack
0x000000000012c1d0  regex_automata::util::pool::inner::Pool<T,F>::guard_stack_transient
0x000000000012c200  regex_automata::util::pool::inner::Pool<T,F>::get
0x000000000012c2b0  regex_automata::util::pool::inner::Pool<T,F>::get::{{closure}}
0x000000000012c2c0  regex_automata::util::pool::inner::Pool<T,F>::get_slow
0x000000000012c8c0  regex_automata::util::pool::inner::Pool<T,F>::put_value
0x000000000012cae0  regex_automata::util::pool::inner::Pool<T,F>::put_value::{{closure}}
0x000000000012caf0  regex_automata::util::pool::inner::PoolGuard<T,F>::put
0x000000000012cc10  <core::iter::adapters::filter_map::FilterMap<I,F> as core::iter::traits::iterator::Iterator>::next
0x000000000012cc20  <core::iter::adapters::filter_map::FilterMap<I,F> as core::iter::traits::iterator::Iterator>::size_hint
0x000000000012cc60  core::iter::traits::iterator::Iterator::collect
0x000000000012cc80  <I as core::iter::traits::collect::IntoIterator>::into_iter
0x000000000012cca0  core::intrinsics::write_bytes::precondition_check
0x000000000012cdc0  <core::ops::range::Range<usize> as core::slice::index::SliceIndex<[T]>>::get_unchecked_mut::precondition_check
0x000000000012ce00  <usize as core::slice::index::SliceIndex<[T]>>::get_unchecked::precondition_check
0x000000000012ce20  <usize as core::slice::index::SliceIndex<[T]>>::get_unchecked_mut::precondition_check
0x000000000012ce40  <() as std::process::Termination>::report
0x000000000012ce50  core::num::<impl u32>::to_be_bytes
0x000000000012ce70  core::num::<impl u64>::to_be_bytes
0x000000000012ce90  core::num::<impl usize>::checked_add
0x000000000012cee0  core::num::<impl usize>::unchecked_add::precondition_check
0x000000000012cf00  core::mem::needs_drop
0x000000000012cf10  core::mem::drop
0x000000000012cf40  core::mem::forget
0x000000000012cf50  core::mem::forget
0x000000000012cf60  core::mem::replace
0x000000000012cf80  alloc::slice::<impl [T]>::join
0x000000000012cfa0  alloc::slice::<impl [T]>::into_vec
0x000000000012cfc0  core::sync::atomic::atomic_store
0x000000000012d0d0  core::sync::atomic::atomic_compare_exchange
0x000000000012d550  core::sync::atomic::atomic_compare_exchange
0x000000000012da30  <core::str::iter::CharIndices as core::iter::traits::iterator::Iterator>::next
0x000000000012db00  <core::str::iter::SplitWhitespace as core::iter::traits::iterator::Iterator>::next
0x000000000012db10  <core::str::iter::SplitWhitespace as core::iter::traits::iterator::Iterator>::size_hint
0x000000000012db30  <&u8 as core::ops::bit::Shr<i32>>::shr
0x000000000012db60  <&u8 as core::ops::bit::BitAnd<u8>>::bitand
0x000000000012db70  core::unicode::unicode_data::white_space::lookup
0x000000000012dc90  <std::fs::Permissions as std::os::unix::fs::PermissionsExt>::from_mode
0x000000000012dca0  <alloc::boxed::Box<F,A> as core::ops::function::Fn<Args>>::call
0x000000000012dcd0  tictactoe::detect_debugger
0x000000000012dd30  tictactoe::decrypt_key
0x000000000012dfd0  tictactoe::sha256_hash
0x000000000012e0d0  tictactoe::secure_zero_memory
0x000000000012e130  tictactoe::execute_c2
0x000000000012e2a0  tictactoe::validate_access_code
0x000000000012e4e0  tictactoe::ask_for_credentials
0x000000000012ead0  tictactoe::obfuscate_pattern
0x000000000012ec80  tictactoe::check_winner
0x000000000012f4c0  tictactoe::is_full
0x000000000012f4f0  tictactoe::main
0x0000000000130040  main
0x0000000000130060  std::sys::backtrace::__rust_begin_short_backtrace
0x0000000000130070  core::array::<impl core::iter::traits::collect::IntoIterator for &[T; N]>::into_iter
0x0000000000130090  core::array::<impl core::iter::traits::collect::IntoIterator for &[T; N]>::into_iter
0x00000000001300b0  std::sync::poison::map_result
0x0000000000130150  generic_array::hex::<impl core::fmt::LowerHex for generic_array::GenericArray<u8,T>>::fmt::{{closure}}
0x00000000001301a0  generic_array::hex::<impl core::fmt::LowerHex for generic_array::GenericArray<u8,T>>::fmt::{{closure}}
0x00000000001303b0  <alloc::alloc::Global as core::clone::Clone>::clone
0x00000000001303c0  alloc::alloc::alloc_zeroed
0x0000000000130410  alloc::alloc::exchange_malloc
...
```

function check ကြည့်တော့ ဒါတွေကျလာတယ်
main တစ်ခုတော့တွေ့တယ်

```c

void main(int param_1,undefined8 param_2)

{
  _ZN3std2rt10lang_start17hdb1fb4bd2383c243E
            (_ZN9tictactoe4main17hd88fd632884bb022E,(long)param_1,param_2,0);
  return;
}


```

မြင်နေကြအရာမျိုးမဟုတ်ဘူး
`gdb` မှာ runကြည့်တော့ debugger detected တဲ့

```c
pwndbg> r
Starting program: /home/Jackfruit/cate/learn/binary/stack/htb/TicTacToed/pwn_tictactoed/tictactoe 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/usr/lib/x86_64-linux-gnu/libthread_db.so.1".
[DEBUG] Debugger detected! Exiting...
[Inferior 1 (process 82162) exited with code 01]
pwndbg> 

```

AI မေးကြည့်တော့
အခု binary မှာတော့ ထူးထူးဆန်းဆန်း `alloc::raw_vec...`, `std::thread...`, `regex_automata...` ဆိုတဲ့ ရှည်လျားထွေပြားလှတဲ့ နာမည်ကြီးတွေ တန်းစီပြီး ထွက်လာတာကို တွေ့ရမယ်
ဘာလို့လဲဆို ကိုင်တွယ်နေရတဲ့ binary က C သို့မဟုတ် C++ နဲ့ ရေးထားတာ မဟုတ်ဘဲ **Rust Language** နဲ့ ရေးထားတဲ့ Binary ဖြစ်နေလို့တဲ့

ဆိုတော့
ငါတို့ Custom logic တွေရှာကြည့်မယ်


