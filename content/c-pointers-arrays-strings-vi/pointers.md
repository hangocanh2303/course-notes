---
title: "Con trỏ và Lỗi"
---

(sec-pointers)=
## Mục tiêu học tập

* Biết cú pháp con trỏ trong C, bao gồm cú pháp con trỏ struct.
* Biết con trỏ `NULL` là gì và tại sao có con trỏ `NULL` là hữu ích.
* Hiểu rằng vì C là ngôn ngữ **truyền theo giá trị (pass-by-value)**, con trỏ giúp cập nhật các giá trị trong bộ nhớ khi thực hiện gọi hàm.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/tS3MOTQraL4
:width: 100%
:enumerated: false
:title: "Lecture 04.1 - C Intro: Pointers, Arrays, Strings: Pointers and Bugs"
:::
1:39 - 4:22
::::

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/LvewxVYZ2vg
:width: 100%
:enumerated: false
:title: "[CS61C FA20] Lecture 04.2 - C Intro: Pointers, Arrays, Strings: Using Pointers Effectively"
:::
::::

Như chúng ta sẽ thấy trong phần này, con trỏ là những trừu tượng cực kỳ mạnh mẽ giúp C hiệu quả. Nhưng sức mạnh đi kèm với trách nhiệm lớn và rất nhiều lỗi. Trong phần này, chúng ta sẽ làm quen với cú pháp con trỏ trong C để hiểu khi nào và ở đâu lỗi xuất hiện.

## Cú pháp con trỏ

(code-ptr-syntax)=
```{code} c
:linenos:
int *p;
int x = 3;
p = &x;
printf("p points to %d\n", *p);
*p = 5;
```

### [Dòng 1-2](#code-ptr-syntax): Khai báo con trỏ

Để khai báo một con trỏ, sử dụng cú pháp:

```c
int *p;
```

Dòng này cho trình biên dịch biết rằng biến `p` là địa chỉ của một `int`. Giống như tất cả các biến C, các biến chưa khởi tạo chứa rác, được biểu diễn bằng dấu hỏi trong @fig-ptr-syntax-line1.


:::{figure} images/ptr-syntax-line1.png
:label: fig-ptr-syntax-line1
:width: 60%
:alt: "Trạng thái ban đầu sau khai báo: biến con trỏ p tồn tại tại địa chỉ 0x100 nhưng chứa giá trị chưa khởi tạo không xác định, trong khi số nguyên x tại 0x104 được đặt là 3."
Khai báo một biến con trỏ.
:::

### [Dòng 3](#code-ptr-syntax): Toán tử địa chỉ, `&`

Để lấy địa chỉ của bất kỳ biến nào, sử dụng cú pháp:

```c
p = &x;
```

Dấu và (`&`) là **toán tử địa chỉ**. Nói một cách thông tục, chúng ta có thể mô tả Dòng 3 là "đặt `p` trỏ đến `x`" hoặc "đặt `p` bằng địa chỉ của `x`."

Trong @fig-ptr-syntax-line4, hai gợi ý trực quan cho thấy phép gán này. Đầu tiên, có một mũi tên xanh đi từ ô của biến `p` và trỏ đến ô của biến `x`. Thứ hai, vì mũi tên sơ đồ không dịch tốt sang các bit, giá trị của `p` đã được cập nhật thành địa chỉ của `x`, hay `0x104`.

:::{figure} images/ptr-syntax-line3.png
:label: fig-ptr-syntax-line3
:width: 60%
:alt: "Sau phép gán p = &x, con trỏ p lưu địa chỉ 0x104 và trỏ đến x."

Đặt `p` trỏ đến `x`.
:::

### [Dòng 4](#code-ptr-syntax): Toán tử giải tham chiếu, `*`

Xem xét dòng tiếp theo:

```c
printf("p points to %d\n", *p);
```

Chúng ta chưa thảo luận chi tiết về chuỗi định dạng. Tóm lại, `%d` đánh dấu một chỗ giữ số nguyên trong chuỗi định dạng. `printf` sau đó diễn giải tham số đầu tiên như một số nguyên để đặt vào chỗ giữ, rồi in chuỗi đã cập nhật ra stdout.

Dấu sao (`*`) ([cũng](#deref-two-ways)) là **toán tử giải tham chiếu**. Nói một cách thông tục, giải tham chiếu có nghĩa là chúng ta đi theo con trỏ và lấy giá trị mà nó trỏ đến. Như trong @fig-ptr-syntax-line4, giải tham chiếu `p` lấy giá trị số nguyên tại `0x104`, là số nguyên `3`.

:::{figure} images/ptr-syntax-line4.png
:label: fig-ptr-syntax-line4
:width: 60%
:alt: "Một mũi tên đen giữa con trỏ p và x cho thấy giải tham chiếu p đi theo địa chỉ 0x104 đến số nguyên x và đọc giá trị 3."

Đi theo con trỏ `p`, tức là truy cập giá trị mà `p` trỏ đến.
:::

Chuỗi được in ra stdout là

```
p points to 3
```

### [Dòng 5](#code-ptr-syntax): Giải tham chiếu và gán

Dòng cuối cùng:

```c
*p = 5;
```

Cú pháp này cũng sử dụng dấu sao `*` để giải tham chiếu. Ở đây, vì nó ở phía bên trái của câu lệnh gán (`=`), C diễn giải Dòng 5 là **đặt** giá trị mà `p` trỏ đến. Như trong @fig-ptr-syntax-line5, điều này cập nhật giá trị tại `0x104` thành số nguyên `5`.

:::{figure} images/ptr-syntax-line5.png
:label: fig-ptr-syntax-line5
:width: 60%
:alt: "Một giá trị mới cho x minh họa phép gán thông qua cập nhật con trỏ: *p = 5 ghi đè x từ 3 thành 5 trong khi p vẫn lưu 0x104."

Cập nhật giá trị mà `p` trỏ đến.
:::

Vì đây là ví dụ đơn giản, lưu ý rằng chúng ta cũng có thể cập nhật giá trị tại `0x104` bằng cách gán `x` bằng 3, ví dụ: `x = 3;`.
Tiếp theo, hãy xem các trường hợp mà giá trị _phải_ được cập nhật bằng con trỏ.

(deref-two-ways)=
:::{warning} Dấu sao (`*`) được sử dụng theo hai cách:

* **Khai báo**: Định nghĩa biến `p` là một con trỏ (Dòng 1)
* **Giải tham chiếu**: Đọc (Dòng 4) hoặc ghi (Dòng 5) giá trị được trỏ bởi p
:::

## C là ngôn ngữ truyền theo giá trị (Pass-by-Value)

Ngôn ngữ lập trình C là **truyền theo giá trị**, nghĩa là các tham số hàm nhận một **bản sao** của giá trị đối số.[^java-pass-by-value] Trong khi thuộc tính này hữu ích để đánh giá các đối số trước khi chúng được truyền vào như tham số, nó hạn chế các giá trị chúng ta có thể cập nhật.

[^java-pass-by-value]: Java cũng là truyền theo giá trị, mặc dù chúng ta nên lưu ý rằng trong Java, các biến giữ đối tượng vốn là object-handles, tức là tham chiếu. Sự phân biệt này giải thích hành vi của các kiểu nguyên thủy Java so với các "đối tượng" Java khi được truyền vào như đối số. Xem thêm trên [Stack Overflow](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value).

Xem xét đoạn code. Cập nhật giá trị của `x` trong hàm `add_one` không cập nhật giá trị của `y` trong `main`.

```{code} c
:linenos:
void add_one(int x) {
  x = x + 1;
}
int main() {
  int y = 3;
  add_one(y);
}
```

::::{note} Giải thích
:class: dropdown

* Dòng 5: Khai báo một biến `y` và đặt giá trị của nó là `3`.
* Dòng 6: Truyền một bản sao của giá trị `y` vào và đặt nó cho tham số `x` của hàm `add_one`.
* Dòng 2: Cập nhật `x` thành `x + 1`, tức là đặt `x` bằng 4. Trả về từ hàm.
* Dòng 6, đã trả về: giá trị tại `y` vẫn là `3`.

:::{figure} images/pass-by-value.png
:label: fig-pass-by-value
:width: 30%
:alt: "Ví dụ truyền theo giá trị cho thấy các hình chữ nhật cho biến x và y trong đó tham số cục bộ x thay đổi từ 3 thành 4, nhưng biến caller y vẫn là 3."

`y` không được cập nhật.
:::

::::

Để thay đổi một giá trị từ bên trong một chương trình con, chúng ta phải sử dụng con trỏ. Xem xét đoạn code cập nhật bên dưới. `main` truyền vào _địa chỉ_ của biến `y`, và `add_one` bây giờ có một tham số kiểu con trỏ. Chương trình con `add_one` bây giờ giải tham chiếu con trỏ để sửa đổi giá trị tại địa chỉ gốc của `y`.

```{code} c
:linenos:
void add_one(int *p) {
  *p = *p + 1;
}
int main() {
  int y = 3;
  add_one(&y);
}
```

::::{note} Giải thích
:class: dropdown

* Dòng 5: Khai báo một biến `y` và đặt giá trị của nó là `3`.
* Dòng 6: `add_one` bây giờ mong đợi một đối số con trỏ. Sử dụng toán tử địa chỉ (`&`) để lấy địa chỉ của `y` và truyền nó vào. Đây là giá trị `0x100`, được đặt cho tham số `p`.
* Dòng 7:
  * Phía bên phải: Giải tham chiếu `p` để lấy `3`, rồi cộng một để được `4`.
  * Phía bên trái: Gán giá trị được trỏ bởi `p` bằng `4`, tức là đặt 4 byte bắt đầu từ địa chỉ `0x100` thành biểu diễn bit của số nguyên `4`. Trả về từ hàm.
* Dòng 6, đã trả về: giá trị tại `y` bây giờ đã cập nhật thành `4`. Điều này là vì `y` nằm tại địa chỉ bộ nhớ `0x100`, và chương trình con `add_one` đã cập nhật giá trị tại địa chỉ này, trong bộ nhớ.

:::{figure} images/pass-by-value-ptr.png
:label: fig-pass-by-value-ptr
:width: 30%
:alt: "Ví dụ truyền theo con trỏ trong đó p lưu địa chỉ 0x100 của y, và ghi thông qua p cập nhật y từ 3 thành 4."

`y` được cập nhật từ bên trong hàm `add_one`.
:::

::::

## Con trỏ: Điều tốt, điều xấu, và điều tồi tệ

Vào thời điểm C được phát minh (đầu những năm 1970), các trình biên dịch không tạo ra mã hiệu quả, nên C được thiết kế để cho các lập trình viên con người nhiều linh hoạt hơn. Với mô hình truyền theo giá trị, việc truyền một con trỏ vào hàm dễ dàng hơn nhiều so với một struct hoặc mảng lớn.

Ngày nay, máy tính nhanh hơn hàng trăm nghìn lần so với các máy tính đầu tiên, và các trình biên dịch hiệu quả hơn nhiều. Tuy nhiên, con trỏ vẫn cực kỳ hữu ích để hiểu mã hệ thống cấp thấp, cũng như triển khai các mô hình đối tượng "truyền theo tham chiếu" trong các ngôn ngữ khác.

Trong khi con trỏ thường có thể cho phép mã sạch hơn, gọn hơn, chúng thường là nguồn lỗi lớn nhất trong C. Hãy cẩn thận! Chúng xuất hiện thường xuyên nhất khi [quản lý bộ nhớ động](#sec-heap) và gây ra các tham chiếu treo và rò rỉ bộ nhớ. Tại sao? Vì con trỏ cho bạn khả năng truy cập các giá trị trong bộ nhớ, _ngay cả khi bạn không nên có quyền truy cập_.

### Địa chỉ rác

Đây là một ví dụ. Giống như tất cả các biến cục bộ trong C, khai báo một biến con trỏ cục bộ **không khởi tạo nó**. Nó chỉ cấp phát không gian để giữ con trỏ!

Ví dụ trong @fig-garbage-addresses cho thấy mã sẽ biên dịch (mặc dù với một vài cảnh báo). Trong trường hợp này, `ptr` được cấp phát cho không gian nên được diễn giải như một `int *`, hoặc như một địa chỉ có một `int`. Bất kỳ byte nào ở đó tại thời điểm khai báo sau đó được diễn giải như địa chỉ để lưu giá trị `5`. Chương trình của bạn sau đó thể hiện hành vi không xác định. Điên rồ!

:::{figure} images/garbage-addresses.png
:label: fig-garbage-addresses
:width: 60%
:alt: "Phác thảo mã với con trỏ int chưa khởi tạo ptr và câu lệnh *ptr = 5, bên cạnh một hộp các bit không xác định cho ptr và một mũi tên đến vị trí không xác định. Hình minh họa hành vi không xác định từ việc ghi thông qua địa chỉ rác."

Các byte được lưu tại `ptr` được diễn giải như một địa chỉ của một `int`. Mã này có thể cập nhật `5` vào một phần ngẫu nhiên của bộ nhớ.
:::

## Sử dụng con trỏ hiệu quả

Tại "điểm" này, chúng tôi hy vọng chúng tôi chưa làm bạn sợ. Bạn vẫn nên mong đợi việc chơi với con trỏ, bất chấp những góc cạnh của chúng! Hãy thảo luận về việc sử dụng con trỏ hiệu quả.

### Con trỏ đến các kiểu dữ liệu khác nhau

Con trỏ được sử dụng để trỏ đến một biến của một kiểu dữ liệu cụ thể:

```c
int *xptr;
char *str;
struct llist *foo_ptr;
```

:::{note} Giải thích
:class: dropdown

* `int *xptr` khai báo một biến gọi là `xptr` trỏ đến một `int`.
* `char *str` khai báo một biến gọi là `str` trỏ đến một `char`.
* `struct llist *foo_ptr` khai báo một biến gọi là `foo_ptr` trỏ đến một `struct llist`.
:::

Khai báo kiểu của con trỏ xác định cách **toán tử giải tham chiếu** (`*`) hoạt động, tức là bao nhiêu byte để đọc/ghi khi chúng ta "đi theo" con trỏ.

Thông thường một con trỏ chỉ có thể trỏ đến một kiểu. Trong một [chương sau](#sec-generics) chúng ta thảo luận về con trỏ `void *`, một **con trỏ tổng quát** có thể trỏ đến bất kỳ thứ gì. Trong khóa học này chúng ta sẽ sử dụng con trỏ tổng quát một cách tiết kiệm để giúp tránh lỗi chương trình...và các vấn đề bảo mật...và những thứ khác... Tuy nhiên, chúng ta sẽ gặp con trỏ tổng quát khi làm việc với các hàm quản lý bộ nhớ trong thư viện chuẩn C (`stdlib`).

Chúng ta cũng có thể có con trỏ đến hàm, mà chúng ta thảo luận trong một [chương sau](#sec-generics). Bây giờ, nếu bạn tò mò về cú pháp:

```c
int (*fn) (void *, void *) = &foo;
(*fn)(x, y);
```

Trong dòng đầu tiên, `fn` là một hàm nhận hai con trỏ `void *` và trả về một `int`. Với khai báo này, chúng ta đặt nó trỏ đến hàm `foo`. Dòng thứ hai sau đó gọi hàm với các đối số `x` và `y`.

### Con trỏ `NULL`

Bất kể kiểu con trỏ, con trỏ đến địa chỉ toàn số không là đặc biệt. Đây là con trỏ `NULL`, giống như `None` của Python hoặc `null` của Java.

:::{figure} images/null.png
:label: fig-null
:width: 40%
:alt: "Khởi tạo con trỏ với NULL: con trỏ char p lưu địa chỉ toàn số không 0x00000000."

Trình biên dịch phân giải `NULL` thành địa chỉ toàn số không, tức là nơi tất cả các bit là 0.
:::

Địa chỉ `0x0...0` được **dành riêng**, nghĩa là không được phép đọc hoặc ghi vào địa chỉ đó; làm như vậy gây ra lỗi runtime. Trong khi bạn có thể nghĩ rằng đọc/ghi vào một "con trỏ null" là tin xấu - vì nó làm chương trình của bạn crash - điều này thực sự cực kỳ hữu ích như một "giá trị canh". Điều chúng tôi muốn nói là đặt một con trỏ thành `NULL` cho chúng ta biết rằng nó không trỏ đến một giá trị hợp lệ trong bộ nhớ.

Nhớ rằng giá trị boolean `false` là toàn số không. Điều này có nghĩa là _rất_ dễ kiểm tra nếu một con trỏ là `NULL` hay không! Trong mã bên dưới, `!p` sẽ chỉ phân giải thành `true` khi và chỉ khi `p` là `NULL`.

```c
if(!p) { /* p là một con trỏ null */ }
if(q) { /* q không phải là một con trỏ null */ }
```

(sec-pointer-arithmetic)=
### Số học con trỏ

Con trỏ có thể xử lý một số phép toán số học: cộng và trừ. Bạn có thể tăng hoặc giảm con trỏ bằng các giá trị số nguyên với một mô hình gọi là **số học con trỏ**.

Trong số học con trỏ, trình biên dịch sử dụng kiểu dữ liệu để xác định khoảng cách "bước" qua bộ nhớ để đến giá trị tiếp theo. Ví dụ, nếu `ptr` là một biến con trỏ và bạn viết `ptr + 5`, C sẽ không luôn cộng 5 vào `ptr`. Thay vào đó, C sẽ cộng 5 nhân với kích thước của kiểu dữ liệu mà `ptr` trỏ đến. Nếu ptr là một `int *` và `int` chiếm 4 byte trong bộ nhớ, `ptr + 5` cộng 20 vào địa chỉ được giữ trong ptr.

Nhớ rằng con trỏ lưu địa chỉ của bộ nhớ địa chỉ theo byte của chúng ta:

* `ptr + n` cộng `n*sizeof(*ptr)` byte vào địa chỉ bộ nhớ được lưu trong `ptr`.

* `pointer - n` trừ `n*sizeof(*ptr)` byte từ địa chỉ bộ nhớ được lưu trong `ptr`.

Số học con trỏ đặc biệt hữu ích để truy cập các phần tử của mảng với [chỉ mục ngoặc vuông](#sec-array-indexing).

Lưu ý bạn không thể cộng hai con trỏ với nhau (cộng hai địa chỉ có nghĩa gì??), nhưng bạn có thể trừ hai con trỏ[^pointer-subtraction].

[^pointer-subtraction]: Phép trừ con trỏ cũng phụ thuộc vào kiểu con trỏ; đọc thêm trên [StackOverflow](https://stackoverflow.com/questions/3238482/pointer-subtraction-confusion).

(foot-multiple-declarations)=
### Cú pháp con trỏ Struct

Chúng ta thường thích sử dụng con trỏ struct vì bản thân struct có thể khá lớn về kích thước. Có một vài "cú pháp đường" hữu ích mà chúng ta sử dụng với struct và con trỏ.

Mã bên dưới khai báo hai con trỏ[^multiple-declarations] `ptr1` và `ptr2` đến các struct `coord1` và `coord2`, tương ứng:


[^multiple-declarations]: Bạn có thể nhận thấy rằng Dòng 8 khai báo hai con trỏ bằng cách gắn `*` cạnh `ptr1` và `ptr2`, tương ứng. Chúng ta chưa thảo luận, nhưng một khai báo đơn `coord_t* ptr1;` cũng hợp lệ. Hầu hết các lập trình viên C hiện đại cố gắng tránh khai báo nhiều biến trên một dòng nếu có thể. Nhưng bạn sẽ thấy nó thường xuyên trong các ứng dụng C cũ. Đọc thêm trên [Reddit](https://www.reddit.com/r/cpp/comments/vm8bwm/how_do_you_declare_pointer_variables/).

```{code} c
:linenos:
typedef struct {
    int x;
    int y;
} coord_t;

/* khai báo */
coord_t coord1, coord2;
coord_t *ptr1, *ptr2;

... /* khởi tạo ở đây... */

/* ký hiệu dấu chấm */
int h = coord1.x;
coord2.y = coord1.y;

/* ký hiệu mũi tên = giải tham chiếu + truy cập struct*/
int k;
k = (*ptr1).x;
k = ptr1->x;  // tương đương
```

* **Toán tử dấu chấm** `.` được sử dụng để truy cập các thành viên struct.
* **Ký hiệu mũi tên** `->` là "cú pháp đường" (tức là viết tắt) cho giải tham chiếu (`*`) và truy cập (`.`). Ở đây, chúng ta lấy giá trị của thành viên `x` của struct được trỏ bởi `ptr1`.

::::{tip} Kiểm tra nhanh

Giả sử rằng chúng ta bắt đầu với trạng thái biến được hiển thị trong @fig-struct-pointers-q. Trạng thái sau khi thực thi đoạn mã (biên dịch được) bên dưới là gì?

```c
/* Đoạn này biên dịch được, nhưng nó làm gì? */
ptr1 = ptr2;
```

:::{figure} images/struct-pointers-q.png
:label: fig-struct-pointers-q
:width: 100%
:alt: "Trạng thái con trỏ struct bắt đầu trước ptr1 = ptr2: ptr2 lưu 0x100 và trỏ đến một struct với các trường x = 3 và y = 4, trong khi ptr1 có giá trị không xác định."

Trạng thái bắt đầu trước khi thực thi dòng `ptr1 = ptr2;`
:::

::::

::::{note} Hiển thị đáp án
:class: dropdown

Trạng thái được cập nhật thành @fig-struct-pointers-choiceb. Nói một cách thông tục, các con trỏ `ptr1` và `ptr2` bây giờ trỏ đến cùng một struct trong bộ nhớ @ địa chỉ `0x100`.

:::{figure} images/struct-pointers-choiceb.png
:label: fig-struct-pointers-choiceb
:width: 100%
:alt: "Trạng thái sau ptr1 = ptr2: cả ptr1 và ptr2 lưu 0x100 và trỏ đến cùng một struct với các trường x = 3 và y = 4."

Trạng thái sau khi thực thi dòng `ptr1 = ptr2;`
:::

Đôi khi dễ dàng hơn để quay lại định nghĩa của con trỏ như các biến lưu **địa chỉ**. Trong trường hợp này, chúng ta đang đọc giá trị tại `ptr2` (`0x100`) và lưu nó vào `ptr1`.

::::

## Con trỏ đôi ("Handles")

Làm thế nào một hàm có thể thay đổi giá trị của một **con trỏ**? Hãy xem một cách tiếp cận (tự nhiên nhưng) sai lầm trước một giải pháp.

**Cách tiếp cận bù nhìn[^strawman]**: Xem xét @code-pointer-handles-fail. Giả sử rằng khi biên dịch, bố cục bộ nhớ _trước Dòng 10_ là trong @fig-ptr-indexing.

[^strawman]: Wikipedia: [Straw Man](https://en.wikipedia.org/wiki/Straw_man)

(code-pointer-handles-fail)=
```{code} c
:linenos:
#include <stdio.h>

void increment_ptr(uint32_t *p) {
    p = p + 1;
}

int main() {
    uint32_t arr[] = {50, 60, 70};
    uint32_t *q = arr;
    increment_ptr(q);
    printf("*q is %d\n", *q);
  return 0;
}
```

:::{figure} images/array-indexing.png
:label: fig-ptr-indexing
:width: 80%
:alt: "Bố cục bộ nhớ trước khi gọi increment_ptr trong phiên bản con trỏ đơn thất bại: arr chứa 50, 60, và 70, và con trỏ q, nằm tại địa chỉ bộ nhớ 0x120, lưu 0x100 trỏ đến arr[0]."

Bố cục bộ nhớ trước khi thực thi Dòng 10 trong @code-pointer-handles-fail. Chúng ta thảo luận về mảng [sau trong chương này](#sec-array).
:::

Kết quả in: `*q is 50`

:::{note} Sai lầm: @code-pointer-handles-fail không cập nhật `q`
Nhớ rằng - C là **truyền theo giá trị**. Khi chúng ta gọi `increment_ptr(q)`, giá trị của `q` được truyền vào. Như trong sơ đồ bên dưới, điều này có nghĩa là tham số `p` nhận một bản sao của giá trị q (**địa chỉ** `0x100`), rồi `p` được cập nhật thành `0x104`. Tuy nhiên, khi hàm trả về, `q` vẫn không đổi.
:::

**Cách tiếp cận tốt hơn**. Mã trong @code-pointer-handles-success khắc phục vấn đề này bằng cách truyền vào một _con trỏ đến con trỏ_, còn được gọi là **con trỏ đôi** hoặc **handle**. Chạy mã bên dưới sẽ in: `*q is 60`.

(code-pointer-handles-success)=
```{code} c
:linenos:
#include <stdio.h>

void increment_ptr(uint32_t **h) {
    *h = *h + 1;
}

int main() {
    uint32_t arr[3] = {50, 60, 70};
    uint32_t *q = arr;
    increment_ptr(&q);
    printf("*q is %d\n", *q);
  return 0;
}
```

:::{figure} images/pointer-handles-success.png
:label: fig-pointer-handles-success
:width: 70%
:alt: "Ví dụ handle hai phần: trong cuộc gọi, con trỏ đôi h lưu địa chỉ 0x120 của con trỏ q, trỏ đến arr[0] (giá trị 50). Sau khi cập nhật con trỏ q từ 0x100 thành 0x104, q bây giờ trỏ đến arr[1] (giá trị 60)."

@code-pointer-handles-success: Bố cục bộ nhớ (trên) trong cuộc gọi `increment_ptr` và (dưới) sau khi trả về `main`.
:::

:::{note} Hiển thị giải thích
:class: dropdown

Thành công: @code-pointer-handles-success cập nhật `q` với con trỏ đôi

* Dòng 10, gọi hàm (@fig-pointer-handles-success, trên): `&q` truyền vào **địa chỉ** của `q`, `0x120`, như đối số cho `increment_ptr`.
* Dòng 3: Tham số `h` được gán cho `0x120`. `h` là một **con trỏ đôi**, nghĩa là đi theo nó hai lần sẽ đưa chúng ta đến một số nguyên không dấu 32-bit:
  * `h` là giá trị `0x120`.
  * ("đi theo một lần") `*h` truy cập giá trị tại địa chỉ `0x120`, bản thân nó là một địa chỉ, `0x100`.
  * ("đi theo hai lần") `**h` là `*(*h)`, truy cập giá trị tại địa chỉ `0x100`, là số nguyên không dấu 32-bit `50`.
* Dòng 4, phía bên phải: `*h + 1` tăng `0x100`, có _kiểu_: nó có địa chỉ của một số nguyên không dấu 32-bit. Tăng `0x100` do đó cho địa chỉ tiếp theo của một giá trị như vậy, là `0x104`.
* Dòng 4, phía bên trái: Gán giá trị `0x104` cho bất kỳ thứ gì `h` trỏ đến, là giá trị tại `0x120`. Nói cách khác, giá trị tại địa chỉ bộ nhớ `0x120` được cập nhật thành `0x104`.
* Dòng 10, trả về (@fig-pointer-handles-success, dưới): Cập nhật trong bộ nhớ vẫn còn!
:::
