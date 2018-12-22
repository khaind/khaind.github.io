---
layout: post
title: "[C# Versus] readonly với const"
image: /img/csharp-icon.png
subtitle: khác nhau ở cái gì ?
bigimg: #TODO
tags: [Programming, C#, Tips, C#Versus]
---
# So sánh việc sử dụng `readonly` với `const` trong C#

## Readonly vs Const ##

Trước hết cùng nhắc lại ý nghĩa của `readonly` và `const` trong C#.

### Readonly ###

> The readonly modifier prevents a field from being modified after construction. A
read-only field can be assigned only in its declaration or within the enclosing type’s
constructor.

### Const ###

> A constant is a static field whose value can never change. A constant is evaluated
statically at compile time, and the compiler literally substitutes its value whenever
used (rather like a macro in C++). A constant can be any of the built-in numeric
types, bool, char, string, or an enum type.

Như vậy ta có thể tạm so sánh `readonly` vs `Const` như sau

1. Const.

    - Là static field (implicitly static)
    - *Phải và chỉ* được khởi tạo tại thời điểm khai báo

    ```C#
    public const string Message = "Hello World";
    ```

    - Compile time evaluation.

    ```C#
    public static double Circumference (double radius)
    {
    return 2 * System.Math.PI * radius;
    }
    ```

    sẽ được biên dịch giống như đoạn code sau.

    ```C#
    public static double Circumference (double radius)
    {
    return 6.2831853071795862 * radius;
    }
    ```

2. Readonly.

    - `readonly` có thể sử dụng giá trị default mà không cần khởi tạo

    `public readonly int z;`

    - `readonly` có thể đc khởi tạo tại thời điểm khai báo, hoặc trong constructor (run time)

    **Ví dụ về `read only`.**

    ```C#
    public class ReadOnlyTest
        {
            class SampleClass
            {
            public int x;
            // Initialize a readonly field
            public readonly int y = 25;
            public readonly int z;

            public SampleClass()
            {
                // Initialize a readonly instance field
                z = 24;
                y = 35; // you can change y value here
            }

            public SampleClass(int p1, int p2, int p3)
            {
                x = p1;
                y = p2;
                z = p3;
            }
            }

            static void Main()
            {
            SampleClass p1 = new SampleClass(11, 21, 32);   // OK
            Console.WriteLine("p1: x={0}, y={1}, z={2}", p1.x, p1.y, p1.z);
            SampleClass p2 = new SampleClass();
            p2.x = 55;   // OK
            Console.WriteLine("p2: x={0}, y={1}, z={2}", p2.x, p2.y, p2.z);
            }
        }
        /*
        Output:
            p1: x=11, y=21, z=32
            p2: x=55, y=25, z=24
        */
    ```

    Ở ví dụ trên, giá trị 2 trường `y, z` có thể đc khởi tạo tại thời điểm khai báo bằng cách gán giá trị `y=25` hoặc sử dụng giá trị mặc định như `z`. Ngoài ra giá trị của `y, z` có thể đc xác định hoặc thay đổi trong constructor. Vì vậy với những `SampleClass` khác nhau có thể có các giá trị khác nhau. Trong khi với `const`, giá trị hằng số sẽ không bao giờ thay đổi trong khi chạy chương trình sau khi được khởi tạo.

## Static readonly vs const ##

Vậy nếu ta sử dụng `static readonly` thì có khác biệt gì với `const` khi lúc này cả 2 đều là trường `static` (const is a implicitly static).

### Static Variable ###

> A field declared with the static modifier is called a static variable. A static variable comes into existence before execution of the static constructor for its containing type, and ceases to exist when the associated application domain ceases to exist.

Sự khác biệt giữa `static readonly` và `const` nằm ở việc giá trị `Static readonly` vẫn sẽ được đánh giá tại runtime. 
Trong khi giá trị `const` là không bao giờ thay đổi.

Giả sử, ta có assembly X với một `const` như sau

`public const decimal ProgramVersion = 2.3;`
  
Nếu một assembly Y khác tham chiếu đến X và dùng giá trị hằng sô này, trình biên dịch sẽ gán giá trị 2,3 vào assembly. Điều này có nghĩa là khi ta thay đổi giá trị `ProgramVersion` trong các phiên bản sau của X, ví dụ như 2,4, thì assembly Y vẫn sử dụng giá trị cũ của X là 2,3 cho đến khi Y được biên dịch lại. Sử dụng một trường `static readonly` sẽ tránh được trường hợp này.

Như vậy tóm gọn lại, khi nghĩ đến việc sử dụng hằng số, nếu giá trị không bao giờ thay đổi, hãy sử dụng `const`, trong trường hợp khác có thể sử dụng `static readonly`.

## Referrence ##

- <https://msdn.microsoft.com/en-us/library/aa691162(v=vs.71).aspx>
- <https://msdn.microsoft.com/en-us/library/acdd6hb7(v=vs.100).aspx>
- <http://stackoverflow.com/questions/755685/static-readonly-vs-const>
- [C# 6.0 in a nutshell](https://www.amazon.com/C-6-0-Nutshell-Definitive-Reference/dp/1491927062)
