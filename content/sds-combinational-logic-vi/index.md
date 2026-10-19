---
title: "Cổng Logic"
---

(sec-logic-gates-vi)=
## Mục Tiêu Học Tập

* Sử dụng bảng chân trị để liệt kê quan hệ vào-ra của cổng logic cơ bản và mạch logic tổ hợp.
* Mô tả chức năng của cổng logic N đầu vào (đặc biệt là cổng XOR 3 đầu vào).

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/KOXgVikIYi4
:width: 100%
:title: "[CS61C FA20] Lecture 16.1 - Combinational Logic: Truth Tables"
:::

::::

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/vX8BhDk3qhc
:width: 100%
:title: "[CS61C FA20] Lecture 16.2 - Combinational Logic: Logic Gates"
:::

::::

Để thiết kế mạch thực hiện các phép toán phức tạp trên tín hiệu nhị phân, trước tiên chúng ta phải định nghĩa các toán tử nguyên thủy gọi là **cổng logic** (logic gates). Cổng logic là các mạch đơn giản (mỗi cổng chỉ có vài transistor) có thể được nối với nhau để hiện thực bất kỳ hàm logic tổ hợp nào. Trong CS 61C, chúng ta coi cổng logic là các phần tử nguyên thủy; chúng là các khối xây dựng cơ bản cho mạch của chúng ta.

Các cổng logic đơn giản nhất là các toán tử **nhị phân** (binary) hoặc **đơn nguyên** (unary), nhận một hoặc hai biến nhị phân làm đầu vào và xuất ra một giá trị nhị phân.

::::::{note} Cổng logic AND

:::::{grid} 4

::::{grid-item}
:::{figure}
:alt: "Mã định nghĩa Boolean AND là y bằng AND của a và b, dòng thứ hai biểu diễn cùng phép toán sử dụng toán tử AND bit của C trên a và b."
```{code} bash
y = AND(a,b)
  = a & b
```
1. Định nghĩa hàm
:::
::::

::::{grid-item}
:::{figure} #tab-and
:width: 70%
:alt: "Bảng chân trị cho AND hai đầu vào liệt kê tất cả tổ hợp của đầu vào a và b với đầu ra y như được in lại từ tài liệu tham khảo logic tổ hợp."
2. Bảng Chân Trị
:::
::::

::::{grid-item}
:::{figure} images/and-gate-mnemonic.png
:width: 70%
:alt: "Ký hiệu cổng AND dạng hoạt hình có hình dạng giống chữ D trong AND, được dùng làm mẹo nhớ liên kết hình dạng cổng với từ AND."
3. Biểu diễn đồ họa[^mnemonic]

[^mnemonic]: Mẹo nhớ: Cổng AND có hình dạng giống chữ "D" trong AN**D**.
:::
::::

::::{grid-item}
:::{figure} images/and-transistor.png
:width: 70%
:alt: "Sơ đồ mức transistor CMOS của cổng AND hai đầu vào hiển thị mạng kéo lên và kéo xuống bù nhau để hiện thực hàm chân trị AND."
4. Mạch Transistor
:::
::::

:::::

Bốn cách biểu diễn khác nhau:

1. Định nghĩa hàm `y = AND(a, b)`. Dòng thứ hai sử dụng phép toán bit C `&`.
1. Bảng chân trị cho `y = AND(a, b)`. Mỗi hàng liệt kê từng tổ hợp đầu vào và giá trị đầu ra tương ứng.
1. Ký hiệu cổng logic cho AND, được dùng làm biểu diễn đồ họa trong sơ đồ mạch số.
1. Mạch transistor CMOS cho cổng logic AND.[^and-cmos]

[^and-cmos]: Ngoài phạm vi khóa học này, nhưng những ai quan tâm, [đọc thêm](https://electronics.stackexchange.com/a/226028) về AND.

::::::

(sec-common-logic-gates-vi)=
## Các Cổng Logic Phổ Biến

Dưới đây là một số cổng logic phổ biến, nhiều trong số đó bạn đã thấy như các [phép toán bit trong C](#sec-c-bitwise-ops). Với mỗi cổng, chúng ta định nghĩa tên, biểu diễn đồ họa, và bảng chân trị xác định chức năng của nó.

<!-- begin grid -->
:::::::{grid} 3

::::::{card}
:header: AND
:::::{tab-set}
::::{tab-item} Cổng
:::{figure} images/and-gate.png
:label: fig-and-gate-vi
:width: 100%
:alt: "Ký hiệu cổng AND hai đầu vào: cạnh trái phẳng với đầu vào a và b, cạnh phải cong với đầu ra y."
:::
::::
::::{tab-item} Bảng Chân Trị
:::{figure} #tab-and
:alt: "Bảng chân trị cho AND hai đầu vào với các hàng cho mỗi mẫu nhị phân của a và b và đầu ra y kết quả."
:::
::::
:::::
::::::

::::::{card}
:header: OR
:::::{tab-set}
::::{tab-item} Cổng
:::{figure} images/or-gate.png
:label: fig-or-gate-vi
:width: 100%
:alt: "Ký hiệu cổng OR hai đầu vào: cạnh đầu vào cong với đầu vào a và b, cạnh đầu ra nhọn với đầu ra y."
:::
::::
::::{tab-item} Bảng Chân Trị
:::{figure} #tab-or
:alt: "Bảng chân trị cho OR hai đầu vào với các hàng cho mỗi mẫu nhị phân của a và b và đầu ra y kết quả."
:::
::::
:::::
::::::

::::::{card}
:header: NOT
:::::{tab-set}
::::{tab-item} Cổng
:::{figure} images/not-gate.png
:label: fig-not-gate-vi
:width: 100%
:alt: "Ký hiệu cổng NOT: tam giác chỉ sang phải với bong bóng đảo ở đỉnh, đầu vào a, và đầu ra y."
:::
::::
::::{tab-item} Bảng Chân Trị
:::{figure} #tab-not
:alt: "Bảng chân trị cho NOT đơn nguyên hiển thị đầu vào a và đầu ra đảo y cho cả hai mức logic."
:::
::::
:::::
::::::

::::::{card}
:header: NAND
:::::{tab-set}
::::{tab-item} Cổng
:::{figure} images/nand-gate.png
:label: fig-nand-gate-vi
:width: 100%
:alt: "Ký hiệu cổng NAND hai đầu vào: hình AND với bong bóng đảo ở phía đầu ra, đầu vào a và b, và đầu ra y."
:::
::::
::::{tab-item} Bảng Chân Trị
:::{table}
:label: tab-nand-vi
:align: center

| a | b | y |
| :--: | :--- | :--- |
| 0 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

:::
::::
:::::
::::::

::::::{card}
:header: NOR
:::::{tab-set}
::::{tab-item} Cổng
:::{figure} images/nor-gate.png
:label: fig-nor-gate-vi
:width: 100%
:alt: "Ký hiệu cổng NOR hai đầu vào: hình OR với bong bóng đảo ở phía đầu ra, đầu vào a và b, và đầu ra y."
:::
::::
::::{tab-item} Bảng Chân Trị
:::{table}
:label: tab-nor-vi
:align: center

| a | b | y |
| :--: | :--- | :--- |
| 0 | 0 | 1 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 0 |

:::
::::
:::::
::::::

::::::{card}
:header: XOR
:::::{tab-set}
::::{tab-item} Cổng
:::{figure} images/xor-gate.png
:label: fig-xor-gate-vi
:width: 100%
:alt: "Ký hiệu cổng XOR hai đầu vào: hình OR với đường cong thêm ở phía đầu vào, đầu vào a và b, và đầu ra y."
:::
::::
::::{tab-item} Bảng Chân Trị
:::{figure} #tab-xor
:alt: "Bảng chân trị cho XOR hai đầu vào với các hàng cho mỗi mẫu nhị phân của a và b và đầu ra y kết quả."
:::
::::
:::::
::::::

:::::::
<!-- end grid -->

:::{warning} Kiểm tra Bảng Chân Trị

Nhấp vào các tab ở trên để xem biểu diễn đồ họa của mỗi cổng và **bảng chân trị** của nó.

Với mỗi mẫu đầu vào gồm 1 và 0, tồn tại một mẫu đầu ra duy nhất. Bảng chân trị liệt kê mối quan hệ đầu vào/đầu ra này. Với các cổng logic 2 đầu vào bên dưới, có bốn hàng.
:::

Ghi chú:

* [AND](#fig-and-gate-vi), [OR](#fig-or-gate-vi), [NOT](#fig-not-gate-vi) và [XOR](#fig-xor-gate-vi) tuân theo các phép toán bit C mà bạn đã học trước đó.
  * Cổng NOT thường được gọi là **cổng đảo** (inverter). Chú ý "bong bóng" (vòng tròn).
* [NAND](#fig-nand-gate-vi) là "NOT" AND. Chú ý bong bóng ở đầu ra.
* [NOR](#fig-nor-gate-vi) là "NOT" OR. Một lần nữa, chú ý bong bóng.

(sec-subset-gates-vi)=
:::{hint} Kết hợp cổng logic để xây dựng mạch

Nói chung, không cần bộ đầy đủ các cổng logic được hiển thị ở trên.
Các tập con chọn lọc là đủ, mặc dù để đơn giản, tập con lớn hơn thường được sử dụng.

Bất kỳ hàm logic tổ hợp nào cũng có thể được hiện thực với:

* Chỉ tập hợp AND và NOT
* Chỉ tập hợp OR và NOT
* Chỉ cổng NAND (NAND được gọi là **cổng phổ quát**)
* Chỉ cổng NOR (cũng là cổng phổ quát)
* **AND, OR, và NOT** đặc biệt hữu ích; chúng ta sẽ thấy tại sao khi thảo luận đại số Boolean trong [phần sau](#sec-boolean-algebra-vi).
:::

## Cổng Logic N Đầu Vào

Ngoại trừ NOT (là toán tử đơn nguyên), chúng ta đã hiển thị các phiên bản 2 đầu vào của các cổng phổ biến này. Các phiên bản của những cổng này với nhiều hơn hai đầu vào cũng tồn tại. Vì lý do hiệu năng, số lượng đầu vào cho cổng logic thường bị giới hạn tối đa khoảng bốn.

:::{figure} images/and-gate-n.png
:label: fig-and-gate-n-vi
:width: 30%
:alt: "Ký hiệu cổng AND bốn đầu vào: các đầu vào xếp chồng a, b, c, và d ở bên trái và đầu ra đơn y ở cạnh phải cong."

Cổng AND 4 đầu vào. Đầu ra `y` là `1` nếu và chỉ nếu `a`, `b`, `c`, và `d` đều là `1`.
:::

Chức năng của các cổng có nhiều hơn hai đầu vào thường tự giải thích và có thể được xác định bằng cách lặp lại kết hợp cổng 2 đầu vào tương đương. Ví dụ, `AND(a, b, c, d) = AND(a, AND(b, AND(c, d))) = AND(AND(a, b), AND(c, d))`, v.v. Có một số ngoại lệ, như cổng exclusive-or; hãy thử suy luận của bạn với một số kiểm tra nhanh.

```{tip} Cổng [NAND](#fig-nand-gate-vi) N đầu vào là gì?
:class: dropdown

NAND N đầu vào là NOT của (AND N đầu vào).
```

```{tip} Cổng [NOR](#fig-nor-gate-vi) N đầu vào là gì?
:class: dropdown

NOR N đầu vào là NOT của (OR N đầu vào).
```

(sec-n-xor-vi)=
```{hint} [Quan trọng] Cổng [XOR](#fig-xor-gate-vi) N đầu vào là gì?

XOR N đầu vào là 1 nếu số lượng số 1 trên tất cả các đầu vào là **lẻ**.

Xét cổng XOR 3 đầu vào, trong đó `y = XOR(a, b, c) = XOR(a, XOR(b, c))`, v.v. Xác minh với bảng chân trị:

:::{table} Cổng XOR 3 đầu vào.
:label: tab-xor-three-vi

| a | b | c | y |
| :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 |
| 0 | 1 | 0 | 1 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 0 | 1 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 0 |
| 1 | 1 | 1 | 1 |
:::
```

## Thiết Kế Mạch Logic Tổ Hợp

Các cổng logic đơn giản có thể được nối với nhau để xây dựng các mạch hữu ích. Thực tế, bất kỳ khối logic tổ hợp nào cũng có thể được hiện thực chỉ với các cổng logic AND, OR, và NOT.

Tuy nhiên, để hiểu mạch thực sự làm gì, chúng ta cần nhiều hơn chỉ sơ đồ mạch: chúng ta cần một mô tả ngắn gọn về hoạt động của nó.
