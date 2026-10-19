---
title: "Đại Số Boolean"
subtitle: Tác giả John Wawrzynek, chỉnh sửa bởi Lisa Yan
---

(sec-boolean-algebra-vi)=
## Mục Tiêu Học Tập

* Nhận biết các phép toán AND, OR, và NOT trong Đại số Boolean.
* Đơn giản hóa biểu thức Đại số Boolean sử dụng các định luật của Đại số Boolean.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/W_WJXNfXBJU
:width: 100%
:title: "[CS61C FA20] Lecture 16.3 - Combinational Logic: Boolean Algebra"
:::

::::

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/8ORLlBY2E7Q
:width: 100%
:title: "[CS61C FA20] Lecture 16.4 - Combinational Logic: Laws of Boolean Algebra"
:::

::::

Trong [phần trước](#sec-logic-gates-vi), chúng ta đã giải thích cách tạo bảng chân trị từ một mạch cho trước (chúng ta chỉ cần đánh giá nó cho tất cả tổ hợp đầu vào). Câu hỏi lớn là: Làm thế nào để đi _theo hướng ngược lại_? Làm thế nào để suy ra mạch cổng logic từ bảng chân trị?

:::{hint} Có sự tương ứng một-một giữa các mạch gồm cổng AND, OR, NOT và các phương trình Đại số Boolean.

Các phép toán cơ bản (tức là các hàm nguyên thủy) của **Đại số Boolean** cũng là AND, OR, và NOT — được định nghĩa với chính xác các bảng chân trị mà chúng ta đã thấy trong [phần cổng logic](#sec-common-logic-gates-vi). Đại số Boolean là một hệ thống toán học (tức là đại số) với các định lý và phương trình. Nhờ có Đại số Boolean, chúng ta có thể áp dụng toán học để cung cấp lý thuyết cho thiết kế phần cứng, tối thiểu hóa, và nhiều hơn nữa.
:::

:::{note} Lịch sử: George Boole và Claude Shannon
:class: dropdown

Vào thế kỷ 19, nhà toán học George Boole (1815-1864), phát triển một hệ thống toán học (đại số) liên quan đến logic, sau này được gọi là **Đại số Boolean**. Các biến của ông nhận các giá trị TRUE và FALSE. Sau đó, Claude Shannon (cha đẻ của lý thuyết thông tin, 1916–2001) viết luận văn Thạc sĩ về cách ánh xạ Đại số Boolean sang mạch số.
:::

## Các Phép Toán Đại Số Boolean

:::{table} Các phép toán cơ bản của Đại số Boolean: AND, OR, và NOT. Xem [phần cổng logic](#sec-common-logic-gates-vi) để biết bảng chân trị.
:label: tab-boolean-operations-vi

| Phép toán | Ký hiệu Đại số Boolean | Ghi chú |
| :-- | :--: | :-- |
| AND(a, b) | $a \cdot b$, $ab$ | Phép toán cơ bản. Dấu $\cdot$ thường được bỏ qua, như trong ký hiệu thứ hai. |
| OR(a, b) | $a + b$ | Phép toán cơ bản. |
| NOT(a) | $\overline{a}$ | Phép toán cơ bản. |
:::

:::{warning} Đại số Boolean không có phép nhân hay phép cộng!

Trong Đại số Boolean, các phép toán $\cdot$ và $+$ **không đại diện cho phép nhân hay phép cộng các giá trị số nguyên**! Thay vào đó, các ký hiệu toán tử này đại diện cho AND và OR, tương ứng, trên các biến **Boolean** $a$ và $b$.
:::

Chúng ta cũng có thể định nghĩa các phép toán cho NAND, NOR, và XOR (mặc dù những phép này không được coi là phép toán cơ bản).

:::{table} NAND, NOR, XOR trong Đại số Boolean.
:label: tab-boolean-operations-advanced-vi

| Phép toán | Ký hiệu Đại số Boolean | Ghi chú |
| :-- | :--: | :-- |
| NAND(a, b) | $\overline{a \cdot b}$ </br>$\overline{ab}$ | "Not AND" |
| NOR(a, b) | $\overline{a + b}$ | "Not OR" |
| XOR(a, b) | $a \oplus b$ | Không phải phép toán cơ bản, nhưng đủ <br/>phổ biến để có ký hiệu riêng.[^xor-bool] |

[^xor-bool]: $a \oplus b = (a + b)\overline{ab} = a\overline{b}+\overline{a}b$
:::



## Tại Sao Sử Dụng Đại Số Boolean

Giá trị của Đại số Boolean cho thiết kế mạch đến từ việc các biểu thức Boolean có thể được thao tác theo toán học. Nói chung, _dễ hơn nhiều_ để thao tác các phương trình so với việc trực tiếp thao tác các mạch.

1. **Đơn giản hóa Mạch.** Một biểu thức Đại số Boolean có thể được đơn giản hóa. Sự đơn giản hóa này sẽ trực tiếp dẫn đến đơn giản hóa mạch. Sử dụng ít cổng hơn có nghĩa là sử dụng ít transistor hơn — rẻ hơn và tiết kiệm năng lượng hơn!

2. **Xác minh mạch.** Một công dụng khác của Đại số Boolean là xác minh mạch. Cho một mạch và một phương trình Boolean, chúng ta có thể đặt câu hỏi, "Hai biểu diễn khác nhau có đại diện cho cùng một hàm không?" Bước đầu tiên là suy ra biểu thức Boolean mới dựa trên mạch. Sau đó thông qua thao tác đại số, chúng ta có thể biến đổi các biểu thức cho đến khi chúng khớp, từ đó "chứng minh" sự tương đương của hai biểu diễn.

(sec-or-simplify-setup-vi)=
### Ví Dụ

Chúng ta có thể sử dụng Đại số Boolean để xác minh sự đơn giản hóa bên dưới. [Mạch đầu tiên](#fig-equivalent-circuit-vi) ($y = ab + a + c$), sử dụng ba cổng, **tương đương** với [mạch thứ hai](#fig-equivalent-circuit-or-vi) ($y = a + c$), chỉ sử dụng một cổng OR. Chúng ta mô tả cách tiếp cận này ở [cuối phần này](#sec-or-simplify-proof-vi).

:::::{grid} 2

::::{grid-item}
:::{figure} images/equivalent-circuit.png
:label: fig-equivalent-circuit-vi
:width: 100%
:alt: "Chuỗi của AND trên b và a vào OR với a, sau đó OR với c để ra y, tương đương với y bằng a OR c sau khi đơn giản hóa."

Mạch đầu tiên: $y = ab + a + c$.
:::
::::

::::{grid-item}
:::{figure} images/equivalent-circuit-or.png
:label: fig-equivalent-circuit-or-vi
:width: 100%
:alt: "Cổng OR hai đầu vào tương đương với đầu vào a và c và đầu ra y."

Mạch tương đương thứ hai: $y = a + c$.
:::
::::
:::::

Chúng ta sẽ xem lại ví dụ này.

## Các Định Luật của Đại Số Boolean

Cùng với Đại số Boolean là một tập hợp các định luật áp dụng cho các biểu thức Boolean. Đây là các đẳng thức đại số đơn giản được biết là đúng. Chúng ta có thể thao tác các biểu thức Boolean khác thông qua việc áp dụng liên tiếp các định luật này.

@tab-boolean-laws-vi liệt kê các định luật Boolean phổ biến quan trọng nhất.

:::{table} Các Định Luật của Đại Số Boolean.
:label: tab-boolean-laws-vi

| Dạng AND | Dạng OR | |
| :---: | :---: | :--- |
| $x \cdot y = y \cdot x$ | $x + y = y + x$ | Giao hoán |
| $(xy)z = x(yz)$ | $(x + y) + z = x + (y + z)$ | Kết hợp |
| $x \cdot 1 = x$ | $x + 0 = x$ | Đồng nhất |
| $x \cdot 0 = 0$ | $x + 1 = 1$ | Định luật về 0 và 1 |
| $xy + x = x$ | $(x + y)x = x$ | Định lý hợp nhất |
| $x(y + z) = xy + xz$ | $x + yz = (x + y)(x + z)$ | Phân phối |
| $x \cdot x = x$ | $x + x = x$ | Lũy đẳng |
| $x \cdot \overline{x} = 0$ | $x + \overline{x} = 1$ | Nghịch đảo (Bù) |
| $\overline{xy} = \overline{x} + \overline{y}$ | $\overline{x + y} = \overline{x} \cdot \overline{y}$ | Định luật DeMorgan |

:::

Một số định luật tương tự với đại số thông thường (không phải Boolean), nhưng nhiều định luật đặc thù cho Đại số Boolean và dựa vào các biến nhận chính xác **hai** giá trị chân trị: 1 (đúng) và 0 (sai).

Do tính đối xứng của Đại số Boolean, tất cả các định luật này có hai phiên bản (hai cột ở trên), một phiên bản được gọi là phiên bản đối ngẫu của phiên bản kia. Trong thực tế, điều này có nghĩa là bạn chỉ cần nhớ một phiên bản của định luật, và phiên bản kia có thể được suy ra tự nhiên.

:::{hint} Sử dụng thẻ tham khảo

Biết cách sử dụng các công thức này để đơn giản hóa và xác minh mạch, nhưng không cần thuộc lòng. Xem thẻ tham khảo CS 61C!

:::

### Chứng Minh

Phân tích các chứng minh bên dưới, được chia thành hai loại:

* Chứng minh vét cạn, tức là tạo bảng chân trị (trong đó bảng chân trị liệt kê mối quan hệ đầu vào/đầu ra cho tất cả giá trị đầu vào có thể) sau đó xác định các cột đại diện cho vế trái và/hoặc vế phải của phương trình.
* Thao tác đại số bằng cách sử dụng các định luật khác.

::::::{note} Định luật về 0, Định luật về 1
:class: dropdown
:::::{grid} 2
::::{grid-item}
:::{card}
:header: $x \cdot 0 = 0$
| $x$ | $x \cdot 0$ |
| :---: | :---: |
| 0 | 0 |
| 1 | 0 |
:::
::::
::::{grid-item}
:::{card}
:header: $x + 1 = 1$
| $x$ | $x + 1$ |
| :---: | :---: |
| 0 | 1 |
| 1 | 1 |
:::
::::
::::::
<!-- note -->

::::::{note} Phân phối
:class: dropdown

:::{card}
:header: $x(y + z) = xy + xz$
Sắp có...
:::

:::{card}
:header: $x + yz = (x + y)(x + z)$
```{math}
\begin{aligned}
    (x + y)(x + z) &= x \cdot x + xz + xy + yz && \text{Phân phối} \\
    &= x + xy + xz + yz && \text{Lũy đẳng (AND)} \\
    &= x + x(y + z) + yz && \text{Phân phối (Tính chất 1)} \\
    &= x + yz && \text{Định lý hợp nhất (AND)}
\end{aligned}
```
:::
::::::
<!-- note -->

::::::{note} Lũy đẳng
:class: dropdown
:::::{grid} 2
::::{grid-item}
:::{card}
:header: $x \cdot x = x$
| $x$ | $x \cdot x$ |
| :---: | :---: |
| 0 | 0 |
| 1 | 1 |
:::
::::
::::{grid-item}
:::{card}
:header: $x + x = x$
| $x$ | $x + x$ |
| :---: | :---: |
| 0 | 0 |
| 1 | 1 |
:::
::::
::::::
<!-- note -->

::::::{note} Định luật DeMorgan
:class: dropdown

Ở mức cao: "phân phối" phép đảo (ví dụ: đối ngẫu) qua x, y và toán tử.

:::::{grid} 2
::::{grid-item}
:::{card}
:header: $\overline{xy} = \overline{x} + \overline{y}$
| $x$ | $y$ | $xy$ | $\overline{xy}$ | $\overline{x}$ | $\overline{y}$ | $\overline{x} + \overline{y}$ |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 1 | 1 | 1 | 1 |
| 0 | 1 | 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 | 0 | 1 | 1 |
| 1 | 1 | 1 | 0 | 0 | 0 | 0 |
:::
::::
::::{grid-item}
:::{card}
:header: $\overline{x + y} = \overline{x} \cdot \overline{y}$
| $x$ | $y$ | $x + y$ | $\overline{x + y}$ | $\overline{x}$ | $\overline{y}$ | $\overline{x} \cdot \overline{y}$ |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 1 | 1 | 1 | 1 |
| 0 | 1 | 1 | 0 | 1 | 0 | 0 |
| 1 | 0 | 1 | 0 | 0 | 1 | 0 |
| 1 | 1 | 1 | 0 | 0 | 0 | 0 |
:::
::::
::::::
<!-- note -->

(sec-or-simplify-proof-vi)=
### Ví Dụ, Xem Lại

Hãy quay lại ví dụ được mô tả [ở trên](#sec-or-simplify-setup-vi) và chứng minh rằng [mạch đầu tiên](#fig-equivalent-circuit-vi) tương đương với [mạch thứ hai](#fig-equivalent-circuit-or-vi).

```{math}
\begin{aligned}
    y &= ab + a + c  && \text{Phương trình suy ra từ mạch gốc} \\
      &= a(b + 1) + c && \text{Phân phối} \\
      &= a(1) + c     && \text{Định luật về 1} \\
      &= a + c        && \text{Đồng nhất (AND)}
\end{aligned}
```
