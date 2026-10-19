---
title: "Tổng Kết"
---

## Kết Luận$\dots$

Mục tiêu của bài giảng này là xây dựng các khối logic tổ hợp. Chúng ta có thể tóm tắt cách tiếp cận này với sơ đồ sau:

:::{figure} #fig-cl-block-representation-vi
:width: 60%
:alt: "Sơ đồ với ba bong bóng mô tả ba biểu diễn của logic tổ hợp: Bảng Chân Trị, Biểu Thức Boolean, và Sơ Đồ Cổng. Các biểu diễn được kết nối bằng các mũi tên cong hiển thị chuyển đổi giữa chúng."

@fig-cl-block-representation-vi trong [phần trước](#sec-cl-practice-vi).
:::

Chúng ta đã thảo luận cách chuyển đổi giữa ba biểu diễn này, được biểu diễn bằng các cung trong sơ đồ. Dưới đây là tóm tắt:

* **Bảng chân trị sang Biểu thức Boolean.** Viết dạng chính tắc (Tổng của Tích) và tiếp theo với đơn giản hóa đại số nếu muốn.
* **Biểu thức Boolean sang Bảng chân trị.** Đánh giá biểu thức cho tất cả tổ hợp đầu vào và ghi lại các giá trị đầu ra.
* **Biểu thức Boolean sang Cổng.** Sử dụng cổng AND cho toán tử AND, cổng OR cho toán tử OR, và cổng đảo cho toán tử NOT. Nối các cổng để khớp với cấu trúc của biểu thức.
* **Cổng sang Biểu thức Boolean.** Đảo ngược quá trình trên.
* **Cổng sang Bảng chân trị.** Chạy qua tất cả tổ hợp đầu vào và đánh giá đầu ra.
* **Bảng chân trị sang Cổng.** Ánh xạ sang biểu thức Boolean rồi sang cổng.


Trong điện tử số, thường quan trọng để có được các đầu ra nhất định dựa trên đầu vào của bạn, như được trình bày bởi bảng chân trị. Bảng chân trị ánh xạ trực tiếp sang biểu thức Boolean, và biểu thức Boolean ánh xạ trực tiếp sang cổng logic. Tuy nhiên, để tối thiểu hóa số lượng cổng logic cần thiết để hiện thực một mạch, thường hữu ích để đơn giản hóa các biểu thức Boolean dài.

Chúng ta có thể đơn giản hóa biểu thức sử dụng chín định luật chính của đại số Boolean:
:::{figure} #tab-boolean-laws-vi
:alt: "In lại bảng chín định luật cơ bản của đại số Boolean với các biểu thức cho đồng nhất, null, lũy đẳng, nghịch đảo, giao hoán, kết hợp, phân phối, hấp thụ, và thuộc tính De Morgan."
Các Định Luật của Đại Số Boolean (in lại của @tab-boolean-laws-vi từ [phần này](#sec-boolean-algebra-vi)).
:::

Ngoài ra, chúng ta có nhiều hàm boolean nhận tín hiệu boolean (0 hoặc 1) làm đầu vào và xuất ra kết quả boolean (0 hoặc 1). Khi thiết kế hệ thống số, các hàm boolean được biểu diễn dưới dạng cổng logic. Các cổng logic phổ biến có thể được tìm thấy trong [phần này](#sec-common-logic-gates-vi)

Có hai loại mạch cơ bản: mạch **logic tổ hợp** và phần tử trạng thái.
Mạch logic tổ hợp đơn giản thay đổi dựa trên đầu vào của chúng sau bất kỳ độ trễ lan truyền nào liên kết với chúng. Ví dụ, nếu một cổng AND (hình bên dưới) có độ trễ lan truyền liên kết là 2ps, đầu ra của nó sẽ thay đổi dựa trên đầu vào như sau:

:::{figure} images/SDS_Precheck_Disc8.png
:width: 60%
:alt: "Sơ đồ bên trái mô tả cổng AND hai đầu vào với đầu vào INPUT_A và INPUT_B và đầu ra thấp; sơ đồ bên phải hiển thị các dạng sóng thời gian liên kết cho a, b, và đầu ra hiển thị hành vi AND với độ trễ lan truyền có thể nhìn thấy sau các cạnh đầu vào."
:::

Bạn nên chú ý rằng đầu ra của cổng AND này luôn thay đổi 2ps sau khi đầu vào của nó thay đổi.
**Phần tử trạng thái**, mặt khác, có thể nhớ đầu vào của chúng ngay cả sau khi đầu vào thay đổi. Phần tử trạng thái thay đổi giá trị dựa trên tín hiệu xung nhịp. Một thanh ghi được kích hoạt bởi cạnh lên, ví dụ, lấy mẫu đầu vào của nó tại cạnh lên của xung nhịp (khi tín hiệu xung nhịp đi từ 0 sang 1).

## Đọc Thêm Từ Sách Giáo Khoa

P&H A.3-A.6

## Tài Liệu Tham Khảo Bổ Sung

Những ghi chú này không thể có được nếu không có tài liệu CS 61C của Giáo sư John Wawrzynek:

* [Tài liệu Boolean](../resources/boolean.pdf)

## Bài Tập
Kiểm tra kiến thức của bạn!

### Ôn Tập Khái Niệm

:::{exercise}
:label: sdscl-01-vi
1. **Đúng/Sai**: Đơn giản hóa biểu thức logic boolean sẽ không ảnh hưởng đến hiệu năng của hiện thực phần cứng.
:::

:::{solution} sdscl-01-vi
:label: sdscl-01-sol-vi
:class: dropdown
**Sai**. Các cách sắp xếp cổng khác nhau hiện thực cùng logic có thể có độ trễ lan truyền khác nhau, có thể ảnh hưởng đến tốc độ xung nhịp cho phép.
:::

:::{exercise}
:label: sdscl-02-vi
2. **Đúng/Sai**: Càng ít cổng logic, mạch càng nhanh (giả sử mỗi cổng có cùng độ trễ lan truyền).
:::

:::{solution} sdscl-02-vi
:label: sdscl-02-sol-vi
:class: dropdown
**Sai**. Một mạch rộng với nhiều cổng song song có thể có ít độ trễ hơn chỉ vài cổng sắp xếp theo chuỗi.
:::

:::{exercise}
:label: sdscl-03-vi
3. **Đúng/Sai**: Thời gian cần thiết cho clock-to-q và thiết lập thanh ghi có thể lớn hơn một chu kỳ xung nhịp.
:::

:::{solution} sdscl-03-vi
:label: sdscl-03-sol-vi
:class: dropdown
**Sai**. Điều này có thể dẫn đến không ổn định nếu các thanh ghi được kết nối với nhau, vì đầu ra thanh ghi có thể chưa lan truyền đúng trước cạnh lên tiếp theo.
:::

:::{exercise}
:label: sdscl-04-vi
4. **Đúng/Sai**: Mọi mạch logic tổ hợp có thể đều có thể được biểu diễn bằng một số tổ hợp của cổng NOR.
:::

:::{solution} sdscl-04-vi
:label: sdscl-04-sol-vi
:class: dropdown
**Đúng**. NOR có thể được sử dụng để biểu diễn cổng AND, OR, và NOT. Do đó, NOR là đầy đủ về chức năng và có thể được sử dụng để biểu diễn bất kỳ biểu thức Boolean nào có thể, và do đó bất kỳ mạch logic tổ hợp nào.
:::

:::{exercise}
:label: sdscl-05-vi
5. **Đúng/Sai**: Đường logic tổ hợp ngắn nhất giữa hai phần tử trạng thái hữu ích trong việc xác định tần số mạch và chu kỳ xung nhịp tối thiểu.
:::

:::{solution} sdscl-05-vi
:label: sdscl-05-sol-vi
:class: dropdown
**Sai**. Chu kỳ xung nhịp tối thiểu phải cho phép đủ thời gian để mọi độ trễ logic tổ hợp ổn định trên đầu ra, vì vậy tần số dựa trên độ trễ logic tổ hợp dài nhất có thể giữa các phần tử trạng thái.
:::

### Bài Tập Ngắn

::::{exercise}
:label: sdscl-06-vi
1. Ghi nhãn mỗi cổng logic sau với hàm boolean tương ứng của chúng, và vẽ bảng chân trị biểu diễn đầu ra của chúng:

:::{figure} images/exercises-1-gates.png
:label: exercises-1-gates-vi
:width: 100%
:alt: "Hàng bảy ký hiệu cổng tiêu chuẩn: NOT, AND, OR, XOR, NAND, NOR, và XNOR."
:::
::::

::::{solution} sdscl-06-vi
:label: sdscl-06-sol-vi
:class: dropdown
**NOT, AND, OR, XOR, NAND, NOR, XNOR**

Dưới đây là đầu ra cho mỗi hàm boolean được kết hợp trong một bảng chân trị duy nhất. Tất cả tổ hợp có thể của đầu vào ${x}$ và ${y}$ được hiển thị ở bên trái, và đầu ra của hàm boolean dựa trên đầu vào hiện tại được hiển thị ở bên phải
| Đầu vào | NOT | AND | OR | XOR | NAND | NOR | XNOR |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| ${x}$ ${y}$ | $\bar{x}$ | ${x}\cdot{y}$ | ${x}+{y}$ | ${x}\oplus{y}$| $\overline{{x}\cdot{y}}$ | $\overline{{x}+{y}}$ | $\overline{{x}\oplus{y}}$ |
| 0  0 | 1 | 0 | 0 | 0 | 1 | 1 | 1 |
| 0  1 | 1 | 0 | 1 | 1 | 1 | 0 | 0 |
| 1  0 | 0 | 0 | 1 | 1 | 1 | 0 | 0 |
| 1  1 | 0 | 1 | 1 | 0 | 0 | 0 | 1 |
::::
