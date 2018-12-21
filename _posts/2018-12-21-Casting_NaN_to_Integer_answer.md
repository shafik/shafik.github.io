---
layout: default
categories: [poll answers] 
title: C++ Poll Answer for December 21th 2018 
---

We are allowed to convert floating point values to integers but it is undefined behavior if the converted floating point value can not be represented and NaN can not be represented as an integer. See [\[conv.fpint\]p1](h`ttp://eel.is/c++draft/conv.fpint#1):

>A prvalue of a floating-point type can be converted to a prvalue of an integer type.
The conversion truncates; that is, the fractional part is discarded.
The behavior is undefined if the truncated value cannot be represented in the destination type.
[ Note: If the destination type is bool, see [conv.bool].— end note]

[This Stackoverflow answer](https://stackoverflow.com/a/10400095/1708801) has the reason for why we see some specific results.

See [this live Wandbox example](https://wandbox.org/permlink/L5x7cHNfri0xS4Xs)

    #include <cmath>
    #include <cstdio>
    #include <limits>

    static int x = static_cast<int>(NAN);
    static int y = static_cast<int>(std::numeric_limits<double>::quiet_NaN());

    int main(){
        float h = NAN;
        printf("%x %d %x %d\n", static_cast<int>(h), static_cast<int>(h), static_cast<int>(NAN), static_cast<int>(NAN));  
        printf("%x %d\n", static_cast<int>(NAN), static_cast<int>(NAN) );
        printf("%x %d\n", static_cast<int>(NAN), static_cast<int>(NAN) );
        printf("%x %d\n", *(int *)&h, *(int *)&h);
        printf("%x %d\n", x, x ) ;
    
        printf( "%x %d\n", static_cast<int>(std::numeric_limits<double>::quiet_NaN()), static_cast<int>(std::numeric_limits<double>::quiet_NaN()));
        printf("%x %d\n", y, y ) ;
    }

for some interesting undefined results.

We can see from [this godbolt](https://godbolt.org/z/1PT7yU) trying to use the result in constant expression is ill-formed due to undefined behavior:

    constexpr int z = static_cast<int>(NAN);

