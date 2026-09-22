---
title: "Generics"
---

:::{warning} ⚠️⚠️⚠️ Bạn đang tìm Tiền tố IEC?
Các tiền tố IEC như MiB, GiB không phải là nội dung C về mặt kỹ thuật nhưng đã được đề cập trong bài giảng này. Chúng được liên kết ở thanh bên phía dưới: [Tiền tố IEC và Cơ số 10](#sec-iec-prefixes).
:::

(sec-generic-swap)=
## Mục tiêu học tập

* Hiểu tại sao con trỏ generic (`void *`) không thể được giải tham chiếu
* Sử dụng `memcpy` và `memmove` để truy cập bộ nhớ trong các hàm generic

Trong phần này, chúng ta tập trung vào việc triển khai hàm swap **generic**. Hãy nhớ lại các mục tiêu của chúng ta cho hàm generic từ [phần giới thiệu](#sec-generics):

1. Generics nên hoạt động bất kể kiểu đối số.
2. Generics nên hoạt động bằng cách truy cập các khối bộ nhớ, bất kể kiểu dữ liệu được lưu trong các khối đó.

Chúng ta chuyển các ý tưởng cấp cao này thành mã giả cho generic swap:

(code-swap-pseudo)=
```c
void swap(void *ptr1, void *ptr2) {
  // 1. lưu một bản sao của data1 vào bộ nhớ tạm

  // 2. sao chép data2 đến vị trí của data1

  // 3. sao chép dữ liệu trong bộ nhớ tạm đến vị trí của data2
}
```

Các tham số trên xấp xỉ đạt được mục tiêu đầu tiên cho các hàm generic. Bằng cách khai báo `ptr1` và `ptr2` là con trỏ generic với `void *`, chúng ta thực tế giả định rằng có một số dữ liệu tại các vị trí mà `ptr1` và `ptr2` trỏ đến cần được hoán đổi. (Chúng ta sẽ thấy sau này rằng một số điều chỉnh cho chữ ký hàm này là cần thiết.)

Còn thân hàm thì sao? [Hàm `swap_int`](#code-swap-int) của chúng ta từ trước đó sử dụng toán tử giải tham chiếu (`*`) để đọc và ghi dữ liệu đến và từ bộ nhớ được trỏ bởi `ptr1` và `ptr2`. Chúng ta sẽ thấy trong phần này rằng giải tham chiếu con trỏ generic _sẽ không hoạt động_, và chúng ta phải khám phá các thao tác khác để truy cập bộ nhớ.

## Bạn không thể giải tham chiếu `void *`

Việc muốn viết một hàm swap generic theo mẫu của `swap_int`, `swap_short`, và `swap_string` từ [trước đó](#sec-swap-motivation) là tự nhiên. Tuy nhiên, code sau đây sẽ _không_ hoạt động:

(code-swap-faulty)=
```{code} c
:linenos:
/* tạo ra lỗi biên dịch */
void swap_faulty(void *ptr1, void *ptr2) {
  void temp = *ptr1;
  *ptr1 = *ptr2;
  *ptr2 = temp;
}
```

Khi chúng ta cố gắng biên dịch với `gcc`, trình biên dịch _không hài lòng_:

```
$ gcc -c -o swap.o swap.c -Wall -std=c99 -g
swap.c: In function 'swap_faulty':
swap.c:10:8: error: variable or field 'temp' declared void
   10 |   void temp = *ptr1;
      |        ^~~~
swap.c:10:15: warning: dereferencing 'void *' pointer
   10 |   void temp = *ptr1;
      |               ^~~~~
swap.c:10:15: error: void value not ignored as it ought to be
   10 |   void temp = *ptr1;
      |               ^
swap.c:11:3: warning: dereferencing 'void *' pointer
   11 |   *ptr1 = *ptr2;
      |   ^~~~~
swap.c:11:11: warning: dereferencing 'void *' pointer
   11 |   *ptr1 = *ptr2;
      |           ^~~~~
swap.c:11:9: error: invalid use of void expression
   11 |   *ptr1 = *ptr2;
      |         ^
swap.c:12:3: warning: dereferencing 'void *' pointer
   12 |   *ptr2 = temp;
      |   ^~~~~
swap.c:12:9: error: invalid use of void expression
   12 |   *ptr2 = temp;
      |         ^
```

Có hai lý do chính khiến code này không hoạt động. Thứ nhất, chúng ta không thể khai báo các biến không có kiểu, vì vậy một khai báo như `void temp;` báo lỗi. Thứ hai, giải tham chiếu con trỏ `void *` _không cho ra_ [_bất cứ thứ gì có thể sử dụng được_](https://www.gnu.org/software/c-intro-and-ref/manual/html_node/Void-Pointers.html).

Hãy xem xét ý nghĩa của việc giải tham chiếu một con trỏ, giả sử, được khai báo và khởi tạo là `int32_t *p = ...;`. Điều này có nghĩa là `p` là địa chỉ của một giá trị `int32_t`, vì vậy giải tham chiếu với `*p` _truy cập 4 byte bộ nhớ đó_. Điều này cho phép trình biên dịch dịch các câu lệnh sau như `(*p) + 4` thành số học số nguyên, _vì nó biết rằng `*p` là một giá trị có kiểu `int32_t`_.

:::{hint} Bạn chỉ có thể giải tham chiếu con trỏ có kiểu!
Để giải tham chiếu một con trỏ, chúng ta phải biết số byte cần truy cập trong bộ nhớ tại **thời điểm biên dịch**. Con trỏ generic (`void *`) **không thể** sử dụng toán tử giải tham chiếu!
:::

:::{warning} Bạn _có thể_ giải tham chiếu `void **`!

Ngược lại, `void **` **không phải** là một con trỏ generic, vì nó trỏ đến một kiểu đã biết. Thật khó hiểu là code sau đây biên dịch và chạy tốt:

```c
void **doubleptr = …;
printf("%p\n", *doubleptr);
```

Lưu ý rằng kích thước của _bất kỳ_ con trỏ nào đều được biết tại thời điểm biên dịch, vì tất cả con trỏ đều có kích thước bằng một word. `sizeof(*doubleptr)` là `sizeof(void *)` tức là kích thước của một địa chỉ. Trong trường hợp này, giải tham chiếu `doubleptr` trong lời gọi `printf` được đánh giá thành một con trỏ generic, tức là một địa chỉ. _Con trỏ generic có kiểu_—chúng có kiểu `void *`!

:::

## Truy cập bộ nhớ generic với `memcpy`, `memmove`

Generics không thể sử dụng toán tử giải tham chiếu (`*`) vì kiểu của dữ liệu được trỏ bởi con trỏ generic là không xác định. Thay vào đó, generics sử dụng hai hàm từ thư viện chuẩn C: `memcpy` và `memmove`.

```c
void *memcpy(void *dest, const void *src, size_t n); 
void *memmove(void *dest, const void *src, size_t n); 
```

Các hàm generic trong `string.h` này **sao chép** `n` byte từ vùng bộ nhớ `src` đến vùng bộ nhớ `dest`. Nói cách khác, chúng cho phép truy cập đọc/ghi dữ liệu trong bộ nhớ sử dụng con trỏ `void *`!

:::{hint} Sử dụng `memcpy` vì lý do hiệu suất, trừ khi bạn biết các vùng bộ nhớ chồng lấn.

Từ trang `man` của Linux cho `memcpy`: "Các vùng bộ nhớ không được chồng lấn."

Ngược lại, `memmove`: "Các vùng bộ nhớ có thể chồng lấn: việc sao chép diễn ra như thể các byte trong `src` trước tiên được sao chép vào một mảng tạm không chồng lấn với `src` hoặc `dest`, và sau đó các byte được sao chép từ mảng tạm đến dest."

Vì điều trên, `memcpy` nói chung nhanh hơn[^memmove-slow]

[^memmove-slow]: [Một số triển khai](https://clc-wiki.net/wiki/memmove) của memmove thực sự sử dụng bộ nhớ tạm (như trong C99), điều này có nguy cơ hết bộ nhớ. Đọc thêm trên [StackOverflow](https://stackoverflow.com/questions/4415910/memcpy-vs-memmove).

:::

## Generic Swap

Hãy xem xét cách `memcpy` generic sẽ thay thế việc giải tham chiếu trong một trong các câu lệnh của [hàm `swap_int`](#code-swap-int):

```c
// trong swap_int:
// ptr1, ptr2 đều được khai báo int *
*ptr1 = *ptr2; 
```

Câu lệnh này đơn giản là một `memcpy`—nó sao chép các byte của `int` tại `ptr2` vào các byte tương ứng tại `ptr1`!

```c
// tương đương với *ptr1 = *ptr2; của swap_int
memcpy(ptr1, ptr2, sizeof(int));
```

`memcpy` yêu cầu biết bao nhiêu byte cần sao chép từ nguồn đến đích. Hãy nhớ lại từ [chương trước](#sec-array-decay) rằng với các tham số con trỏ, hàm _sẽ không biết_ có bao nhiêu dữ liệu được trỏ đến. Do đó chúng ta phải cập nhật [mã giả](#code-swap-pseudo) để thêm một tham số kích thước bổ sung:

(code-swap-pseudo-size)=
```c
void swap(void *ptr1, void *ptr2, size_t nbytes) {
  // 1. lưu một bản sao của data1 vào bộ nhớ tạm

  // 2. sao chép data2 đến vị trí của data1

  // 3. sao chép dữ liệu trong bộ nhớ tạm đến vị trí của data2
}
```

### Triển khai

Cuối cùng, chúng ta đã sẵn sàng triển khai hàm **generic swap**, `swap`!

:::{card}
Hàm generic swap
^^^
(code-swap-generic)=
```{code} c
:linenos:
void swap(void *ptr1, void *ptr2, size_t nbytes) {
  // 1. lưu một bản sao của data1 vào bộ nhớ tạm
  char temp[nbytes];
  memcpy(temp, ptr1, nbytes);

  // 2. sao chép data2 đến vị trí của data1
  memcpy(ptr1, ptr2, nbytes);

  // 3. sao chép dữ liệu trong bộ nhớ tạm đến vị trí của data2
  memcpy(ptr2, temp, nbytes);
}
```
:::

Hàm `swap` ở trên sử dụng `memcpy` và do đó giả định rằng `ptr1` trỏ đến `nbytes` byte bộ nhớ và `ptr2` trỏ đến nbytes byte bộ nhớ, và không có sự chồng lấn giữa hai vùng bộ nhớ này.

:::{warning} Khai báo `char temp[nbytes]` là gì vậy?

[Hàm `swap_int`](#code-swap-int) không generic của chúng ta đã khai báo `temp` là một biến cục bộ đủ lớn để chứa `sizeof(int)` byte. Để làm điều tương tự trong một hàm generic, chúng ta phải khai báo một bộ nhớ tạm "generic" tương tự.

Vì `sizeof(char)` được định nghĩa là một byte trong C, chúng ta có thể đạt được bộ nhớ tạm generic này bằng cách khai báo một mảng `char` cục bộ, tức là một **buffer**.
:::

Bây giờ, hãy sử dụng `swap` generic mới này để hoán đổi hai `int`:

(code-swap-generic-main)=
```{code} c
:linenos:
int data1 = 22;
int data2 = 61; 
swap(&data1, &data2, sizeof(data1));
```

Lời gọi hàm trông gần như giống hệt [lời gọi không generic đến `swap_int`](#code-swap-int-main)! Giống như trước, chúng ta truyền vào vị trí của hai giá trị mà chúng ta muốn hoán đổi. Sự thay đổi duy nhất là bây giờ chúng ta bổ sung thêm kích thước của (các) giá trị, mà chúng ta giả định là giống nhau.

Bộ slide dưới đây theo dõi `swap` đang hoạt động sử dụng một ví dụ đơn giản. Ban đầu, `data1` lưu `22` tại địa chỉ `0x100`, và `data2` lưu `61` tại địa chỉ `0x104`. Giả sử `sizeof(int)` là 4.


:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vTd9HX8X0gtVYXfqM9Fv7-BkzzX7HHsRTjQLMV8zu9FqBTT0-NKepc9RWIJW9lKReYLZAPo16cX3G4V/pubembed?start=false&loop=false
:width: 100%
:title: "Animation that steps through the enumerated text in Section C Generics Implementation. Access [original Google Slides](https://docs.google.com/presentation/d/1pIqociLW0G65W5Bm4kPh9Kjq7UB64VdcOkhaA1zStxg/edit?usp=sharing)"
:::

## Ứng dụng: `swap_ends`

Cuối cùng, hãy xem xét generics hoạt động trên một mảng generic các giá trị. Chúng ta sẽ cần thực hiện số học con trỏ!

Chúng ta muốn sử dụng hàm `swap` để viết một hàm `swap_ends`, hoán đổi phần tử đầu tiên và cuối cùng trong một mảng.

[Code dưới đây](#code-swap-ends-main) sẽ cập nhật mảng `arr` như được hiển thị trong @fig-swap-ends:

(code-swap-ends-main)=
```{code} c
int main() {
  ...
  int32_t arr[] = {1, 2, 3, 4, 5};
  int32_t n = sizeof(arr)/sizeof(arr[0]);
  swap_ends(arr, n, sizeof(arr[0])); // cần triển khai
  ...
}
```

:::{figure} images/swap-ends.png
:label: fig-swap-ends
:width: 60%
:alt: "Array example after swap_ends: five consecutive integers at addresses 0x100 through 0x110 now read 5, 2, 3, 4, 1, showing that the first and last elements were exchanged."

`swap_ends` hoán đổi các phần tử `1` và `5` trong mảng `arr`.
:::

Hàm `swap_ends` có các đầu vào và đầu ra sau:

```c
void swap_ends(void *arr, size_t nelems, size_t nbytes);
```

* `arr`: Một con trỏ generic đến một khối bộ nhớ (nhớ rằng tất cả các đối số "mảng" đều suy biến thành con trỏ)
* `size_t nelems`: Số phần tử trong mảng
* `size_t nbytes`: Kích thước của mỗi phần tử, tính bằng byte

Hai tham số cuối chỉ định (1) khối bộ nhớ ("mảng") tại `arr` lớn bao nhiêu và (2) cách truy cập các phần tử tuần tự trong khối. Kết hợp lại, `nelems` và `nbytes` sẽ giúp chúng ta truy cập phần tử cuối cùng trong mảng.

:::{tip} Điền vào chỗ trống

Xem xét triển khai một phần của `swap_ends` dưới đây. Điền gì vào chỗ trống?

```c
void swap_ends(void *arr, size_t nelems, size_t nbytes) {
    swap(arr, ______ , nbytes); 
}
```

* **A.** `arr + nelems - 1`
* **B.** `arr + (nelems - 1)*nbytes`
* **C.** `(char *) arr + (nelems - 1) * nbytes`
* **D.** `(char *) (arr + (nelems - 1) * nbytes)`
* **E.**  Thứ gì khác

:::

**Đáp án**: Hãy xem xét `swap` làm gì. Nó nhận hai con trỏ và hoán đổi `nbytes` tại các vị trí đó. Để hoán đổi các đầu của mảng trong @fig-swap-ends, chúng ta muốn truyền vào `0x100` và `0x110` để hoán đổi.

Lựa chọn C thực hiện điều này một cách rõ ràng:

* `(char *) arr`: Ép kiểu `arr` generic thành `char *` để thực hiện **số học theo byte**.
* Cộng `(nelems - 1) * nbytes` vào con trỏ `(char *) arr`.
  * Ở đây, vì con trỏ trỏ đến `char`, số học con trỏ **thực tế là theo byte**.
  * Để trỏ đến phần tử cuối cùng, chúng ta nhảy đến cuối mảng (tính bằng byte), sau đó quay lại một độ dài phần tử, tính bằng byte.

Triển khai cuối cùng của `swap_ends`:

```c
void swap_ends(void *arr, size_t nelems, size_t nbytes) {
    swap(arr, (char *) arr + (nelems - 1) * nbytes, nbytes); 
}
```

:::{note} Các lựa chọn khác: C không chuẩn

Các lựa chọn B và D cũng hoạt động (không cần ép kiểu `(char *)`) nhưng [không phải C chuẩn](https://stackoverflow.com/questions/10058234/void-vs-char-pointer-arithmetic), chuẩn C không định nghĩa số học con trỏ trên con trỏ `void *`. Tuy nhiên, các lựa chọn này sẽ biên dịch và chạy tốt trên máy của khóa học với `gcc`, đó là một trình biên dịch cho _GNU_ C.

:::


(sec-generic-strings)=
## Các Generics khác trong `string.h`

Header `string.h` trong thư viện chuẩn C chứa các hàm chuỗi _và_ các hàm xử lý bộ nhớ, như `memcpy`, `memmove`, và `memset`.

Thoạt nhìn, bạn có thể thấy tên header hơi gây hiểu lầm. Tuy nhiên, như chúng ta đã thấy, các hàm xử lý bộ nhớ hoạt động từng byte một. Hơn nữa, theo định nghĩa, chuỗi là các mảng byte có kết thúc bằng null! Vì mảng byte có thể được truy cập với con trỏ `char *`, nhiều triển khai của hàm chuỗi đơn giản giả định các toán hạng `char *`.

Ví dụ, triển khai glibc của `strncpy` sử dụng `memset` và `memcpy`. Từ trang `man` của Linux:

```
... tối đa n byte của src được sao chép. Cảnh báo: Nếu không có byte null trong
n byte đầu tiên của src, chuỗi được đặt trong dest sẽ không được
kết thúc bằng null. Nếu độ dài của src nhỏ hơn n, strncpy() ghi
thêm các byte null vào dest để đảm bảo tổng cộng n byte được ghi.

...[trả về] một con trỏ đến chuỗi đích dest.
```

Đây là một phiên bản đơn giản hóa của [triển khai đầy đủ](https://codebrowser.dev/glibc/glibc/string/strncpy.c.html) trong glibc:

```{code} c
:linenos:
char *strncpy(char *dest, const char *src, size_t n) {
  size_t size = strnlen(src, n); // min(strlen(src), n)
  if (size != n) 
    memset(dest + size, '\0', n - size);
  return memcpy(dest, src, size);
}
```

:::{note} Giải thích
:class: dropdown

Giải thích từng dòng:

1. Khai báo hàm.
2. `size` được đặt thành $\min ($`strlen(src)`, `n` $)$. Từ trang `man` của Linux: "Hàm `strnlen()` trả về `strlen(s)`, nếu giá trị đó nhỏ hơn `maxlen`, hoặc `maxlen` nếu không có ký tự kết thúc null (`'\0'`) trong `maxlen` ký tự đầu tiên được trỏ bởi `s`."
3. Nhớ rằng các câu lệnh điều kiện không có dấu ngoặc nhọn coi câu lệnh tiếp theo là câu lệnh duy nhất của thân điều kiện (ở đây, Dòng 4).
4. Ghi vào các ký tự kết thúc null vượt quá độ dài của `src`. Nếu `n` ít nhất là `strlen(src) + 1` byte, dòng này kết thúc null cho kết quả.
5. Sao chép `size` byte từ `src` đến `dest`.

:::
