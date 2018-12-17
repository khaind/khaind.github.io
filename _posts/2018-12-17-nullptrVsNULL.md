---
layout: post
title: "[Cpp Versus] nullptr có giống NULL ?"
image: https://c1.staticflickr.com/8/7282/8743059428_a4733e06b7_b.jpg
subtitle: Hiểu rõ hơn về nullptr trong C++11
bigimg: https://external-preview.redd.it/fOELi1tRrcZwCi1aRkD1qQ3mXX1HDjF-E6J3GQXxn1Y.jpg?auto=webp&s=e919d930d17cd605cf9eae7feb01fcd36ffc853e
tags: [Programming, C++, nullptr]
---
# Tại sao chúng ta nên sử dụng **nullptr** thay vì **NULL**

![""](https://hownot2code.files.wordpress.com/2016/06/16k4l4.jpg)

Trong C++11 xuất hiện một từ khóa (**keyword**) mới là **nullptr**.
**nullptr** được sử dụng để chỉ một con trỏ là null (**null pointer**).
Trước đây chúng ta hay sử dụng **NULL** cho mục đích này (kế thừa từ C ^^)
Tuy nhiên **NULL** về bản chất chỉ là một số nguyên có giá trị = 0 (zero integer).

Vậy tại sao ta nên sử dụng **nullptr** thay cho **NULL**.

```cpp
int f(int value)
{
    // ...
}

int f(foo* parameter)
{
    // ...
}

int main()
{
    return f(NULL); // which overload are we calling?
}
```

Trong trường hợp trên, hàm được gọi trong hàm *main()* sẽ là hàm *f(int value)*, bởi vì đơn giản **NULL** ko phải là một con trỏ mà chỉ là một số nguyên.
Nêu developer chủ định muốn gọi hàm overload *f(foo\* parameter)* thì cần sử dụng **nullptr**

```cpp
int main()
{
    return f(nullptr);
}
```

## Tóm lại như sau

!["ptr vs nullptr"](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRXI7HSCPpfBpi5RMjnJ2I82q2DevkhuWbMNLvzCh3SEqbPRDF4 "ptr vs nullptr")

!["NULL"](https://i1.wp.com/blog.feabhas.com/wp-content/uploads/2015/06/image7.png "NULL")

Tham khảo:
<https://www.moderncplusplus.com/nullptr-vs-null/>