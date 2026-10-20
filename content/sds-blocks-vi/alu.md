---
title: "Đơn Vị Số Học Logic (Arithmetic Logic Unit)"
subtitle: Bởi John Wawrzynek, chỉnh sửa bởi Lisa Yan
---

(sec-alu-vi)=
## Mục Tiêu Học Tập

* Thiết kế Đơn Vị Số Học Logic (ALU) cơ bản.
* Sử dụng mạch mux trong ALU.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/oVc3yo3cZj8
:width: 100%
:title: "[CS61C FA20] Lecture 17.2 - Combinational Logic Blocks: ALU"
:::

::::

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/QFPvVUKTCFk
:width: 100%
:title: "[CS61C FA20] Lecture 17.4 - Combinational Logic Blocks: Subtractor Design"
:::

::::

Hầu hết các triển khai bộ xử lý bao gồm một khối logic tổ hợp đặc biệt gọi là đơn vị số học logic (ALU - arithmetic logic unit). Trong RISC-V, ALU được sử dụng để tính kết quả trong các lệnh R-type, như, `add`, `sub`, `and`, hoặc `addi`, `ori`, v.v.

## Khối ALU

Chúng ta sẽ xem xét thiết kế một phiên bản ALU đơn giản hơn so với ALU trong bộ xử lý RISC-V của chúng ta. ALU của chúng ta sẽ chỉ bao gồm bốn chức năng cơ bản:

* ADD (Cộng)
* SUB (Trừ)
* AND (theo bit)
* OR (theo bit)

ALU này được triển khai như một khối logic tổ hợp trong @fig-alu-vi:

* hai đầu vào dữ liệu 32-bit rộng, `A` và `B`;
* một đầu ra dữ liệu 32-bit rộng, `R`; và
* một đầu vào điều khiển 2-bit rộng, `S`

:::{figure} images/alu.png
:label: fig-alu-vi
:width: 55%
:alt: "Ký hiệu khối ALU cơ bản với đầu vào 32-bit A và B, điều khiển 2-bit S, và kết quả 32-bit R. Ký hiệu ALU hộp đen có thể bao gồm các phép toán như add, subtract, and, or."

ALU cơ bản: ADD, SUB, AND, và OR
:::

:::{hint} Đầu vào S của ALU là gì?

S là bộ chọn và được sử dụng để chỉ ra phép toán nào trong bốn phép toán khả thi mà ALU sẽ thực hiện (xem [phần trước](#sec-mux-vi) về mux). Thông thường, đầu vào S sẽ được điều khiển bởi bộ xử lý dựa trên phép toán của lệnh hiện đang thực thi trên bộ xử lý (với RISC-V R-type, các trường `func7` và `func3`).

:::

Trong ALU cơ bản của chúng ta, S được sử dụng để chọn R là một trong bốn phép toán:

```{math}
:label: eq-alu-vi
:enumerated: true

\texttt{R} = 
\begin{cases}
\texttt{A + B} & \text{khi } \texttt{S} = 00 \\
\texttt{A - B} & \text{khi } \texttt{S} = 01 \\
\texttt{A \& B} & \text{khi } \texttt{S} = 10 \\
\texttt{A | B} & \text{khi } \texttt{S} = 11 \\
\end{cases}
```

## Mạch ALU

Thiết kế bên trong của ALU đơn giản của chúng ta được hiển thị trong @fig-alu-circuit-vi:

:::{figure} images/simple-alu.png
:label: fig-alu-circuit-vi
:width: 100%
:alt: "Datapath ALU bên trong hiển thị các khối AND, OR, và cộng-trừ song song. Mux 4-to-1 chọn kết quả 32-bit cuối cùng từ các đầu ra của các khối logic song song."

Mạch ALU cơ bản với ba khối (AND, OR, cộng/trừ) và mux 4-to-1.
:::

Cho ALU đơn giản của chúng ta, chúng ta sẽ cần một khối cộng/trừ, một khối AND, và một khối OR. Mỗi khối này sẽ nhận hai đầu vào 32-bit và tạo ra một đầu ra 32-bit. Đọc thêm về việc triển khai các khối này [bên dưới](#sec-alu-internal-vi).

:::{note} Giải Thích Thêm
:class: dropdown

Bit cao của `S` ($s_1$) và bit thấp của `S` ($s_0$) được sử dụng để triển khai Phương trình @eq-alu-vi:

* $s_1=0$: Khối cộng/trừ
  * $s_0=0$: Chỉ định cộng; lưu ý rằng $s_0$ do đó được sử dụng cho cả MUX và khối cộng/trừ.
  * $s_0=1$: Chỉ định trừ
* $s_1=1$: Một trong các khối AND hoặc OR
  * $s_0=0$: Chỉ định AND
  * $s_0=1$: Chỉ định OR
:::

:::{warning} Trong ALU, tất cả kết quả đều được tính nhưng chỉ một được chọn.

Cách phổ biến để triển khai ALU là cung cấp một instance của khối logic tổ hợp cho mỗi chức năng ALU khả thi. Các đầu vào, A và B, được phân phối đến tất cả các khối, và đầu ra của khối thích hợp được chọn bằng mux.

Trong cấu hình này, mọi chức năng của ALU đều được tính toán bên trong ALU ở mỗi chu kỳ, nhưng chỉ một trong các kết quả được gửi đến đầu ra.

:::

(sec-alu-internal-vi)=
### Triển Khai Các Khối Bên Trong

Các phép toán logic như được định nghĩa bởi RISC-V ISA là các phép toán theo bit.

* AND: $r_i$, là $a_i$ `&` $b_i$, cho các bit thứ $i$ của đầu ra `R`, `A`, và `B`, tương ứng. Thực hiện phép toán theo bit này như một tập hợp 32 cổng AND, trong đó mỗi cổng AND chịu trách nhiệm cho một trong 32 bit kết quả.
* OR: Tương tự, khối OR là một tập hợp 32 cổng OR.
* Khối cộng/trừ là khối phức tạp hơn đáng kể so với các khối AND hoặc OR; thiết kế của nó là chủ đề của [phần tiếp theo](#sec-adder-subtractor-vi). Hiện tại, chúng ta lưu ý rằng mạch trừ rất giống với bộ cộng, do đó tại sao chúng ta cung cấp một mạch đơn có khả năng thực hiện cả hai phép toán.
