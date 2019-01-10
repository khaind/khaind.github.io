---
layout: post
title: "[Cpp FAQ] Sử dụng 'using namespace std'"
image: /img/cpp_icon.png
subtitle: Nên hay không nên ?
tags: [Programming, C++, namespace, cpp faq]
---

# Có nên sử dụng `using namespace std` hay không ?

# Không nên

Chúng ta thường không thích việc gõ đi gõ lại `std::`, nên nhiều khi thích sử dụng `using namespace std` hơn. Khi sử dụng `using namespace std`, compiler sẽ nhìn thấy tất cả các hàm khai báo trong namespace `std`, kể cả những thứ mà chúng ta không nghĩ tới sẽ dùng đến. Nói một cách khác, điều này dẫn đến việc xung đột tên (name conflicts).

Ví dụ, bạn có một đoạn code dùn để đếm và bạn sử dụng biến hoặc hàm có tên `count`. Tuy nhiên `std library` cũng sử dụng `count` (trong std algorithms), điều này dẫn đến việc không rõ ràng về ngữ nghĩa (ambiguties)

Ý nghĩa của việc sử dụng `namespace` là để tránh xung đột namespace giữa hai source code phát triển độc lập nhau. `Using-directive` (using namespace XYZ) sẽ import toàn bộ nội dung của 1 namespace vào 1 namespace khác, điều này vô tình đi ngược lại mục đích trên. `Using-directive` tồn tại cho các đoạn mã legacy C++, nhưng bạn có lẽ không lên sử dụng nó thường xuyên ít nhất là với các đoạn mã phát triền mới.

Nếu bạn thực sự muốn tránh phải gõ `std::`, bạn có thẻ sử dụng `using-declaration`. Hoặc đơn giản hơn là khiến bản thân không ngại gõ `std::` nữa :v

Sử dụng một `using-declaration` sẽ specific cho 1 name cụ thể. Ví dụ, để sử dụng `std::cout`, bạn có thể thêm `using std::cout`

```c++
#include <vector>
#include <iostream>
void f(const std::vector<double>& v)
{
 using std::cout;  // ← a using-declaration that lets you use cout without qualification
 cout << "Values:";
 for (auto value : v)
   cout << ' ' << value;
 cout << '\n';
}
```

Hoặc trở về phương án nguyên thuỷ (the un-solution):

```c++
#include <vector>
#include <iostream>
void f(const std::vector<double>& v)
{
 std::cout << "Values:";
 for (auto value : v)
   std::cout << ' ' << value;
 std::cout << '\n';
}
```

Bạn có thể sử dụng cách nào cũng được, nhưng nên nhớ thống nhất với coding standards đã được lựa chọn sử dụng trong team. Bởi dù sao bạn cũng chỉ là 1 phần của một team lớn hơn mà thôi.

## Tham Khảo

<https://isocpp.org/wiki/faq/coding-standards>