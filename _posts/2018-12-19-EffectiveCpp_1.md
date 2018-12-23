---
layout: post
title: "[Tóm tắt] Effective C++ by Scott Mayers - Phần 1"
image: /img/cpp_icon.png
subtitle: Bước đầu sử dụng C++ sao cho hiệu quả
tags: [Programming, Books, Best Practices, C++]
---

# Tổng kết, tóm tắt nội dung sách Effective C++ by Scott Mayers

## Phần 1: Bước đầu làm quen với C++

1. Item 1: C++ là 1 ngôn ngữ đa mẫu hình (**multiparadigm programming language**)

    Có nhiều loại mẫu hình lập trình (**paradigm programming**) khác nhau có thể kể đến như object-oriented programming, aspect-oriented programming, functional programming, etc .. Mỗi hình mẫu lập trình đem đến 1 hướng tiếp cận khác nhau cho lập trình viên hiện thực hoá chương trình của mình. Một số ngôn ngữ được thiết kế để hỗ trợ một mẫu hình đặc thù (Smalltalk hỗ trợ lập trình hướng đối tượng trong khi Haskell hỗ trợ lập trình chức năng). Số ngôn ngữ khác lại hỗ trợ nhiều mẫu hình (như Python, Common Lisp, C#, etc..). Một chương trình C++ có thể được viết kết hợp 1 hoặc nhiều loại hình mẫu lập trình như: procedural, object-oriented, functional, generic và metaprogramming. Điều này thể hiện tính linh hoạt của C++ nhưng ngược lại có thể gây khó hiểu với những người mới bắt đầu. Do đó, chúng ta nên nhìn C++ giống như là 1 ngôn ngữ mẹ của 4 ngôn ngữ con (sublanguage):
    {: .text-justify}

    - C

        C with classes là tên ban đầu của C++ thể hiện rằng nó là người kế thừa, mở rộng của C. Các kiểu dữ liệu, mảng, con trỏ .. đều đc kế thừa từ C.

    - Object-oriented C++

        Đây là lý do vì sao C++ được gọi là C with Classes: classes (including constructors and destructors), encap- sulation, inheritance, polymorphism, virtual functions (dynamic binding), etc..

    - Template C++

        Đây là phần liên quan đến generic programming của C++, cũng có lẽ sẽ không quen thuộc giống như lập trình hướng đối tượng. Lập trình template rất mạnh, và có hẳn 1 mẫu hình lập trình mang tên nó: template metapro- gramming (TMP)
    - The STL

        STL là 1 thư viện template đặc biệt của C++. Nó xoay quanh khái niệm: containers, iterators, algorithms, and function objects. Sử dụng STL thành thạo đem đến nhiều lợi thế rút ngắn thời gian phát triển, tăng hiệu suất thực thi của chương trình.

    **Cần nhớ**:
    - Tuỳ thuộc vào việc chúng ta sử dụng phần nào của C++ mà sẽ có những rules riêng giúp chúng ta sử dụng c++ hiệu quả hơn

2. Item 2:  Ưu tiên sử dụng **const**, **enum**, **inline** thay cho **#define**.

    ```cpp
    #define ASPECT_RATIO 1.653
    ```

    - **#define** - preprocessor sẽ thay thế toàn bộ **ASPECT_RATIO** với giá trị **1.653** trước khi source code được build bởi **compiler** -> **ASPECT_RATIO** sẽ ko được đưa vào symbol table -> khó tracking sau này khi có bug phát sinh
    - **const**, **enum**, **inline** - compiler xử lý, đưa vào symbol table

    ```cpp
    const double AspectRatio = 1.653;
    ```

    *Note*:
    **const pointer** khai báo trong header thông thường cần đảm bảo tính constness cho bản thân nó và dữ liệu nó trỏ đến. Với **class-specific const**, để đảm bảo tối đa có 1 bản copy của const, chúng ta cần khai báo nó là static.

    ```cpp
    const char* const authorName = "Scott Meyers"; // nên sử dụng const std::string

    class GamePlayer {
    private:
        static const int NumTurns = 5;             // const declaration
        int scores[NumTurns];                      // use of const
        // ...
    };
    ```

    Sử dụng **const** trong trường hợp trên có thể sinh ra mã code nhỏ hơn so với sử dụng **preprocessor**

    *Note*:
    **Preprocessor** cũng thường được dùng để khai báo macros, tuy nhiên việc này cần rất thận trọng vì có thể phát sinh nhiều lỗi

    ```cpp
    // cal f with the maximum of a and b
    #define CALL_WITH_MAX(a,b) f((a) > (b) ? (a) : (b))

    int a = 5, b = 0;
    CALL_WITH_MAX(++a, b);        // a is incremented twice
    CALL_WITH_MAX(++a, b+10);        // a is incremented once

    // inline function
    template<typename T>
    inline void callWithMax(const T& a, const T& b)
    {
        f(a > b ? a : b);
    }
    // because we don’t know what T is, we pass by reference-to-const — see Item 20
    ```

    **Cần nhớ**:
    - Với hằng số đơn giản, ưu tiên dùng **const** objects hoặc **enum** so với **#define**
    - Với function-like macros, ưu tiên dùng **inline** functions to **#define**

3. Item 3: Sử dụng **const** bất cứ khi nào có thể.

    **Const** chỉ rõ ràng buộc về ngữ nghĩa (semantic constraint) rằng 1 đối tượng không được thay đổi. Compiler sẽ tuân theo ràng buộc này

    - For pointer

    ```cpp
    char greeting[] = "Hello";
    char *p = greeting;                             // non-const pointer, non-const data
    const char *p = greeting;                       // non-const pointer, const data
    char * const p = greeting;                      // const pointer, non-const data
    const char * const p = greeting;                // const pointer, const data
    ```

    - STL

    ```cpp
    std::vector<int> vec;
    // ...
    const std::vector<int>::iterator iter = vec.begin();        // iter acts like a T* const
    *iter = 10;                                                 // OK, change what iter points to
    ++iter;                                                     // error! iter is const

    std::vector<int>::const_iterator cIter = vec.begin();       // cIter acts like a const T*
    *cIter = 10;                                                // error~ *cIter is const
    ++cIter;                                                    // fine, changes cIter
    ```

    For function: **const** có thể áp dụng cho return của hàm, tham số, hoặc bản thân hàm
    Ưu điểm: Giảm thiểu khả năng lỗi từ client

    ```cpp
    class Rational {
        //..
    };
    const Rational operator*(const Rational& lhs, const Rational& rhs);

    Rational a, b, c;
    // ...
    (a*b) = c;                        // invoke operator= on the result of a*b!

    if (a*b = c) {                    // opps, meant to do a comparison!
        // ..
    }
    ```

    **Const** member function:

    - Make interface class easier to understand
    - Make it possible to work with const object

    **bitwise constness** vs **logical constness**

    ```cpp
    class CTextBlock {
    public:
    // ..
    char& operator[](std::size_t position) const {      // inappropriate (but bitwise const)
        return pText[position];
    }

    private:
    char *pText;
    };

    const CtextBlock cctb("Hello");
    char* pc = &cctb[0];
    *pc = 'J';                          // cctb now has the value "Jello"
    ```

    ```cpp
    class CTextBlock {
    public:
        //...
        std::size_t length() const;
        private:
        char *pText;
        mutable std::size_t textLength;   // these data members may
        mutable bool lengthIsValid;       // always be modified, even in
    };                                  // const member functions

    std::size_t CTextBlock::length() const
    {
        if (!lengthIsValid) {
            textLength = std::strlen(pText);      // now fine
            lengthIsValid = true;                 // also fine
        }
        return textLength;
    }
    ```

    **Cần nhớ**:
    - Khai báo **const** giúp trình biên dịch phát hiện các lỗi trong quá trình code. **const** có thể được dùng ở nhiều phạm vi khác nhau từ function parameters, return types đến member functions
    - Trình biên dịch luôn muốn bitwise constness, nhưng lập trình viên nên sử dụng logical constness
    - Khi const và non- const member functions có cài đặt giống nhau, việc lặp code có thể tránh bằng cách gọi const version từ non-const version.

4. Item 4: Đảm bảo rằng **objects** được khởi tạo trước khi được sử dụng.

    Luôn luôn khởi tạo các **objects** trước khi dùng chúng

    For non-member objects of buil-in type -> manually

    ```cpp
    int x = 0;　                                  // manual initialization of an int
    const char * text = "A C-style string";       // manual initialization of a pointer (see also Item 3)
    double d;                                     // “initialization” by reading from
    std::cin >> d;                                // an input stream
    ```

    Other, init by contructors (contructor khởi tạo mọi thứ có trong object)
    Note: Sử dụng initilization list thay vì assigment

    ```cpp
    class PhoneNumber { ... };
    class ABEntry {
        // ABEntry = “Address Book Entry”
        public:
            ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones);
        private:
            std::string theName;
            std::string theAddress;
            std::list<PhoneNumber> thePhones;
            int numTimesConsulted;
    };

    ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones) 
                    : theName(name), theAddress(address),                                 // these are now all initializations
                    thePhones(phones), numTimesConsulted(0)
    {} // the ctor body is now empty

    ABEntry::ABEntry() : theName(),               // call theName’s default ctor;
                        theAddress(),             // do the same for theAddress;
                        thePhones(),              // and for thePhones;
                        numTimesConsulted(0)      // but explicitly initialize numTimesConsulted to zero
    {}
    ```

    **static object** là đối tượng tồn tại từ khi chúng được khởi tạo, đến khi chương trình kết thúc:

    - Global object
    - object được định nghĩa với scope namespace
    - object khai báo dạng static bên trong class
    - object khai báo dạng static bên trong function (local static object)
    - object khai báo dạng static với scope của file

    **translation unit** là mã nguồn có thể build ra 1 file object đơn lẻ (a single source file, plus its "#include" files)

    **Cần nhớ**:
    - Khởi tạo bằng tay các đối tượng kiểu có sẵn (**builtin type**), vì C++ chỉ thi thoảng mới khởi tạo chúng.
    - Trong hàm khởi tạo (**constructor**), ưu tiên dùng **member initialization list** so với **assignment** bên trong thân hàm. Và sử dụng thành phần trong **initialization list** theo đúng thứ tự được khai báo trong class.
    - Tránh lỗi liên quan đến thứ tự khởi tạo (**initialization order**) giữa các **translation units** bằng cách thay thê non-local static objects với local static objects.
