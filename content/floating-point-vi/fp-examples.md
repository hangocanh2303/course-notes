---
title: "Số chuẩn hóa: Thực hành"
---

## Mục tiêu học tập

* Thực hành chuyển đổi giữa định dạng IEEE 754 floating point độ chính xác đơn sang số thập phân.


::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/7MRtSYK1IOI
:width: 100%
:title: "[CS61C FA20] Lecture 06.4 - Floating Point: Examples, Discussion"
:::

::::

Trong khóa học này, chúng tôi kỳ vọng bạn có thể chuyển đổi giữa định dạng IEEE 754 Floating Point của một số và biểu diễn thập phân của chúng. Chúng tôi đưa ra một số ví dụ dưới đây.

Trong thực tế, bạn có thể và nên sử dụng các bộ chuyển đổi floating point. [Ứng dụng web này](https://www.h-schmidt.net/FloatConverter/IEEE754.html) cung cấp một bộ chuyển đổi tuyệt vời mà bạn có thể và nên sử dụng để khám phá các số ngoài những số được thảo luận dưới đây!

## Ví dụ 1: Floating Point sang Thập phân

:::{tip} Ví dụ 1

Số thập phân được biểu diễn bởi định dạng IEEE 754 floating point nhị phân độ chính xác đơn này là gì?

| s | số mũ | significand |
| :--: | :--: | :--: |
| `1` | `1000 0001` | `111 0000 0000 0000 0000 0000` |

* **A.** $-7 \times 2^{129}$
* **B.** -3.5
* **C.** -3.75
* **D.** 7
* **E.** -7.5
* **F.** Thứ gì khác

:::

:::{note} Hiển thị đáp án
:class: dropdown

**E.** -7.5.

Chúng ta tách 32 bit này thành các trường bit dấu (1 bit), số mũ (8 bit), và significand (23 bit) trước tiên và dịch từng phần riêng biệt.

* s: 1, nên dấu là âm
* số mũ: `1000 0001` là $128 + 1 = 129$, nên giá trị số mũ là $129 - 127 = 2$
* significand: `1110....0` là `111`, nên giá trị mantissa là `1.111` (cơ số 2)

Thay vào công thức của chúng ta, lưu ý rằng các thành phần là thập phân trừ khi được ghi chú khác với chỉ số dưới:

$$
\begin{align}
(-1)^\text{s} \times (1 + \text{significand})_{\text{hai}} \times 2^{(\text{số mũ}-127)} \\
= (-1)^1 \times (1 + .111)_{\text{hai}} \times 2^{(129-127)} \\
= -1 \times (1.111)_{\text{hai}} \times 2^2 \\
= -111.1_{\text{hai}} \\
= -7.5
\end{align}
$$

* Dòng thứ hai từ cuối: $(1.111)_{\text{hai}} \times 2^2$ liên quan đến việc di chuyển dấu chấm nhị phân sang trái hai vị trí, ví dụ: $111.1_{\text{hai}}$.
* Dòng cuối: Thành phần nguyên `111` là $7$; thành phần phân số `.1` là $\texttt{1} \times 2^{-1} = 1/2 = 0.5$.

Với những ai quan tâm, điều này có nghĩa là viết câu lệnh C `float x = -7.5;` dẫn đến `x` có mẫu bit `0xC0F00000`.

:::

## Ví dụ 2: Bước nhảy với độ chính xác hạn chế

Vì chúng ta có số bit cố định (độ chính xác), chúng ta không thể biểu diễn tất cả các số trong một phạm vi. Với số floating point, trường số mũ cho biết bước nhảy của chúng ta.

:::{tip} Ví dụ 2
Giả sử `y` có định dạng floating point dưới đây. **Bước nhảy** xung quanh `y` là bao nhiêu?

| s | số mũ | significand |
| :--: | :--: | :--: |
| `0` | `1000 0001` | `111 0000 0000 0000 0000 0000` |

_Gợi ý_: Xem xét sự khác biệt giữa mẫu bit của `y` và số có thể biểu diễn tiếp theo sau (hoặc trước) `y`.

:::

:::{note} Hiển thị đáp án
:class: dropdown

Chúng ta xem xét số có thể biểu diễn tiếp theo **sau** `y`. Điều này liên quan đến việc tăng significand thêm `1` ở bit có trọng số thấp nhất, tương ứng với mức tăng nhỏ nhất có thể ("bước nhảy"):

| s | số mũ | significand |
| :--: | :--: | :--: |
| `0` | `1000 0001` | `111 0000 0000 0000 0000 0001` |

Số mới này là `y + z`, với một bước nhảy nhỏ `z`:

$$
= y + \left( (.0...01)_{\text{hai}} \times 2^{(\text{số mũ}-127)} \right)\\
$$

Thay vì dịch `y` và số mới `y + z`, sau đó lấy hiệu của chúng, chúng ta thay vào đó lưu ý rằng chúng ta đang cố tìm chính `z`. Hãy tìm ra chính xác lũy thừa 2 mà `z` biểu diễn, với số mũ đã cho.

Bit có trọng số thấp nhất `.0....01`, cho bất kỳ mantissa nào của dạng chuẩn hóa nhị phân:

* Số 1 đứng đầu ẩn không được biểu diễn bởi bất kỳ bit nào trong 23 bit nhưng tương ứng với $2^0$
* bit 22 (bit có trọng số cao nhất) của significand là $2^{-1}$
* bit 0 (bit có trọng số thấp nhất) của significand là $2^{-23}$

Nói cách khác, với giá trị số mũ là $127 - 127 = 0$ (không có dịch chuyển), `z` sẽ là $2^{-23}$. Tuy nhiên, với trường số mũ của chúng ta, chúng ta dịch bit này sang lũy thừa thích hợp.

Giá trị số mũ cho `10000001` là $129 - 127 = 2$. Điều này dịch $2^{-23}$ sang phải hai vị trí. Bước nhảy `z` của chúng ta do đó là

$$\left(2^{-23} \times 2^2\right) = 2^{-21}.$$

:::

Số mũ lớn hơn có nghĩa là bước nhảy lớn hơn, và ngược lại. Đây thực sự là hành vi mong muốn: khi chúng ta có số siêu lớn, sự khác biệt phân số trở nên vô cùng nhỏ. Tuy nhiên, với số nhỏ, bước nhảy nhỏ hơn có giá trị hơn và độ chính xác của chúng ta (được biểu diễn bởi các bit của significand) phải hướng tới việc biểu diễn sự khác biệt.

## Ví dụ 3: Floating Point sang Thập phân

:::{tip} Ví dụ 3

Số thập phân được biểu diễn bởi định dạng IEEE 754 floating point nhị phân độ chính xác đơn này là gì?

| s | số mũ | significand |
| :--: | :--: | :--: |
| `0` | `0110 1000` | `101 0101 0100 0011 0100 0010` |

:::

::::{note} Hiển thị đáp án
:class: dropdown

$$1.666115 \times 2^{-23} \approx 1.986 \times 10^{-7}$$

<!-- TODO: change image to translated example -->

Giải thích (hiện tại) bằng hình ảnh (@fig-float-ex3):

:::{figure} images/float-ex3.png
:label: fig-float-ex3
:width: 100%
:alt: "Worked conversion for Example 3 from IEEE 754 binary to decimal, parsing sign, exponent, and significand from 0b00110100010101010100001101000010 and concluding the decimal answer is 1.986 times 10^-7."

Ví dụ 3, giải thích
:::

::::

## Ví dụ 4: Thập phân sang Floating Point


:::{tip} Ví dụ 4
$-2.340625 \times 10^1$ trong định dạng IEEE 754 floating point nhị phân độ chính xác đơn là gì?
:::

::::{note} Hiển thị đáp án
:class: dropdown

| s | số mũ | significand |
| :--: | :--: | :--: |
| `1` | `1000 0011` | `011 1011 0100 0000 0000 0000` |

<!-- TODO: change image to translated example -->

Giải thích (hiện tại) bằng hình ảnh (@fig-float-ex4):

:::{figure} images/float-ex4.png
:label: fig-float-ex4
:width: 100%
:alt: "Worked conversion for Example 4 from decimal -23.40625 to IEEE 754 binary fields, showing normalization, biasing the exponent, and resulting sign-exponent-significand bit pattern."

Ví dụ 4, giải thích
:::

::::

## Ví dụ 5: Thập phân sang Floating Point

Bài tập này cho thấy những hạn chế của biểu diễn chính xác sử dụng chuẩn IEEE 754 có _độ chính xác_ cố định. Xét cho cùng, độ chính xác cố định có nghĩa là chúng ta chỉ có 32 bit, và biểu diễn nhị phân đôi khi không đủ.

:::{tip} Ví dụ 5
$\frac{1}{3}$ trong định dạng IEEE 754 floating point nhị phân độ chính xác đơn là gì?
:::

::::{note} Hiển thị đáp án
:class: dropdown

| s | số mũ | significand |
| :--: | :--: | :--: |
| `0` | `0111 1101` | `010 1010 1010 1010 1010 1010` |

<!-- TODO: change image to translated example -->

Giải thích (hiện tại) bằng hình ảnh (@fig-float-ex5):

:::{figure} images/float-ex5.png
:label: fig-float-ex5
:width: 100%
:alt: "Worked example 5 showing representation of one-third in IEEE 754 floating point with repeating binary fraction 0.010101..., normalized form, exponent -2 plus bias, and truncated significand bits."

Ví dụ 5, giải thích
:::

::::
