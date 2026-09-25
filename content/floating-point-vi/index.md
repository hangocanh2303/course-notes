---
title: "Động lực: Dấu phẩy cố định"
---

## Mục tiêu học tập

* Hiểu các giới hạn của biểu diễn "dấu phẩy cố định" cho các số có phần thập phân.
* Tính toán phạm vi và bước nhảy cho các biểu diễn số nguyên và dấu phẩy cố định.
* Nhận diện các hạn chế của biểu diễn dấu phẩy cố định.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/-nB32FCrlTA
:width: 100%
:title: "[CS61C FA20] Lecture 06.1 - Floating Point: Basics & Fixed Point"
:::

::::

Học kiến trúc có nghĩa là học trừu tượng hóa nhị phân! Hãy xem lại biểu diễn số. Bây giờ, mục tiêu của chúng ta là sử dụng 32 bit để biểu diễn các số có phần thập phân—như 4.25, 5.17, v.v. Chúng ta sẽ học chuẩn **IEEE 754 Floating Point Độ chính xác đơn (Single-Precision)**.

Để bắt đầu, tôi muốn chia sẻ một câu nói của James Gosling (người tạo ra Java) từ năm 1998.[^gosling-cite]

> "95% những người ngoài kia hoàn toàn không hiểu gì về floating-point."

[^gosling-cite]: Từ một bài phát biểu quan trọng của James Gosling vào ngày 28 tháng 2 năm 1998. Liên kết gốc đã mất, nhưng bài ["How Java's Floating-Point Hurts Everyone Everywhere"](https://people.eecs.berkeley.edu/~wkahan/JAVAhurt.pdf) của Giáo sư Will Kahan cung cấp bối cảnh hợp lý cho bài nói chuyện, đề xuất và các lựa chọn thay thế của Gosling.

Bạn sẽ thuộc nhóm 5% sau chương này :-)

## Các thước đo: Phạm vi và Bước nhảy

Nhớ lại rằng chuỗi N-bit có thể biểu diễn tối đa $2^\texttt{N}$ giá trị riêng biệt. Chúng ta đã đề cập một số hệ thống N-bit để biểu diễn số nguyên cho đến nay, như được hiển thị trong @tab-int-systems.

:::{table} Một tập con của các biểu diễn số nguyên N-bit mà chúng ta đã thảo luận trong lớp này.
:label: tab-int-systems
:align: center

| Hệ thống | \# bit | Giá trị nhỏ nhất | Giá trị lớn nhất | Bước nhảy |
| :--- | :-: | :---: | :---: | :---: |
| Số nguyên không dấu | 32 | 0 | $2^{32} - 1$, tức là <br/> $4,294,967,295$ | 1 |
| Số nguyên có dấu với bù hai | 32 | $-2^{31}$, tức là <br/> $-2,147,483,648$ | $2^{31} - 1$, tức là <br/> $2,147,483,647$ | 1 |

:::

**Phạm vi** của một hệ thống biểu diễn số có thể được định lượng bằng cách tính số nhỏ nhất và lớn nhất có thể biểu diễn. Giống như với các biểu diễn số nguyên, chúng ta sẽ tính toán rõ ràng phạm vi và so sánh nó với một số ứng dụng mục tiêu. Trong @tab-int-systems, chúng ta thấy số nguyên không dấu cho giá trị lớn nhất gấp khoảng hai lần so với bù hai—vì hệ thống sau dành khoảng một nửa hệ thống để biểu diễn số âm.

**Bước nhảy** của một hệ thống biểu diễn số được định nghĩa là khoảng cách giữa hai số liên tiếp. Đối với các biểu diễn số nguyên mà chúng ta xem xét trong lớp này, bước nhảy luôn là 1 giữa các số nguyên liên tiếp. Mặt khác, với các biểu diễn số có phần thập phân, chúng ta sẽ không thể biểu diễn phạm vi vô hạn của các số (thực) giữa hai số liên tiếp, do đó bước nhảy trở thành một thước đo quan trọng của **độ chính xác**.[^precision-footnote]

[^precision-footnote]: Chúng ta sẽ định nghĩa độ chính xác đúng cách sớm thôi; hiện tại, hãy coi độ chính xác là một thước đo khả năng biểu diễn các thay đổi nhỏ trong số của biểu diễn.

## Dấu phẩy cố định (Fixed Point)

Hãy tạo động lực cho floating point bằng cách trước tiên khám phá một cách tiếp cận tạm thời[^strawman]: **dấu phẩy cố định**. Hệ thống "dấu phẩy cố định" cố định số chữ số được sử dụng để biểu diễn phần nguyên và phần thập phân.

[^strawman]: Wikipedia: [Straw Man](https://en.wikipedia.org/wiki/Straw_man)

Hãy nhớ lại khi bạn lần đầu học về **dấu thập phân**. Bên trái của dấu chấm, bạn có hàng đơn vị ($10^0$), hàng chục ($10^1$), hàng trăm ($10^2$), v.v. Bên phải của dấu chấm, các số biểu diễn phần mười ($10^{-1}$), phần trăm ($10^{-2}$), phần nghìn ($10^{-3}$), v.v. **Dấu chấm nhị phân** hoạt động tương tự cho một _chuỗi bit_. Bên trái của dấu chấm nhị phân là các lũy thừa của hai: 1 ($2^{0}$), 2 ($2^{1}$), 4 ($2^{2}$), v.v. Bên phải của dấu chấm nhị phân là các lũy thừa âm của hai: 1/2 ($2^{-1}$), 1/4 ($2^{-2}$), 1/8 ($2^{-3}$), v.v.

Biểu diễn dấu phẩy cố định 6-bit cố định dấu chấm nhị phân ở một vị trí cụ thể trong mẫu bit 6-bit. Xem xét biểu diễn dấu phẩy cố định 6-bit được hiển thị trong @fig-fixed-point. Điều này cố định dấu chấm nhị phân ở vị trí 4 bit từ bên phải, và giả định rằng chúng ta chỉ biểu diễn số không âm.

:::{figure} images/fixed-point.png
:label: fig-fixed-point
:width: 80%
:alt: "Six-bit fixed-point layout with two integer bits and four fractional bits, where the binary point is fixed between them. Labels bit positions to powers 2^1, 2^0, 2^-1, 2^-2, 2^-3, and 2^-4."
Biểu diễn dấu phẩy cố định 6-bit
:::

Theo hệ thống này, số 2.625 có mẫu bit `101010`:

$$
\begin{align}
\texttt{1} × 2^1 + \texttt{0} × 2^0 &+ \texttt{1} \times 2^{-1} +  \texttt{0} \times 2^{-2} + \texttt{1} \times 2^{-3} + \texttt{0} \times 2^{-4} \\
    &= 2 + 	0.5 + 0.125 \\
    &= 2.625_{\text{mười}}
\end{align}
$$

### Dấu phẩy cố định: Phạm vi và Bước nhảy

:::{tip} Kiểm tra nhanh
Sử dụng biểu diễn dấu phẩy cố định này, phạm vi của các số có thể biểu diễn là gì?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Phạm vi:

* Số nhỏ nhất `000000`: không.
* Số lớn nhất: `111111`, hay 3.9375 = $4 - \texttt{1} \times 2^{-4}$

:::

:::{tip} Kiểm tra nhanh
Sử dụng biểu diễn dấu phẩy cố định này, **bước nhảy** giữa hai số liên tiếp bất kỳ là bao nhiêu?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Bước thập phân nhỏ nhất chúng ta có thể thực hiện là tăng bit có trọng số thấp nhất, đó là $2^{-4} = 1/16$.

:::

### Phép toán

@fig-fixed-point-add cho thấy phép cộng dấu phẩy cố định rất đơn giản. Bằng cách căn chỉnh các dấu chấm nhị phân, các định dạng dấu phẩy cố định có thể tái sử dụng bộ cộng số nguyên.

:::{figure} images/fixed-point-add.png
:label: fig-fixed-point-add
:width: 40%
:alt: "Fixed-point addition example with aligned binary points: 01.1000 plus 00.1000 equals 10.0000, corresponding to 1.5 + 0.5 = 2.0."

$1.5 + 0.5 = 2$ sử dụng biểu diễn dấu phẩy cố định 6-bit từ @fig-fixed-point.
:::

Tuy nhiên, @fig-fixed-point-mul cho thấy phép nhân dấu phẩy cố định phức tạp hơn. Giống như trong phép nhân thập phân, chúng ta phải dịch dấu chấm nhị phân dựa trên số chữ số thập phân trong các thừa số.

:::{figure} images/fixed-point-mul.png
:label: fig-fixed-point-mul
:width: 40%
:alt: "Fixed-point multiplication example showing partial products for 01.1000 times 00.1000 and a final binary-point adjustment to obtain 00.1100, which represents 0.75."

$1.5 \times 0.5 = 0.75$ sử dụng biểu diễn dấu phẩy cố định 6-bit từ @fig-fixed-point.
:::

### Các trường hợp sử dụng

Biểu diễn dấu phẩy cố định rất hữu ích trong các lĩnh vực cụ thể ưu tiên tính toán rất nhanh các giá trị với các đặc điểm cụ thể. Một số ứng dụng đồ họa ưa thích dấu phẩy cố định để tính toán nhanh khi render hình ảnh.

## Các số khoa học

Các nhà phát triển đã xem xét những gì cần thiết để định nghĩa một hệ thống số có thể biểu diễn các số được sử dụng trong các ứng dụng khoa học phổ biến:

* Các số rất lớn, ví dụ: số giây trong một thiên niên kỷ là $31,556,926,010 = 3.155692610 \times 10^{10}$
* Các số rất nhỏ, ví dụ: bán kính Bohr xấp xỉ $0.000000000052917710 = 5.2917710 \times 10^{-11}$ mét
* Các số có cả phần nguyên và phần thập phân, ví dụ: $2.625$

Một biểu diễn dấu phẩy cố định có thể biểu diễn cả ba giá trị ví dụ này sẽ cần ít nhất 92 bit[^fp-huge], lớn hơn nhiều so với các hệ thống số nguyên 32-bit được thảo luận trong @tab-int-systems.

[^fp-huge]: $31,556,926,010 = 3.155692610 \times 10^{10}$ cần 34 bit cho phần nguyên, và $0.000000000052917710 = 5.2917710 \times 10^{-11}$ cần 58 bit cho phần thập phân.

Vấn đề với dấu phẩy cố định là một khi chúng ta xác định vị trí đặt dấu chấm nhị phân, chúng ta bị mắc kẹt. Trong biểu diễn sáu bit của chúng ta từ @fig-fixed-point, chúng ta không có cách nào để biểu diễn các số lớn hơn nhiều so với $3.9375$ hoặc các số giữa, ví dụ, $0$ và $1/16$, chứ chưa nói đến nhiều số trong các ứng dụng khoa học.

Điều gì sẽ xảy ra nếu chúng ta có cách để "di chuyển" dấu chấm nhị phân xung quanh và thay vào đó chọn vị trí của nó tùy thuộc vào số mục tiêu? Các nhà phát triển chuẩn floating point IEEE 754 đã tìm ra giải pháp trong một thực hành khoa học rất phổ biến để ký hiệu các số thập phân: **ký hiệu khoa học**. Hãy đọc tiếp!
