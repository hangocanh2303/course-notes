---
title: "Phép toán Bitwise trong C"
short_title: "Phép toán Bitwise"
---

(sec-c-bitwise-ops)=
## Mục tiêu học tập

* Hiểu cách các phép toán bitwise "đảo" hoặc "giữ" các bit từ các toán hạng.
* Xác định các trường hợp sử dụng của AND, OR, XOR, và NOT.

::::{note} 🎥 Video hướng dẫn
:class: dropdown

Vui lòng truy cập [video YouTube này](https://www.youtube.com/watch?v=Q7w9wXs8ZBM) bằng đăng nhập UC Berkeley.

Xin chân thành cảm ơn Adelson Chua, một trong những thành viên đội ngũ CS 61C mùa Thu 2022, về bản ghi video này.
::::

Cho đến nay, chúng ta đã học về các phép toán số học liên quan đến [số nhị phân](#sec-integer-reps). Trong phần này, chúng ta sẽ học về **các phép toán bitwise**.

## Các phép toán Bitwise

Chúng ta ký hiệu các phép toán bitwise AND, OR, XOR, và NOT là các toán tử `&`, `|`, `^`, và `~`, tương ứng. Giả sử `a` và `b` là các giá trị một bit, các **bảng chân trị** sau đây chỉ định kết quả `y` của mỗi phép toán.[^bool-logic]

[^bool-logic]: Tên "bảng chân trị" đến từ logic boolean. Chúng ta thảo luận trong [phần sau](#sec-logic-gates) về thiết kế logic, khi hữu ích để biểu diễn tín hiệu là "cao" hoặc "thấp".

@tab-and là bitwise AND. `a & b` là `1` chỉ khi **cả** `a` **và** `b` đều là 1. Nếu không, nó là `0`.

:::{table} AND: `y = a & b`
:label: tab-and
:align: center

| `a` | `b` | `y` |
| :--: | :--: | :--: |
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |
:::

@tab-or là bitwise OR. `a | b` là `1` chỉ khi **một trong** `a` **hoặc** `b` là 1. Nếu không, nó là `0`.

:::{table} OR: `y = a | b`
:label: tab-or
:align: center

| `a` | `b` | `y` |
| :--: | :--: | :--: |
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |
:::

@tab-not là bitwise NOT. Nó là toán tử một ngôi vì nó chỉ nhận một toán hạng. `~a` là `1` chỉ khi `a` là 0. Nếu `a` là 1, thì `~a` là 0.

:::{table} NOT: `y = ~a`
:label: tab-not
:align: center

| `a` |`y` |
| :--: | :--: |
| 0 | 1 |
| 1 | 0 |
:::

@tab-xor là bitwise XOR ("exclusive OR" - OR loại trừ). `a ^ b` là `1` chỉ khi **một trong** `a` và `b` là 1. Nếu không, nếu cả `a` và `b` đều là `0` hoặc `1`, `a^b` là `0`.

:::{table} XOR: `y = a ^ b`
:label: tab-xor
:align: center

| `a` | `b` | `y` |
| :--: | :--: | :--: |
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |
:::

(sec-bitwise-props)=
### Tính chất của các phép toán Bitwise

Các tính chất dưới đây trong @tab-bitwise-props đúng cho giá trị một bit `x`. Chúng tôi để phần chứng minh cho bạn.

:::{table} Tính chất của các phép toán bitwise AND, OR, và XOR.
:label: tab-bitwise-props
:align: center

| Phép toán Bitwise | Ví dụ | Kết quả | Diễn giải |
| :--: | :--: | :--: | :-- |
| AND | `x & 0` | `0` | đặt về 0 |
| AND | `x & 1` | `x` | giữ nguyên |
| OR | `x \| 0` | `x` | giữ nguyên |
| OR | `x \| 1` | `1` | đặt về 1 |
| XOR | `x ^ 0` | `x` | giữ nguyên |
| XOR | `x ^ 1` | `~x` | đảo bit |
:::

Vì hành vi của nó, chúng ta cũng gọi XOR là "bộ đảo có điều kiện". Chúng ta thảo luận thêm khi thiết kế các cổng logic (xem [phần sau](#sec-adder-subtractor)).

(sec-bitwise-ops-defined)=
## C: Phép toán Bitwise vs. Phép toán Logic

Các toán tử bitwise `&`, `|`, và `~` được sử dụng trong C. Với các toán hạng n-bit, các phép toán bitwise được thực hiện trên (các) số nhị phân **từng bit một**; độ rộng bit của kết quả phụ thuộc vào các toán hạng đầu vào. Xem @tab-bitwise để có các ví dụ về giá trị `char` 8-bit.

:::{table} Ví dụ phép toán bitwise với giá trị `char` 8-bit.
:label: tab-bitwise
:align: center

| Phép toán Bitwise | Ví dụ C | Kết quả |
| :---: | :--- | :--- |
| AND| `0b00001001 & 0b00000011` | `0b00000001` |
| OR | `0b00001001 \| 0b00000011` | `0b00001011` |
| XOR | `0b00001001 ^ 0b00000011` | `0b00001010` |
| NOT | `~0b00001001` | `0b11110110` |
:::

Tuy nhiên, lưu ý rằng các toán tử bitwise C **không** nên nhầm lẫn với các **toán tử logic** C `&&`, `||`, và `!`. Ngược lại, **các phép toán logic** chuyển đổi các giá trị bit thành giá trị truthy và falsy. Các loại phép toán này còn được gọi là **toán tử kết hợp boolean**[^python-bool].

[^python-bool]: Trong Python, các toán tử boolean là `and`, `or`, và `not`. Python cũng hỗ trợ các toán tử bitwise `&`, `|`, `~`, và `^`.

:::{note} Ví dụ

Cho các số nguyên có dấu 4-bit `0b0101` và `0b0100`:

* Bitwise AND: `0b0101 & 0b0100` là `0b0100`, hay 4.
* Logical AND: `0b0101 && 0b0100` là true, vì cả hai giá trị đều true.
* Bitwise NOT: `~0b0101` là `0b1010`.
* Logical NOT: `!0b0101` là false.
:::

(sec-bitmasks)=
## Bitmask

Đây là bước đầu tiên của chúng ta vào các phép toán bitwise. Tại sao chúng ta quan tâm?

Nhớ rằng `n` bit có thể biểu diễn $2^n$ thứ, và chúng ta thường muốn sử dụng các bit để biểu diễn nhiều thứ hơn chỉ là số. Giả sử chúng ta muốn sử dụng các mẫu bit để biểu diễn liệu `n` thứ có hiện diện hay không. Chúng ta có thể sử dụng mỗi trong số `n` bit của chúng ta để biểu diễn liệu mỗi thứ có hiện diện (`1`) hay không (`0`).

Sử dụng các phép toán bitwise (và [các tính chất của các phép toán bitwise đó](#sec-bitwise-props)) sẽ rất tiện lợi để cập nhật trạng thái sử dụng các mẫu bit gọi là **bitmask**.

:::{note} Ví dụ 1

Giả sử trạng thái của bốn thứ A, B, C, D được biểu diễn bởi mẫu 4-bit `x`, trong đó A được biểu diễn bởi bit trái cùng, B bởi bit thứ hai từ trái, và cứ tiếp tục.

* Nếu `x` là `0b0110`, thì B và C hiện diện.
* Giả sử chúng ta muốn cập nhật `D` thành hiện diện. `x | 0b0001` làm điều này. `0b0001` được gọi là bitmask.
* Giả sử chúng ta muốn đặt lại A, B, C thành không hiện diện. `x & 0b0001` làm điều này. `0b0001` cũng được gọi là bitmask.
:::

:::{note} Ví dụ 2
Giả sử `N` là một giá trị 32-bit (tức là 4 byte). Chúng ta có thể lấy byte ít quan trọng nhất (LSB) của `N` bằng cách sử dụng bitmask `0xFF`: `N & 0xFF`. Xác minh điều này hoạt động với bất kỳ giá trị 32-bit nào của `N`!

:::

Các phép toán bitwise sẽ rất hữu ích cho logic boolean sau này khi chúng ta giới thiệu các cổng logic.

## Thêm các toán tử Bitwise C: Dịch trái và Dịch phải

Hai phép toán bổ sung mô tả **dịch bit**.

Giả sử bạn có các mẫu bit 8-bit (nơi chúng ta đặt khoảng trắng giữa các nibble để dễ đọc):

* `x`, với mẫu bit `0001 0001`
* `y`, với mẫu bit `1111 0001`

(sec-left-shift)=
**Dịch trái** `x << n` dịch các bit của `x` sang trái `n` bit, điền `n` bit thấp hơn ("đi vào từ bên phải") bằng `0`. Về mặt toán học, điều này tương đương với nhân `x` với $2^{\texttt{n}}$.

  * Ví dụ, `x << 2` cho mẫu bit `0100 0100`. Nếu chúng ta diễn giải `x` như một số nguyên có dấu 8-bit 17, thì `x << 2` thực sự là $17 \times 4 = 68$.
  * Dịch trái cũng gặp overflow: `x << 4` cho mẫu bit `0001 0000` nơi bit `1` trái cùng bị "dịch ra ngoài" kiểu 8-bit (để tạo ra số nguyên có dấu 8-bit 16, chắc chắn không phải $17 \times 2^4$).

(sec-right-shift)=
**Dịch phải**, `x >> n` dịch các bit của `x` sang phải `n` bit. Về mặt toán học, điều này tương đương với lấy phần nguyên của phép chia cho $2^{\texttt{n}}$. Chúng ta vẫn cần điền các bit cao đi vào từ bên phải bằng cách nào đó, nhưng phép toán chính xác trong C phụ thuộc vào kiểu của `x`.

(sec-right-shift-logical)=
* **Dịch phải logic** "mở rộng bằng không" và điền các bit cao bằng `0`. Nếu `x` là một số nguyên không dấu 8-bit, thì `x >> 2` cho mẫu bit `0000 0100`, nơi bit `1` phải cùng bị dịch ra. Điều này tương đương với `(unsigned char) 17 >> 2` cho kết quả `4`.

(sec-right-shift-arithmetic)=
* **Dịch phải số học** "mở rộng dấu" và điền các bit cao bằng bit dấu của `x`. Do đó dịch phải số học bảo toàn bit dấu của kết quả khi sử dụng các toán hạng có dấu.

:::{note} Ví dụ

Nếu `y` là một số nguyên có dấu 8-bit two's complement với mẫu bit `1111 0001`, thì `y >> 2` cho mẫu bit `1111 1100`, nơi bit `1` phải cùng bị dịch ra. Điều này tương đương với `(char) -15 >> 2` cho kết quả `-4`.

:::

(sec-why-shift-left)=
:::{warning} Kiểm tra nhanh

Tại sao chúng ta không định nghĩa logic và số học cho dịch _trái_?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Về mặt logic, dịch trái dịch một mẫu bit sang trái và chèn các số không. Về mặt số học, dịch trái một số (bất kể dấu của nó) nhân nó với lũy thừa của hai, cũng chèn các số không. Do đó dịch trái số học và logic là tương đương.
:::

(sec-c-bitwise-practice)=
## Thực hành

:::{tip} Thực hành
Sau khi mã bên dưới được thực thi, `y` là gì?

```c
uint32_t N = 0x34FF;
uint32_t y = N & ((N << 0x10) >> 0x8);
```

* **A.** `0x0`
* **B.** `0x3400`
* **C.** `0x4F0`
* **D.** `0xFF00`
* **E.** `0x34FF`
* **F.** Khác

:::

::::{note} Hiển thị đáp án
:class: dropdown

**B.** `0x3400`.

* `N` là `0x000034FF`
* `N << 0x10` là `0x34FF 0000`. Chúng ta dịch trái `N` 16 bit.
* `(N << 0x10) >> 0x8` là `0x0034 FF00`. Vì `N` là một số nguyên không dấu, dịch phải số học 8 bit chèn các số không và tương đương với dịch phải logic.
* `N & (N << 0x10) >> 0x8` là `0x0000 3400`, như trong @fig-bit-shift. Lưu ý rằng `0x34 & 0x00` và `0xFF & 0x00` đều là không, vì AND bất cứ thứ gì với 0 đặt tất cả các bit về không.


:::{figure} images/bit-shift.png
:label: fig-bit-shift
:width: 50%
:alt: "Phép toán bitwise AND giữa 0x000034FF và 0x0034FF00, tạo ra 0x00003400. Các byte giữa được tô sáng để hiển thị bitwise AND giữa các byte 0x34 và 0xFF nơi chỉ các bit 1 chồng lên nhau được giữ."

Kết quả của `N & (N << 0x10) >> 0x8`, tức là `0x000034FF & 0x0034 FF00`
:::

::::
