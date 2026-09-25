---
title: "Tóm tắt"
---

## Và để kết luận$\dots$


Chuẩn IEEE 754 định nghĩa một biểu diễn nhị phân cho các giá trị floating point sử dụng ba trường.

:::{figure} #fig-float
:alt: "Reprint of the 32-bit IEEE 754 single-precision layout: sign bit, exponent field, and fraction bits ordered from most significant to least significant as in the floating-point representation section."
Biểu diễn Floating Point Độ chính xác đơn (32-bit) (in lại từ @fig-float từ [phần này](#sec-fp)). Bit ngoài cùng bên trái là bit có trọng số cao nhất; bit ngoài cùng bên phải là bit có trọng số thấp nhất.
:::

* *Dấu* xác định dấu của số ($0 $ cho dương, $1 $ cho âm).
* *Số mũ* ở dạng ký hiệu thiên lệch. Đối với số floating point độ chính xác đơn, thiên lệch là $127$, đến từ $(2^{(8−1)} −1)$. Đối với số floating point độ chính xác kép, thiên lệch là $1023$.
  * Số mũ `00000000` biểu diễn *số không*, nếu có mantissa bằng không, hoặc *số không chuẩn hóa*, nếu có mantissa khác không.
  * Số mũ `11111111` biểu diễn *NaN*, nếu có mantissa khác không, hoặc *vô cực*, nếu có mantissa bằng không.
* *Significand* được sử dụng để lưu trữ một **phân số** thay vì số nguyên và đề cập đến các bit bên phải của số "`1`" đứng đầu khi chuẩn hóa. Ví dụ, nếu một mantissa là `1.010011`, significand của nó là `010011`.


* Đối với [float chuẩn hóa](#sec-normalized):

$$\text{Giá trị} = (−1)^{\text{Dấu}} × 2^{\text{Số mũ}-\text{Thiên lệch}} × 1.\text{Significand}_2$$

* Đối với [float không chuẩn hóa](#sec-denorms), bao gồm [số không](#sec-zero):

$$\text{Giá trị} = (−1)^{\text{Dấu}} × 2^{\text{Số mũ}-\text{Thiên lệch}+1} × 0.\text{Significand}_2$$

Khi chuyển đổi giữa các giá trị floating point nhị phân và thập phân, chúng ta phải nhớ rằng có một thiên lệch cho số mũ.

:::{figure} #tab-float-exp-fields
:alt: "Reprint of the table summarizing IEEE 754 single-precision exponent encodings for normalized, denormalized, infinity, and NaN cases from the special floating-point values section."
Trường số mũ IEEE 754 độ chính xác đơn có giá trị từ $0$ đến $255 (in lại từ @tab-float-exp-fields từ [phần này](#sec-special-floats)).
:::

## Tài liệu đọc thêm

P&H 3.5, 3.9

## Tham khảo bổ sung

* [Trình mô phỏng IEEE 754](https://www.h-schmidt.net/FloatConverter/IEEE754.html)

## Bài tập
Kiểm tra kiến thức của bạn!

### Ôn tập khái niệm

:::{exercise}
:label: fp-01
**Đúng hay Sai**: Ý tưởng của floating point là sử dụng khả năng di chuyển dấu cơ số (thập phân) đến bất kỳ đâu để biểu diễn một phạm vi rộng các số thực một cách chính xác nhất có thể.
:::

:::{solution} fp-01
:label: fp-01-sol
:class: dropdown

**Đúng.** Floating point:

* Cung cấp hỗ trợ cho một phạm vi rộng các giá trị. (Cả rất nhỏ và rất lớn)
* Giúp lập trình viên xử lý lỗi trong số học thực vì floating point có thể biểu diễn $+\infty$, $-\infty$, $\text{NaN}$ (Không phải số)
* Giữ độ chính xác cao. Nhớ rằng precision là số bit trong một word máy tính
được sử dụng để biểu diễn một giá trị. IEEE 754 phân bổ phần lớn bit cho significand, cho phép
sử dụng kết hợp các lũy thừa âm của hai để biểu diễn phân số.

<!--See: [Lecture 2 Slide 13](https://docs.google.com/presentation/d/1dmCk2fZz-P8VedzAXnVmJiYPKszVka5NKmTuLJ6hqZc/edit?slide=id.g2af3b38b3e2_1_154#slide=id.g2af3b38b3e2_1_154)-->
:::

:::{exercise}
:label: fp-02
**Đúng hay Sai**: Floating Point và Bù hai có thể biểu diễn cùng tổng số lượng số (bất kỳ số thực, số nguyên, v.v.) với cùng số bit.
:::

:::{solution} fp-02
:label: fp-02-sol
:class: dropdown

**Sai.** Floating Point có thể biểu diễn vô cực cũng như NaN, vì vậy tổng số lượng số có thể biểu diễn
thấp hơn Bù hai, trong đó mọi tổ hợp bit ánh xạ đến một giá trị số nguyên duy nhất.
:::


:::{exercise}
:label: fp-03
**Đúng hay Sai**: Khoảng cách giữa các số floating point tăng khi giá trị tuyệt đối của số tăng.
:::

:::{solution} fp-03
:label: fp-03-sol
:class: dropdown

**Đúng.** Khoảng cách không đều là do biểu diễn số mũ của số floating point. Có một số bit cố định trong significand. Trong lưu trữ IEEE $32$-bit có $23 $ bit cho significand, có nghĩa là LSB biểu diễn $2^{−23}$ nhân 2 mũ số mũ. Ví dụ, nếu số mũ là không (sau khi cho phép offset) thì sự khác biệt giữa hai float liền kề sẽ là $2^{−23}$. Nếu số mũ là $8 $, sự khác biệt giữa hai float liền kề sẽ là $2^{−15}$ vì mantissa được nhân với $2 ^{8}$. Precision hạn chế làm cho số floating point nhị phân không liên tục; có khoảng trống giữa chúng.

:::

:::{exercise}
:label: fp-04
**Đúng hay Sai**: Phép cộng Floating Point có tính kết hợp.
:::

:::{solution} fp-04
:label: fp-04-sol
:class: dropdown
**Sai.** Vì lỗi làm tròn, bạn có thể tìm thấy số Lớn và Nhỏ sao cho: `(Nhỏ + Lớn) + Lớn != Nhỏ + (Lớn + Lớn)`

FP xấp xỉ kết quả vì nó chỉ có 23 bit cho significand.
:::

:::{exercise}
:label: fp-05
Tại sao ký hiệu khoa học chuẩn hóa luôn bắt đầu bằng 1 trong cơ số 2?
:::

:::{solution} fp-05
:label: fp-05-sol
:class: dropdown
Một chữ số khác không được yêu cầu trước dấu cơ số trong ký hiệu khoa học, và vì chữ số khác không duy nhất
trong cơ số 2 là 1, giá trị chuẩn hóa sẽ luôn bắt đầu bằng 1.
:::
