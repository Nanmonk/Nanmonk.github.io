+++
date = '2026-05-17T00:00:00+08:00'
draft = false
title = 'C++ inline & static'
tags = ['C++']
+++

# inline
[inline cppreference](https://zh.cppreference.com/w/cpp/language/inline)

## 重定义

举例：

![](https://img2024.cnblogs.com/blog/3391146/202504/3391146-20250401135424512-1189673066.png)

```asm
.LC0:
        .string "\345\245\207"
.LC1:
        .string "\345\201\266"
num_check(int):
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], edi
        mov     edx, DWORD PTR [rbp-4]
        mov     eax, edx
        sar     eax, 31
        shr     eax, 31
        add     edx, eax
        and     edx, 1
        sub     edx, eax
        mov     eax, edx
        test    eax, eax
        jle     .L2
        mov     eax, OFFSET FLAT:.LC0
        jmp     .L4
.L2:
        mov     eax, OFFSET FLAT:.LC1
.L4:
        pop     rbp
        ret
.LC2:
        .string "%02d %s\n"
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     DWORD PTR [rbp-4], 0
        jmp     .L6
.L7:
        mov     eax, DWORD PTR [rbp-4]
        mov     edi, eax
        call    num_check(int)
        mov     rdx, rax
        mov     eax, DWORD PTR [rbp-4]
        mov     esi, eax
        mov     edi, OFFSET FLAT:.LC2
        mov     eax, 0
        call    printf
        add     DWORD PTR [rbp-4], 1
.L6:
        cmp     DWORD PTR [rbp-4], 99
        jle     .L7
        mov     eax, 0
        leave
        ret
```

可以明显发现，此时是正常的函数调用，那如何才能内联优化呢？

可以在函数前使用 `[[gnu::always_inline]]` 或者 `__attribute__((always_inline))`，比如

```cpp
[[gnu::always_inline]] inline const char* num_check(int v) {
    return (v % 2 > 0) ? "奇" : "偶";
}
```

其实只要设置编译优化 `-O1` 就会自动优化。

![](https://img2024.cnblogs.com/blog/3391146/202504/3391146-20250401135438367-34701593.png)

## 内联 const 变量默认拥有外部链接

## 类成员函数与模板函数默认 inline

- 类中直接定义的成员函数默认是 `inline` 的（即允许多重定义）

```cpp
struct S {
    int get() const { return 42; } // 默认 inline
};
```

- 模板函数直接定义时也是默认 `inline` 的

```cpp
template<typename T>
T square(T x) { return x * x; } // 默认 inline
```

# static

让被修饰内容仅在当前翻译单元（Translation Unit）可见。

[ODR](https://zh.cppreference.com/w/cpp/language/definition)

# Other

> **Note**
> 头文件可以使用 `#pragma once`，虽然被很多编译器支持，但这并不是标准的内容。
