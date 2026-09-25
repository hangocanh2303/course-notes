---
title: "Số chuẩn hóa"
---

(sec-fp)=
## Mục tiêu học tập

* Hiểu cách biểu diễn số chuẩn hóa trong chuẩn IEEE 754 floating point độ chính xác đơn được lấy cảm hứng từ ký hiệu khoa học.
* Xác định cách định dạng IEEE 754 floating point độ chính xác đơn sử dụng ba trường:
  * Bit dấu biểu diễn dấu
  * Trường số mũ biểu diễn giá trị số mũ.
  * Trường significand biểu diễn phần thập phân của giá trị mantissa.
* Đối với số chuẩn hóa:
  * Trường số mũ biểu diễn giá trị số mũ dưới dạng số được mã hóa thiên lệch (bias-encoded)
  * Mantissa luôn có số 1 đứng đầu ẩn.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/GzOMIRj1yO0
:width: 100%
:title: "[CS61C FA20] Lecture 06.2 - Floating Point: Floating Point"
:::

Đã bỏ qua (cho mục đích của phần tiếp theo): Overflow và Underflow, 6:54 - 8:40

::::

## Trực giác: Ký hiệu khoa học

Thay vì xác định vị trí dấu chấm nhị phân cố định cho mọi số trong biểu diễn của chúng ta, mô hình **floating point** xác định vị trí dấu chấm nhị phân cho _mỗi_ số.

Xem xét số thập phân $0.1640625$ có biểu diễn nhị phân[^exercise]

[^exercise]: Chúng tôi để việc chuyển đổi này như một bài tập cho người đọc.

$$
\dots\texttt{000000.001010100000}\dots.
$$

Có nhiều số không trong biểu diễn nhị phân này. Ví dụ, không có thành phần nguyên, vì vậy mọi thứ bên trái dấu chấm nhị phân đều là không. Thực tế, thực sự chỉ có một phạm vi "thú vị" với một số "năng lượng", ví dụ: các số một và không thay đổi: `10101`. Mẫu bit này nằm ở vị trí hai giá trị bên phải của dấu chấm nhị phân.

Với hai mảnh thông tin này—các **bit có nghĩa** là gì, và **số mũ** nào mà các bit có nghĩa được liên kết với—chúng ta đột nhiên có thể biểu diễn cả các số rất lớn và rất nhỏ. Đây là trực giác đằng sau **ký hiệu khoa học**.

### Ký hiệu khoa học (Cơ số 10)

Bạn có thể đã thấy ký hiệu khoa học trong khóa vật lý hoặc hóa học, nhưng chúng tôi ôn lại ở đây và giới thiệu thuật ngữ cốt lõi được chuyển sang trường hợp nhị phân. Ký hiệu khoa học cho số $0.1640625$ là $1.640625 \times 10^{-1}$ (@fig-scientific-notation-base10):

:::{figure} images/scientific-notation-base10.png
:label: fig-scientific-notation-base10
:width: 40%
:alt: "Decimal scientific-notation diagram labeling mantissa and exponent for 1.640625 times 10^-1, with arrows marking the decimal point and base-10 radix."

Ký hiệu khoa học giả định dạng **chuẩn hóa**, trong đó mantissa có đúng một chữ số khác không bên trái của dấu thập phân.
:::

* **Cơ số (Radix)**: Cơ số. Trong thập phân, cơ số 10.
* **Mantissa**: "Năng lượng" của số, tức là các [chữ số có nghĩa](https://en.wikipedia.org/wiki/Significant_figures) (1.640625 trong @fig-scientific-notation-base10)
* **Số mũ (Exponent)**: Lũy thừa mà cơ số được nâng lên ($-1$ trong @fig-scientific-notation-base10).

Ký hiệu khoa học giả định **dạng chuẩn hóa** của các số, trong đó mantissa có đúng một chữ số bên trái của dấu thập phân.

Mỗi số được biểu diễn trong ký hiệu khoa học có đúng **một dạng chuẩn hóa** cho một số chữ số có nghĩa nhất định. Ví dụ, số $1/1000000000$ có dạng chuẩn hóa (cho hai chữ số có nghĩa) $1.0 \times 10^{-9}$ và các dạng không chuẩn hóa $0.1 \times 10^{-8}, 10 \times 10^{-10}$, v.v.

### Dạng chuẩn hóa nhị phân

Số $0.1640625$ ở dạng chuẩn hóa nhị phân là $1.0101_{\text{hai}} \times 2^{-3}$ (@fig-scientific-notation-base2):

:::{figure} images/scientific-notation-base2.png
:label: fig-scientific-notation-base2
:width: 40%
:alt: "Binary normalized-notation diagram labeling mantissa and exponent for 1.0101 base two times 2^-3, with arrows marking the binary point and base-2 radix."

Trong nhị phân, chúng ta cũng giả định dạng **chuẩn hóa**, trong đó mantissa có đúng một chữ số khác không bên trái của dấu chấm nhị phân.
:::

* **Cơ số (Radix)**: Cơ số trong nhị phân là cơ số 2.
* **Mantissa**: $1.0101$ trong @fig-scientific-notation-base2.
* **Số mũ (Exponent)**: $-3$ trong @fig-scientific-notation-base2.

Chúng tôi kết thúc thảo luận này với hai điểm quan trọng:

* Một **biểu diễn floating point** 32-bit của dạng chuẩn hóa nhị phân nên phân bổ hai trường riêng biệt trong 32 bit—một để biểu diễn mantissa, và một để biểu diễn số mũ.
* Với dạng chuẩn hóa nhị phân, mantissa **luôn có số 1 đứng đầu**.

:::{warning} Dạng chuẩn hóa nhị phân có nghĩa là mantissa luôn có dạng `1.xyz...`!
:class: dropdown

Tại sao? Xem xét định nghĩa của dạng chuẩn hóa:

> Mantissa có đúng một chữ số khác không bên trái của dấu thập phân.

Trong nhị phân, chỉ có một chữ số khác không: `1`. Điều này có nghĩa là một biểu diễn như $0.1101_{hai} \times 2^{2}$ không được chuẩn hóa và phải được chia _xuống_ một lũy thừa của 2 thành dạng chuẩn hóa của nó, $1.101_{hai} \times 2^{1}$.

:::

## IEEE 754 Floating Point Độ chính xác đơn

Thảo luận này dẫn chúng ta đến định nghĩa của **IEEE 754 Floating Point Độ chính xác đơn (Single-Precision)**, được sử dụng cho kiểu biến `float` trong C. Định dạng này tận dụng dạng chuẩn hóa nhị phân để biểu diễn một phạm vi rộng các số cho sử dụng khoa học sử dụng **32 bit**.[^double]

[^double]: IEEE 754 Floating Point Độ chính xác kép được sử dụng cho kiểu biến `double` trong C, là 64 bit. Đọc thêm trong [phần bonus](#sec-double).

Chuẩn này được tiên phong bởi Giáo sư William ("Velvel") Kahan của UC Berkeley. Trước hệ thống của Kahan, hệ sinh thái để biểu diễn floating point rất hỗn loạn, và các tính toán trên một máy sẽ cho kết quả khác trên máy khác. Bằng cách dẫn đầu nỗ lực tập trung số học floating point vào chuẩn IEEE 754, Giáo sư Kahan [đã nhận giải thưởng Turing năm 1988](https://people.eecs.berkeley.edu/~wkahan/ieee754status/754story.html).

Ba trường trong chuẩn IEEE 754 được thiết kế để tối đa hóa **độ chính xác** của việc biểu diễn nhiều số sử dụng cùng **độ chính xác** hạn chế của 32 bit (cho độ chính xác đơn, và 64 bit cho độ chính xác kép). Điều này dẫn chúng ta đến hai định nghĩa quan trọng để giúp chúng ta định lượng hiệu quả của biểu diễn số này:

* **Precision (Độ chính xác bit)** là số bit được sử dụng để biểu diễn một giá trị.
* **Accuracy (Độ chính xác giá trị)** là sự khác biệt giữa giá trị thực của một số và biểu diễn máy tính của nó.

Không cần nói thêm, @fig-float định nghĩa ba trường trong IEEE 754 floating point độ chính xác đơn cho 32 bit.

:::{figure} images/float.png
:label: fig-float
:width: 100%
:alt: "IEEE 754 single-precision field layout across 32 bits: 1 sign bit at the most significant position, 8 exponent bits, and 23 significand bits."

Các trường bit trong IEEE 754 floating point độ chính xác đơn. Bit có trọng số thấp nhất (ngoài cùng bên phải) được đánh chỉ số 0; bit có trọng số cao nhất (ngoài cùng bên trái) được đánh chỉ số 31.
:::

(sec-normalized)=
## Số chuẩn hóa

Ba trường trong @fig-float có thể được sử dụng để biểu diễn số chuẩn hóa có dạng trong Phương trình @eq-float-normalized.

(eq-float-normalized)=
:::{math}
:enumerated: true
(-1)^\text{s} \times (1 + \text{significand}) \times 2^{(\text{số mũ}-127)}
:::

Thiết kế này sẽ có vẻ khó hiểu lúc đầu. Tại sao có $1+$? Mantissa đi đâu rồi—significand là gì? Tại sao số mũ được dịch bởi $-127$? Như đã nói trước đó, ba trường này được định nghĩa để tối đa hóa độ chính xác của các số có thể biểu diễn trong 32 bit. Nói cách khác, _không có mẫu bit nào trong ba trường này sử dụng bù hai_! Thay vào đó, chuẩn chỉ định cách diễn giải mẫu bit của mỗi trường để biểu diễn số chuẩn hóa (@tab-float):

:::{table} Các trường dấu, số mũ, và significand cho số chuẩn hóa.
:label: tab-float
:align: center

| Tên trường | Biểu diễn | Số chuẩn hóa[^exp-norm] |
| :--- | :-- | :--- |
| s | Dấu | 1 là âm; 0 là dương |
| exponent (số mũ) | Số mũ mã hóa thiên lệch | Trừ 127 từ trường số mũ để có giá trị số mũ. |
| significand | Thành phần phân số của Mantissa | Diễn giải significand như một phân số 23-bit (`0.xx...xx`) và cộng 1 để có giá trị mantissa. |

:::

[^exp-norm]: Chỉ hợp lệ khi trường số mũ nằm trong khoảng `0000001` (1) đến `1111110` (254), tức là khi nó không phải 0 cũng không phải 255.

:::{note} Tại sao sử dụng số mũ mã hóa thiên lệch?
:class: dropdown

Các nhà thiết kế muốn số floating point được hỗ trợ ngay cả khi phần cứng floating point chuyên dụng không tồn tại. Ví dụ, vẫn có thể sắp xếp các số floating point chỉ bằng các phép so sánh số nguyên, miễn là chúng ta biết trường bit nào nên được xem xét.

Để hỗ trợ sắp xếp các số cùng dấu chỉ với phần cứng số nguyên, trường số mũ lớn hơn nên biểu diễn số lớn hơn. Với bù hai, số âm sẽ trông lớn hơn (vì chúng bắt đầu bằng `1`, bit dấu). Với mã hóa thiên lệch, mặt khác, tất cả số floating point được sắp xếp theo giá trị của số mũ: trường số mũ `000...000` biểu diễn giá trị số mũ nhỏ nhất (các số là $\times 2^{-127}$), và trường số mũ `111....111` biểu diễn giá trị số mũ lớn nhất (các số là $\times 2^{128}$).

:::

:::{note} Tại sao không biểu diễn trực tiếp mantissa? Tại sao giả định có số 1 ẩn?
:class: dropdown

Nhớ rằng trong dạng chuẩn hóa nhị phân, mantissa _luôn_ bắt đầu bằng 1. IEEE 754 biểu diễn số chuẩn hóa bằng cách giả định rằng _luôn_ có số 1 ẩn, sau đó significand biểu diễn rõ ràng các bit sau dấu chấm nhị phân. Nói cách khác, luôn đúng là với số chuẩn hóa, 0 < significand < 1.

Giả định này cho số chuẩn hóa giúp IEEE 754 độ chính xác đơn đóng gói nhiều số (chuẩn hóa) có thể biểu diễn hơn vào cùng 23 bit, vì bây giờ chúng ta biểu diễn mantissa (chuẩn hóa) 24-bit! Nói cách khác, độ chính xác là 24 bit, mặc dù chúng ta chỉ sử dụng 23 bit.
:::

## Số không, Vô cực, và Nhiều hơn nữa

[Số chuẩn hóa](#sec-normalized) không phải là giá trị duy nhất có thể được biểu diễn bởi chuẩn IEEE 754. Chúng ta thảo luận về số không, vô cực, và các số khác trong [phần sau](#sec-special-floats).

## Sử dụng bộ chuyển đổi Floating Point

Hãy xem [ứng dụng web này](https://www.h-schmidt.net/FloatConverter/IEEE754.html) để có bộ chuyển đổi đơn giản giữa số thập phân và định dạng IEEE 754 floating point độ chính xác đơn của chúng.


(sec-double)=
## IEEE 754 Floating Point Độ chính xác kép

Chuẩn IEEE 754 floating point độ chính xác kép được sử dụng cho kiểu biến `double` trong C. Nó có ba trường, bây giờ trên 64 bit:

* Dấu: Vẫn 1 bit dấu (bit có trọng số cao nhất, chỉ số bit 63)
* Số mũ: 11 bit với thiên lệch 1023
* Significand: bây giờ 52 bit

Ưu điểm chính là độ chính xác lớn hơn do significand lớn hơn. Dạng chuẩn hóa có thể biểu diễn các số từ khoảng $2.0 \times 10^{-308}$ đến $2.0 \times 10^{308}$.

Mặc dù chúng tôi rất muốn bạn sử dụng bộ chuyển đổi để hiểu comic trong @fig-smbc-float, cần lưu ý rằng robot đang giả định định dạng IEEE 754 **độ chính xác kép** (xem [phần khác](#sec-double)).

:::{figure} images/smbc-float.png
:label: fig-smbc-float
:alt: "SMBC comic where a robot login screen asks 0.1 + 0.2 and accepts 0.30000000000000004, illustrating floating-point rounding behavior."
:align: center
:width: 50%

Chào mừng đến với Internet Robot Bí mật ([SMBC Comics](https://www.smbc-comics.com/comic/2013-06-05)).
:::

Vì floating point dựa trên lũy thừa của hai, nó không thể biểu diễn hầu hết các phân số thập phân một cách chính xác. Ở đây, 0.3 được biểu diễn không chính xác là `0.29999999999`, và $0.1 + 0.2$ với `double` là `0.30000000000000004`. Đọc thêm về `double`, phép cộng, và độ chính xác trong [phần khác](#sec-float-discussion).
