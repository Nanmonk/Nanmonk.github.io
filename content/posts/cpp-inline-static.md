+++
date = '2026-05-17T00:00:00+08:00'
draft = false
title = 'C++ inline & static'
tags = ['C++']
+++

# inline
[inline cppreference](https://zh.cppreference.com/w/cpp/language/inline)

## 内联展开

以下面这段代码为例：

```cpp
const char* num_check(int v) {
    return (v % 2 > 0) ? "奇" : "偶";
}

int main() {
    for (int i = 0; i <= 99; i++)
        printf("%02d %s\n", i, num_check(i));
}
```

未开启优化时编译，可以看到 `num_check` 是正常的函数调用（`call num_check(int)`）：

![](https://github.com/Nanmonk/Nanmonk.github.io/blob/main/assets/593692100-12467e52-378b-4a70-86c6-d718ef9cb072.png)

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

开启 `-O1` 后编译器会自动将函数内联展开，消除函数调用开销：

![](https://github.com/Nanmonk/Nanmonk.github.io/blob/main/assets/593692126-5d5e8c23-c275-4961-ab34-594902a11cb9.png)

如果需要在 `-O0` 下（例如调试构建中的热路径）也强制内联，可以使用 `[[gnu::always_inline]]` 或 `__attribute__((always_inline))`：

```cpp
[[gnu::always_inline]] inline const char* num_check(int v) {
    return (v % 2 > 0) ? "奇" : "偶";
}
```

> [!NOTE]
> 通常情况下让编译器决定（开 `-O1` 及以上）是更好的选择。`always_inline` 适用于必须控制内联行为的特殊场景，滥用可能导致代码体积膨胀。

## 内联 const 变量默认拥有外部链接

普通的 `const` 变量在命名空间作用域下默认是**内部链接**（相当于隐式 `static`），而加了 `inline` 之后则拥有**外部链接**，可以在多个翻译单元中共享同一份定义：

```cpp
// constants.h
inline const int kMaxSize = 1024; // 外部链接，多个 .cpp 包含也不会重复定义
const int kOther = 42;            // 内部链接，每个 .cpp 各有一份副本
```

这让头文件中定义全局常量成为可能，而不需要 `extern` 声明 + `.cpp` 定义的分离写法。

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

这也是为什么模板定义通常放在头文件里——多个翻译单元包含同一份定义不会触发 ODR 违规。

# static

`static` 修饰命名空间作用域的函数或变量时，让其仅在**当前翻译单元**（Translation Unit）可见，即内部链接。

```cpp
// a.cpp
static void foo() {}  // 仅 a.cpp 可见

// b.cpp
static void foo() {}  // 与 a.cpp 的 foo 互不干扰，不违反 ODR
```

这与 `inline` 的"允许多重定义"不同：`static` 是"每个翻译单元各有一份独立定义"，`inline` 是"多个翻译单元共享同一份定义"。

[ODR cppreference](https://zh.cppreference.com/w/cpp/language/definition)

# 头文件保护

> [!NOTE]
> 头文件可以使用 `#pragma once` 代替传统的 include guard，虽然被大多数主流编译器支持，但它不属于 C++ 标准，移植性上需留意。
