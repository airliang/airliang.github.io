---
title: Right value reference and std::move
tags: C++
---

In C++, a right value reference (denoted as `T&&`) is a type of reference that can bind to rvalues, which are temporary objects that do not persist beyond a single expression. Right value references enable move semantics and perfect forwarding, improving performance by avoiding unnecessary copying of temporary objects.

### Key Concepts:
#### Rvalues vs. Lvalues:

Lvalue: An object with a persistent memory address (`int a = 10; a = 20;`).
Rvalue: A temporary object or a value without a persistent memory address (`a + b, 10, std::string("temp")`).

#### Move Semantics
Move semantics allow the transfer of resources (like memory) from a temporary object to another object without executing __copy constructor__.

See this example:
```cpp
class TestA
{
public:
    TestA() = default;
    explicit TestA(const char *str) {
        int length = strlen(str);
        _a = new char[length + 1];
        _a[length] = 0;
        memcpy(_a, str, length);
    }
    ~TestA()
    {
        if (_a)
        {
            delete[] _a;
            _a = nullptr;
        }
    }

    //copy construtor
    TestA(const TestA& other) {
        if (_a != nullptr)
        {
            delete[] _a;
        }

        if (other._a != nullptr)
        {
            int length = strlen(other._a);
            _a = new char[length + 1];
            _a[length] = 0;
            memcpy(_a, other._a, length);
        }
        std::cout << "Copy constuctor." << std::endl;
    }
    
    //move constructor
    TestA(TestA&& rvalue) noexcept
    {
        std::cout << "Move constuctor." << std::endl;
        _a = std::move(rvalue._a);
        rvalue._a = nullptr;
    }
    TestA& operator=(const TestA& other)
    {
        if (_a != nullptr) {
            delete[] _a;
        }

        if (other._a != nullptr) {
            int length = strlen(other._a);
            _a = new char[length + 1];
            _a[length] = 0;
            memcpy(_a, other._a, length);
        }
        std::cout << "Copy assignment constuctor." << std::endl;
        return *this;
    }

    TestA &operator=(TestA &&rvalue) noexcept
    {
        _a = std::move(rvalue._a);
        rvalue._a = nullptr;
        std::cout << "Move assignment constuctor." << std::endl;
        return *this;
    }

public: 
    char* _a = nullptr;
};
```
Notice that we have two constructors, the __copy constructor__ and the __move constructor__. In the __copy constructor__, there is a new operator which could cost more performance.
Only in this case could we call the __move constructor__ instead of the __copy constructor__.
```cpp
std::vector<TestA> test;
{
    TestA a("test right value");
    //TestA a = "test right value"; compile error because we declare the constructor as explicit
    test.emplace_back(std::move(a));  //call the move constructor
}
```
std::move function converts the left value reference to the right value reference, so it will call the move constructor and avoid the new operator.
I summarize several rules for using the right value reference:
- Only memory alloc operations exist in the constructor should declare move constructor. If the class members are all int, float, char, etc. We don't need the move constructor.
- Call std::move to convert the left value reference to the right value reference to call the move constructor.

### Universal reference
A __universal reference__ occurs when:
1. It is declared as `T&&` (where T is a template parameter or deduced with auto).
2. Type deduction is involved.

Example:

```cpp
template <typename T>
void func(T&& param); // param is a universal reference

int&& r = 42; // r is an rvalue reference, only binds to rvalues

auto&& u = r; // u is a universal reference, binds to an lvalue (r)
```
In this case,
`r` is an rvalue reference, but `r` as a named variable is an lvalue.

When `u` is declared as `auto&&`, it is a universal reference, meaning it can bind to both lvalues and rvalues.

During type deduction:
Since `r` is an lvalue, `T` is deduced as `int&`.
Therefore, `u` becomes `int& &&`, which collapses to `int&`.
`u` is now an lvalue reference to `r`.

### Reference
[Universal References in C++11 -- Scott Meyers](https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers)

[Value categories](https://en.cppreference.com/w/cpp/language/value_category)
