---
title: "Bộ Ghép Kênh Dữ Liệu (Data Multiplexors)"
subtitle: Bởi John Wawrzynek, chỉnh sửa bởi Lisa Yan
---

(sec-mux-vi)=
## Mục Tiêu Học Tập

* Vẽ mạch mux n-bit rộng, k-to-1.
* Giải thích cách mux sử dụng tín hiệu điều khiển để chọn đầu ra từ tập các đầu vào dữ liệu.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/iaJVhHQN0ys
:width: 100%
:title: "[CS61C FA20] Lecture 17.1 - Combinational Logic Blocks: Data Multiplexors"
:::

::::

Lần trước chúng ta đã thấy cách biểu diễn và thiết kế **các khối logic tổ hợp**. Trong phần này chúng ta sẽ nghiên cứu một số khối logic đặc biệt: bộ ghép kênh dữ liệu, mạch cộng/trừ, và đơn vị số học/logic.

## Mux

Bộ **ghép kênh dữ liệu** (data **multiplexor**), thường gọi là **mux** hoặc **bộ chọn** (selector), là mạch chọn giá trị đầu ra từ tập các giá trị đầu vào. Dưới đây là hai mạch mux.

:::::{grid} 2

::::{grid-item}
:::{figure} images/mux-2.png
:label: fig-mux-2-vi
:width: 50%
:alt: "Ký hiệu bộ ghép kênh 2-to-1 1-bit với đầu vào dữ liệu a và b, đầu vào chọn s, và đầu ra y."

MUX 1-bit rộng, 2-to-1.
:::
::::
::::{grid-item}
:::{figure} images/mux-n.png
:label: fig-mux-n-vi
:width: 57%
:alt: "Ký hiệu bộ ghép kênh 2-to-1 n-bit với đầu vào n-bit A và B, select 1-bit s, và đầu ra n-bit Y."

MUX n-bit rộng, 2-to-1.
:::
:::::
<!-- end grid -->

Cả hai mux này đều có hai đầu vào **dữ liệu** và một đầu ra. Ngoài ra, mỗi mux có một **tín hiệu điều khiển** đặc biệt được gắn nhãn s, viết tắt của **select** (chọn). Tín hiệu s cũng là đầu vào, nhưng nó được dùng để điều khiển giá trị đầu vào nào trong hai giá trị được chuyển đến đầu ra.

@fig-mux-2-vi cho thấy mạch mux **1-bit rộng**, **2-to-1**:

* 2-to-1 vì nó nhận hai đầu vào dữ liệu `a` và `b` và xuất ra một trong số chúng.
* Nó rộng 1 bit vì tất cả tín hiệu dữ liệu (`a` và `b`) đều có độ rộng 1 bit.
* Tuy nhiên, lưu ý rằng tín hiệu `s` chỉ rộng một bit. Điều này vì nó phải chọn giữa 2 đầu vào.

@fig-mux-n-vi cho thấy mạch mux **n-bit rộng**, **2-to-1**:

* 2-to-1 vì nó nhận hai đầu vào dữ liệu `A` và `B` và xuất ra một trong số chúng.
* Nó rộng n bit vì tất cả tín hiệu dữ liệu (`A` và `B`) đều có độ rộng n bit.
* Tín hiệu `s` vẫn chỉ rộng một bit vì nó phải chọn giữa 2 đầu vào.

Chức năng của, ví dụ, [mux 2-to-1 1-bit rộng](#fig-mux-2-vi) có thể được mô tả với hai quy tắc:

```{math}
\texttt{y} = 
\begin{cases}
\texttt{a} & \text{khi } \texttt{s} = 0 \\
\texttt{b} & \text{khi } \texttt{s} = 1 \\
\end{cases}
```

Để nhắc nhở chúng ta giá trị nào của s tương ứng với đầu vào nào, trong ký hiệu mux chúng ta thường gắn nhãn mỗi đầu vào với giá trị s tương ứng của nó.

:::{hint} Sử dụng mux để chọn dữ liệu!
Mux được sử dụng bất cứ khi nào một mạch phải chọn dữ liệu từ nhiều nguồn.

MUX N-to-1 n-bit rộng có N đầu vào dữ liệu, 1 đầu vào điều khiển, và 1 đầu ra. Đầu vào điều khiển S chọn giữa N đầu vào khác.

:::

:::{tip} Kiểm Tra Nhanh: Điền vào chỗ trống

> MUX 4-to-1 32-bit rộng chọn giữa `__(1)__` tín hiệu đầu vào, mỗi tín hiệu có độ rộng `__(2)__` bit. Mạch mux này có `__(3)__` bit chọn.

:::

::::{note} Hiện Đáp Án
:class: dropdown

:::{figure} images/mux-4.png
:label: fig-mux-4-vi
:width: 30%
:alt: "Khối mux 4-to-1 32-bit với bốn đầu vào dữ liệu 32-bit A, B, C, D, select 2-bit S, và một đầu ra n-bit Y."

Mạch mux 4-to-1 32-bit.
:::

1. 4 tín hiệu đầu vào
2. 32 bit rộng
3. 2 bit chọn

::::

Mux được sử dụng phổ biến trong thiết kế vi xử lý, ví dụ, những vi xử lý triển khai RISC-V.

## MUX: Triển Khai

Trong hầu hết các ứng dụng, bạn sẽ có sẵn mux; bạn sẽ không cần xây dựng từ đầu. Tuy nhiên, tốt để nhớ rằng giống như tất cả các khối logic tổ hợp, chức năng của mux có thể được mô tả bằng bảng chân lý và do đó được triển khai như một mạch cổng logic.

### MUX 2-to-1 1-bit rộng

Làm thế nào để triển khai mux trong @fig-mux-2-vi, đã thảo luận ở trên?

Mạch này có thể được biểu diễn bằng đại số Boolean:

$$ y = \overline{s} a + s b $$

Mở rộng các mục dropdown bên dưới để xem sơ đồ cổng và suy ra biểu thức trên.

::::{note} Hiện Sơ Đồ Cổng
:class: dropdown

@fig-mux-2-circuit-vi cho thấy sơ đồ cổng cho mux 2-to-1 1-bit rộng với đầu vào `a` và `b` và đầu ra `c`. Bên dưới, $c = \overline{s} a + s b$.

:::{figure} images/mux-2-circuit.png
:label: fig-mux-2-circuit-vi
:width: 100%
:alt: "Mux 2-to-1 1-bit mức cổng triển khai c bằng (not s and a) OR (s and b)."

Sơ đồ cổng cho mux 2-to-1 1-bit rộng.
:::
::::
<!-- ::::: -->

::::{note} Hiện Bảng Chân Lý
:class: dropdown

:::{table} Bảng chân lý cho mux 2-to-1 1-bit trong @fig-mux-2-vi. Lưu ý vì có ba đầu vào 1-bit (`a`, `b`, và điều khiển `s`), bảng chân lý có $2^3 = 8$ hàng.
:label: tab-mux-2-vi

| s | ab | y |
| :--: | :--: | :--: |
| 0 | 00 | 0 |
| 0 | 01 | 0 |
| 0 | 10 | 1 |
| 0 | 11 | 1 |
| 1 | 00 | 0 |
| 1 | 01 | 1 |
| 1 | 10 | 0 |
| 1 | 11 | 1 |

:::

::::

::::{note} Hiện Giải Thích Đại Số Boolean
:class: dropdown

Để đưa ra phương trình logic và sơ đồ mạch mức cổng liên quan, chúng ta có thể áp dụng kỹ thuật đã học [chương trước](#sec-boolean-algebra). Chúng ta viết dạng chuẩn tổng các tích và đơn giản hóa qua thao tác đại số:

```{math}
\begin{aligned}
     c 
     &= \overline{s} a \overline{b} + \overline{s} a b + s \overline{a} b + sab && \text{Tổng các Tích} \\
     &= \overline{s} (a \overline{b} + ab) + s (\overline{a} b + ab) && \text{Tính chất Phân phối} \\
     &= \overline{s} (a (\overline{b} + b)) + s ((\overline{a} + a) b) && \text{Tính chất Phân phối} \\
     &= \overline{s} (a \cdot 1) + s (1 \cdot b) && \text{Nghịch đảo (OR)} \\
     &= \overline{s} a + sb && \text{Đồng nhất (AND)} \\
\end{aligned}
```

Trực giác, kết quả này có nghĩa; Khi đầu vào điều khiển, `s`, là `0`, biểu thức bên phải của phương trình rút gọn thành `a`, và khi nó là `1`, biểu thức rút gọn thành `b`.
::::

### Mux 4-to-1 1-bit rộng

Thường xuyên chúng ta cần mở rộng số lượng đầu vào dữ liệu của bộ ghép kênh. Ví dụ xem xét bộ ghép kênh 4-to-1 trong @fig-mux-4-bits-vi:

:::{figure} images/mux-4-bits.png
:label: fig-mux-4-bits-vi
:width: 55%
:alt: "Ký hiệu mux 4-to-1 1-bit với đầu vào a đến d, bit chọn s1 s0, và đầu ra 1-bit e."

MUX 4-to-1 1-bit rộng.
:::

@fig-mux-4-block-vi cho thấy cách mux lớn hơn này có thể được tạo thành bằng cách nối các MUX nhỏ hơn với nhau.

:::{figure} images/mux-4-block.png
:label: fig-mux-4-block-vi
:width: 60%
:alt: "Mux 4-to-1 được xây dựng từ ba mux 2-to-1. Ban đầu, hai mux 2-to-1 được chọn bằng bit chọn s0, và các đầu ra đưa vào mux 2-to-1 thứ ba sử dụng bit chọn s1 để có đầu ra cuối cùng e."

Sơ đồ mạch bộ ghép kênh (MUX) 4-to-1.
:::

Thiết kế mạch này tận dụng bản chất phân cấp của việc ghép kênh. Lớp mux đầu tiên sử dụng đầu vào $s_0$ để thu hẹp bốn đầu vào xuống còn hai, sau đó lớp thứ hai sử dụng $s_1$ để chọn đầu ra cuối cùng.

::::{note} Hiện Cách Tiếp Cận Đại Số Boolean
:class: dropdown

```{math}
\texttt{e} = 
\begin{cases}
\texttt{a} & \text{khi } \texttt{S} = 00 \\
\texttt{b} & \text{khi } \texttt{S} = 01 \\
\texttt{c} & \text{khi } \texttt{S} = 10 \\
\texttt{d} & \text{khi } \texttt{S} = 11 \\
\end{cases}
```

Một cách tiếp cận thay thế có thể bắt đầu bằng cách liệt kê bảng chân lý—trong trường hợp này hàm có 4 đầu vào dữ liệu một bit và một đầu vào điều khiển 2-bit rộng, tổng cộng 6 đầu vào một bit. Bảng chân lý sẽ có $2^6 = 64$ hàng. Chắc chắn là cách tiếp cận khả thi. Nếu chúng ta làm điều này, chúng ta sẽ có phương trình logic sau:

$$e = \overline{s_1 s_0} a
     + \overline{s_1} s_0 b
     + s_1 \overline{s_0} c
     + s_1 s_0 d$$

::::
