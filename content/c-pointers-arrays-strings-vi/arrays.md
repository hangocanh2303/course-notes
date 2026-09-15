---
title: "Mảng"
---

(sec-array)=
## Mục tiêu học tập

* Khai báo và khởi tạo mảng C.
* Hiểu rằng mảng C nên được xem như các khối bộ nhớ liền kề, không phải con trỏ. Tên mảng đồng nghĩa với vị trí của phần tử đầu tiên trong mảng.
* Chuyển đổi chỉ mục mảng thành số học con trỏ theo sau là thao tác giải tham chiếu.
* Suy biến (decay) mảng thành con trỏ khi được sử dụng như tham số hình thức cho định nghĩa hàm hoặc đối số cho hàm.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/hJNoW4hlZDg
:width: 100%
:enumerated: false
:title: "Lecture 04.3 - C Intro: Pointers, Arrays, Strings: Arrays"
:::
::::


::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/J6mhHw7UTPM
:width: 100%
:enumerated: false
:title: "Lecture 05.1 - C Memory Management: Dynamic Memory Allocation"
:::
Từ 9:36 trở đi: Ví dụ mảng không phải con trỏ
::::

Chúng ta tiếp tục khám phá bộ nhớ bằng cách nghiên cứu mảng C. Bề ngoài, mảng C có vẻ khá giống với những gì bạn có thể nhận ra từ Java. Trong phần này, chúng ta học rằng mảng trong C **không phải là biến cũng không phải là con trỏ**. Khi được sử dụng trong các câu lệnh C, tên mảng thường hoạt động như tên biến con trỏ, vì những lý do chúng ta sẽ mô tả ngay sau đây.

:::{hint} Một câu ngạn ngữ từ K&R
Tên mảng không phải là một biến.
:::

:::{hint} Một câu ngạn ngữ từ chúng tôi
Mảng C thực sự chỉ là một khối lớn các thứ liên tiếp trong bộ nhớ với các thuộc tính nhất định.
:::


Để **khai báo** một mảng hai phần tử mà không khởi tạo giá trị của nó, chúng ta có thể sử dụng câu lệnh bên dưới. Câu lệnh này khai báo một khối bộ nhớ đủ lớn để chứa hai `int` liền kề. Nó không khởi tạo giá trị, nên chúng ta có thể giả sử các phần tử chứa rác:

```c
int arr_uninitialized[2];
```

Để **khởi tạo và khai báo** một mảng hai phần tử 795 và 635, theo thứ tự đó:

```c
int arr2[] = {795, 635};
```

hoặc tương đương

```c
int arr2[2] = {795, 635};
```

**Chỉ mục ngoặc vuông** là một cách để truy cập các phần tử của mảng. Giống như nhiều ngôn ngữ, C quy định mảng đánh chỉ mục từ không:

```c
arr2[0]; // 795
```

(sec-array-indexing)=
## Chỉ mục mảng sử dụng số học con trỏ

Có cách khác để truy cập các phần tử mảng không? Có, nếu không chúng tôi đã không bí ẩn như vậy trước đó.

Chỉ mục ngoặc vuông cho mảng C là cái chúng tôi gọi là "cú pháp đường" - nghĩa là, nó tồn tại để con người dễ đọc, nhưng trình biên dịch C sẽ dịch nó thành hai thao tác: [số học con trỏ](#sec-pointer-arithmetic) theo sau là **giải tham chiếu**:

Biểu thức `arr[i]` **tương đương** với biểu thức `*(arr+i)`. Biểu thức sau xử lý tên mảng `arr` như một con trỏ, tăng nó, rồi giải tham chiếu.

### Ví dụ

Giả sử rằng khi biên dịch, @code-array-indexing bên dưới tạo ra bố cục bộ nhớ trong @fig-array-indexing. `q` là một con trỏ đến một số nguyên không dấu 32-bit, trong khi `arr` là một mảng, tức là một khối 24-byte liền kề của các số nguyên không dấu 32-bit.

(code-array-indexing)=
```{code} c
:linenos:
#include <stdio.h>
#include <stdint.h>

int main () {
  uint32_t arr[] = {50, 60, 70}; // mảng không dấu 32-bit
  uint32_t *q = arr;

  printf("    *q: %d is %d\n", *q, q[0]);
  printf("*(q+1): %d is %d\n", *(q+1), q[1]);
  printf("*(q-1): %d is %d\n", *(q-1), q[-1]);
}
```

:::{figure} images/array-indexing.png
:label: fig-array-indexing
:width: 80%
:alt: "Bố cục bộ nhớ cho con trỏ q và mảng arr, trong đó arr chứa 50, 60, và 70 trong các word liên tiếp tại 0x100, 0x104, và 0x108. Con trỏ q lưu 0x100, nên q[0] và *q đọc 50, q[1] và *(q+1) đọc 60, và q[-1] tham chiếu đến word không xác định phía trước."

Bố cục bộ nhớ cho @code-array-indexing.
:::

Vì chỉ mục ngoặc vuông là cú pháp đường cho số học con trỏ và giải tham chiếu:

* Dòng 7: Con trỏ `q` trỏ đến một số nguyên không dấu 32-bit tại địa chỉ `0x100`, là `50`. In `    *q: 50 is 50`.
* Dòng 8: Tăng `q` trỏ đến số nguyên không dấu 32-bit **tiếp theo**. Nếu `q` trỏ đến số nguyên không dấu 32-bit tại địa chỉ `0x100`, thì tăng `q` trỏ đến *số nguyên không dấu 32-bit tiếp theo* tại địa chỉ `0x104`, là `60`. In `*(q+1): 60 is 60`.
* Dòng 9: Vì chỉ mục ngoặc vuông là cú pháp đường, *chỉ mục âm không tạo ra bất kỳ lỗi nào*. Thay vào đó, giảm `q` trỏ đến số nguyên không dấu 32-bit **trước đó** tại địa chỉ `0xFC`, là một giá trị không xác định. Dòng này có thể in rác, ví dụ: `*(q-1): 32490 is 32490`.

## Mảng không phải là con trỏ

Từ K&R:

> Có một sự khác biệt giữa tên mảng [(như `a`)] và một con trỏ [(như `pa`)] cần phải nhớ. Một con trỏ là một biến, nên `pa=a` và `pa++` là hợp lệ. Nhưng tên mảng không phải là một biến; các cấu trúc như `a=pa` và `a++` là không hợp lệ.

Cũng từ K&R:

> Tên của một mảng là đồng nghĩa với vị trí của phần tử ban đầu.

Do đó, con trỏ và mảng khác nhau về cách chúng hoạt động với toán tử địa chỉ, `&`. Xem xét @code-array-addressing:[^fstring]

[^fstring]: `%d`: số thập phân có dấu, `%x`: hex. [Wikipedia](https://en.wikipedia.org/wiki/Printf)

(code-array-addressing)=
```{code}c
:linenos:

int *p, *q, x;
int a[4];
p = &x;
q = a + 1;

*p = 1;
printf("*p:%d, p:%x, &p:%x\n", *p, p, &p);

*q = 2;
printf("*q:%d, q:%x, &q:%x\n", *q, q, &q);

*a = 3;
printf("*a:%d, a:%x, &a:%x\n", *a, a, &a);

```

Với bố cục bộ nhớ trong @fig-array-addressing, đầu ra là:

```
*p:1, p:108, &p:100
*q:2, q:110, &q:104
*a:3, a:10c, &a:10c
```

:::{figure} images/array-addressing.png
:label: fig-array-addressing
:width: 80%
:alt: "Bố cục bộ nhớ cho con trỏ p và q, vô hướng x, và mảng a, cho thấy các địa chỉ được sử dụng trong các ví dụ printf. Nó minh họa rằng p và q là biến con trỏ với địa chỉ riêng của chúng, trong khi a đặt tên cho một khối liền kề có phần tử đầu tiên được đặt là 3 tại 0x10c và phần tử thứ hai được đặt là 2 tại 0x110."

Bố cục bộ nhớ cho @code-array-addressing.
:::

Địa chỉ của mảng `a` là địa chỉ của chính mảng, tức là địa chỉ của khối bộ nhớ liền kề lớn của các `int`!

:::{note} Hiển thị giải thích
:class: dropdown

* Chúng ta thảo luận về khai báo nhiều biến trong một [phần trước](#foot-multiple-declarations).

[^multiple-declarations]: Bạn có thể nhận thấy rằng Dòng 8 khai báo hai con trỏ bằng cách gắn `*` cạnh `ptr1` và `ptr2`, tương ứng. Chúng ta chưa thảo luận, nhưng một khai báo đơn `coord_t* ptr1;` cũng hợp lệ. Hầu hết các lập trình viên C hiện đại cố gắng tránh khai báo nhiều biến trên một dòng nếu có thể. Nhưng bạn sẽ thấy nó thường xuyên trong các ứng dụng C cũ. Đọc thêm trên [Reddit](https://www.reddit.com/r/cpp/comments/vm8bwm/how_do_you_declare_pointer_variables/).

* Dòng 3: Con trỏ `int`, `p`, được khởi tạo thành địa chỉ của biến `int` `x`.
  * Dòng 6: Lấy giá trị `p` trỏ đến; đặt nó bằng 1.
  * `*p` giải tham chiếu `p` và lấy giá trị tại địa chỉ `0x108`, là `1`.
  * `p` là một biến con trỏ; giá trị của `p` là một địa chỉ, là `0x108`.
  * `&p` là địa chỉ của biến `p`, là `0x100`.
* Dòng 4: Con trỏ `int` `q` được khởi tạo thành kết quả của `a + 1`, là **số học con trỏ**! Trong biểu thức, tên mảng `a` là địa chỉ của phần tử đầu tiên trong `a`; tăng thêm một cho địa chỉ của phần tử _thứ hai_ của `a`, tại `0x110`.
  * Dòng 9: Lấy giá trị `q` trỏ đến; đặt nó bằng 2.
  * `*q` giải tham chiếu `q` và lấy giá trị tại địa chỉ `0x110`, là `2`.
  * `q` là một biến con trỏ; giá trị của `q` là một địa chỉ, là `0x110`.
  * `&q` là địa chỉ của biến `q`, là `0x104`.
* Dòng 2: Mảng `a` là một khối bộ nhớ của 4 `int`. Mảng bắt đầu tại địa chỉ `0x10c`, cũng là địa chỉ của phần tử đầu tiên của nó.
  * Dòng 12: Tên mảng `a` là địa chỉ của phần tử đầu tiên trong `a`; câu lệnh `*a = 3;` lấy giá trị này và đặt nó bằng 3.
  * `*a` là **số học con trỏ theo sau là giải tham chiếu**. Tên mảng `a` là địa chỉ của phần tử đầu tiên trong `a`; giải tham chiếu lấy chính phần tử, là `3`.
  * `a` là địa chỉ của phần tử đầu tiên trong `a` theo định nghĩa, là `0x10c`.
  * `&a` **lấy địa chỉ của mảng `a`**, là `0x10c`.[^address-of-sizeof]

  [^address-of-sizeof]: Chúng tôi đã suy nghĩ rất nhiều về cách giải thích `&a` và `sizeof(a)` (nó liên quan đến việc ngồi trong phòng tối với nhạc lớn). Cả hai thao tác có thể đều do thiết kế C hợp lý. Sau cùng, phải có _cách nào đó_ để tham chiếu đến địa chỉ và kích thước của một mảng. Thay vì báo lỗi, hai biểu thức này có lẽ là ngoại lệ duy nhất đối với việc xử lý tên mảng như đồng nghĩa với địa chỉ của phần tử đầu tiên. Nếu bạn, người đọc, có giải thích tốt hơn, chúng tôi rất muốn sử dụng nó. Gửi pull request!
:::

(sec-array-decay)=
## Tên mảng "suy biến" với hàm

Khi sử dụng với hàm, mảng **suy biến (decay)** thành con trỏ theo hai cách. Chúng ta sử dụng @code-decay bên dưới làm ví dụ.

(code-decay)=
```{code}c
:linenos:
int bar(int arr[], size_t nelems){
   … arr[…] … 
}
int main(void) {
    int a[5], b[10];
    … 
    bar(a, 5);
    …
}
```

**1. Khi được sử dụng như tham số hình thức cho định nghĩa hàm.** Ở Dòng 2 của @code-decay, định nghĩa `int arr[]` là cú pháp đường cho định nghĩa `int *arr`. Chúng tôi khuyến nghị sử dụng cách sau nếu có thể để tránh nhầm lẫn.

**2. Khi được truyền vào như đối số cho các lời gọi hàm**. Ở Dòng 7 của @code-decay, đối số `a` là một mảng nhưng suy biến thành con trỏ khi hàm được gọi. Sự suy biến này hiệu quả truyền vào địa chỉ của `a` như đối số đầu tiên của `bar`.

:::{warning} Luôn truyền vào độ dài mảng

Các hàm được gọi sẽ không bao giờ biết giới hạn của các mảng được truyền vào như đối số. Thực tế, chúng thậm chí sẽ không biết rằng các tham số con trỏ của chúng là mảng đã suy biến.
Một con trỏ tự nó không đủ để suy ra độ dài của mảng, vì vậy nếu bạn cần theo dõi độ dài của một mảng, bạn phải sử dụng một biến hoặc tham số khác (xem Dòng 1 của @code-decay).
:::

(sec-array-sizeof)=
## `sizeof` với mảng

Chúng ta đã thảo luận về `sizeof` nhiều lần. Đối với mảng, toán tử thời gian biên dịch sẽ đánh giá thành kích thước của mảng, tính bằng byte.[^address-of-sizeof] Quan sát này thông tin về hành vi của @code-array-sizeof:

(code-array-sizeof)=
```{code} c
:linenos:
void mystery(short arr[], int len) {
    printf("%d ", len);
    printf("%d\n", sizeof(arr));
}

int main() {
    short nums[] = {1, 2, 3, 99, 100};
    printf("%d ", sizeof(nums));
    mystery(nums, sizeof(nums)/sizeof(short));
    return 0;
}
```

:::{tip} Kiểm tra "nhanh"
Giả sử kiến trúc 64-bit trong đó `short` là 16 bit. Khi @code-array-sizeof được chạy, cái gì được in?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Kết quả in: `10 5 8`

* Ở Dòng 10, `sizeof(nums)` nằm trong phạm vi khai báo của mảng. Đánh giá thành tổng kích thước mảng của năm `short`, tức là 10.
* Ở Dòng 4, giá trị `len` là kết quả của việc đánh giá `sizeof(nums)/sizeof(short)` trong `main`, tức là 10/2 = 5.
* Ở Dòng 8, `arr` là một tham số hàm. Khai báo hình thức `short arr[]` là cú pháp đường cho `short *arr`, nên `arr` là một *con trỏ*. Kích thước của một con trỏ là 64 bit, nên `sizeof(arr)` là 8.

:::

Trong thực tế, các lập trình viên C thường sử dụng `sizeof(nums)/sizeof(short)` để đếm **số phần tử** trong mảng `nums`. Lưu ý rằng `nums` phải được khai báo trong cùng phạm vi, nếu không nó suy biến thành con trỏ.

## Mảng là nguyên thủy! Nhắc nhở

Hy vọng phần này đã thuyết phục bạn rằng mảng là các cấu trúc tương đối nguyên thủy:

* Khai báo mảng dành riêng các khối liền kề trong bộ nhớ.
* Tên mảng đồng nghĩa với vị trí của phần tử đầu tiên trong mảng.
* Mảng suy biến thành con trỏ khi được sử dụng như tham số hàm hoặc đối số hàm.

Chúng ta kết thúc với một vài nhắc nhở cuối cùng về cách bản chất nguyên thủy này đòi hỏi các thực hành C có trách nhiệm.

:::{warning} Nhắc nhở 1
Giữ kích thước mảng trong các hằng số nếu có thể.

Thay vì mã quản lý nhiều bản sao của các hằng số nguyên,

```c
int i, arr[10];
for(i = 0; i < 10; i++) { ... }
```

chọn khai báo "nguồn sự thật duy nhất":

```c
const int ARRAY_SIZE = 10;
int i, a[ARRAY_SIZE];
for(i = 0; i < ARRAY_SIZE; i++) { ... }
```
:::

:::{warning} Nhắc nhở 2
Giới hạn mảng không được kiểm tra trong quá trình truy cập phần tử.

Truy cập phần tử chỉ là số học con trỏ với giải tham chiếu, nên rất dễ vô tình truy cập ra ngoài cuối mảng. Bạn có thể tìm thấy lỗi tinh vi trong đoạn mã này không?

```c
const int N = 100;
int foo[N];
int i;
...
for(i = 0; i <= N; ++i) {
   foo[i] = 0;
}
```

Truy cập không đúng ra ngoài cuối mảng được gọi là **tràn bộ đệm (buffer overflow)**[^buffer-overflow]. Lỗi rất phổ biến này có thể làm hỏng các phần khác của chương trình, bao gồm dữ liệu nội bộ C. Khai thác tràn bộ đệm là các lỗ hổng bảo mật có thể làm crash chương trình.

[^buffer-overflow]: Học Computer Security để tìm hiểu thêm! [Wikipedia](https://en.wikipedia.org/wiki/Buffer_overflow)
:::

:::{warning} Nhắc nhở 3
Chọn định nghĩa tham số hình thức một cách khôn ngoan.

`char *str` và `char str[]` tương đương _khi được sử dụng như tham số hình thức trong định nghĩa hàm._ Bạn sẽ thấy cả hai trong thực tế. K&R đề nghị sử dụng cách trước nếu có thể vì nó nói rõ ràng hơn rằng biến là một con trỏ.
:::

:::{warning} Nhắc nhở 4

Luôn truyền vào độ dài mảng. Xem @sec-array-decay.
:::
