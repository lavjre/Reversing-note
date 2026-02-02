_difficulty: 2.0_
## Intro
Let's starting with the description of challenge:
Welcome to the Existential Vending Machine. This is no ordinary machine; it doesn't just dispense snacks, it judges the weight of your soul. To obtain the "Ultimate Snack" (item C3), you must prove your worth by entering the correct access code that resonates with your unique identity. Designed for beginners, with no dirty tricks or packers.
Goal: Find a valid key for your specific username (Keygening)
## Debugging and Analyst
First we need go through `main()` to understand flow of this program because all C/C++ program always start with `main()` first then it call other `func()` inside `main()`![[{18401582-1B07-47B0-A987-C64B57729104}.png]]
and we can see here in left panel mingw -> this program compiled by mingw
### Digging inside `main()`
![[{CFD61DC3-D977-453B-9D86-DF6AD1905E42}.png]]
Ok so we see first thing in the `main()` is ask for a name input where i rename it as `Name_input`
for easy looking and all above it just printing description for visual so we skip that. Right under `Name_input` it has an if-else to check `Name_input` if it empty then print out a sentence.
Let come with else of it:
```C++
Key = calculate_soul_weight((__int64)Name_input);
    v4 = std::operator<<<std::char_traits<char>>(refptr__ZSt4cout, "\nANALYZING SOUL METRICS for ");
    v5 = std::operator<<<char>(v4, Name_input);
    v6 = std::operator<<<std::char_traits<char>>(v5, "...");
    std::ostream::operator<<(v6, refptr__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_);
    v30 = 1000;
    std::chrono::duration<long long,std::ratio<1ll,1000ll>>::duration<int,void>(v29, &v30);
    std::this_thread::sleep_for<long long,std::ratio<1ll,1000ll>>(v29);
    v7 = std::operator<<<std::char_traits<char>>(refptr__ZSt4cout, "DETECTED SOUL WEIGHT: ");
    v8 = std::ostream::operator<<(v7, Key);
    v9 = std::operator<<<std::char_traits<char>>(v8, " cosmic units.");
    std::ostream::operator<<(v9, refptr__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_);
    v50 = &v32;
    std::string::basic_string<std::allocator<char>>(
      v31,
      "\nWARNING: TO UNLOCK ITEM [C3], YOU MUST PROVIDE THE ACCESS CODE.",
      &v32);
    slow_print((__int64)v31, 30);
    std::string::~string(v31);
    std::__new_allocator<char>::~__new_allocator(&v32);
    v49 = &v34;
    std::string::basic_string<std::allocator<char>>(
      v33,
      "HINT: The code is hidden in the silence between the stars... or just basic math.",
      &v34);
    slow_print((__int64)v33, 30);
    std::string::~string(v33);
    std::__new_allocator<char>::~__new_allocator(&v34);
    std::operator<<<std::char_traits<char>>(refptr__ZSt4cout, "\n[ENTER ACCESS CODE]: ");
    v10 = (_QWORD *)std::istream::operator>>(refptr__ZSt3cin, &v19);
```
ok now we'll driving in this. Firstly, we got a key from calculate soul weight when we call `calculate_soul_weight()` func:
```C
__int64 __fastcall calculate_soul_weight(__int64 name)
{
  __int64 v2; // [rsp+28h] [rbp-38h] BYREF
  _QWORD v3[3]; // [rsp+30h] [rbp-30h] BYREF
  char v4; // [rsp+4Fh] [rbp-11h]
  __int64 v5; // [rsp+50h] [rbp-10h]
  int v6; // [rsp+5Ch] [rbp-4h]

  v6 = 0;
  v5 = name;
  v3[0] = std::string::begin(name);
  v2 = std::string::end(v5);
  while ( 1 )
  {
    v3[2] = v3;
    v3[1] = &v2;
    if ( v3[0] == v2 )
      break;
    v4 = *(_BYTE *)v3[0];
    v6 += v4;
    ++v3[0];
  }
  return (unsigned int)(13 * v6 + 7);
}
```
look at this func to see what it do get sum of acsii from each character in `Name_input` then take that sum x 13 +7 then we got the soul weight(Key).
And after got Key the program ask us about sth call access_code to get the sercet prize 
```C
    std::operator<<<std::char_traits<char>>((__int64)refptr__ZSt4cout, (__int64)"\n[ENTER ACCESS CODE]: ");
    v10 = (_QWORD *)std::istream::operator>>((__int64)refptr__ZSt3cin, (__int64)&v19);
    if ( (unsigned __int8)std::ios::operator!((__int64)v10 + *(_QWORD *)(*v10 - 24LL)) )
    {
      v11 = std::operator<<<std::char_traits<char>>(
              (__int64)refptr__ZSt4cout,
              (__int64)"INVALID INPUT. THE MACHINE EATS YOUR COIN.");
      std::ostream::operator<<(v11, (__int64)refptr__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_);
      v3 = 0;
    }
    else
    {
      v57 = 64206;
      v56 = 123;
      v55 = (int)((v58 ^ 0xFACE) - 123);
      v12 = std::operator<<<std::char_traits<char>>((__int64)refptr__ZSt4cout, (__int64)"\nVERIFYING...");
      std::ostream::operator<<(v12, (__int64)refptr__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_);
      v36 = 1500;
      std::chrono::duration<long long,std::ratio<1ll,1000ll>>::duration<int,void>(&v35, &v36);
      std::this_thread::sleep_for<long long,std::ratio<1ll,1000ll>>((__int64)&v35);
      if ( v55 == v19 )
      {
        v13 = std::operator<<<std::char_traits<char>>((__int64)refptr__ZSt4cout, (__int64)"\nACCESS GRANTED!");
        std::ostream::operator<<(v13, (__int64)refptr__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_);
        v14 = std::operator<<<std::char_traits<char>>(
                (__int64)refptr__ZSt4cout,
                (__int64)"\n"
                         "          .  o ..\n"
                         "          o . o o.o\n"
                         "               ...\n"
                         "              \\   /\n"
                         "               \\ /\n"
                         "              ( * )  <- The Ultimate Snack\n"
                         "               \\_/\n"
                         "        ");
        std::ostream::operator<<(v14, (__int64)refptr__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_);
        v48 = &v38;
        std::string::basic_string<std::allocator<char>>(v37, "CONGRATULATIONS! You have solved the riddle.");
        slow_print((__int64)v37, 30);
        std::string::~string((__int64)v37);
        std::__new_allocator<char>::~__new_allocator();
        v47 = &v40;
        std::string::basic_string<std::allocator<char>>(v39, "The universe makes slightly more sense now.");
        slow_print((__int64)v39, 30);
        std::string::~string((__int64)v39);
      }
      else
      {
        v15 = std::operator<<<std::char_traits<char>>((__int64)refptr__ZSt4cout, (__int64)"\nACCESS DENIED!");
        std::ostream::operator<<(v15, (__int64)refptr__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_);
        v16 = std::operator<<<std::char_traits<char>>((__int64)refptr__ZSt4cout, (__int64)"Required Code was not: ");
        v17 = std::ostream::operator<<(v16, v19);
        std::ostream::operator<<(v17, (__int64)refptr__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_);
        v46 = &v42;
        std::string::basic_string<std::allocator<char>>(v41, "The machine laughs at your poverty of logic.");
        slow_print((__int64)v41, 30);
        std::string::~string((__int64)v41);
        std::__new_allocator<char>::~__new_allocator();
        v45 = &v44;
        std::string::basic_string<std::allocator<char>>(v43, "Try again, brave hacker.");
        slow_print((__int64)v43, 30);
        std::string::~string((__int64)v43);
      }
      std::__new_allocator<char>::~__new_allocator();
      std::operator<<<std::char_traits<char>>((__int64)refptr__ZSt4cout, (__int64)"\n[Press Enter to exit]");
      std::istream::ignore(refptr__ZSt3cin);
      std::istream::get(refptr__ZSt3cin);
      v3 = 0;
    }
  }
  std::string::~string((__int64)Name_input);
  return v3;
}
```
right under asking about access code is a checking input from access code if it's true format just unsigned int if not just end the program and if it true do the math to check our input access code is true/false. let see the math transfer name to access code
```
Key(soul_weight) XOR 64206(0xFace) - 123
```
So to get the sercet prize we just do like program to get true access_code
### Overall
Just do a simple math to get sercet prize
![[{9A4CFDF3-77D3-4B3D-B75B-7DAD822A6FA0}.png]]
```python
name = "weat"  # Thay tên khác nếu cần
sum_ascii = sum(ord(c) for c in name)
soul_weight = sum_ascii * 13 + 7
face = 0xFACE  # 64206
access_code = (soul_weight ^ face) - 123
print(f"Soul weight: {soul_weight}")
print(f"Access code: {access_code}")
#weat
#60495
```
