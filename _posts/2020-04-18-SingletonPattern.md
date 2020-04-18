---
layout: post
title: "[Design Pattern] Singleton trong C++"
image: /img/cpp_icon.png
subtitle: C++ Singleton the right way
tags: [Programming, Design Pattern, C++, C++11]
---

# Sử dụng Singleton đúng trong C++

## Nguồn gốc

Singleton là một mẫu thiết kế (**design pattern**) được đề xuất bởi *Gang of Four* (bốn đồng tác gỉa) qua cuốn sách nổi tiếng *Design Patterns: Elements of Reusable Object-Oriented Software* xuất bản năm 1994. Singleton là mẫu thiết kế thuộc nhóm Creational Pattern, tức là liên quan tới việc khởi tạo object
{: .text-justify}
Các tác giả giới giới thiệu singleton như sau

> Ensure a class has one instance, and provide a global point of access to it.

Có 2 điểm cần lưu ý ở đây:

1. Khi class chỉ cần sử dụng 1 và duy nhất 1 instance

2. Cung cấp một điểm truy cập toàn cục tới instance này thay vì qua constructor

Vậy cùng phân tích 2 điểm này ở các hướng tiếp cận cài đặt singleton sau trong C++

## Eager initialization

```cpp
class Singleton {
public:
    Singleton(const Singleton&) = delete;   // 1c. delete copy constructor
    static Singleton& getInstance() {       // 2a. public function for client code usage
        std::cout << "You want my instance, i'm yours" << std::endl;
        return _instance;
    }
    void doOneThing() {
        std::cout << "Do thing #1" << std::endl;
    }
private:
    Singleton() = default;          // 1a. Don't public constructor function
    static Singleton _instance;     // 1b. static private instance
};

Singleton Singleton::_instance;     // define static global variable

int main()
{
    std::cout << "Hello singleton" << std::endl;
    // Singleton s;                             // failed due to private constructor
    // Singleton s1 = Singleton::getInstance(); // failed due to deleted copy constructor
    Singleton::getInstance().doOneThing();      // use directly

    auto& s = Singleton::getInstance();
    s.doOneThing();                             // use via reference variable
    std::cout << "Good bye!" << std::endl;
    return 0;
}
```

Cách cài đặt đầu tiên này sử dụng 1 biến **static member variable** *_instance*, khởi tạo biến này ở **global scope**. API public *getInstance()* đơn giản trả ra instance đã được khởi tạo đó, và API này là duy nhất để có thể truy xuất được instance. Lưu ý ở đây constructor được đưa vào **private scope**, **copy constructor** thì được *delete*, như vậy client user không thể khởi tạo hay copy từ instance duy nhất -> đảm bảo điều 1.
Tuy nhiên cách cài đặt này yêu cầu chúng ta phải khai báo *_instance_* của class ở bên ngoài, khiến code trông không được đẹp, và instance sẽ được khởi tạo ngay khi chương trình chạy.
{: .text-justify}

## Lazy initialization

Bây giờ chúng ta sẽ đưa khởi tạo của instance vào bên trong public API như sau

```cpp
class Singleton {
public:
    Singleton(const Singleton&) = delete;   // 2b. delete copy constructor
    static Singleton& getInstance() {       // 2a. public function for client code usage
        std::cout << "You want my instance, i'm your" << std::endl;
        static Singleton _instance;         // 1b. static local variable
        return _instance;
    }
    void doOneThing() {
        std::cout << "Do thing #1" << std::endl;
    }
private:
    Singleton() = default;          // 1a. Don't public constructor function
};
```

Như vậy class Singleton đã trông gọn hơn nhiều, và chỉ khi hàm *getInstance()* được gọi thì một instance mới được khởi tạo. Lưu ý instance này sẽ chỉ được khởi tạo 1 lần duy nhất, và việc khởi tạo này cũng là **thread safe** kể từ C++11
{: .text-justify}
 > If control enters the declaration concurrently while the variable is being initialized, the concurrent execution shall wait for completion of the initialization.

Note: một số compiler không cài đặt toàn bộ C++11 standard có thể sẽ không hoạt động đúng tỉ dụ như Visual Studio 2013

## Classless approach

Cuối cùng là cách cài đặt Singleton mà hoàn toàn không sử dụng tới class

```cpp
namespace SingletonNS {
    namespace {
        // custom data goes here
        static int meaningOfLife = 42;
    }
    static void doOneThing() {
        std::cout << "Do thing and found " << meaningOfLife << std::endl;
    }
}

// Usage
SingletonNS::doOneThing();
```

Sử dụng namespace chúng ta hoàn toàn có thể có được *behavior* giống như cài đặt một class *Singleton*, và việc viết code cũng đơn giản hơn mặc dù code lúc này có vẻ không có cấu trúc rõ ràng như khi đưa vào class.
{: .text-justify}

## Kết luận

Singleton là một mẫu thiết kế khá nổi tiếng, mặc dầu vậy nó cũng khá tai tiếng. Nhiều developer coi Singleton như 1 anti-pattern cần tránh. Và thực sự thì điều gì cũng có 2 mặt. Singleton cũng vậy, cũng có nhược điểm như sử dụng global scope (luôn cần cân nhắc và tránh dùng), và có vẻ vi phạm nguyên tắc Single Responsibility Principle. Chính vì vậy, trước khi thực sự cần sử dụng Singleton hay cân nhắc xem mình có thực sự cần nó hay không, nếu thực sự cần thì có thể sử dụng 1 trong các cài đặt như trên (khuyến cáo sử dụng #2).
{: .text-justify}

### Tham Khảo

<http://gameprogrammingpatterns.com/singleton.html>
<https://preshing.com/20130930/double-checked-locking-is-fixed-in-cpp11/>
<https://www.youtube.com/watch?v=PPup1yeU45I>
