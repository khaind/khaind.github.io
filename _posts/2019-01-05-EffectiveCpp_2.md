---
layout: post
title: "[Tóm tắt] Effective C++ by Scott Mayers - Phần 2"
image: /img/cpp_icon.png
subtitle: Constructors, destructor, assignment operators
tags: [Programming, Books, Best Practices, C++]
---

# Tổng kết, tóm tắt nội dung sách Effective C++ by Scott Mayers

## Phần 2: Constructor, destructor, assignment operator

### Item 5: Các hàm C++ tự động tạo và gọi

Nếu programer không khai báo, compiler sẽ tự khai báo 1 bản **copy constructor**, 1 bản **copy assignment operator**, và 1 bản **destructor**. Ngoài ra nếu ko khai báo **constructor**, compiler cũng sẽ tự khai báo 1 **default constructor**
{: .text-justify}

Vì thế khai báo sau:

```cpp
class Empty {};
```

về cơ bản giống như khi ta viết:

```c++
class Empty {
public:
  Empty() { ... }                             // default constructor
  Empty(const Empty& rhs) { ... }             // copy constructor
  ~Empty() { ... }                            // destructor — see below for whether it’s virtual
  Empty& operator=(const Empty& rhs) { ... }  // copy assignment operator
};
```

Khi các hàm trên được gọi, compiler sẽ tự sinh ra 1 bản nếu chưa được khai báo

```c++
Empty e1;         // default constructor;
                  // destructor
Empty e2(e1);     // copy constructor
e2 = e1;          // copy assignment operator
```

Note:

- Với **copy constructor** và **copy assignment operator** được khai báo tự động bởi compiler đơn giản là copy mỗi thành phần non-static của source object sang target object
- Compiler sẽ không tự động khai báo default constructor nếu đã có 1 constructor được khai báo bởi programer
- Compiler có thể không compile code nếu không thẻ generate được các hàm trên

#### Cần nhớ

- Trình biên dịch (**Compilers**) có thể sinh ra **default constructor**, **copy constructor**, **copy assignment operator**, và **destructor** của một class.

### Item 6: Ngăn cản việc compiler tự động sinh ra một số hàm của class

Như đã trao đổi ở mục trước, **copy constructor** hay **copy assignment operator** có thể được compiler tự động sinh ra nếu chúng ta không khai báo những hàm này cho class. Vấn đề nảy sinh khi thực tế có những trường hợp chúng ta không muốn một object có thể được copy hay gán qua một object khác như ví dụ sau:
{: .text-justify}

```cpp
HomeForSale h1;
HomeForSale h2;
HomeForSale h3(h1); // attempt to copy h1 — should not compile!
h1 = h2; // attempt to copy h2 — should not compile!
```

Trường hợp trên khi chúng ta khai báo class *HomeForSale* nhưng không muốn client code có thể copy hay gán giá trị của object h1, h2.

Để thực hiện được mục đích trên chúng ta sẽ khai báo **copy constructor** và **copy assignment operator** cho class *HomeForSale* những đặt những hàm này là hàm private, như vậy client code ko thể gọi những hàm này được, tránh được việc compiler tự dộng sinh ra 2 hàm này public.
{: .text-justify}
Tuy nhiên dù chúng ta có đưa 2 hàm này vào private thì vẫn còn một khả năng là những hàm cài đặt khác bên trong class *HomeForSale* hoặc những hàm là friend của class vẫn có thể gọi được, và nếu những hàm này là public thì client code về thực tế vẫn gọi được 2 hàm trên. Để giải quyết phát sinh trên, chúng ta sẽ chỉ khai báo mà không đưa ra định nghĩa (implementation) cho **copy constructor** và **copy assignment operator**.
{: .text-justify}

```cpp
class HomeForSale {
  public:
    //...
  private:
    //...
    HomeForSale(const HomeForSale&);              // declarations only
    HomeForSale& operator=(const HomeForSale&);
};
```

Như vậy, nếu một hàm friend chẳng hạn nếu vô tình hoặc hữu ý gọi 2 hàm trên sẽ dính link error.
Một cách khác compiler có thể

#### Cần nhớ

- Để loại bỏ các hàm được tự động cung cấp bởi trình biên dịch, hãy khai báo các hàm đó private và không định nghĩa cài đặt cho chúng.

### Item 7: Khai báo **destructor** dạng **virtual** cho **base class** mang tính đa hình (**polymorphic**)

> C++ specifies that when a derived class object is deleted through a pointer to a base class with a non-virtual destructor, results are undefined

Giả sử ta có khai báo class sau:

```cpp
class TimeKeeper {
  public:
    TimeKeeper( );
    ~TimeKeeper( ); ///...
};

class AtomicClock: public TimeKeeper { ... };
class WaterClock: public TimeKeeper { ... };
class WristWatch: public TimeKeeper { ... };
```

đồng thời sử dụng một factory function như bên dưới

```cpp
TimeKeeper* getTimeKeeper();      // returns a pointer to a dynamic
                                  // ally allocated object of a class
                                  // derived from TimeKeeper

// Using factory function
TimeKeeper *ptk = getTimeKeeper();  // get dynamically allocated object
                                    // from TimeKeeper hierarchy
//...
delete ptk;      // release it to avoid resource leak
```

Trong trường hợp trên giả sử *getTimeKeeper()* trả về một object *AtomicClock*, khi giải phóng (delete) biến *ptk*, chỉ có destructor class cha *TimeKeeper* được gọi, dẫn đến một số phần được khai báo trong *AtomicClock* có thể sẽ không được giải phóng -> resources leak.
{: .text-justify}
Để tránh tình trạng trên chúng ta phải khai báo destructor của *TimeKeeper* dạng **virtual**

```cpp
class TimeKeeper {
  public:
    TimeKeeper( );
    virtual ~TimeKeeper();
    //...
};

TimeKeeper *ptk = getTimeKeeper();
//...
delete ptk;     // now behaves correctly
```

Lúc này khi delete biến *ptk* toàn bộ object sẽ được giải phóng đúng cách bao gồm cả **base class** và những vùng nhớ cấp phát riêng cho **derived class**

Note:
> Declare a virtual destructor in a class if and only if that class contains at least one virtual function.

Khi một class chứa ít nhất 1 hàm ảo (**virtual function**) đồng nghĩa với việc người viết có ý định cài đặt nó ở một class kế thừa nào đó. Ngược lại, không nên lạm dụng khai báo virtual destructor tràn lan kể cả với những class thông thường. Một class được thiết kế không nhất thiết phải trở thành base class cho những class khác ví dụ như std::string hay một số STL container (vector, list ..), thậm chí một số base class không nhất thiết phải có tính đa hình (thao tác với class kế thừa qua interface của class cha). Những class này không cần virtual destructor.
{: .text-justify}

#### Cần nhớ

- Những base class thiết kế đa hình thì cần khai báo virtual destructor. Nếu một class có hàm ảo, cần khai báo virtual destructor
- Những class không được thiết kế như base class hay cần sử dụng đa hình không nên khai báo virtual destructor

### Item 8: Ngăn chặn exceptions liên quan đến destructors

> Premature program termination or undefined behavior can result from destructors emitting exceptions even without using containers and arrays. C++ does not like destructors that emit exceptions!

Giả sử ta có một class sau:

```cpp
class Widget {
  public:
    //...
    ~Widget() { ... }     // assume this might emit an exception
};
void doSomething() {
  std::vector<Widget> v;
  //...
} // v is automatically destroyed here
```

Khi kết thúc hàm *doSomething()* vector v sẽ được giải phóng, gọi destructor tương ứng với một phần tử trong vector. Nếu destructor của những phần tử này phát sinh exception thì chương trình sẽ bị ngắt thực thi đột ngột (**premature program termination**) hoặc phát sinh lỗi không thể lường trước được (**undefined behavior**)
{: .text-justify}
Vậy nếu gặp trường hợp destructor gọi hàm có khả năng ném ra exception thì nên xử lý ra sao. Thông thường để tránh điều trên, chúng ta sẽ sử dụng try catch rồi xử lý exception theo 1 trong 2 cách sau:

- Chủ động ngắt chương trình nếu chương trình của chúng ta không thể tiếp tục chạy trong trường hợp lỗi phát sinh

```cpp
DBConn::~DBConn( ) {  // a destructor
  try {
    db.close();
  }
  catch (...) {
    // make log entry that the call to close failed;
    std::abort();
  }
}
```

- Nuốt ngoại lệ (**swallow exception**) nếu chương trình vẫn có khả năng thực hiện đúng nghiệp vụ trong trường hợp tiếp tục thực thi

```cpp
DBConn::~DBConn( ) {
  try {
    db.close();
  }
  catch (...) {
    //make log entry that the call to close failed;
  }
}
```

Hai cách xử lý trên chưa phải là tối ưu. Để tốt nhất, chúng ta nên thiết kế interface của class sao cho client có thể chủ động xử lý exception thay vì trông chờ vào destructor.

```cpp
class DBConn {
  public:
    //...
    void close() {      // function for client usse
      db.close( );
      closed = true;
    }
  ~DBConn( ) {
    if (!closed) {
      try {
        db.close( );    // close the connection if the client didn’t
      }
      catch (...) {   // if closing fails, note that and terminate or swallow
        //make log entry that call to close failed;
        //...
      }
    }
  }
  private:
    DBConnection db;
    bool closed;
};
```

Như vậy chúng ta đã cung cấp thêm 1 hàm close() để client chủ động xử lý (dù có exception hay không) thay vì chờ đợi destructor tự đóng kết nối.

#### Cần nhớ

- Destructors không nên ném ra exceptions. Destructor nên bắt các exception ném ra hoặc chủ động ngắt chương trình.
- Cân nhắc đưa các thao tác có thể sinh ra exception ở 1 hàm thông thường không phải destructor, như vậy client có thể chủ động xử lý khi thực thi thao tác nó muốn.
{: .text-justify}
### Item 9: Không được gọi hàm ảo trong constructor hay destructor

> Base class parts of derived class objects are constructed before derived class parts are
> During base class construction, virtual functions never go down into derived classes. Instead, the object behaves as if it were of the base type

```cpp
class Transaction { // base class for all transactions
  public:
    Transaction( );
    virtual void logTransaction() const = 0; // make type-dependent log entry
    //...
};

Transaction::Transaction() {  // implementation of base class ctor
  //...
  logTransaction();           // as final action, log this transaction
}

class BuyTransaction: public Transaction {    // derived class
  public:
    virtual void logTransaction() const;      // how to log transactions of this type
    //...
};

BuyTransaction b;
```

Ở ví dụ trên, khi tạo đối tượng *b*, hàm khởi tạo của *Transaction* (base class) sẽ được gọi trước, và trong hàm khởi tạo này gọi *logTransaction()* của chính nó chứ không phải *logTransaction()* **override** bởi *BuyTransaction*. Điều này cũng dễ hiểu bởi bản thân đối tượng *BuyTransaction b* vẫn chưa khởi tạo xong thì không thể có bất kỳ cài đặt nào của *logTransaction()* bên trong nó cả mà gọi được. Quay lại việc hàm khởi tạo của *Transaction* gọi *logTransaction()*, lúc này sẽ nguy hiểm nếu *logTransaction()* là hàm **pure virtual**, nghĩa là sẽ không có bất kỳ cài đặt nào của nó bên trong class *Transaction*. Trường hợp này lại tuỳ compiler, một số sẽ thông báo warnings, số khác lại không.
{: .text-justify}
Trường hợp destructor cũng hoàn toàn tương tự.

#### Cần nhớ

- Không được gọi **virtual functions** trong **constructor** hay **destructor**, because such calls will never go to a more derived class than that of the currently executing constructor or destructor.

### Item 10: Trả về tham chiếu (**reference**) tới *this trong cài đặt của **assignment operators**

Trong c++ bạn có thể thực hiện phép gán (**assignment**) liên tục như sau bởi assignment có tính chất kết hợp **right-associative**

```cpp
intx,y,z;
x = y = z = 15; //chain of assignments
// ~ x=(y=(z=15));
```

Để thực hiện điều này với 1 object ta cần cài đặt nó như sau:

```cpp
class Widget {
  public:
    // ..
    Widget& operator=(const Wiget& rhs) {     // return type is reference to current class
      //..
      return *this;                           // return the left-hand object
    }
    // ...
};
```

Tương tự với các toán tử khác như +=, -=, *=, ... ta cũng áp dụng theo quy ước này.

#### Cần nhớ

- Cài đặt **assignment operators** bằng cách trả về một reference tới con trỏ this

### Item 11: Xử lý việc gán **assignment** cho chính bản thân object trong toán tử operator=

Nghe có vẻ buồn cười nhưng thực tế chúng ta cũng hay gặp trường hợp gán một đối tượng cho chính nó, tỉ dụ như những đoạn code dưới đây

```cpp
a[i] = a[j];      // potential assignment to self
*px = *py;        // potential assignment to self
class Base { ... };
class Derived: public Base { ... };
void doSomething(const Base& rb, Derived* pd);  // rb and *pd might actually be the same object
```

Để xử lý việc này một cách hiệu quả chúng ta cần thêm đoạn code gọi là **identity test** vào cài đặt của assignment operator như sau:

```cpp
class Bitmap { ... };
class Widget { ...
  private:
    Bitmap *pb;   // ptr to a heap-allocated object

};

Widget& Widget::operator=(const Widget& rhs) {
  if (this == &rhs)         // identity test: if a self-assignment, do nothing
    return *this;

  delete pb;
  pb = new Bitmap(*rhs.pb);

  return *this;
}
```

Một số kỹ thuật khác cũng có thể được sử dụng có thể kể đến như **statement ordering** hay **copy and swap**

```cpp
Widget& Widget::operator=(const Widget& rhs) {
  Bitmap *pOrig = pb;         // Remember old poiter
  pb = new Bitmap(*rhs.pb);   // copy data from rhs object
  delete pOrig;               // clean old resource

  return *this;
}
```

```cpp
class Widget {
  //...
  void swap(Widget& rhs); // exchange *this’s and rhs’s data;
  //...
};

Widget& Widget::operator=(const Widget& rhs) {
  Widget temp(rhs);     // make a copy of rhs’s data
  swap(temp);           // swap *this’s data with the copy’s
  return *this;
}
```

#### Cần nhớ

- Đảm bảo rằng assignment operator hoạt động đúng trong trường hợp gán cho chính nó sử dụng **identity test**, **statement ordering** hay **copy and swap**

### Item 12: Đảm bảo copy toàn bộ các thành phần của một đối tượng

> When you’re writing a copying function, be sure to (1) copy all local data members and (2) invoke the appropriate copying function in all base classes, too

Trong C++ có 2 hàm đảm nhiệm việc copy các đối tượng là **copy constructor** và **copy assignment operator**, từ đây gọi chung là các hàm copy (**copying functions**).

Trong quá trình code, nhiều lúc chúng ta cần update những class có sẵn bằng việc thêm các trường data mới vào class. Trong trường hợp này, nhất thiết phải update  các hàm copy để các hàm này cũng xử lý các trường data ta mới thêm vào. Điều này cần lưu ý bởi compiler sẽ không đưa ra bất kỳ cảnh báo nào nếu chúng ta quên và việc quên sẽ khiến chương trình chạy không chuẩn xác nhất là với những class có class kế thừa
{: .text-justify}
Ngoài ra, vì bản thân các hàm copy thường có những đoạn xử lý giống nhau (mặc dù mục đích sử dụng của copy constructor và assignment operator là khác nhau), chúng ta thường muốn gom những đoạn xử lý này lại để tránh lặp code. Tuy nhiên tuyệt đối không xử lý bằng cách để 2 hàm copy gọi nhau (copy constructor gọi assignment operator hoặc ngược lại), mà cần đưa những đoạn code giống nhau qua 1 hàm thứ ba (thường đặt là init()) dạng private.
{: .text-justify}
