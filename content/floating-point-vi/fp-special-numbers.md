---
title: "Số đặc biệt"
---

(sec-special-floats)=
## Mục tiêu học tập

* Hiểu cách chuẩn IEEE 754 biểu diễn số không, vô cực, và NaN
* Hiểu ý nghĩa của overflow hoặc underflow với số floating point
* Hiểu cách số không chuẩn hóa (denormalized) triển khai underflow "từ từ"
* Chuyển đổi số không chuẩn hóa sang dạng thập phân tương ứng


::::{note} 🎥 Video bài giảng (overflow và underflow)
:class: dropdown

:::{iframe} https://www.youtube.com/embed/GzOMIRj1yO0
:width: 100%
:title: "[CS61C FA20] Lecture 06.2 - Floating Point: Floating Point"
:::

Overflow và Underflow, 6:54 - 8:40

::::

::::{note} 🎥 Video bài giảng (mọi thứ khác)
:class: dropdown

:::{iframe} https://www.youtube.com/embed/Gs0ARZzY-gM
:width: 100%
:title: "[CS61C FA20] Lecture 06.3 - Floating Point: Special Numbers"
:::

::::

**Số chuẩn hóa** chỉ là một _phần_ (hehe) của các biểu diễn floating point. Đối với độ chính xác đơn (32-bit), IEEE định nghĩa @tab-float-exp-fields dựa trên trường số mũ (ở đây, "số mũ thiên lệch"):


:::{table} Các giá trị trường số mũ cho IEEE 754 độ chính xác đơn.
:label:  tab-float-exp-fields
:align: center

| Số mũ thiên lệch | Trường significand | Mô tả |
| :--- | :--- | :--- |
| 0 (`0000000`) | tất cả là không | [số không](#sec-zero) |
| 0 (`0000000`) | khác không | Số không chuẩn hóa, còn gọi là [denorm](#sec-denorms) |
| 1 – 254 | bất kỳ | [Floating point chuẩn hóa](#sec-normalized) (mantissa có số 1 đứng đầu ẩn) |
| 255 (`1111111`) | tất cả là không | [vô cực](#sec-infty) |
| 255 (`1111111`) | khác không | [`NaN`](#sec-nans) |

:::

Trong phần này, chúng ta sẽ giải thích tại sao các "số đặc biệt" này tồn tại bằng cách xem xét các cạm bẫy của overflow **và** underflow. Sau đó, chúng ta sẽ định nghĩa từng số đặc biệt.

## Overflow và Underflow

Vì 0 và 255 là các trường số mũ dành riêng, phạm vi của floating point **chuẩn hóa** độ chính xác đơn là $[-3.4\times 10^{38}, -2^{-126}]$ và $[+2^{-126}, +3.4\times 10^{38}]$. Lưu ý $2^{-126} \approx 1.2 \times 10^{-38}$.

:::{note} Hiển thị giải thích
:class: dropdown

Số có độ lớn lớn nhất: (`1.1…1`) $\times 2^{(254 - 127)} \approx 3.4 × 10^{38}$. Lưu ý số mũ thiên lệch lớn nhất là $254$, vì $255$ được dành riêng cho [vô cực](#sec-infty) và [NaN](#sec-nans).

| s | số mũ | significand |
| :--: | :--: | :--: |
| `s` | `1111 1110` | `111 1111 1111 1111 1111 1111` |

(sec-float-range)=
Số có độ lớn nhỏ nhất: (`1.0`) $\times 2^{(1-127)} = 2^{-126} \approx 1.2 × 10^{-38}$. Lưu ý số mũ thiên lệch nhỏ nhất là $1$, vì $0$ (tất cả là không) được dành riêng cho [số không](#sec-zero) và [denorm](#sec-denorms).

| s | số mũ | significand |
| :--: | :--: | :--: |
| `s` | `0000 0001` | `000 0000 0000 0000 0000 0000` |

:::

Vì chuẩn floating point biểu diễn các thành phần phân số, nó bây giờ phải xem xét cả **overflow** và **_underflow_** (@fig-over-under-flow):

* Overflow: Độ lớn của giá trị quá lớn để biểu diễn.
* Underflow: Độ lớn của giá trị quá _nhỏ_ để biểu diễn.

:::{figure} images/over-under-flow.png
:label: fig-over-under-flow
:width: 100%
:alt: "Number-line sketch of floating-point range showing overflow beyond about plus or minus 3.4 times 10^38 and an underflow gap around zero between approximately minus 1.2 times 10^-38 and plus 1.2 times 10^-38."

Biểu diễn floating point có thể gặp cả overflow và underflow.
:::

:::{warning} Tại sao _số nguyên_ không gặp underflow?

Trong biểu diễn floating point chuẩn hóa IEEE 754, underflow đề cập đến khoảng trống giữa hai phạm vi rời rạc của các số có thể biểu diễn. Với biểu diễn số nguyên, chúng ta có thể biểu diễn tất cả các số hợp lệ (tức là số nguyên) trong một phạm vi nhất định [`INT_MIN`, `INT_MAX`] vì biểu diễn số nguyên có cùng bước nhảy "1". Do đó không có "khoảng trống" trong biểu diễn trong phạm vi có thể biểu diễn này.

:::

Nhớ lại rằng với số nguyên, integer overflow khiến kết quả số học "quay vòng". Điều này có nghĩa là cộng các số nguyên dương lớn có thể dẫn đến số nguyên âm. Không giống như biểu diễn số nguyên, biểu diễn floating point tương tự như chuẩn IEEE 754 có thể xử lý overflow, underflow, và lỗi một cách "duyên dáng" hơn với các số đặc biệt, mà chúng ta thảo luận tiếp theo.

Khi số học floating point gây ra overflow, chúng ta báo hiệu [vô cực](#sec-infty) hoặc biểu diễn trực tiếp lỗi số học với [NaN](#sec-nans). Với underflow, chúng ta _từ từ_ di chuyển về [số không](#sec-zero) với [denorm](#sec-denorms).

## Số đặc biệt

Xem bốn danh mục không chuẩn hóa được hiển thị trong @tab-float-exp-fields.

(sec-zero)=
### Số không

Giống như trong số không dấu-độ lớn, IEEE 754 floating point có hai số không (@tab-float-zero). Nhớ rằng chuẩn được xây dựng cho tính toán khoa học! Có hai số không là hữu ích về mặt toán học. Hai ví dụ: giới hạn về không và tính toán $\pm \infty$, cái sau chúng ta thảo luận [tiếp theo](#sec-infty)).

:::{table} IEEE 754 độ chính xác đơn: Số không
:label: tab-float-zero

| giá trị | s | số mũ | significand |
| :-- | :--: | :--: | :--: |
| +0 | `0` | `0000 0000` | `000 0000 0000 0000 0000 0000` |
| -0 | `1` | `0000 0000` | `000 0000 0000 0000 0000 0000` |

:::

Nếu chúng ta xem xét biểu diễn toán học của nó, số không là lần gặp đầu tiên của chúng ta với biểu diễn floating point **không chuẩn hóa**. Xét cho cùng, $0.0$ trong ký hiệu khoa học không có số 1 đứng đầu!

Phần cứng floating point thường triển khai số không bằng cách dành riêng giá trị số mũ thiên lệch không `00000000` để báo hiệu không chuẩn hóa, tức là không cộng thêm 1 ẩn. Nếu significand cũng là tất cả không, thì phần cứng biết đó là số không. Nếu significand _khác_ không, chúng ta biểu diễn _các số không chuẩn hóa khác_, mà chúng ta thảo luận dưới đây là [số không chuẩn hóa](#sec-denorms).

(sec-infty)=
### Vô cực


Chuẩn IEEE 754 định nghĩa vô cực dương ($+\infty$) và vô cực âm ($-\infty$), như được hiển thị trong @tab-float-zero. Để biểu diễn vô cực, chúng ta dành riêng giá trị số mũ thiên lệch `11111111` và đặt significand thành không.

:::{table} IEEE 754 độ chính xác đơn: Vô cực
:label: tab-float-infty

| giá trị | s | số mũ | significand |
| :-- | :--: | :--: | :--: |
| $+\infty$ | `0` | `1111 1111` | `000 0000 0000 0000 0000 0000` |
| $-\infty$ | `1` | `1111 1111` | `000 0000 0000 0000 0000 0000` |

:::

Vì vô cực là một khái niệm quan trọng trong toán học, chuẩn phân biệt vô cực với các lỗi số học khác (mà chúng ta thảo luận [tiếp theo](#sec-nans)). Quan trọng là, chia cho $\pm 0$ cho ra $\pm \infty$. Các tính toán như $x / 0 > y$ nên có thể biểu diễn được,[^math] ngay cả khi không phải là "số" thực sự.

[^math]: Chúng tôi nhường cho sinh viên toán.

(sec-nans)=
### Không phải số (NaN)

Điều gì xảy ra nếu chúng ta cố tính toán số học không hợp lệ, như $\sqrt{-4}$ hoặc $0/0$? Đối với tính toán khoa học, có thể có giá trị hơn khi "nổi bọt" các lỗi này lên người dùng–thay vì làm crash chương trình một cách rõ ràng hoặc tính toán các giá trị không chính xác do quay vòng (ví dụ: trong integer overflow).

**NaN** (**N**ot **a** **N**umber - Không phải số) là các giá trị có dạng sau (@tab-float-nan):

:::{table} IEEE 754 độ chính xác đơn: NaN
:label: tab-float-nan

| s | số mũ | significand |
| :--: | :--: | :--: |
| một trong hai | `1111 1111` | khác không |

:::

Vì các giá trị này được kích hoạt khi overflow (lưu ý số mũ cao), chúng _lây lan_: $\text{op}(\text{NaN}, x) = \text{NaN}$.

Một số phần cứng độc quyền cho floating point đi xa hơn và sử dụng significand để mã hóa hoặc xác định nơi xảy ra lỗi. Thực hành mã lỗi này không được định nghĩa trong chuẩn.

(sec-denorms)=
### Số không chuẩn hóa (Denormalized Numbers)

#### Khoảng trống xung quanh số không

Trong trường hợp overflow, vô cực có vẻ hợp lý—xét cho cùng, nó là một bước nhảy qua float chuẩn hóa lớn nhất có thể biểu diễn (xấp xỉ $3.4\times 10^{38}$). Tương tự, với underflow, số không thực sự là một bước nhảy qua float chuẩn hóa _nhỏ nhất_ có thể biểu diễn ($2^{-126}$).

Tuy nhiên, khi chúng ta xem xét phạm vi toán học trong câu hỏi trong @fig-underflow, chúng ta quan sát một **khoảng trống** lớn xung quanh số không.

:::{figure} images/underflow.png
:label: fig-underflow
:width: 100%
:alt: "Zoomed number line near zero showing the smallest normalized positive value at 2^-126 and a much smaller local spacing of 2^-149 between nearby normalized values."

Vì underflow, có một "khoảng trống" của các số có thể biểu diễn xung quanh số không.
:::

Về độ lớn, khoảng trống này _không lớn_—$2^{-126}$ là rất nhỏ! Tuy nhiên, xem xét độ chính xác 23-bit của `float`. Đối với số chuẩn hóa trong khu vực này, chúng ta có thể sử dụng độ chính xác của mình để thực hiện bước nhảy nhỏ $2^{-149}$.


:::{note} Hiển thị giải thích
:class: dropdown

Số chuẩn hóa nhỏ nhất [từ trước](#sec-float-range): (`1.0`) $\times 2^{(1-127)} = 2^{-126}$

| s | số mũ | significand |
| :--: | :--: | :--: |
| `0` | `0000 0001` | `000 0000 0000 0000 0000 0000` |

Số chuẩn hóa nhỏ thứ hai: (`1.00...001`) $\times 2^{(1-127)} = (1 + 2^{-23}) \times 2^{-126} = 2^{-126} + 2^{-149}$

| s | số mũ | significand |
| :--: | :--: | :--: |
| `0` | `0000 0001` | `000 0000 0000 0000 0000 0001` |

Bước nhảy chuẩn hóa nhỏ nhất là hiệu này: $2^{-149}$

:::

Trong phạm vi này, chúng ta muốn duy trì độ chính xác cao để biểu diễn các bước nhỏ giữa các số nhỏ đó. Tuy nhiên, vì **số 1 ẩn trong mantissa chuẩn hóa**—và sự thiếu vắng của nó ở số không—có một sự khác biệt tương đối lớn về bước nhảy giữa 0 và số chuẩn hóa nhỏ nhất so với số nhỏ nhất và số nhỏ thứ hai.

#### Underflow từ từ

Với những điều trên, chuẩn IEEE 754 chỉ định một phạm vi số vẫn có thể được sử dụng khi chúng ta gặp underflow, để không phải tất cả phép tính bị mất. **Số không chuẩn hóa** trong chuẩn giúp hỗ trợ **underflow từ từ**.[^gradual]

[^gradual]: Nếu một số không chuẩn hóa là kết quả từ phép tính của hai số chuẩn hóa, chúng ta _vẫn nói rằng underflow đã xảy ra_. Nói cách khác, denorm giúp bảo toàn độ chính xác số học trong quá trình underflow.

Chuẩn IEEE 754 định nghĩa số không chuẩn hóa có dạng trong Phương trình @eq-float-denorm.

(eq-float-denorm)=
:::{math}
:enumerated: true
(-1)^\text{s} \times (\text{significand}) \times 2^{-126}
:::

Chuẩn chỉ định cách diễn giải các trường để biểu diễn số không chuẩn hóa, còn được gọi là **denorm** (@tab-denorm):

:::{table} Các trường dấu, số mũ, và significand cho denorm
:label: tab-denorm
:align: center

| Tên trường | Biểu diễn | Số không chuẩn hóa |
| :--- | :-- | :--- |
| s | Dấu | 1 là âm; 0 là dương |
| exponent (số mũ) | `0000 0000` | Số mũ cho số không chuẩn hóa luôn ẩn là $-126$. |
| significand | Thành phần phân số của Mantissa | Diễn giải significand như một phân số 23-bit (`0.xx...xx`). **Không cộng thêm 1 ẩn** để có giá trị mantissa. |

:::

:::{warning} Tại sao nó được gọi là không chuẩn hóa?

Nhớ lại trực giác của chúng ta từ ký hiệu khoa học: số chuẩn hóa trong nhị phân phải có số 1 đứng đầu ẩn trong mantissa. Ngược lại, số không chuẩn hóa có biệt danh này vì chúng không có số 1 này.

Nói cách khác, denorm chắc chắn là ngoại lệ. Chỉ có $2^{23}$ trong số chúng, và chúng biểu diễn các số nhỏ trong phạm vi underflow không thể biểu diễn. Số chuẩn hóa cho đến nay là phần lớn các giá trị có thể biểu diễn, và khi chúng ta nói "float" chúng ta thường có nghĩa là số chuẩn hóa.

:::


"Số mũ ẩn" cho denorm là số mũ chuẩn hóa nhỏ nhất: $2^{1 - 127} = 2^{-126}$. Số mũ không chuẩn hóa này do đó thực thi một bước nhảy đồng đều $2^{-149}$ trên phạm vi không chuẩn hóa và các số chuẩn hóa nhỏ nhất[^at-home]. Sự nhất quán này cũng mang lại **underflow từ từ** mà chúng ta muốn, như được hiển thị trong @fig-underflow-gradual:

[^at-home]: Chúng tôi để bạn tự tìm hiểu điều này.

:::{figure} images/underflow-gradual.png
:label: fig-underflow-gradual
:width: 100%
:alt: "Gradual-underflow diagram showing denormalized values filling the region between zero and the smallest normalized value, with uniform tiny steps that connect smoothly to the normalized range."

Underflow từ từ bằng cách chỉ định **số không chuẩn hóa** trong chuẩn IEEE 754.
:::
