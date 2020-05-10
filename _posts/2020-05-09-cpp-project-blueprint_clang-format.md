---
layout: post
title: "Tại sao nên dùng clang-format"
image: /img/cpp_icon.png
subtitle: Format C++ code the right way
tags: [Programming, coding convention, C++, formatting, llvm, clang]
---

## Coding convention

Trước hết, cần nhắc lại về việc quy định và sử dụng **coding convention** trong mỗi dự án.
Thông thường bất cứ dự án lập trình nào ban đầu đều cần quy định sử dụng **coding convention** trong team ra sao, sau khi đã lựa chọn công nghệ và ngôn ngữ lập trình sử dụng cho dự án. Vậy cụ thể coding convention là gì, và tại sao cần sử dụng nó.
{: .text-justify}

### Khái niệm

**Coding convention** là 1 tập hợp các hướng dẫn khuyên dùng liên quan đến phong cách lập trình (**programming style**) hay các phướng pháp tiếp cận khi lập trình một chương trình với một ngôn ngữ lập trình nhất định nào đó. Thông thường coding convention đề cập đến việc tổ chức project như thế nào, quy định viết code style ra sao (comment, spacing, indentation, naming ...), các nguyên tắc khi lập trình hay thậm chí cả các best practices về kiến trúc. Bài viết này chỉ tập trung vào coding style.
{: .text-justify}

### Why

Việc sử dụng coding convention là cực kỳ cần thiết trong bất kỳ dự án nào bởi điều này tạo sự thống nhất và nhiều lợi ích lớn trong team:

- Việc đọc code trở nên dễ dàng hơn với bất kỳ ai (improve readability)

- Việc bảo trì phần mềm về sau cũng thuận lợi hơn (easier for maintenance)

- Việc peer-review code cũng thuận tiện cho người review hơn

Tuy nhiên có 1 điều là coding convention lại không phải là yêu cầu bắt buộc phải thực thi với compiler, nghĩa là lập trình viên là người cần chủ động đảm bảo việc tuân thủ coding convention khi commit source lên repo.
Do đó cần có các tool giúp lập trình viên tự động hoá quy trình nhàm chán này bởi lập trình viên cũng là con người, còn người thì luôn mắc sai sót :))
Và clang-format chính là vị cứu tinh xuất sắc.
{: .text-justify}

## Clang-format

Clang-format là một thư viện (**library**) và cũng là một chương trình có thể chạy độc lập (**stand-alone tool**) với mục đích tự động format mã nguồn C++ (clang-format hiện cũng đã support một số ngôn ngữ khác: C, Java, JS, Objective-C, C#) theo một file cấu hình cho trước. Do là một tool và library nên clang-format có thể tích hợp với các IDE hay editor như Vim, Emacs, VS, VSCode, CLion ...
Bài viết này sẽ thực hành cấu hình với Vim.
{: .text-justify}

1. Cài đặt

    ```shell
    # Ubuntu
    sudo apt-get install clang-format

    # MacOS
    brew install clang-format
    ```

2. Cấu hình

    Trước hết cần tạo ra file cấu hình .clang-format. Một số style support sẵn có như: LLVM, Google, Chromium, Mozilla, WebKit, Microsoft. Ở đây mình sử dụng style của LLVM.
    {: .text-justify}

    ```shell
    clang-format -style=llvm -dump-config > .clang-format
    ```

    Lúc này chúng ta sẽ có một file text theo định dạng YAML

    ```yaml
    ---
    Language:        Cpp
    # BasedOnStyle:  LLVM
    AccessModifierOffset: -2
    AlignAfterOpenBracket: Align
    AlignConsecutiveMacros: false
    AlignConsecutiveAssignments: false
    AlignConsecutiveDeclarations: true
    AlignEscapedNewlines: Right
    AlignOperands:   true
    # ...
    ```

    Từ đây bạn có thể chỉnh sửa file này theo style riêng của team.
    Ở đây mình có liệt kê một số options ví dụ, bạn có thể tham khảo thêm ở trang chủ bên dưới.

    - AlignConsecutiveAssignments (bool)

    ```cpp
    int aaaa = 12;
    int b    = 23;
    int ccc  = 23;
    ```

    - AlignTrailingComments (bool)

    ```cpp
    true:                                   false:
    int a;     // My comment a      vs.     int a; // My comment a
    int b = 2; // comment  b                int b = 2; // comment about b
    ```

    - FixNamespaceComments (bool)

    ```cpp
    true:                                  false:
    namespace a {                  vs.     namespace a {
    foo();                                 foo();
    } // namespace a                       }
    ```

    - IndentWidth (unsigned)
    The number of columns to use for indentation.

    - ColumnLimit (unsigned)
    The column limit.

3. Format mã nguồn

    Bạn có thể sử dụng clang-format trực tiếp trên terminal như sau

    ```shell
    # format file abc.cpp
    clang-format -i abc.cpp

    # format toàn bộ file với đuôi .cpp
    clang-format -i *.cpp

    # format toàn bộ file đuôi .cpp, .hpp, .cu, .c, .h với file cấu hình
    find . -regex '.*\.\(cpp\|hpp\|cu\|c\|h\)' -exec clang-format -style=file -i {} \;
    ```

4. Tích hợp với editor Vim

    ```vim
    " .vimrc

    " plugin
    Plugin 'autozimu/LanguageClient-neovim', { 'branch': 'next', 'do': 'bash install.sh', }

    " ...

   " format một đoạn code đang chọn với gq
    set formatexpr=LanguageClient#textDocument_rangeFormatting()

    " format toàn bộ buffer đang mở
    nnoremap <silent> <leader>q :call LanguageClient#textDocument_formatting()<CR>

    " tự động format khi lưu file c++
    autocmd BufWritePre *.cpp :call LanguageClient#textDocument_formatting_sync()
    ```

### Tham Khảo

<https://en.wikipedia.org/wiki/Coding_conventions>

<https://clang.llvm.org/docs/ClangFormat.html>

<https://clang.llvm.org/docs/ClangFormatStyleOptions.html>

<https://leimao.github.io/blog/Clang-Format-Quick-Tutorial/>
