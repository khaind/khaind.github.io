---
layout: post
title: "[Tóm tắt] Effective C++ by Scott Mayers - Phần 3"
image: /img/cpp_icon.png
subtitle: Resource Management
tags: [Programming, Books, Best Practices, C++, RAII]
---

## Phần 3: Quản lý tài nguyên

**Resource** (Tài nguyên hệ thống) thường dùng ví dụ như **dynamic memory allocated** (bộ nhớ cấp phát động), **file descriptor**, **mutex lock**, **font**, **network socket**, **database connection**. Cũng giống như tài nguyên thiên nhiên, những resource này là hữu hạn, vì vậy khi chúng ta sử dụng chúng xong thì phải trả về cho hệ thống để khi cần hệ thống có thể cấp phát cho những chương trình khác sử dụng.
{: .text-justify}

### Item 13: Sử dụng objects để quản lý resource dễ dàng hơn

Trong C++, chúng ta thường dùng con trỏ (**pointer) để tận dụng bộ nhớ heap, rồi giải phóng bộ nhớ đó sau khi đã sử dụng xong như sau:

```cpp
void f() {
  Investment *pInv = createInvestment();  // call factory function
  // ...                                  // use pInv
  delete pInv;                            // release object
}
```

Cách làm trên không sai, nhưng chưa đủ an toàn, bởi câu lệnh giải phóng bộ nhớ có thể không được thực thi trong nhiều trường hợp:

- Premature return statement

  Dù chúng ta có thể cẩn thận phòng tránh trường hợp này, nhưng không thể đảm bảo 1 người khác khi thay đổi code hàm *f()* cũng cẩn thận như chúng ta

- Exception throw

  Hoàn toàn có khả năng những hàm xử lý trước đó có thể ném ra ngoại lệ. Khi đó, chúng ta sẽ bị leak vùng nhớ của *pInv* đang trỏ tới

Giải pháp an toàn hơn là sử dụng **smart pointer** để đóng gói tài nguyên vào một object. Như vậy chúng ta có để lợi dụng destructor của object để giải phóng vùng nhớ đang dùng. Ý tưởng này thường được gọi là RAII (Resource Acquisition Is Initialization). Và những object này được gọi là resource-managing objects.
{: .text-justify}

> Resources are acquired and immediately turned over to resource-managing objects.
> Resource-managing objects use their destructors to ensure that resources are released

```cpp
void f() {
  //...
  std::tr1::shared_ptr<Investment> pInv(createInvestment( )); // call factory function
  //...           // use pInv as before
}                 // automatically delete pInv via shared_ptr's destructor
```

Lưu ý:
  Không sử dụng smart pointer cho một mảng dữ liệu, bởi destructor của pointer sử dụng *delete* để giải phóng dữ liệu chứ không dùng *delete[]*

```cpp
std::tr1::shared_ptr<int> spi(new int[1024]); // bad idea! the wrong delete form will be used
```

#### Cần nhớ

- Để đảm bảo không bị lủng tài nguyên, sử dung RAII objects để khởi tạo resource trong hàm constructor của nó và có thể giải phóng resource trong destructor.
{: .text-justify}

### Item 14: Cẩn trọng về copying behavior trong các lớp resource-managing classes

Ở mục trước, chúng ta dùng đến shared_ptr để quản lý resource. Ở phần này chúng ta sẽ tự viết 1 RAII class tương tự như sau:

```cpp
class Lock {
  public:
    explicit Lock(Mutex *pm) : mutexPtr(pm)
      {
        lock(mutexPtr);               // acquire resource
      }
    ~Lock() { unlock(mutexPtr); }     // release resource
  private:
    Mutex *mutexPtr;
};
```

Class *Lock* trên quản lý resource của một con trỏ *Mutex*. Nó lock mutex đó trong hàm khởi tạo và unlock mutex trong hàm destructor.
Như vậy client code có thể sử dụng *Lock* theo kiểu RAII một cách khá an toàn như dưới đây:

```cpp
Mutex m;              // define the mutex you need to use
//...
{                     // create block to define critical section
  Lock ml(&m);        // lock the mutex
  //...               // perform critical section operations
}                     // automatically unlock mutex at end of block
```

Tuy nhiên, có 1 trường hợp phát sinh khi client code muốn copy 2 đối tượng *Lock*

```cpp
Lock ml1(&m); // lock m
Lock ml2(ml1); // copy ml1 to ml2 — what should happen here?
```

Như vậy khi thiết kế các class RAII ngoài việc khởi tạo/giải phóng resource trong constructor/destructor, chúng ta cần cân nhắc các copying function hoạt động ra sao. Thông thường tuỳ theo ý nghĩa của từng class mà nhu cầu sử dụng sẽ khác nhau. Dưới đây là 1 trong các hướng cài đặt chúng ta nên cân nhắc:
{: .text-justify}

- **Prohibit copying**

  Với class *Lock* như ở trên sẽ không có nhiều ý nghĩa khi copy đối tượng mutex. Do đó, chúng ta có thể loại bỏ copying function, để client code không thể thực hiện việc copy 2 đối tượng. Cách làm đã được trình bày ở Item 6.
  {: .text-justify}
- **Reference-count the underlying resource**

  Giống như shared_ptr sử dụng 1 biến đếm (reference count) để biết hiện có bao nhiêu smart pointer đang trỏ đến vùng nhớ đó. Khi có 1 smart pointer mới trỏ đến vùng nhớ đó, biến đếm tăng lên 1, ngược lại khi 1 smart pointer được sử dụng xong, biến đếm sẽ giảm đi 1. Khi biến đếm trở về 0, vùng nhớ liên quan sẽ được giải phóng. Chúng ta cũng có thể tận dụng luôn shared_ptr để quản lý resource trong class. Ví dụ với class *Lock* trên sẽ như sau:
  {: .text-justify}

  ```cpp
  class Lock {
    public:
      explicit Lock(Mutex *pm) : mutexPtr(pm, unlock)   // init shared_ptr with the Mutex
      {                                                 // to point to and the unlock func as the deleter
        lock(mutextPtr.get());                          // see Item 15 for info on “get”
      }
      private:
        std::tr1::shared_ptr<Mutex> mutexPtr; // use shared_ptr instead of raw pointer
  };
  ```

  Lưu ý, chúng ta truyền *unlock* vào khởi tạo của shared_ptr *mutextPtr* để dùng như một **custom deleter**, bởi mặc định, shared_ptr sẽ gọi *delete* (điều này không đúng mong muốn của class *Lock*).
- **Copy the underlying resource**

  Nếu chúng ta thực sự cần nhiều bản copy của resource, hãy thực hiện **deep copy** tức là copy toàn bộ resource đang được quản lý. Một ví dụ của trường hợp này là std::string
- **Transfer ownership of the underlying resource**

  Trường hợp này khá hiếm, khi copy thì đối tượng được copy sẽ là chủ sở hữu duy nhất của resource được quản lý bởi đối tượng cũ, ví dụ như trường hợp auto_ptr (sử dụng auto_ptr là không đc khuyến cáo nhé)

#### Cần nhớ

- Khi sử dụng RAII class, tuỳ thuộc vào nhu cầu copy resource như thế nào sẽ quyết định việc cài đặt copying function của RAII object ra sao
- Một số hành vi xử lý của copying functions phố biến có thể kể đến: ngăn chặn hoàn toàn việc copy, và thực hiện reference counting

### Item 15: Cung cấp phương thức truy cập resources trong các lớp resource-managing classes

Sử dụng resource-managing classes được khuyến cáo để giảm thiểu việc lủng tài nguyên (resource leak). Tuy nhiên khi tương tác với những module khác (legacy code chẳng hạn), có một số API muốn truy xuất trực tiếp vào resource (dùng con trỏ). Vậy để đáp ứng yêu cầu này, thông thường các RAII classes cần cung cấp hàm trả ra raw pointer trỏ tới vùng nhớ đang quản lý.
{: .text-justify}
Ví dụ shared_ptr cung cấp hàm *get()* trả về con trỏ trỏ tới resource, ngoài ra cũng **overload** các *operator->* và *operator\** để client code truy xuất vào resource một cách dễ dàng

```cpp
class Investment {                      // root class for a hierarchy of investment types
  public:
    bool isTaxFree() const;
  //...
};

std::tr1::shared_ptr<Investment> pInv(createInvestment());

// sample API
int daysHeld(const Investment *pi);       // return number of days investment has been held

// int days = daysHeld(pInv);             // error!

int days = daysHeld(pInv.get());          // fine, passes the raw pointer in pInv to daysHeld

bool taxable1 = !(pInv->isTaxFree());     // access resource via operator->
bool taxable2 = !((*pInv).isTaxFree());   // access resource via operator*
```

Có 2 cách để cung cấp việc truy xuất tới raw resource cho client code: **explicit conversion function** và **implicit conversion function**

```cpp
FontHandle getFont();             // from C API — params omitted // for simplicity
void releaseFont(FontHandle fh);  // from the same C API

class Font {                              // RAII class
  public:
    explicit Font(FontHandle fh) : f(fh)  // acquire resource;
    {}
    ~Font( ) {
      releaseFont(f );                    // release resource
    }
    //...                                 // handle copying

    FontHandle get() const {              // explicit conversion function
      return f;
    }

    operator FontHandle() const {         // implicit conversion function
      return f;
    }
  private:
    FontHandle f;                         // the raw font resource
};

// Usage
void changeFontSize(FontHandle f, int newSize);
Font f(getFont()); int newFontSize;
// ...
changeFontSize(f.get(), newFontSize);     // explicitly convert Font to FontHandle
// or
changeFontSize(f, newFontSize);           // implicitly convert Font to FontHandle
```

Như vậy nếu sử dụng **explicit conversion function** thì client code luôn luôn phải gọi hàm *get()*, ngược lại dùng **implicit conversion function** sẽ tiện hơn cho client code khi chỉ cần gọi qua smart pointer object. Tuy nhiên cần sử dụng thận trọng **implicit conversion function** vì dễ nhầm lẫn giữa 2 lớp *Font* và *FontHandle*

```cpp
Font f1(getFont());
//...
FontHandle f2 = f1;   // oops! meant to copy a Font object, but instead implicitly
                      // converted f1 into its underlying FontHandle, then copied that
```

#### Cần nhớ

- Các APIs thường sẽ cần truy cập tới raw resources, vì vậy RAII class nên cung cấp những cách thức thể client code truy xuất đc tới resource dễ dàng.
- Có 2 cách cài đặt thông qua explicit conversion (an toàn hơn) hoặc implicit conversion (thuận tiện hơn cho client code)

### Item 16: Sử dụng **new** và **delete** đúng cách

> If you use [] in a new expression, you must use [] in the corresponding delete expression. If you don’t use [] in a new expression, don’t use [] in the matching delete expression

Chúng ta sử dụng operator **new** để tạo một object hoặc 1 mảng objects trên bộ nhớ heap. Tuy nhiên có sự khác biệt về **memory layout** giữa 1 object và 1 mảng objects

Single object

| Object  |

Array

|   n     | Object  | Object  | Object  | ...   |

Bởi vậy, để giải phóng 1 mảng objects chúng ta cần sử dụng *delete []* thay vì *delete*

```cpp
std::string *stringPtr1 = new std::string;
std::string *stringPtr2 = new std::string[100];
// ...
delete stringPtr1; // delete an object
delete [] stringPtr2; // delete an array of objects
```

#### Cần nhớ

- Nếu chúng ta dùng **operator new** để khởi tạo 1 mảng object, thì phải dùng **delete []** khi giải phóng bộ nhớ. Nếu chỉ khởi tạo 1 object đơn lẻ, thì không được sử dụng **delete []** khi giải phóng bộ nhớ

### Item 17: Store newed objects in smart pointers in standalone statements

Giả sử chúng ta có đoạn code sau:

```cpp
int priority();
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);

// ...
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
```

Trước khi thực hiện hàm processWidget, compilers phải thực hiện các bước sau

1. Gọi hàm priority()
2. Thực hiện lệnh “new Widget”
3. Gọi constructor của tr1::shared_ptr

Tuy nhiên compiler chỉ đảm bảo thứ tự thực hiện *new Widget* trước khi gọi constructor của tr1::shared_ptr, việc gọi hàm *priority()* hoàn toàn không theo 1 thứ tự cố định nào. Điều sau đây hoàn toàn có thể xảy ra

1. Thực hiện lệnh “new Widget”
2. Gọi hàm priority()
3. Gọi constructor của tr1::shared_ptr

Vậy trong trường hợp này, nếu *priority()* ném ra exception, chương trình của chúng ta sẽ bị memory leak.

Để tránh tình trạng trên chúng ta cần khởi tạo smart pointer bên ngoài trước, rồi mới truyền vào hàm *processWidget()*

```cpp
std::tr1::shared_ptr<Widget> pw(new Widget); // store newed object in a smart pointer in a standalone statement
processWidget(pw, priority());               // this call won’t leak
```

#### Cần nhớ

- Khởi tạo đối tượng smart pointers ở một câu lệnh đơn lẻ để tránh tình trạng lủng resource trong 1 số trường hợp
