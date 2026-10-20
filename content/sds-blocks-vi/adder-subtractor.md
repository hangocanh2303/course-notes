---
title: "Bộ Cộng/Trừ (Adder/Subtractor)"
subtitle: Bởi John Wawrzynek, chỉnh sửa bởi Lisa Yan
---

(sec-adder-subtractor-vi)=
## Mục Tiêu Học Tập

* Triển khai bộ cộng n-bit dưới dạng chuỗi các bộ cộng 1-bit.
* Nhớ rằng XOR là bộ đảo có điều kiện và có thể được sử dụng với SUB để mở rộng khối cộng thành khối cộng/trừ.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/NkgFywLV65U
:width: 100%
:title: "[CS61C FA20] Lecture 17.3 - Combinational Logic Blocks: Adder/Subtractor"
:::

:::{iframe} https://www.youtube.com/embed/QFPvVUKTCFk
:width: 100%
:title: "[CS61C FA20] Lecture 17.4 - Combinational Logic Blocks: Subtractor Design"
:::

::::

@fig-add-sub-block-vi cho thấy khối cộng/trừ được sử dụng trong [ALU cơ bản](#sec-alu-vi) của chúng ta:

:::{figure} images/adder-sub-block.png
:label: fig-add-sub-block-vi
:width: 55%
:alt: "Khối cộng-trừ với đầu vào 32-bit A và B, tín hiệu điều khiển SUB, đầu ra 32-bit Y, và cờ overflow."

Khối Cộng/Trừ.
:::

Mạch đơn này có khả năng thực hiện cả phép cộng và phép trừ:

* hai đầu vào dữ liệu 32-bit rộng `A` và `B`
* một đầu vào điều khiển 1-bit rộng `SUB`.
* một đầu ra dữ liệu 32-bit rộng `Y`
* một đầu ra 1-bit rộng `overflow`.

Khi SUB=1, mạch thực hiện phép trừ `A - B`. Khi SUB=0 mạch thực hiện phép cộng `A + B`.

Đầu ra "đặc biệt", được gắn nhãn **overflow** báo hiệu [tràn số nguyên](#sec-integer-overflow). Khi thực hiện phép cộng hoặc trừ, đầu ra này sẽ là 1 nếu kết quả quá lớn để vừa trong 32 bit. Chúng ta sẽ quay lại điều này [sau](#sec-adder-overflow-vi).

Để thiết kế mạch cộng/trừ, chúng ta sẽ bắt đầu bằng cách nghiên cứu thiết kế mạch cộng, sau đó mở rộng để cũng thực hiện phép trừ.

## Trực Giác Về Chuỗi (Cascading)

Một phương pháp strawman[^strawman] để đạt được mạch mức cổng chi tiết cho bộ cộng sẽ là theo quy trình chúng ta đã học ở chương trước[^cl-practice]. Thật không may, kỹ thuật đó chỉ hiệu quả cho các bộ cộng rất hẹp (ví dụ, 1-bit hoặc 2-bit rộng), vì kích thước của bảng chân lý quá lớn cho các bộ cộng rộng hơn.

Do đó, chúng ta chuyển sang **thiết kế theo module**: Tìm cách chia thiết kế thành các phần nhỏ hơn dễ quản lý hơn. Chúng ta sẽ thiết kế các phần nhỏ hơn riêng lẻ, sau đó nối chúng với nhau để tạo ra bộ cộng rộng hoàn chỉnh.

[^strawman]: Wikipedia: [Straw Man](https://en.wikipedia.org/wiki/Straw_man)
[^cl-practice]: Như đã thảo luận trong [chương trước](#sec-cl-practice): Bắt đầu với bảng chân lý, viết phương trình Boolean chuẩn, sau đó đơn giản hóa.

:::{warning} Chuỗi các bộ cộng 1-bit
Thay vì tính dạng chuẩn cho bảng chân lý, phân rã bộ cộng n-bit thành một **chuỗi** các bộ cộng 1-bit.

**Phép cộng thủ công** cho chúng ta hướng dẫn tốt về cách chia bộ cộng thành các phần.
:::

Xem xét việc cộng hai số 4-bit `A`= a{sub}`3`a{sub}`2`a{sub}`1`a{sub}`0` và `B`= b{sub}`3`b{sub}`2`b{sub}`1`b{sub}`0`:

```{math}
:label: eq-add-4-vi
:enumerated: true
\begin{array}{c c c c c}
    & \texttt{c}_{\texttt{3}} & \texttt{c}_{\texttt{2}} & \texttt{c}_{\texttt{1}} & \\
    & \texttt{a}_{\texttt{3}} & \texttt{a}_{\texttt{2}} & \texttt{a}_{\texttt{1}} & \texttt{a}_{\texttt{0}} \\
    + & \texttt{b}_{\texttt{3}} & \texttt{b}_{\texttt{2}} & \texttt{b}_{\texttt{1}} & \texttt{b}_{\texttt{0}} \\
    \hline
    & \texttt{y}_{\texttt{3}} & \texttt{y}_{\texttt{2}} & \texttt{y}_{\texttt{1}} & \texttt{y}_{\texttt{0}}
\end{array}
```

Ví dụ, với `A` là `1011` và `B` là `0001`:

```{math}
\begin{array}{c c c c c}
    & \texttt{0} & \texttt{1} & \texttt{1} & \\
    & \texttt{1} & \texttt{0} & \texttt{1} & \texttt{1} \\
    + & \texttt{0} & \texttt{0} & \texttt{0} & \texttt{1} \\
    \hline
    & \texttt{1} & \texttt{1} & \texttt{0} & \texttt{0}
\end{array}
```

:::{note} Giải Thích

Khi chúng ta cộng bằng tay:

* Bắt đầu bằng cách cộng hai bit ít quan trọng nhất của A và B, a{sub}`0` và b{sub}`0`, tương ứng (gọi đây là vị trí thứ không). Từ phép cộng đó, chúng ta tạo ra **bit kết quả**, y{sub}`0`, và có thể là bit **carry out** (nhớ ra) c{sub}`1`.
* Sau đó chúng ta di chuyển sang cột quan trọng hơn tiếp theo (vị trí thứ nhất), cộng bit **carry in** (nhớ vào) c{sub}`1` từ giai đoạn trước cùng với hai bit tiếp theo của `A` và `B`, a{sub}`1` và b{sub}`1`, tương ứng.
* Chúng ta tiếp tục quy trình bằng cách di chuyển sang trái một cột mỗi lần cho đến khi hoàn thành tất cả n cột. Trong Phương trình @eq-add-4-vi, n là 4.
:::

## Bộ Cộng 1-bit

Phép cộng ở vị trí thứ i tạo ra hai đầu ra:

* **Bit kết quả**, tức là, y{sub}`i`
* **Bit nhớ** (ví dụ, c{sub}`i+1` đưa vào phép cộng ở vị trí thứ (i+1). Chúng ta đặt tên bit nhớ theo giai đoạn mà nó là đầu vào.

### Vị trí thứ không

Chúng ta xây dựng trực giác để triển khai bộ cộng 1-bit bằng cách xem xét bộ cộng ở vị trí thứ không (giai đoạn ít quan trọng nhất). Điều này tính toán bit kết quả ít quan trọng nhất, $\texttt{y}_{\texttt{0}}$, và bit **carry-out** của giai đoạn ít quan trọng nhất, $\texttt{c}_{\texttt{1}}$.

:::{figure} images/adder-1-zero.png
:label: fig-adder-1-zero-vi
:width: 70%
:alt: "Khối bộ cộng 1-bit minh họa phép cộng theo bit đơn giữa a0 và b0, tạo ra đầu ra y0 và bit nhớ c1 được bao gồm trong phép cộng bit đơn tiếp theo. Bên trái mô tả phép cộng 4-bit đầy đủ giữa 4-bit a và 4-bit b, kết quả là 4-bit y. Không có bit carry-in trong phép cộng bit đơn đầu tiên này giữa a0 và b0."

Bộ cộng 1-bit ở vị trí thứ không, _không có_ bit carry-in.
:::

Bằng cách kiểm tra bảng chân lý trong @tab-adder-1-zero-vi, chúng ta có thể trực tiếp viết các phương trình logic cho giai đoạn này[^xor-and].

[^xor-and]: `^`: XOR, `&`: AND

:::::{grid} 2
::::{grid-item}
:::{table} Bảng chân lý cho bộ cộng 1-bit ở vị trí thứ không.
:label: tab-adder-1-zero-vi

| $\texttt{a}_{\texttt{0}}$ | $\texttt{b}_{\texttt{0}}$ | $\texttt{y}_{\texttt{0}}$ | $\texttt{c}_{\texttt{1}}$ |
| :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

:::
::::
::::{grid-item}
```{math}
\begin{aligned}
\\ \\ \\
    \texttt{s}_{\texttt{0}} &= \texttt{a}_{\texttt{0}} \texttt{ \^ \ } \texttt{b}_{\texttt{0}} \\
    \texttt{c}_{\texttt{1}} &= \texttt{a}_{\texttt{0}} \texttt{ \& } \texttt{b}_{\texttt{0}}
\end{aligned}
```
::::
:::::
<!-- end grid -->

### Vị trí thứ nhất

Tiếp theo, xem xét logic cho giai đoạn quan trọng hơn tiếp theo (và tất cả các giai đoạn tiếp theo):

:::{figure} images/adder-1-one.png
:label: fig-adder-1-one-vi
:width: 70%
:alt: "Phép cộng bit đơn giữa bit a1 và b1 với bit nhớ c1 từ phép cộng trước, và tạo ra bit tổng y1 và bit carry-out thứ hai c2."

Bộ cộng 1-bit ở vị trí thứ nhất, _có_ bit carry-in.
:::

Mặc dù bảng chân lý lớn hơn, các phương trình logic vẫn đơn giản để viết—ít nhất, nếu chúng ta nhớ những gì đã học từ các phần trước.

:::::{grid} 2
::::{grid-item}
:::{table} Bảng chân lý cho bộ cộng 1-bit ở vị trí thứ nhất.
:label: tab-adder-1-one-vi

| $\texttt{a}_{\texttt{1}}$ | $\texttt{b}_{\texttt{1}}$ | $\texttt{c}_{\texttt{1}}$ | $\texttt{y}_{\texttt{1}}$ | $\texttt{c}_{\texttt{2}}$ |
| :---: | :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 0 |
| 0 | 1 | 0 | 1 | 0 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 | 0 |
| 1 | 0 | 1 | 0 | 1 |
| 1 | 1 | 0 | 0 | 1 |
| 1 | 1 | 1 | 1 | 1 |

:::
::::
::::{grid-item}
```{math}
\begin{aligned}
\\ \\ \\ \\ \\ 
\begin{aligned}
    \texttt{y}_{\texttt{1}} &= \texttt{XOR}(\texttt{a}_{\texttt{1}}, \texttt{b}_{\texttt{1}}, \texttt{c}_{\texttt{1}}) \\
    \texttt{c}_{\texttt{2}} &= \texttt{MAJ}(\texttt{a}_{\texttt{1}}, \texttt{b}_{\texttt{1}}, \texttt{c}_{\texttt{1}}) \\
    &= \texttt{a}_{\texttt{1}}\texttt{b}_{\texttt{1}} + \texttt{a}_{\texttt{1}}\texttt{c}_{\texttt{1}} + \texttt{b}_{\texttt{1}}\texttt{c}_{\texttt{1}}
\end{aligned}
\end{aligned}
```
::::
:::::
<!-- end grid -->

:::{tip} Tại sao bảng chân lý có 8 hàng?
:class: dropdown

Bảng chân lý bây giờ có 8 hàng, vì có ba đầu vào vào giai đoạn này:

* một bit từ mỗi A và B, cùng với
* bit **carry-in** cho giai đoạn hiện tại, cũng là carry-out từ giai đoạn trước.
:::

:::{tip} Tại sao XOR cho bit kết quả, và MAJ (Majority - Đa số) cho bit nhớ?
:class: dropdown

Bằng cách kiểm tra cẩn thận bảng chân lý bạn sẽ nhận thấy:

* **Bit kết quả** $\texttt{y}_{\texttt{1}}$: Đầu ra tổng là **XOR của ba đầu vào**. Nhớ rằng [XOR N-bit](#sec-n-xor) là 1 khi số lượng 1 trong đầu vào là lẻ.
* **Bit carry-out** $\texttt{c}_{\texttt{2}}$: Hàm carry-out là **hàm majority (đa số)**. Nhớ rằng [mạch majority 3 đường](#sec-majority-circuit) là 1 khi số lượng 1 trong đầu vào lớn hơn số lượng 0.
:::

### Trường hợp tổng quát

Cuối cùng hãy xem xét logic cho vị trí thứ i, với $i = 0, ..., n-1$ cho phép cộng $n$-bit rộng. Chúng ta sẽ đóng gói các hoạt động của một giai đoạn, hoặc cột, của phép cộng vào một khối nhỏ, như trong @fig-adder-1-i-vi:

:::{figure} images/adder-1-i.png
:label: fig-adder-1-i-vi
:width: 70%
:alt: "Mô tả tổng quát của khối full-adder thứ i với đầu vào ai, bi, và bit nhớ ci, cũng như đầu ra yi và c_i+1. Điều này cho thấy bước bộ cộng bit đơn trung gian trong phép cộng n-bit với n-bit."

Bộ cộng 1-bit ở vị trí thứ i, _có_ bit carry-in.
:::

Bạn có thể nghĩ khối này như một **bộ cộng 1-bit**. Tên chính thức của nó là full-adder cell (ô cộng đầy đủ), nhưng hầu hết mọi người nhầm lẫn điều này với bộ cộng n-bit, mà chắc chắn nó không phải.

Sơ đồ mạch mức cổng cho bên trong bộ cộng 1-bit được hiển thị trong @fig-adder-1-i-circuit-vi. Phương trình logic có thể được pattern-match từ thảo luận trước đó và là Phương trình @eq-adder-1-i-vi.

:::::{grid} 2
::::{grid-item}
:::{figure} images/adder-1-i-circuit.png
:label: fig-adder-1-i-circuit-vi
:width: 100%
:alt: "Triển khai full-adder mức cổng sử dụng XOR cho bit tổng và logic tổ hợp với 3 cổng AND và một cổng OR để tính bit carry-out."

Mạch bộ cộng 1-bit.
:::
::::
::::{grid-item}
```{math}
:label: eq-adder-1-i-vi
:enumerated: true

\begin{aligned}
\\ \\ \\
\begin{aligned}
    \texttt{y}_{\texttt{i}} &= \texttt{XOR}(\texttt{a}_{\texttt{i}}, \texttt{b}_{\texttt{i}}, \texttt{c}_{\texttt{i}}) \\
    \texttt{c}_{\texttt{i+1}} &= \texttt{MAJ}(\texttt{a}_{\texttt{i}}, \texttt{b}_{\texttt{i}}, \texttt{c}_{\texttt{i}}) \\
    &= \texttt{a}_{\texttt{i}}\texttt{b}_{\texttt{i}} + \texttt{a}_{\texttt{i}}\texttt{c}_{\texttt{i}} + \texttt{b}_{\texttt{i}}\texttt{c}_{\texttt{i}}
\end{aligned}
\\ \\ \\
\end{aligned}
```
::::
:::::

:::{warning} Còn vị trí thứ không thì sao?
Chúng ta sẽ không bận tâm tạo khối đặc biệt cho cột đầu tiên của bộ cộng, vì bất cứ điều gì nó có thể làm thì khối bộ cộng 1-bit cũng có thể làm được. Chúng ta chỉ cần tìm ra nơi để gắn carry-in (tức là, $\texttt{c}_{\texttt{0}}$). Chi tiết hơn sau.
:::

## Bộ Cộng N-bit

Bước tiếp theo trong thiết kế mạch cộng của chúng ta là nối một tập hợp các bộ cộng 1-bit với nhau để tạo ra **bộ cộng n-bit**. Tất cả những gì chúng ta cần làm là nối đầu ra carry-out của một giai đoạn vào đầu vào carry-in của giai đoạn tiếp theo, từ ít quan trọng nhất đến quan trọng nhất, như trong @fig-cascade-add-vi:

:::{figure} images/cascade-add.png
:label: fig-cascade-add-vi
:width: 100%
:alt: "Bộ cộng ripple-carry n-bit được tạo bằng cách xếp chuỗi các bộ cộng 1-bit từ bit 0 đến bit n-1, với bit carry-in ban đầu c0 được nối đất."

Bộ cộng chuỗi n-bit (Chưa hoàn chỉnh). Bit carry-in c{sub}`0` được đặt là GND, nhưng bit carry-out c{sub}`n` hiện chưa được sử dụng.
:::

Đầu vào được áp dụng ở phía trên và _sau một độ trễ nào đó_, liên quan đến độ trễ qua các cổng logic của các khối, và độ trễ của carry từ giai đoạn này sang giai đoạn khác, kết quả cuối cùng xuất hiện ở phía dưới.

**Làm gì với bit carry-in c{sub}`0`?** Để bộ cộng tạo ra kết quả đúng, hãy tự thuyết phục rằng c{sub}`0` phải được kết nối với nguồn logic 0 (GND trong mạch). Nhưng chúng ta sẽ tìm điều gì đó thú vị hơn để làm với c{sub}`0` khi thảo luận về [bộ trừ](#sec-subtractor-vi) bên dưới.

**Làm gì với bit carry-out c{sub}`n`?** Carry-out từ giai đoạn quan trọng nhất, c{sub}`n`, có thể được sử dụng như bit thứ (n+1) của kết quả (nhớ rằng, cộng hai số n-bit có thể cho ra kết quả (n+1)-bit). Tuy nhiên, trong hầu hết các ứng dụng cho bộ cộng, chúng ta phải tạo ra kết quả n bit. Ví dụ, thanh ghi RV32I chỉ có thể lưu trữ 32 bit, nên kết quả của lệnh add phải là số 32-bit. Trong trường hợp này, chúng ta phải báo hiệu **overflow** (tràn), mà chúng ta thảo luận tiếp theo.

(sec-adder-overflow-vi)=
### Tràn (Overflow)

Như đã thảo luận trong [thiết kế cộng/trừ ban đầu](#fig-add-sub-block-vi), đầu ra overflow phải được tạo ra để chỉ ra khi kết quả không thể vừa trong n bit. Nhớ lại từ [phần trước](#sec-integer-overflow) rằng overflow có thể xảy ra khi cộng một cặp số âm hoặc khi cộng một cặp số dương.

Chúng ta có thể đưa ra các quan sát sau về phép cộng trong mạch này. Nhớ rằng, giai đoạn quan trọng nhất là giai đoạn liên quan đến bit dấu, tức là, giai đoạn n-1.

* Nếu có **carry in** vào giai đoạn quan trọng nhất, **nhưng không có carry out** từ giai đoạn đó, thì A và B đều _dương_ và kết quả của phép cộng **tràn**, tạo ra một số 1 sai ở vị trí bit dấu.
* Nếu có **carry out** từ giai đoạn quan trọng nhất, **nhưng không có carry in** vào giai đoạn đó, thì A và B đều _âm_ và kết quả của phép cộng **tràn**.
* Trong tất cả các trường hợp khác, giá trị của carry in vào giai đoạn quan trọng nhất khớp với carry out, và **không có overflow**.

Chúng ta minh họa trực giác này trong @fig-2bit-add-overflow-table-vi cho bộ cộng 2-bit.

:::{figure} images/2bit-add-overflow-table.png
:label: fig-2bit-add-overflow-table-vi
:width: 70%
:alt: "Sơ đồ bảng hiển thị các đầu ra khác nhau từ phép cộng 2-bit với các bit carry khác nhau và diễn giải có dấu, làm nổi bật các trường hợp được sử dụng để suy luận về overflow."

Sơ đồ bảng overflow bộ cộng 2-bit. Ba phép cộng được đánh dấu bị tràn và tạo ra kết quả không chính xác.
:::

::::{note} Hiện Giải Thích của @fig-2bit-add-overflow-table-vi
:class: dropdown

@fig-2bit-add-overflow-overview-vi cung cấp ý tưởng tổng quát về phép cộng với các số 2-bit `A` và `B`. Hãy ghi nhớ bánh xe số nguyên bên trái khi cộng các số nguyên.

:::{figure} images/2bit-add-overflow-overview.png
:label: fig-2bit-add-overflow-overview-vi
:width: 100%
:alt: "Bên trái hiển thị kết quả cộng có dấu 2-bit trên bánh xe giá trị, bao gồm hành vi wraparound. Bên phải hiển thị bảng cộng có dấu 2-bit tương tự cho thấy các khả năng đầu ra khác nhau và diễn giải của chúng ở dạng thập phân."

Sơ đồ bảng overflow bộ cộng 2-bit tổng quát.
:::

Ba kết quả overflow được đóng khung trong @fig-2bit-add-overflow-table-vi:

* $1 + 1 = -2$, không phải $2$
* $-2 + (-2) = 0$, không phải $-4$
* $-1 + (-2) = 1$, không phải $-3$

@fig-2bit-add-overflow-table-vi gốc hiển thị các mẫu bit của ba phép cộng này (trong số những phép khác), dẫn đến ba quan sát trong phần này.

Lưu ý rằng vẫn được phép cộng các toán hạng có dấu khác nhau. Ví dụ, $1 + (-2) = -1$ và $0 + 1 = 1$ không tạo ra overflow.

::::

Dựa trên các quan sát này, chúng ta có thể thiết kế một mạch đơn giản tạo ra tín hiệu đầu ra overflow bằng cách **so sánh** carry-in và carry-out của giai đoạn n-1. Một mạch đơn giản chỉ ra khi hai tín hiệu có giá trị khác nhau là **cổng XOR**.

$$ \texttt{overflow} = \texttt{c}_{\texttt{n}} XOR \texttt{c}_{\texttt{n-1}} $$

@fig-cascade-add-overflow-vi hiển thị bộ cộng n-bit đầy đủ của chúng ta:

:::{figure} images/cascade-add-overflow.png
:label: fig-cascade-add-overflow-vi
:width: 100%
:alt: "Bộ cộng ripple-carry hoàn chỉnh được biểu diễn với các khối bộ cộng bit đơn tuần tự cho các bit a0 và b0 đến a_n-1 và b_n-1. Phát hiện overflow được tính sau logic bộ cộng cho bit a_n-1 và b_n-1 sử dụng cổng XOR với đầu vào c_n và c_n-1 từ trước khối bộ cộng cuối cùng."

Bộ cộng chuỗi n-bit với overflow.
:::

(sec-subtractor-vi)=
## Bộ Trừ (Subtractor)

Chúng ta đã đề cập trước đó rằng phép cộng và phép trừ có liên quan chặt chẽ, và do đó chúng ta mong đợi rằng các mạch tương ứng của chúng tương tự nhau. Do đó chúng ta có thể xây dựng trên mạch cộng để tích hợp phép trừ, từ đó phục vụ cả hai mục đích.

Như đã thảo luận trong [thiết kế cộng/trừ ban đầu](#fig-add-sub-block-vi), bit điều khiển SUB báo hiệu nếu khối cộng/trừ nên thực hiện phép trừ. Chúng ta lưu ý điều sau đúng, do bù hai:

* Khi SUB=0 mạch thực hiện phép cộng `A + B`.
* Khi SUB=1, mạch thực hiện phép trừ `A - B`. Các phát biểu sau là tương đương:
  * Tính `A + (-B)`, với `B` là bù hai.
  * Tính `A + ~B + 1`. Đây là định nghĩa của bù hai: đảo bit và cộng 1.

Quan sát cuối cùng này cho chúng ta cách rất gọn gàng để mở rộng mạch cộng thành bộ cộng và trừ kết hợp:

1. Cộng 1 vào kết quả.
1. Đảo các bit của `B`.

:::{hint} Nối SUB vào c{sub}`0`

Chúng ta không thực sự cần bộ cộng thứ hai để thực hiện +1, vì chúng ta có đầu vào chưa sử dụng c{sub}`0` (bit carry in cho vị trí thứ không).

  * Khi SUB=0, nối c{sub}`0` với 0.
  * Khi SUB=1, nối c{sub}`0` với 1 để cộng thêm 1 vào cột ít quan trọng nhất, đạt được +1 bổ sung. 

Nói cách khác, nối SUB vào c{sub}`0`.

:::

:::{hint} XOR SUB với các bit của B
Sự mở rộng khác cần thiết cho phép trừ là đảo tất cả các bit của B trước khi đưa chúng vào đầu bộ cộng. Tất nhiên, chúng ta muốn phép đảo là **có điều kiện** dựa trên phép toán nào chúng ta muốn mạch thực hiện: ADD hoặc SUB.

Một lần nữa, chúng ta quay lại người bạn cũ cổng XOR. Nếu nhìn đúng cách, cổng XOR là bộ đảo có điều kiện (xem lại [tính chất của các phép toán bitwise](#sec-bitwise-props)!).

* Nếu một trong các đầu vào của cổng XOR là 0, thì đầu ra nhận giá trị của đầu vào còn lại.
* Mặt khác, nếu một trong các đầu vào của cổng XOR là 1, thì đầu vào còn lại đi qua, **bị đảo**.

Do đó chúng ta có thể đảo B có điều kiện, bằng cách cho mỗi bit của nó đi qua cổng XOR với SUB trên đường đến đầu vào của bộ cộng!
:::

Thiết kế bộ cộng được mở rộng được hiển thị bên dưới. Khi đầu vào SUB là 1 khối thực hiện phép trừ, khi SUB=0 khối thực hiện phép cộng.

:::{figure} images/nbit-add-sub.png
:label: fig-nbit-sub-design-vi
:width: 100%
:alt: "Cùng mạch bộ cộng n-bit với phát hiện overflow, nhưng bây giờ với thiết kế bộ trừ sử dụng cổng XOR để đảo B có điều kiện dựa trên đầu vào điều khiển SUB quyết định bit carry-in ban đầu c0."

Sơ đồ mạch thiết kế bộ cộng/trừ N-bit.
:::
