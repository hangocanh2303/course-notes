---
title: "Giới thiệu"
---

:::{warning} ⚠️⚠️⚠️ Bạn đang tìm Tiền tố IEC?
Các tiền tố IEC như MiB, GiB không phải là nội dung C về mặt kỹ thuật nhưng đã được đề cập trong bài giảng này. Chúng được liên kết ở thanh bên phía dưới: [Tiền tố IEC và Cơ số 10](#sec-iec-prefixes).
:::

(sec-generics)=
## Mục tiêu học tập

* Nhận diện các hàm generic được sử dụng trong quản lý bộ nhớ heap.
* Hiểu tại sao generics hỗ trợ code đa dụng trong C.

Trong chương này, chúng ta thực hành kỹ năng bộ nhớ C với thêm hai khái niệm liên quan đến con trỏ.

Như đã đề cập trong [chương trước](#sec-pointers), thông thường con trỏ chỉ có thể trỏ đến một kiểu. Khai báo `int *p;` cho chúng ta biết `p` nên trỏ đến một giá trị `int`, và nó cũng xác định cách `p` hoạt động với các toán tử như số học con trỏ (bước nhảy bao nhiêu byte) và giải tham chiếu (đọc bao nhiêu byte).

Trong phần này, chúng ta thảo luận về con trỏ `void *`, một **con trỏ generic** có thể trỏ đến bất cứ thứ gì. Generics hỗ trợ code đa dụng bằng cách giảm **sự trùng lặp code**. Generics được sử dụng khắp nơi trong C để sắp xếp mảng bất kỳ kiểu nào, tìm kiếm mảng bất kỳ kiểu nào, giải phóng bộ nhớ chứa dữ liệu bất kỳ kiểu nào, và nhiều hơn nữa.

Bằng cách định nghĩa một hàm cho mỗi trường hợp sử dụng này, chúng ta có thể gọi hàm đó trên nhiều kiểu biến khác nhau. Điều này giúp chúng ta cải tiến ở một nơi duy nhất, thay vì nhiều nơi rất giống nhau. Khi làm vậy, chúng ta phải hiểu sâu hơn về bộ nhớ để tránh lỗi khi viết generics của riêng mình.

Chúng ta có hai mục tiêu cho generics:

1. Generics nên hoạt động bất kể kiểu đối số.
2. Generics nên hoạt động bằng cách truy cập các khối bộ nhớ, bất kể kiểu dữ liệu được lưu trong các khối đó.

Mục tiêu đầu tiên là hiển nhiên; mục tiêu thứ hai sẽ trở nên rõ ràng [sau này](#sec-generic-swap).

## Các hàm generic trong thư viện chuẩn C

Mặc dù chúng ta nói sẽ sử dụng generics một cách tiết kiệm để tránh lỗi chương trình, bạn đã gặp các generics đầu tiên rồi! Hãy xem các hàm quản lý bộ nhớ heap trong `stdlib.h`:

* `void *malloc(size_t n)`
* `void free(void *ptr)`
* `void *realloc(void *ptr, size_t size)`

Các hàm này là **hàm generic** (hay gọi tắt là **generics**[^java]) vì chúng không giả định gì về kiểu của bộ nhớ được cấp phát hoặc giải phóng. Như đã mô tả trong [phần trước](#sec-heap), chúng ta ép kiểu giá trị trả về của các lời gọi `malloc` và `realloc` sang kiểu con trỏ phù hợp và sử dụng chúng trong các biến con trỏ cục bộ có kiểu.

[^java]: Java cũng hỗ trợ generics để (trong số những thứ khác) hỗ trợ tạo cấu trúc dữ liệu có thể chứa bất kỳ kiểu tham chiếu nào, ví dụ: `DataStructure<T>`

(sec-swap-motivation)=
## Động lực: `swap_int`, `swap_short`, `swap_string`

### `swap_int`

Giả sử chúng ta viết một hàm `swap_int` để hoán đổi các số nguyên:

(code-swap-int)=
```{code} c
:linenos:
void swap_int(int *ptr1, int *ptr2) {
  int temp = *ptr1;
  *ptr1 = *ptr2;
  *ptr2 = temp;
}
```

Vì C truyền theo giá trị, để hoán đổi các số nguyên được khai báo trong phạm vi hàm khác, chúng ta truyền đối số dưới dạng con trỏ. Ví dụ, chúng ta có thể gọi `swap_int` như sau:

(code-swap-int-main)=
```{code} c
int x = 2;
int y = 5;
swap_int(&x, &y);
```

Bộ slide dưới đây theo dõi một ví dụ đơn giản giả định rằng ban đầu, `x` lưu `2` tại địa chỉ `0x100` và `y` lưu `5` tại `0x104`. Sau lời gọi `swap_int`, `x` và `y` vẫn ở cùng địa chỉ nhưng đã hoán đổi giá trị thành `5` và `2` tương ứng.

:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vSPi9zeMb9_6MHbefYmj3qLUG360ZXXl6jFvy4nCSf5dhSJN7BmIVoT5x2LWBnNAUktlzvhtYoNuZ2G/pubembed?start=false&loop=false
:width: 100%
:title: "Animation that steps through the enumerated text in this section motivating swap_int, swap_short, and swap_string. Access [original Google Slides](https://docs.google.com/presentation/d/1pjgFhJh-Rx3CapS7gbUSkjjqCC3OrL9k7MMT14lau8o/edit?usp=sharing)"
:::

Nhấp vào bên dưới để hiển thị giải thích của animation.

:::{note} Giải thích
:class: dropdown

Hàm `swap_int` tận dụng giải tham chiếu con trỏ và biến cục bộ `temp` của riêng nó để cập nhật các giá trị đúng trong bộ nhớ.

* Lời gọi hàm: _Địa chỉ_ của các biến cục bộ `x` và `y` được truyền cho lời gọi hàm `swap_int` dưới dạng `ptr1` và `ptr2` tương ứng. `ptr1` lưu địa chỉ `0x100`, và `ptr2` lưu địa chỉ `0x104`.
* [Dòng 2](#code-swap-int): Biến cục bộ `temp` tạo một bản sao của giá trị tại `ptr1`, đó là `2` (hiện đang được lưu tại địa chỉ `0x100`).
* [Dòng 3](#code-swap-int): Đặt giá trị tại `ptr1` thành một bản sao của giá trị tại `ptr2`. Vế phải, `*ptr2`, giải tham chiếu `ptr2` và có giá trị `5` (vì đó là các byte `int` tại địa chỉ `0x104`). Vế trái, `*ptr1`, biểu thị vị trí đích—các byte số nguyên tại địa chỉ `0x100`.
* [Dòng 4](#code-swap-int): Đặt giá trị tại `ptr2` thành một bản sao của `temp`. Vế phải có giá trị là một `int`—giá trị `2`. Vế trái, `*ptr2`, biểu thị vị trí đích để lưu các byte này—tại địa chỉ `0x104`.
:::

### `swap_short`

Tiếp theo chúng ta viết một hàm `swap_short` để hoán đổi các short (kiểu số nguyên, không phải quần đùi):

(code-swap-short)=
```{code} c
:linenos:
void swap_short(short *ptr1, short *ptr2) {
  short temp = *ptr1;
  *ptr1 = *ptr2;
  *ptr2 = temp;
}
```

Ngoài các khai báo kiểu của `ptr1`, `ptr2`, và `temp`, logic vẫn tương tự.

### `swap_string`

Logic vẫn tương tự với `swap_string`, hàm "hoán đổi chuỗi".

(code-swap-string)=
```{code} c
:linenos:
void swap_string(char **ptr1, char **ptr2) {
  char *temp = *ptr1;
  *ptr1 = *ptr2;
  *ptr2 = temp;
}
```

Thay vì tạo bản sao của các byte `char`, hàm này hoán đổi địa chỉ của hai biến `char *` (tức là con trỏ đến chuỗi C). Chúng ta có thể gọi `swap_string` với code dưới đây.

(code-swap-string-main)=
```{code} c
char *s1 = "CS";
char *s2 = "61C";
swap_string(&s1, &s2);
```

@fig-swap-string-before minh họa một ví dụ đơn giản về bộ nhớ tại thời điểm bắt đầu lời gọi `swap_string`; @fig-swap-string-after minh họa bộ nhớ ngay trước khi lời gọi trả về.

:::{figure} images/swap-string-before.png
:label: fig-swap-string-before
:width: 60%
:alt: "Initial swap_string state: ptr1 and ptr2 point to variables s1 and s2, where s1 stores address 0x0FACE0 for string CS and s2 stores address 0x0ABBA0 for string 61C, both with a null terminator."

`swap_string` được gọi.
:::

:::{figure} images/swap-string-after.png
:label: fig-swap-string-after
:width: 60%
:alt: "Final swap_string state before return: s1 now stores 0x0ABBA0 and points to string 61C, while s2 stores 0x0FACE0 and points to string CS; the string data in memory is unchanged."

Ngay trước khi lời gọi `swap_string` trả về.
:::

Nhấp vào bên dưới để hiển thị giải thích của @fig-swap-string-before và @fig-swap-string-after.

:::{note} Giải thích
:class: dropdown

@fig-swap-string-before:

* [Lời gọi hàm](#code-swap-string-main): _Địa chỉ_ của các biến `s1` và `s2` được truyền cho lời gọi hàm `swap_string` dưới dạng `ptr1` và `ptr2` tương ứng. Các biến có kiểu `char *`, do đó con trỏ đến các biến này có kiểu `char **`. `ptr1` lưu địa chỉ `0x7F...F0`; `ptr2` lưu địa chỉ `0x7F...F4`.
* `s1` là một con trỏ đến một chuỗi nằm tại `0x0FACE0`. `s1` lưu giá trị `0x0FACE0` tại địa chỉ `0x7F...F0`.
* `s2` là một con trỏ đến một chuỗi nằm tại `0x0ABBA0`. `s2` lưu giá trị `0x0ABBA0` tại địa chỉ `0x7F...F4`.

@fig-swap-string-after:

* [Dòng 2](#code-swap-string): Biến cục bộ `temp` tạo một bản sao của giá trị tại `ptr1`, đó là `0x0FACE0`.
* [Dòng 3](#code-swap-string): Đặt giá trị tại `ptr1` thành một bản sao của giá trị tại `ptr2`. Vế phải, `*ptr2`, giải tham chiếu `ptr2` và có giá trị `0x0ABBA0` (vì đó là các byte tại địa chỉ `0x7F...F4`). Vế trái, `*ptr1`, biểu thị vị trí đích–các byte tại địa chỉ `0x7F...F0`.
* [Dòng 4](#code-swap-string): Đặt giá trị tại `ptr2` thành một bản sao của `temp`. Vế phải có giá trị là `0x0FACE0`. Vế trái, `*ptr2`, biểu thị vị trí đích để lưu các byte này–tại địa chỉ `0x7F...F4`.

:::

### Chúng ta có thể viết một hàm swap generic không?

Ba hàm này chứng minh rằng ở mức cao, việc hoán đổi hai giá trị cùng kiểu dữ liệu tuân theo cùng một logic. Chúng ta muốn viết một hàm có thể thực hiện **hoán đổi generic**. Hãy đọc tiếp!
