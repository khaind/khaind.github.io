---
layout: post
title: "[Cpp Versus] Private inheritance vs Composition"
image: /img/cpp_icon.png
subtitle: Use composition when you can, private inheritance when you have to.
tags: [Programming, C++, private inheritance, composition, cppversus]
---

# So sánh giữa `private inheritance` và `composition`

## Nhắc lại về thừa kế trong c++

```c++
class A
{
public:
    int x;
protected:
    int y;
private:
    int z;
};

class B : public A
{
    // x is public
    // y is protected
    // z is not accessible from B
};

class C : protected A
{
    // x is protected
    // y is protected
    // z is not accessible from C
};

class D : private A    // 'private' is default for classes
{
    // x is private
    // y is private
    // z is not accessible from D
};
```

### Lưu ý 1

Class B, C, D đều chứa biến x, y, z, nhưng việc truy xuất đến những biến này là khác nhau như trên.

## Ý nghĩa của việc thừa kế

**public** inheritance

`is-a` relationship

```c++
class Button : public Widget {};
```

Một Button là một Widget, bất kể hàm nào nhận tham số đầu vào là Widget đều có thể truyền vào Button

**protected** inheritance

`Protected implemented-in-term-of` relationship. (Ít sử dụng)

Protected inheritence thường được sử dụng khi ta có một base class chứa các chức năng hữu ích mà derive class có thể sử dụng, tuy nhiên ta chỉ muốn duy nhất derive class sử dụng được nó (không bàn đến friend)

**private** inheritance

`implemented-in-term-of` or `has-a` relationship

Sử dụng chức năng từ base class để cài đặt derived class, nhưng không muốn các lớp khác biết đến và sử dụng các chức năng của base class.

### Lưu ý 2

```c++
class Person {};
class Student:private Person {};     // private
void eat(const Person& p){}         // anyone can eat
void study(const Student& s){}      // only students study

int main()
{
    Person p;       // p is a Person
    Student s;      // s is a Student
    eat(p);         // fine, p is a Person
    eat(s);         // error if using protected or private inheritance
    return 0;
}
```

Do protected inheritance hay private inheritance không phải là quan hệ `is-a` nên câu lệnh ```eat(s);``` sẽ không compile được.

## Private inheritance vs Composition

Như vậy `private inheritance` khá giống với `composition` (aggregation) bởi đều là quan hệ `has-a`

Ví dụ: Mối quan hệ "Car `has-a` Engine" có để được cài đặt 1 trong 2 cách sau:

### Sử dụng composition

```c++
class Engine {
public:
  Engine(int numCylinders);
  void start();                 // Starts this Engine
};
class Car {
public:
  Car() : e_(8) { }             // Initializes this Car with 8 cylinders
  void start() { e_.start(); }  // Start this Car by starting its Engine
private:
  Engine e_;                    // Car has-a Engine
};
```

### Sử dụng *private inheritance*

```c++
class Car : private Engine {    // Car has-a Engine
public:
  Car() : Engine(8) { }         // Initializes this Car with 8 cylinders
  using Engine::start;          // Start this Car by starting its Engine
};
```

### Sự giống nhau giữa 2 biến thể trên

- Có duy nhất một đối tượng Engine được chứa trong đối tượng Car
- Client không thể convert từ Car* thành Engine*
- Lớp Car có phương thức start() gọi phương thức start() của đối tượng Engine mà nó chứa.

### Sự khác nhau giữa 2 biến thể trên

- Khi dùng composition thì mỗi đối tượng Car có thể chưa nhiều Engine nếu muốn
- Dùng private-inheritance có thể dẫn tới đa kế thừa không cần thiết
- Dùng private-inheritance cho phép member của Car có thể convert một con trỏ Car* thành một con trỏ Engine*
- Dùng private-inheritance cho phép truy xuất đến protected members của base class
- Dùng private-inheritance cho phép Car có thể override các hàm ảo (virtual function) của Engine
- Dùng private-inheritance làm cho code trông có vẻ gọn hơn 1 chút : ))

## Nên sử dụng biến thể nào: composition hay  private inheritance

"Use composition when you can, private inheritance when you have to"

Thông thường chúng ta không muốn truy xuất vào quá nhiều lớp khác, và `private inheritance` có thể giúp chúng ta một chút về việc này. Tuy nhiên `private inheritance` lại khó maintain hơn, vì nó tăng khả năng mà một người khác khi thay đổi một số thứ sẽ có thể gây hại tới code của chúng ta.

Một cách sử dụng tin cậy của `private inheritance` là khi chúng ta muốn xây dựng một *class Fred* sử dụng code của một *class Wilma* khác, và code của *class Wilma* đó cần gọi phương thức từ *class Fred* mới này. Trong trường hợp này, *Fred* gọi hàm non-virtuals trong *Wilma*, và *Wilma* gọi hàm (thường là pure virtuals) của bản thân nó, hàm mà được override bởi *Fred*. Việc này sẽ khó khăn nếu chúng ta sử dụng `composition`.

```c++
class Wilma {
protected:
  void fredCallsWilma()
    {
      std::cout << "Wilma::fredCallsWilma()\n";
      wilmaCallsFred();
    }
  virtual void wilmaCallsFred() = 0;   // A pure virtual function
};
class Fred : private Wilma {
public:
  void barney()
    {
      std::cout << "Fred::barney()\n";
      Wilma::fredCallsWilma();
    }
protected:
  virtual void wilmaCallsFred()
    {
      std::cout << "Fred::wilmaCallsFred()\n";
    }
};
```

## Tham Khảo

<https://stackoverflow.com/questions/860339/difference-between-private-public-and-protected-inheritance>

<https://isocpp.org/wiki/faq/private-inheritance>

<https://www.bogotobogo.com/cplusplus/private_inheritance.php>