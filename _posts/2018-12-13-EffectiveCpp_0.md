---
layout: post
title: "[Tóm tắt] Effective Cpp by Scott Meyers"
image: /img/cpp_icon.png
subtitle: Bước đầu sử dụng Cpp sao cho hiệu quả
tags: [Programming, Books, Best Practices, C++]
---
# Tổng kết, tóm tắt nội dung sách Effective C++ by Scott Meyers

## Phần 0: Giới thiệu

Cuốn sách kinh điển tổng hợp và mô tả những điều nên làm hoặc nên tránh trong quá trình phát triển phần mềm với C++
Một số thuật ngữ cần lưu ý:

- Một **declaration** (Khai báo) chỉ cho **compiler** (trình biên dịch) biết về name (*tên*) và type (*kiểu*) của một object (*đối tượng*), function (*hàm*), class (*lớp*) mà không đưa ra mô tả chi tiết

  ```cpp
  extern int x;                          // object declaration
  int numDigits(int number);             // function declaration
  class Clock;                           // class declaration
  template<class T>
  class SmartPointer;                    // template declaration
  ```

- Ngược lại, một **definition** (định nghĩa) đưa ra mô tả chi tiết cho trình biên dịch (compiler). Ví dụ với một object, compiler sẽ biết khởi tạo object, cấp phát bộ nhớ ở đâu, với function hoặc function template, một definition sẽ cung cấp phần thân hàm (code body). Với class hay class template definition sẽ là các member functions hoặc trường bên trong class hoặc template

  ```cpp
  int x;                                  // object definition
  int numDigits(int number)               // function definition
  {                                       // (this function returns
    int digitsSoFar = 1;                  // the number of digits in
                                          // its parameter)
    if (number < 0) {
      number = -number;
      ++digitsSoFar;
    }
    while (number /= 10) ++digitsSoFar;
    return digitsSoFar;
  }

  class Clock {                           // class definition
  public:
    Clock();
    ~Clock();
    int hour() const;
    int minute() const;
    int second() const;
    ...
  };
  template<class T>
  class SmartPointer {                    // template definition
  public:
    SmartPointer(T *p = 0);
    ~SmartPointer();
    T * operator->() const;
    T& operator*() const;
    ...
  };

  ```

**Constructor** - Hàm khởi tạo.

**Default Constructor** - Hàm khởi tạo mặc định là hàm khởi tạo đc gọi mà không cần bất kỳ tham số nào ( không có tham số hoặc nhận giá trị mặc định cho mọi tham số ). Thông thường nếu muốn khai báo mảng của 1 đối tượng thì cần có default constructor.

```cpp
class A {
  public:
    A();                              // default constructor
};

A arrayA[10];                         // 10 constructors called

class B {
  public:
    B(int x = 0);                     // default constructor
};

B arrayB[10];                          // 10 constructor called, each with an arg of 0

class C {
  public:
    C(int x);                         // not a default constructor
};

C arrayC[10];                         // error!
```

Để tạo mảng của đối tượng ko có hàm khởi tạo mặc định, ta có thể định nghĩa mảng của **pointer** (*con trỏ*), sau đó khởi tạo từng con trỏ với từ khóa **new**:

```cpp
C *ptrArray[2];                       // no constructor called
ptrArray[0] = new C(22);              // allocate and construct 1 C object
ptrArray[1] = new C(4);               // ditto
```

**Copy constructor** được dùng để khởi tạo một đối tượng sử dụng 1 đối tượng khác có cùng **kiểu** (**type**):

```cpp
class String
{
  public:
    String();                          // default constructor
    String(const String& rhs);         // copy constructor

  private:
    char *data;
};

String s1;                            // call default constructor
String s2(s1);                        // call copy constructor
String s3 = s2;                       // ditto
```

**Initialization** - Khởi tạo (object). Object's initialization xảy ra khi object được gán giá trị ngay khi khai báo đầu tiên.
**Assignment** - Gán (object). Object's assignment xảy ra khi một object đã được khởi tạo và được gán giá trị mới:

```cpp
string s1;                            // initilization
string s2("Hello");                   // ditto
string s3 = s2;                       // ditto

s1 = s3;                              // asignment
```

Có thể hiểu sự khác nhau giữa **initialization** mà **assignment** nằm ở việc initialization được thực hiện bằng cách gọi **constructor**, trong khi với assignment là **operator=**

**Client** - là 1 người sử dụng code mà ta đã viết

**Casting** - ép kiểu

- **const_cast** dùng để loại bỏ tính constness của đối tượng hoặc con trỏ
- **dynamic_cast** dùng cho "safe downcasting"
- **reinterpret_cast** dùng để casting giữa các kiểu con trỏ hàm (ít dùng)
- **static_cast** catch-all cast, dùng khi những kiểu casting khác ko phù hợp
