---
title: "Dạng Chính Tắc, Thiết Kế Logic Tổ Hợp"
subtitle: Tác giả John Wawrzynek, chỉnh sửa bởi Lisa Yan
---

(sec-cl-practice-vi)=
## Mục Tiêu Học Tập

* Thực hành chuyển đổi giữa ba biểu diễn của khối logic tổ hợp: bảng chân trị, biểu thức boolean, và sơ đồ cổng.
* Thực hành sử dụng các định luật của Đại số Boolean.
* Thực hành thiết kế module: kết hợp các mạch phức tạp từ các mạch nhỏ hơn.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/aZ6o8dDwgmM
:width: 100%
:title: "[CS61C FA20] Lecture 16.5 - Combinational Logic: Canonical Forms"
:::

::::

Mục tiêu của bài giảng này là xây dựng các khối logic tổ hợp. Chúng ta có thể tóm tắt cách tiếp cận này với sơ đồ sau:

:::{figure} images/cl-block-representation.png
:label: fig-cl-block-representation-vi
:width: 60%
:alt: "Sơ đồ với ba bong bóng mô tả ba biểu diễn của logic tổ hợp: Bảng Chân Trị, Biểu Thức Boolean, và Sơ Đồ Cổng. Các biểu diễn được kết nối bằng các mũi tên cong hiển thị chuyển đổi giữa chúng."

Ba biểu diễn của một khối logic tổ hợp: bảng chân trị, biểu thức Boolean, và sơ đồ cổng. Chúng ta có thể chuyển đổi giữa các biểu diễn khác nhau.
:::

@fig-cl-block-representation-vi hiển thị ba biểu diễn có thể cho một khối logic tổ hợp.

* Dạng **bảng chân trị** là duy nhất — với mỗi khối logic tổ hợp, chỉ có một bảng chân trị. Tính duy nhất của bảng chân trị làm cho nó trở thành cách hữu ích để định nghĩa rõ ràng chức năng của khối logic tổ hợp.
* Có nhiều **biểu thức Boolean** có thể có cho bất kỳ khối logic tổ hợp nào. Biểu thức Boolean hữu ích cho thao tác đại số, đặc biệt là đơn giản hóa.
* Nhiều biểu diễn **mức cổng** cho bất kỳ khối logic tổ hợp nào. Sơ đồ cổng cho chúng ta đơn thuốc để chuyển đổi sang mạch transistor thực tế; mỗi cổng trong sơ đồ có thể được thay thế bằng một mạch transistor nhỏ để đạt được chức năng mức cổng tương ứng.

:::{warning} Bảng chân trị là duy nhất
Với một khối logic tổ hợp cho trước, chỉ có một bảng chân trị. Ngược lại, một khối logic tổ hợp có thể có nhiều biểu thức Boolean và nhiều sơ đồ mạch.
:::

Phần này thảo luận cách chuyển đổi giữa các biểu diễn khác nhau của khối logic tổ hợp. Phần này kết thúc với cách _kết hợp_ các mạch từ các khối nhỏ hơn.

(sec-majority-circuit-vi)=
### Ví Dụ: Mạch Đa Số

Xét phương trình bên dưới. Phương trình này mô tả một **mạch đa số ba đầu vào** vì đầu ra y nhận giá trị khớp với đa số các giá trị đầu vào.

```{math}
:label: eq-majority-circuit-vi
:enumerated: true
y = ab + bc + ac
```

Phương trình @eq-majority-circuit-vi có bảng chân trị tương ứng

:::{table}
:label: tab-majority-circuit-vi

| a | b | c | y |
| :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 |
| 0 | 1 | 0 | 0 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 1 |
| 1 | 1 | 0 | 1 |
| 1 | 1 | 1 | 1 |
:::

và có mạch cổng tương ứng

:::{figure} images/majority-circuit.png
:label: fig-majority-circuit-vi
:width: 60%
:alt: "Ba cổng AND hai đầu vào ghép cặp trên a, b, và c nạp vào một cổng OR ba đầu vào để ra y, hiện thực hàm đa số với ghi chú về các điểm nối dây."

Mạch tương ứng với $y = ab + bc + ac$.
:::

Phương trình nói rằng đầu ra y là OR ($+$) của ba số hạng, trong đó mỗi số hạng là AND của hai biến đầu vào ($\cdot$ hoặc nối). Sự tương ứng giữa @eq-majority-circuit-vi và mạch trong @fig-majority-circuit-vi nên rõ ràng. Ba số hạng tích được hiện thực sử dụng cổng AND và tổng cuối cùng bằng cổng OR ba đầu vào.



(sec-sum-product-vi)=
## Dạng Chính Tắc

Hãy quay lại bảng chân trị như định nghĩa của hàm mong muốn. Bắt đầu từ bảng chân trị, bằng cách quan sát, có thể viết một biểu thức Boolean khớp với hành vi của nó. Loại biểu thức Boolean mà chúng ta sẽ viết được gọi là dạng chính tắc **"tổng của tích"** (sum-of-products).

Dạng chính tắc có tên như vậy vì nó được hình thành bằng cách lấy tổng (OR) của một tập các số hạng, mỗi số hạng là một tích (AND) của các biến đầu vào hoặc bù của chúng.

* Mỗi nơi (hàng) có số 1 xuất hiện ở đầu ra, chúng ta sẽ sử dụng một số hạng tích (AND) trong biểu thức Boolean.
* Mỗi số hạng AND bao gồm tất cả các biến đầu vào ở dạng bù hoặc không bù.
* Một biến xuất hiện ở dạng bù nếu nó là 0 trong hàng của bảng chân trị, và là 1 nếu nó xuất hiện là 1 trong hàng.

:::{warning} Tại sao Tổng của Tích hoạt động?

Trong dạng chính tắc, các số hạng tích là tất cả các liệt kê khác nhau của các đầu vào. Đầu ra là 1 nếu ít nhất một[^input] trong các tích là một. Đầu ra là không nếu không có tích nào là một. Về mặt logic, đây là cùng chức năng được định nghĩa bởi bảng chân trị!

[^input]: Thực tế, "chính xác" một, vì các tổ hợp đầu vào là duy nhất. Trong Tổng của Tích, chúng ta chọn OR (thay vì "XOR") để đơn giản hóa biểu thức với các định luật của Đại số Boolean.

Trừ tính kết hợp và giao hoán của các số hạng, dạng chính tắc tổng của tích là duy nhất.
:::

(sec-sum-product-example-vi)=
## Ví Dụ: Bảng Chân Trị $\rightarrow$ Biểu Thức Boolean $\rightarrow$ Sơ Đồ Cổng

Bất kỳ mạch CL nào có thể được đặc tả thực tế dưới dạng bảng chân trị đều có thể được biểu diễn bằng biểu thức Boolean bằng cách viết **dạng chính tắc** của nó. Sau đó, thông qua thao tác đại số, nó có thể được đơn giản hóa, rồi chuyển đổi thành mạch cổng logic.

:::{tip} Ví dụ
Thiết kế một mạch hiện thực hàm 3 đầu vào được mô tả bởi bảng chân trị bên dưới.

| $a$ | $b$ | $c$ | $y$ |
| :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 1 |
| 0 | 0 | 1 | 1 |
| 0 | 1 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 0 | 1 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |
| 1 | 1 | 1 | 0 |
:::

**1. Sử dụng bảng chân trị để xây dựng dạng chính tắc.** Phương trình dạng chính tắc có bốn số hạng tích, một cho mỗi số 1 trong đầu ra $y$.

$$y = \overline{a} \overline{b} \overline{c} + \overline{a} \overline{b} c + a \overline{b} \overline{c} + a b \overline{c}$$

:::{note} Nhấp để xem bảng chân trị có chú thích
:class: dropdown

| $a$ | $b$ | $c$ | $y$ | Tích |
| :---: | :---: | :---: | :---: | :--- |
| 0 | 0 | 0 | 1 | $\overline{a} \cdot \overline{b} \cdot \overline{c}$ |
| 0 | 0 | 1 | 1 | $\overline{a} \cdot \overline{b} \cdot c$ |
| 0 | 1 | 0 | 0 | |
| 0 | 1 | 1 | 0 | |
| 1 | 0 | 0 | 1 | $a \cdot \overline{b} \cdot \overline{c}$ |
| 1 | 0 | 1 | 0 | |
| 1 | 1 | 0 | 1 | $a \cdot b \cdot \overline{c}$ |
| 1 | 1 | 1 | 0 | |

:::

**2. Đơn giản hóa sử dụng các định luật Đại số Boolean, nếu cần.**

```{math}
\begin{aligned}
    y &= \overline{a} \overline{b} \overline{c} + \overline{a} \overline{b} c + a \overline{b} \overline{c} + a b \overline{c} && \text{Tổng của Tích} \\
      &= \overline{a} \overline{b} (\overline{c} + c) + a \overline{c} (\overline{b} + b) && \text{Phân phối} \\
      &= \overline{a} \overline{b} (1) + a \overline{c} (1) && \text{Nghịch đảo (OR) } x + \overline{x} = 1 \\
      &= \overline{a} \overline{b} + a \overline{c} && \text{Đồng nhất (AND) } x \cdot 1 = x
\end{aligned}
```

**3. Vẽ sơ đồ cổng.** Mạch được đặc tả bởi dạng chính tắc là một trong nhiều mạch hiện thực hàm 3 đầu vào gốc.


:::{figure} images/sum-product-example.png
:label: fig-sum-product-example-vi
:width: 50%
:alt: "Mạch kiểu tổng của tích: cổng NOT trên a, b, và c; một AND kết hợp NOT a với NOT b, AND khác kết hợp a với NOT c; cả hai đầu ra nạp vào cổng OR để ra y."

Mạch được đặc tả bởi dạng chính tắc trong [ví dụ](#sec-sum-product-example-vi).
:::

:::{tip} Bài tập thêm
Thử thiết kế mạch này chỉ với cổng NOR.
:::


## Ví Dụ: Đơn Giản Hóa Mạch

Đơn giản hóa mạch bao gồm lấy sơ đồ cổng và viết biểu thức Boolean của nó, đơn giản hóa nó, rồi chuyển đổi lại thành sơ đồ cổng.

:::{tip} Xem đơn giản hóa mạch trong [phần trước](#sec-boolean-algebra-vi).

* [Thiết lập](#sec-or-simplify-setup-vi)
* [Chứng minh](#sec-or-simplify-proof-vi)
:::

## Ví Dụ: Xác Minh Mạch (Mạch Đa Số, Xem Lại)

Trong [mạch đa số ba đầu vào](#sec-majority-circuit-vi), chúng ta có thể sử dụng bảng chân trị để xây dựng dạng chính tắc:

$$y = \overline{a}bc + a\overline{b}c + ab\overline{c} + abc$$

::::{note} Nhấp để xem bảng chân trị có chú thích
:class: dropdown

:::{table}

| $a$ | $b$ | $c$ | $y$ | Tích |
| :---: | :---: | :---: | :---: | :--- |
| 0 | 0 | 0 | 0 | |
| 0 | 0 | 1 | 0 | |
| 0 | 1 | 0 | 0 | |
| 0 | 1 | 1 | 1 | $\overline{a} \cdot b \cdot c$ |
| 1 | 0 | 0 | 0 | |
| 1 | 0 | 1 | 1 | $a \cdot \overline{b} \cdot c$ |
| 1 | 1 | 0 | 1 | $a \cdot b \cdot \overline{c}$ |
| 1 | 1 | 1 | 1 | $a \cdot b \cdot c$ |
:::

::::

Sau đó chúng ta có thể sử dụng đại số Boolean để xác minh rằng dạng chính tắc tương đương với [phương trình mạch đa số](#eq-majority-circuit-vi) trực quan:

```{math}
\begin{aligned}
    y &= \overline{a}bc + a\overline{b}c + ab\overline{c} + abc \\
      &= (\overline{a}bc + abc) + a\overline{b}c + ab\overline{c} + abc && \text{Lũy đẳng: } x = x + x \\
      &= (\overline{a} + a)bc + a\overline{b}c + ab\overline{c} + abc && \text{Phân phối} \\
      &= (1)bc + a\overline{b}c + ab\overline{c} + abc && \text{Nghịch đảo (OR)} \\
      &= bc + (a\overline{b}c + abc) + ab\overline{c} + abc && \text{Lũy đẳng (lần nữa)} \\
      &= bc + a(\overline{b} + b)c + ab\overline{c} + abc && \text{Phân phối} \\
      &= bc + ac + ab(\overline{c} + c) && \text{Phân phối} \\
      &= bc + ac + ab(1) && \text{Nghịch đảo (OR)} \\
      &= bc + ac + ab && \text{Biểu thức đơn giản hóa}
\end{aligned}
```

## Ví Dụ: Sơ Đồ Cổng $\rightarrow$ Bảng Chân Trị

Để tạo bảng chân trị từ một mạch cho trước, chúng ta chỉ cần đánh giá đầu ra cho tất cả tổ hợp đầu vào có thể.

::::{tip} Bảng chân trị của mạch bên dưới là gì?

:::{figure} images/compare-2-circuit.png
:label: fig-compare-2-circuit-vi
:width: 70%
:alt: "Mạch so sánh bằng một bit: AND của a và b song song với AND của NOT a và NOT b, cả hai đầu ra nạp vào cổng OR để ra y, đại diện cho logic mà y cao khi cả hai bit khớp."
Mạch cho khối so sánh 2 bit.
:::

::::

Bảng Chân Trị:

| a | b | y |
| :--: | :--- | :--- |
| 0 | 0 | 1 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

:::{note} Hiển thị Giải thích
:class: dropdown

* Cổng AND trên: Đầu ra của cổng này là `1` chỉ nếu `a` và `b` đều là `1`.
* Cổng AND dưới. Do hai cổng NOT ngay trước đầu vào, đầu ra của cổng này là `1` chỉ nếu NOT `a` và NOT `b` đều là `1`, tức là `a` và `b` đều là `0`.[^not-nand]
* Cổng OR. Đầu ra `y` là `1` nếu ít nhất một trong các cổng AND là `1`.

[^not-nand]: Đáng chú ý, mạch này không phải là cổng NAND; chúng tôi để phần chứng minh cho bạn. Chúng tôi đề nghị viết mọi thứ ra với bảng chân trị.
:::

(sec-compare-2-vi)=
### So Sánh 2 Bit

Sau một chút suy nghĩ, bạn có thể nhận ra rằng mạch logic tổ hợp này có thể được mô tả như một **so sánh**: Nhận hai đầu vào và trả về `1` nếu hai đầu vào bằng nhau và `0` nếu không. Chúng ta sử dụng chức năng này trong [ví dụ sau](#sec-compose-circuits-vi).[^compare-xor]

[^compare-xor]: Chúng tôi lưu ý rằng đầu ra của hai cổng AND sẽ không bao giờ _cả hai_ đều là `1`, vì vậy chúng ta có thể thay thế cổng OR bằng cổng XOR.

(sec-compose-circuits-vi)=
## Kết Hợp Mạch: Bộ So Sánh 32 Bit

Chúng ta muốn một mạch **so sánh hai giá trị 32 bit**; mạch so sánh 32 bit này có thể được sử dụng trong bộ xử lý của chúng ta để hiện thực so sánh bằng cho lệnh RV32I `beq rs1 rs2 Label`.

:::{figure} images/compare-32-block.png
:label: fig-compare-32-block-vi
:width: 30%
:alt: "Ký hiệu khối với bus gạch chéo 32 bit cho A và B đi vào đỉnh khối. Một đầu ra đơn Z kết quả từ khối; nội dung khối hiển thị bằng với chỉ số dưới 32 và dấu hỏi, đại diện cho logic tổ hợp kiểm tra bằng."

Mạch so sánh 32 bit.
:::

:::{hint} Sức mạnh của **thiết kế module**

Bảng chân trị của bộ so sánh 32 bit quá lớn để chúng ta viết bằng tay (nó có $2^{64}$ hàng, cho mỗi tổ hợp của các bit của `A` và `B`). Điều này có nghĩa là viết dạng chính tắc, chưa nói đến xây dựng mạch từ nó, là không khả thi.

Nếu bảng chân trị quá lớn để xây dựng, hãy phân rã bằng cách định nghĩa các mạch nhỏ hơn trước, sau đó kết hợp mạch lớn hơn với các khối mạch nhỏ hơn này. Đây là sức mạnh của **thiết kế module**.
:::

Do đó, chúng ta muốn sử dụng mạch so sánh 2 bit mà chúng ta đã xây dựng trong [ví dụ trước](#sec-compare-2-vi). Chúng ta biểu diễn mạch như một **khối logic tổ hợp**, như bên dưới. Khối này giúp trừu tượng hóa; bất cứ khi nào bạn thấy khối trong @fig-compare-2-block-vi, hãy biết rằng nó được hiện thực như mạch trong @fig-compare-2-circuit-vi.

:::{figure} images/compare-2-block.png
:label: fig-compare-2-block-vi
:width: 30%
:alt: "Khối hình chữ nhật với đầu vào một bit a và b ở trên và đầu ra y bên dưới. Dấu bằng với chỉ số dưới một với dấu hỏi bên trong khối biểu thị so sánh một bit kiểm tra bằng."

Mạch so sánh từng bit; biểu diễn khối của mạch bí ẩn trong @fig-compare-2-circuit-vi.
:::

Để so sánh, với mạch so sánh 32 bit (@fig-compare-32-block-vi):

* Giá trị `A` là giá trị 32 bit, vì **bó dây** (dây được đánh dấu bằng vết khía). Tương tự, giá trị `B` là 32 bit.
* Giá trị `z` là giá trị 1 bit.

Về chức năng, `z` là `1` chỉ nếu tất cả các bit của `A` bằng với tất cả các bit của `B`. Nói cách khác, chúng ta có thể thực hiện 32 so sánh từng bit, sau đó AND chúng lại với nhau. Mạch cơ bản cho khối so sánh của chúng ta do đó như sau:

:::{figure} images/compare-32-circuit.png
:label: fig-compare-32-circuit-vi
:width: 80%
:alt: "Ba mươi hai khối so sánh bằng một bit song song trên các cặp đầu vào a31 b31 đến a0 b0, tất cả nạp vào một cổng AND 32 đầu vào có đầu ra là z. Đầu ra z cao khi mọi cặp bit khớp."

Sơ đồ mạch so sánh 32 bit, giả sử chúng ta đã hiện thực mạch so sánh từng bit. Cổng AND 32 đầu vào có thể được hiện thực đệ quy với các cổng AND 2 hoặc 4 đầu vào.
:::
