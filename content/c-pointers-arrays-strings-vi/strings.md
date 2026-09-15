---
title: "Chuỗi C"
---

(sec-strings)=
## Mục tiêu học tập

* Phân biệt giữa mảng ký tự và chuỗi C.
* Biết cách sử dụng các hàm thư viện chuẩn C trong `string.h`.

Không có video.

## Chuỗi C vs. mảng `char`

Một **chuỗi C** (tức là "string") chỉ là một mảng các ký tự, theo sau là một **ký tự kết thúc null**. Một **ký tự kết thúc null** là byte toàn số 0, tức là ký tự `'\0'`. Giá trị ASCII của ký tự kết thúc null là `0`.

Ký tự kết thúc null cho phép chúng ta xác định độ dài của chuỗi C chỉ từ một con trỏ đến đầu chuỗi.

Khi bạn tạo một mảng ký tự, bạn nên kết thúc mảng bằng ký tự kết thúc null. Ví dụ, đoạn mã

```c
char my_str[] = {'e', 'x', 'a', 'm', 'p', 'l', 'e', '\0'};
```

khai báo một mảng `char` 8-byte trên stack, rồi khởi tạo mảng với 8 `char` được chỉ định. Đọc về stack trong [phần khác](#sec-stack).

Nếu bạn đang sử dụng dấu ngoặc kép (`"`) để tạo chuỗi, ký tự kết thúc null được thêm ngầm định, nên bạn không nên tự thêm nó. Ví dụ đoạn mã

```c
char *my_str = "example";
```

khai báo một con trỏ đến chuỗi ký tự 8-byte (bao gồm ký tự kết thúc null).

:::{tip} Kiểm tra nhanh

Trong đoạn mã bên dưới, `arr` có phải là chuỗi C không?

```c
char arr[] = {'h', 'e', 'l', 'l', 'o'};
```
:::

:::{note} Hiển thị đáp án
:class: dropdown

Không. Trong khi `arr` là một mảng `char`, nó không kết thúc bằng ký tự kết thúc null và theo định nghĩa không phải là chuỗi C.
:::

:::{warning} Cấp phát đủ bộ nhớ cho ký tự kết thúc null

Khi cấp phát bộ nhớ cho một chuỗi, phải có đủ bộ nhớ để lưu các ký tự trong chuỗi và ký tự kết thúc null. Nếu không, bạn có thể gặp hành vi không xác định. Tuy nhiên, mảng có thể lớn hơn chuỗi mà nó lưu trữ.

:::

## `<string.h>`

Chuỗi C có các hàm trong thư viện chuẩn C, được nhập qua header `<string.h>`. Xem [Wikibooks](https://en.wikibooks.org/wiki/C_Programming/String_manipulation#The_%3Cstring.h%3E_standard_header) để biết mô tả các hàm `<string.h>` thường dùng. Đây là hai hàm bạn có thể gặp trong khóa học này:

* `strlen`: tính độ dài của chuỗi bằng cách đếm số ký tự trước ký tự kết thúc null
* `strcpy`: sao chép một chuỗi từ một vị trí bộ nhớ sang vị trí khác, từng ký tự một cho đến khi gặp ký tự kết thúc null (ký tự kết thúc null cũng được sao chép).

Để đọc về bất kỳ hàm chuỗi chuẩn nào, chúng tôi khuyến nghị các trang hướng dẫn ("man pages"). Bạn có thể gõ lệnh sau vào terminal:

```
man strlen
```

Xem xét đoạn mã sau, là một triển khai hợp lý của `strlen`[^strlen-practical]. Hàm `strlen` là hàm thư viện chuẩn C tính độ dài của chuỗi, trừ ký tự kết thúc null.

[^strlen-practical]: Xem [glibc](https://github.com/lattera/glibc/blob/master/string/strlen.c) để có triển khai thực tế, hiệu quả hơn của `strlen`.

```{code} c
:linenos:

int strlen(char s[]) {
    size_t n = 0; 
    while (*(s++) != 0) { n++; } 
    return n;
}
```

:::{note} Giải thích

* Dòng 1: Cú pháp mảng trong tham số là cú pháp đường cho con trỏ; ở đây, nó tương đương với `char *s`, khai báo `s` như một con trỏ đến `char`. Ở đây, chúng ta giả sử thêm rằng `s` trỏ đến một chuỗi C, nhưng không có cách nào để mô tả ràng buộc này một cách rõ ràng qua khai báo kiểu.
* Dòng 2: Khai báo một số nguyên không dấu cục bộ `n` đủ lớn để giữ bất kỳ số byte nào trong bộ nhớ (đây là typedef `size_t`)
* Dòng 3: Nhiều thứ đang diễn ra trong vòng lặp while này.
  * Thân: Tăng `n` thêm một.
  * Điều kiện:
    * Tăng giá trị của `s` thêm một. Trước khi làm điều đó, giải tham chiếu `s` để lấy ký tự hiện tại.[^post-increment] Giá trị của biểu thức là ký tự hiện tại.
    * Đánh giá thành `true` nếu ký tự hiện tại không phải là ký tự null (`'\0'` có giá trị nhị phân `0`).

[^post-increment]: `*(s++)` sử dụng tăng **sau** (post-increment): trước tiên giải tham chiếu `s` để đọc ký tự hiện tại, rồi tăng `s` thêm một. C cũng hỗ trợ toán tử tăng (và giảm) **trước** (pre-increment) `(++s)` (và `(--s)`), đánh giá thành giá trị sau khi thao tác hoàn thành. Sự khác biệt nằm ngoài phạm vi của khóa học này.
:::

## Chuỗi ký tự (String literals)

Các chuỗi được tạo với cú pháp sau là chỉ đọc, hoặc **bất biến (immutable)**. Sau khi **chuỗi ký tự** này được tạo, chương trình C có thể giải tham chiếu `my_immutable_str` và đọc dữ liệu của nó, nhưng không thể thay đổi giá trị của chuỗi trong quá trình thực thi.

```c
char *my_immutable_str = "Hello";
```

Ngược lại, khai báo bên dưới tạo một chuỗi _có thể_ thay đổi:

```c
char my_str[] = "hello";
```

:::{note} Giải thích

Tại sao chuỗi đầu tiên bất biến trong khi chuỗi thứ hai có thể thay đổi? Câu trả lời là [bố cục bộ nhớ](#sec-mem-layout). Chuỗi đầu tiên được lưu trong phân đoạn dữ liệu chỉ đọc[^rodata] của bộ nhớ, trong khi chuỗi thứ hai được lưu trên stack.

:::

[^rodata]: Chuỗi ký tự được đặt trong phân đoạn dữ liệu chỉ đọc, mà chúng ta không thảo luận trong lớp này. Đọc thêm trong phần chú thích khi chúng ta thảo luận về [bố cục bộ nhớ](#sec-mem-layout).
