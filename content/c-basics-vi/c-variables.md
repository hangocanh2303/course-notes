---
title: "Biến trong C"
subtitle: "Biến có Kiểu và Danh sách các Chủ đề"
---

## Mục tiêu học tập

Bạn không được mong đợi học nhiều, nhiều chi tiết của C ngay lập tức. Nhưng bạn nên biết cách khai báo và khởi tạo biến.

* Khai báo và khởi tạo các kiểu biến cơ bản trong C.
* Sử dụng các typedef của `stdint.h` khi có thể, vì độ rộng của các kiểu số nguyên cơ bản phụ thuộc vào bộ xử lý.
* Luôn nhớ rằng các biến chưa khởi tạo chứa rác.
* Nhớ rằng tất cả giá trị trong C có thể được cast thành boolean, và các giá trị `false` duy nhất là `0`, `NULL`, và `false`.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/euf_2BqbdIw?si=a2uoMhvfi4gXmZ2_
:width: 100%
:title: "[CS61C FA20] Lecture 03.1 - C Intro: Basics: Intro and Background"
:::

::::

## Các Kiểu Cơ bản trong C

Giống như trong Java, tất cả biến C đều có kiểu.

:::{table} Các Kiểu Cơ bản trong C; xem [Wikibooks](https://en.wikibooks.org/wiki/C_Programming/Language_Reference#Table_of_data_types).
:label: tab-c-types
:align: center

| Kiểu | Mô tả | Ví dụ |
| :--- | :--- | :--- |
| `int` | Số nguyên (bao gồm âm) | `0`, `78`, `-217`, `0x7337` |
| `unsigned int` | Số nguyên không dấu (tức là, không âm) | `0`, `6`, `35102` |
| `float` | Số thập phân dấu phẩy động | `0.0`, `3.14159`, `6.02e23` |
| `double` | Dấu phẩy động có độ chính xác bằng hoặc cao hơn | `0.0`, `3.14159`, `6.02e23` |
| `char` | Ký tự đơn | `'a'`, `'D'`, `'\n'` |
| `short` | int ngắn hơn | `-7` |
| `long` | int dài hơn | `0`, `78`, `-217`, `301720971` |
| `long long` | int còn dài hơn nữa | `3170519272109251` |
:::

### Tại sao biến có kiểu?

Kiểu của biến được cố định tại compile-time và không thể thay đổi trong suốt thời gian chương trình. Điều này thực sự giúp compiler xác định cách dịch chương trình thành machine code được thiết kế cho kiến trúc của máy tính:

* Biến này chiếm bao nhiêu byte trong bộ nhớ?
* Biến này có thể hỗ trợ những toán tử nào?

:::{card}
`uint16_t y = 38;`
^^^
* `y` lưu trữ số nguyên không dấu 16-bit. (Xem [bên dưới](#inttypes) về `uint16_t`)
* `y` được khởi tạo thành biểu diễn không dấu 16-bit của 38, tức là, 16 bits `0000 0000 0010 0110`.
:::

### `sizeof`

:::{warning} `sizeof` là toán tử compile-time

**Kích thước của biến được biết tại compile time**. Chúng ta sẽ thấy tại sao khi chúng ta thảo luận về quản lý bộ nhớ C trong vài bài giảng nữa.

`sizeof(arg)` **không phải là lời gọi hàm**! Thay vào đó, compiler C giải quyết mọi giá trị `sizeof(arg)` thành kích thước của kiểu dữ liệu hoặc biến `arg`, tính bằng **byte** và tiếp tục biên dịch chương trình kết quả.

:::

Trong khi kiểu của biến không thể thay đổi, bạn có thể typecast giá trị và định nghĩa các kiểu biến mới. Điều này được thảo luận trong [danh sách các chủ đề](sec-laundry-list=).

### Kích thước của các Kiểu Số nguyên


:::{tip} Kiểm tra nhanh

Điều nào sau đây đúng về kiểu dữ liệu `int` trong C?
Chọn tất cả áp dụng.

* A. Số nguyên Two's complement
* B. Chứa tất cả số nguyên trong phạm vi $[−32767, +32767]$
* C. `sizeof(int) = 2`
* D. `sizeof(int) = 16`
* E. `sizeof(int) = 4`
* F. `sizeof(int) = 64`
* G. Không có đáp án nào ở trên

Lưu ý: $2^{15} = 32768$.

:::

:::{note} Hiển thị đáp án
:class: dropdown

Chỉ (A) luôn đúng trên các bộ xử lý. Chuẩn C không định nghĩa kích thước của `int`; nó chỉ đảm bảo rằng nó có độ rộng ít nhất 8 bit.
:::

Chuẩn C không định nghĩa kích thước tuyệt đối của tất cả các kiểu số nguyên! Chuẩn chỉ định nghĩa rằng **`char` có độ rộng 1 byte**. Tất cả các kiểu khác có đảm bảo kích thước _tương đối_:

$$
\texttt{sizeof(long long)} \geq \texttt{sizeof(long)}  \geq \texttt{sizeof(int)} \geq \texttt{sizeof(short)}
$$

Nhớ rằng, C được xây dựng để hiệu quả. Từ sớm, họ xác định rằng kích thước của `int` là kích thước hiệu quả nhất để đọc, ghi, và thao tác trên các số two's complement. Vì vậy máy 32-bit thường sẽ có số nguyên 4-byte (4 byte = 32 bit) nếu datapath được xây dựng cho các giá trị 32-bit, và máy 64-bit sẽ có số nguyên 8-byte (8 byte = 64 bit), nhưng không phải lúc nào cũng vậy.

:::{table} Các kiểu số nguyên trong C, Java và Python
:label: tab-int-types
:align: center

| Ngôn ngữ | kích thước của số nguyên (tính bằng bit) |
|:--- | :--- |
| Python | $\geq$ 32 bits (plain ints), vô hạn (long ints) |
| Java | 32 bits |
| C | Phụ thuộc vào máy tính; 16 hoặc 32 hoặc 64 |
:::

Để viết một chương trình C, khi đó, người ta _thực sự_ cần biết các chi tiết phức tạp của phần cứng. Nhưng điều này làm mất lợi ích của tính di động; code giả định một kiểu dữ liệu rộng $N$-bit (giả sử, vì chúng ta muốn biểu diễn $2^N$ thứ không phải số nguyên) có thể sử dụng `int`, sau đó cần thay đổi kiểu để hoạt động trên máy khác.

(inttypes)=
:::{hint} Sử dụng `inttypes.h` hoặc `stdint.h`

Chúng tôi khuyến khích bạn sử dụng `inttypes.h` hoặc `stdint.h`, một phần của thư viện chuẩn C[^inttypes-vs-stdint]. Nó chỉ định các kiểu không dấu và có dấu[^typedef-int] như `uint8_t` và `int32_t`, trong đó độ rộng được chỉ định bằng bit. Vì vậy `int32_t x;` sẽ khai báo `x` là số nguyên có dấu rộng 32-bit sử dụng biểu diễn two's complement.

:::

[^typedef-int]: Chính xác hơn, `inttypes.h` khai báo nhiều tên `typedef` có dạng `intN_t` và `uintN_t` chỉ định các kiểu số nguyên two's complement và không dấu, tương ứng, với độ rộng bit cụ thể `N`.

[^inttypes-vs-stdint]: Xem [StackOverflow](https://stackoverflow.com/questions/7597025/difference-between-stdint-h-and-inttypes-h) để biết sự khác biệt giữa `inttypes.h` và `stdint.h`. Với mục đích của lớp này, cái nào cũng được.

## Khai báo và khởi tạo biến

**Lưu ý cảnh báo**: Nhiều thứ trong C có "hành vi không xác định." Hoàn toàn có thể cho một chương trình C chạy một cách trên một máy tính và cách khác trên máy khác. Thậm chí có thể chạy khác nhau mỗi lần chương trình được thực thi trên cùng một máy![^heisenbug]

[^heisenbug]: "Heisenbugs" là các bug có vẻ ngẫu nhiên/khó tái tạo, và có vẻ biến mất hoặc thay đổi khi debug. So sánh, "Bohrbugs" là có thể lặp lại và tái tạo được.

Xem xét code sau. In ra gì?

```{code} c
:linenos:
#include <stdio.h>
int main(int argc, char *argv[]) {
    int32_t x = 0;
    int32_t y;
    
    printf("before: x=%d, y=%d\n", x, y);
    x++;
    y += x;

    printf(" after: x=%d, y=%d\n", x, y);
    return 0;
}
```

Tôi thử chạy điều này trên các máy CS 61C và nhận được đầu ra sau lần đầu tiên, và đầu ra khác sau khi tôi chèn một số chú thích:

```
before: x=0, y=22621
 after: x=1, y=22622
```

:::{warning} Không giống Java, khai báo biến C **không** khởi tạo biến thành giá trị mặc định.

Hãy nhìn vào dòng 4. Dòng 4 chỉ **khai báo** `y` có kiểu `int`; nó không **khởi tạo** `y` thành bất kỳ giá trị mặc định nào. 

Tại sao Dòng 8 không gây ra lỗi? Nhớ lại rằng bất kỳ giá trị 32-bit nào cũng có thể được hiểu là số nguyên two's complement. Tại compilation time, compiler nhận ra rằng `y += x;` là một phép toán hợp lệ dựa trên các kiểu của `x` và `y`. Tại runtime, chương trình chỉ đơn giản lấy bất kỳ 32 bit nào ở `y` và cộng giá trị của `x`, sau đó cập nhật những 32 bit đó.
:::


Sau khi chỉnh sửa chú thích và chạy lại, tôi nhận được đầu ra khác lần thứ hai. Chuyện gì đang xảy ra?

## Kiểu `bool` trong C

Kiểu `bool` là kiểu tích hợp kể từ C23. Với các phiên bản cũ hơn (ví dụ, C17), chúng ta cần `#include <stdbool.h>` để có định nghĩa của `true` và `false`.

Các giá trị trong C là truthy—nghĩa là, mọi giá trị có thể được hiểu là true hoặc false.

* Các giá trị False:
  * `0`, tức là, tất cả bit của giá trị này là 0
  * `NULL`, cũng được định nghĩa là `0`, nhưng thường được sử dụng cho con trỏ. Thêm sau.
  * `false`, nếu `stdbool.h` được sử dụng
* Các giá trị True: Mọi thứ khác.[^truthy-python]
  
[^truthy-python]: Python cũng có các giá trị truthy và falsy. Xem [Stack Overflow](https://stackoverflow.com/questions/39983695/what-is-truthy-and-falsy-how-is-it-different-from-true-and-false).


:::{tip} Kiểm tra nhanh

Code sau làm gì?

```c
if (42) {
  printf("meaning of life\n");
}
```

:::

:::{note} Hiển thị đáp án
:class: dropdown

Nó in `meaning of life` (cộng một dòng mới) vì bất kể độ rộng số nguyên được sử dụng, hằng số `42` sẽ có biểu diễn bit khác không. Tất cả biểu diễn bit khác không là `true`.
:::
