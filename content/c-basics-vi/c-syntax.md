---
title: "Cú pháp C"
subtitle: "Biến có Kiểu và Danh sách các Chủ đề"
---

(sec-laundry-list)=
## Mục tiêu học tập

Coi ghi chú này như tài liệu tham khảo cho đến khi bạn cần. 

* Điều hướng danh sách dài các chủ đề C ở đây, và sử dụng tài liệu tham khảo K&R để bổ sung chi tiết.
* Đặc biệt chú ý đến `typedef` và `struct`, được sử dụng để khai báo và tổ chức các kiểu dữ liệu phức tạp hơn.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/euf_2BqbdIw?si=a2uoMhvfi4gXmZ2_
:width: 100%
:title: "[CS61C FA20] Lecture 03.1 - C Intro: Basics: Intro and Background"
:::

::::

:::{warning}
Coi phần còn lại của ghi chú này như tài liệu tham khảo cho đến khi bạn cần. Các từ khóa `typedef` và `struct` sẽ đặc biệt hữu ích để biết sớm.
:::

### `typedef`

`typedef` cho phép bạn tạo một tên bổ sung (bí danh) cho một kiểu dữ liệu khác.

```c
typedef uint8_t BYTE;
BYTE b1, b2;
```

Code trên định nghĩa `BYTE` là tên khác cho `uint8_t`, cho phép chúng ta khai báo `b1` và `b2` đều có kiểu `BYTE`. Lưu ý phụ: Chúng tôi không khuyến nghị khai báo nhiều biến trong cùng một giá trị như trên; nó dẫn đến các kiểu khó hiểu khi chúng ta giới thiệu con trỏ lần sau.

### Structs

`struct` là các nhóm biến có cấu trúc. Một `struct` là định nghĩa kiểu dữ liệu trừu tượng. Nó giống rất nhiều với Python nơi bạn có class và các trường dot, nhưng bạn có nhiều quyền kiểm soát hơn.

Struct và `typedef` thường được sử dụng song song[^typedef-struct]. Ví dụ dài hơn:

[^typedef-struct]: Đọc thêm trên [StackOverflow](https://stackoverflow.com/questions/1675351/typedef-struct-vs-struct-definitions).

```{code} c
:linenos:
typedef struct {
    uint16_t length_in_seconds;
    uint16_t year_recorded;
} SONG;

SONG song1;
song1.length_in_seconds  =  213;
song1.year_recorded      = 1994;

SONG song2;
song2.length_in_seconds  =  248;
song2.year_recorded      = 1988;
```

:::{note} Code, giải thích
:class: dropdown

* Dòng 1 - 4: `SONG` là bí danh cho `typedef struct {uint16_t length_in_seconds; uint16_t year_recorded; }`.
* Dòng 6: Khai báo `song1` là struct có hai biến `uint16_t`, `length_in_seconds` và `year_recorded`.
* Dòng 7-8: Khởi tạo dữ liệu trong biến `song1`.
* Dòng 10-12: Làm điều tương tự cho `song2`.
:::

Quan trọng:
* Struct **không phải** là object.
* Toán tử chấm (`.`) do đó không phải là lời gọi method; nó chỉ truy cập dữ liệu tại một vị trí cụ thể. Thêm sau.

(sec-preprocessor)=
### Macro C Preprocessor, `#define`

`#define PI (3.14159)` là macro CPP (C Preprocessor). Trước khi biên dịch, tiền xử lý bằng cách thực hiện thay thế chuỗi trong chương trình dựa trên tất cả `#define macros`. Dòng trên thay thế tất cả `PI` bằng `(3.14159)` và hiệu quả làm cho `PI` là một "hằng số."

Bạn thường thấy các macro CPP được định nghĩa để tạo các "hàm" nhỏ. Nhưng nhớ rằng vì `#define` về cơ bản là thay thế chuỗi, đây không phải là các hàm thực sự—thay vào đó, bạn đơn giản đang thay đổi văn bản của chương trình.

Vì `#define` về cơ bản là thay thế chuỗi, điều này có thể tạo ra các lỗi thú vị. Ví dụ:

```c
#define min(X,Y) ((X)<(Y)?(X):(Y))
next = min(w, foo(z));
```

được dịch thành code này, trước khi biên dịch:

```c
next = ((w)<(foo(z))?(w):(foo(z)));
```

Nếu `foo(z)` có side effect, side effect đó sẽ xảy ra hai lần!

:::{note} Thêm về CPP

Các file nguồn C trước tiên đi qua macro preprocessor (C Preprocessor hoặc CPP) trước khi compiler thấy code. Ví dụ, CPP thay thế các chú thích bằng một khoảng trắng đơn.

Tất cả các lệnh CPP bắt đầu bằng `#`:
* `#include "file.h"`: Chèn `file.h` vào output
* `#include <stdio.h>`: Tìm `stdio.h` tại vị trí chuẩn, nhưng ngoài ra tương đương với mục trước
* `#define PI (3.14159)`: Định nghĩa hằng số
* `#if/#endif`: Bao gồm văn bản có điều kiện. Hữu ích nếu chương trình C này sẽ được biên dịch trên các máy khác nhau và do đó yêu cầu các thư viện phụ thuộc kiến trúc

Để xem kết quả của tiền xử lý, bạn có thể sử dụng tùy chọn `-save-temps` trong `gcc`. Đọc tài liệu GCC để biết thêm về [CPP](http://gcc.gnu.org/onlinedocs/cpp/) và [macros](https://gcc.gnu.org/onlinedocs/cpp/Macros.html).

:::

### Hằng số và Enums

Từ khóa `const` khai báo một **hằng số**; biến được gán một giá trị có kiểu một lần trong khai báo. Bạn có thể có phiên bản hằng số của bất kỳ kiểu biến C chuẩn nào, nhưng giá trị không thể thay đổi trong toàn bộ quá trình thực thi chương trình.

```c
const float  golden_ratio = 1.618;
const int    days_in_week = 7;
const double the_law      = 2.99792458e8;
```

Một **enum** là tính năng hay cho các hằng số liệt kê. Nó khai báo một nhóm các ràng buộc số nguyên liên quan, như red=0, green=1, blue=2:

```c
enum cardsuit {CLUBS, DIAMONDS, HEARTS, SPADES};
enum color {RED, GREEN, BLUE};
```

Khuyến nghị mạnh mẽ của tôi: đừng nhìn bên dưới để tìm ra những bit đó là gì. Thay vào đó, sử dụng trừu tượng hóa; code của bạn nên hoạt động ngay cả khi bạn sắp xếp lại tập hợp có thứ tự.

### Luồng Điều khiển

Rất giống Java. Không có gì nhiều để nói ở đây ngoài hai mục:

**Dấu ngoặc nhọn**: Thân của các điều kiện if-else và vòng lặp có thể được bao quanh bởi dấu ngoặc nhọn hoặc đứng một mình. Xem [phần trước](#c-vs-java-sec).

**Vòng lặp While**: Ngoài vòng lặp `while` chuẩn, C cũng có vòng lặp `do-while`:

```c
do statement while (expression);
```

**Switch**: Cho đến khi bạn đến câu lệnh `break`, bạn sẽ tiếp tục thực thi các câu lệnh, ngay cả những câu trong các `case` tiếp theo.

```c
// khối này chạy qua tất cả các câu lệnh
switch (expression){
  case const1:    statements
  case const2:    statements
  default:        statements
}

```

Khi viết code C, chúng tôi không khuyến nghị vòng lặp `do-while`, cũng như `goto` đáng sợ. Nhưng chúng ta sẽ thấy cả hai ý tưởng hữu ích khi chúng ta thảo luận về luồng điều khiển trong ngôn ngữ assembly. Thêm (nhiều) sau.


### Hàm

Hai hàm ví dụ:

```c
int number_of_people(int class1, int class2) {
  return class1 + class2;
}

float dollars_and_cents(float cost) { return cost; }
```

* Bạn phải khai báo kiểu dữ liệu bạn dự định trả về từ một hàm.
* Kiểu `return` có thể là bất kỳ kiểu biến C nào (hoặc `void`), và được đặt bên trái tên hàm. Kiểu trả về `void` ngụ ý rằng không có giá trị nào sẽ được trả về; chúng ta quay lại điều này sau trong khóa học.
* Các tham số cũng phải có kiểu.

Biến và hàm phải được khai báo trước khi sử dụng. Trong các phiên bản C cũ hơn, điều này có nghĩa là tất cả các khai báo hàm cần phải ở đầu file, hoặc được bao gồm trong headers. Các implementation hàm có thể được mô tả sau trong file. Trong C gần đây hơn, hàm có thể được sử dụng miễn là chúng được khai báo trong file.

## File header

File header cho phép bạn chia sẻ hàm và macro giữa các file nguồn khác nhau. Để biết thêm thông tin, xem [tài liệu header GCC](https://gcc.gnu.org/onlinedocs/cpp/Header-Files.html#Header-Files).
