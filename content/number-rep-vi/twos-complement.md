---
title: "Biểu diễn Có dấu: Two's Complement"
short_title: "Two's Complement"
---

(sec-twos-complement)=
## Mục tiêu học tập

* Chuyển đổi giữa số thập phân và biểu diễn two's complement
* So sánh two's complement với các biểu diễn có dấu và không dấu khác

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/opFCs4m7pW8?si=u3uolJHtdKzZRsBB
:width: 100%
:title: "[CS61C FA20] Lecture 02.4 - Number Representation: Two's Complement, Bias, and Summary"
:::

::::

## Two's Complement

Ones' complement:
* Vấn đề: Các ánh xạ âm "chồng lấn" với các ánh xạ dương, tạo ra hai số 0.
* Giải pháp: **Dịch các ánh xạ âm sang trái một đơn vị.**

:::{figure} images/twos-complement-number-line.png
:label: fig-twos-complement-number-line
:width: 100%
:align: center
:alt: "Một trục số nằm ngang màu xanh hiển thị các giá trị nhị phân 4-bit và các tương đương thập phân tương ứng từ -8 đến 7 để minh họa biểu diễn two's complement. Hai mũi tên màu vàng chỉ sang phải trên toàn bộ thang, chỉ ra rằng cộng vào giá trị nhị phân liên tục tăng giá trị số trên cả phạm vi âm và dương."

"Đồng hồ đo nhị phân" cho twos' complement 4-bit.
:::

Những điểm đáng chú ý:
* Giống như trong ones' complement, tăng đồng hồ đo nhị phân tương ứng với cộng số nguyên thêm một.
* `0b0000` vẫn là $0$.
* Số dương giống như ones' complement.
* Số âm được dịch sang! Ví dụ, `0b1111` bây giờ ánh xạ đến $-1$. Điều này cho chúng ta thêm một số âm.
* Bit có ý nghĩa nhất (bit ngoài cùng bên trái) vẫn có thể được hiểu như **sign bit**.

> Trong Two's Complement, một mẫu bit toàn một là $-1$.

:::{tip} Kiểm tra nhanh

Giả sử bạn dùng $N$ bits để biểu diễn số nguyên với two's complement. Có bao nhiêu số dương? số âm? số không?
:::

:::{note} Hiển thị đáp án
:class: dropdown

* Số không: 1
* Số dương: $2^{N-1} - 1$
* Số âm: $2^{N-1}$
:::

## Số học và chuyển đổi

Phần cứng cho two's complement giờ đơn giản.

> Phép cộng hoàn toàn giống như với số không dấu.

Các số $5$ và $-5$ được biểu diễn trong two's complement 4-bit với `0b0101` và `0b1011`, tương ứng. Cộng chúng lại nên cho kết quả $0$, hoặc `0b0000`.

:::{figure} images/twos-complement-addition.png
:label: fig-twos-complement-addition
:width: 100%
:align: center
:alt: "Một sơ đồ so sánh phép cộng thập phân của năm dương và năm âm với phép toán tương đương trong nhị phân two's complement, cộng 0101 và 1011. Phép tính nhị phân hiển thị các bit nhớ màu đỏ và kết quả trong tổng năm-bit trong đó bit đứng đầu bị loại bỏ để đạt được giá trị bốn-bit đúng là 0000, tức là số không."

Phép cộng trong two's complement tuân theo trực giác thập phân.
:::

:::{note} Giải thích
:class: dropdown

Làm việc từ phải sang trái:

1. `1+1=0` nhớ `1`
1. `1+0+1=0` nhớ `1`
1. `1+1+0=0` nhớ `1`
1. `1+0+1=0` nhớ `1`
1. `1` (bị cắt bỏ và loại bỏ trong biểu diễn 4-bit)

(Kiểm tra lại rằng toán nhị phân khớp với toán thập phân: $5 + -5 = 0$)
:::


## Định nghĩa chính thức

Chúng ta có thể viết giá trị của một số two's complement $n$-digit là

$$
- 2^{n-1} d_{n-1} + \sum_{i=0}^{n-2} 2^i d_i
$$

Số dương và số âm có thể được tính bằng cùng một công thức. Ở trên, dấu được tính bằng cách nhân bit cao nhất với $(-2^{n-1})$.

:::{card}
Ví dụ: $5$ và $-5$ trong two's complement 4-bit
^^^

$$
\begin{align}
\texttt{0b1011}
&= (1 \times -2^3)  + (0 \times 2^2)  + (1 \times 2^1)  + (1 \times 2^0) \\
&=  -8    +  0          +  2            +  1 \\
&=  -5 \\
\end{align}
$$

$$
\begin{align}
\texttt{0b0101}
&= (0 \times -2^3)  + (1 \times 2^2)  + (0 \times 2^1)  + (1 \times 2^0) \\
&=  0           +  4            +  0            +  1 \\
&=  5 \\
\end{align}
$$

:::

<!--TODO: có cách nghĩ đơn giản hơn về mặt sư phạm. Chúng ta thảo luận sau (tức là, sao chép công thức từ precheck summary).-->

## Two's Complement: Đảo dấu

Phần cứng để chuyển dương sang âm (& ngược lại) đơn giản.

1. Bù tất cả bits
1. Sau đó cộng 1

:::{figure} images/twos-complement-flip-shift.png
:label: fig-twos-complement-flip-shift
:width: 70%
:align: center
:alt: "Hai quy trình minh họa quá trình đổi dấu two's complement bằng cách chuyển năm dương sang năm âm và ngược lại. Mỗi ví dụ minh họa việc đảo bits của giá trị nhị phân bắt đầu và sau đó cộng một để có kết quả cuối cùng với dấu ngược lại."

Two's Complement: Để đổi dấu, đảo bits và cộng một.
:::

Ở nhà: Chứng minh thuật toán tương đương với công thức!

Trực giác đến từ biểu diễn "bánh xe số" của đồng hồ đo nhị phân (Hình @fig-twos-complement-number-wheel). Bánh xe này cũng giúp chúng ta xác định nơi integer overflow xảy ra:

:::{figure} images/twos-complement-number-wheel.png
:label: fig-twos-complement-number-wheel
:width: 100%
:align: center
:alt: "Một trục số nằm ngang và một bánh xe số tròn trực quan biểu diễn tính liên tục và các điểm overflow của số nguyên two's complement 4-bit. Các tam giác cảnh báo màu vàng đánh dấu ranh giới quan trọng giữa giá trị dương tối đa 0111 và giá trị âm tối thiểu 1000 để chỉ ra nơi arithmetic overflow xảy ra."

Trên: Một trục số chỉ ra nơi integer overflow xảy ra. Dưới: Một "bánh xe" số chỉ ra cùng vị trí integer overflow.
:::

Trong @fig-twos-complement-number-wheel, 0 đến 7 vẫn giữ nguyên như mọi biểu diễn. Nhưng sau đó nó nhảy đến $-8$. Số hạng cấp cao thú vị đó ($-8$) kéo tất cả số âm xuống **một** để không có chồng lấn tại số không.


## Two's Complement: Chuẩn C (tính đến 2025)

Two's complement là biểu diễn số chuẩn C23 cho số nguyên có dấu. Một lần nữa, `int` tích hợp là nhập nhằng vì nó không chỉ định bitwidth. Và một lần nữa, header `stdint.h` cung cấp các typedefs như `int8_t`, `int16_t`, `int32_t`, v.v., cho các biểu diễn số nguyên có dấu.
