---
title: "Tóm tắt"
---

## Và Tóm lại$\dots$

* Chúng ta biểu diễn "thứ" trong máy tính như các mẫu bit cụ thể:
  * Với $N$ bits, bạn có thể biểu diễn tối đa $2^N$ thứ.
* Hôm nay, chúng ta đã thảo luận năm encoding khác nhau cho số nguyên:
  * Số nguyên không dấu (Unsigned integers)
  * Số nguyên có dấu:
    * Sign-Magnitude
    * Ones' Complement
    * Two's Complement
  * Bias Encoding
* Kiến trúc sư máy tính đưa ra quyết định thiết kế để làm phần cứng đơn giản
  * Unsigned và Two's complement là chuẩn C. Học chúng!!
* Integer overflow: Kết quả của phép toán số học nằm ngoài phạm vi có thể biểu diễn của số nguyên.
  * Số có vô số chữ số, nhưng máy tính có độ chính xác hữu hạn. Điều này có thể dẫn đến lỗi số học. Thêm sau!

Bài học tổng kết: Chúng ta đưa ra quyết định thiết kế để làm **phần cứng đơn giản**. Chúng ta loại bỏ **sign magnitude** và **ones' complement** vì phần cứng sẽ khó. Nhưng đây là bí mật: phần cứng cho toán học trên **số không dấu và two's complement** là giống nhau. Sự khác biệt duy nhất là cách bạn tính overflow.

<!--Để bạn xem xét:
Làm sao chúng ta có thể biểu diễn -12.75?-->

## Đọc thêm từ Textbook

P&H: 2.4

## Tài liệu tham khảo bổ sung

[Dan Garcia's Binary Slides, Spring 2021](https://inst.eecs.berkeley.edu/~cs61c/sp21/resources-pdfs/garcia_binary_slides.pdf)

Amazing Illustrations by Ketrina (Yim) Thompson: [CS Illustrated](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2009/EECS-2009-79.html) Number Rep Handouts

* [Comparing Binary Integer Representations](https://csillustrated.berkeley.edu/PDFs/handouts/integer-representations-1-handout.pdf)
* [Negation and Zeroes](https://csillustrated.berkeley.edu/PDFs/handouts/integer-representations-2-comparing-handout.pdf)
* [Increments and Monotonicity](https://csillustrated.berkeley.edu/PDFs/handouts/integer-representations-3-comparing-handout.pdf)
* [The Thrilling Conclusion!](https://csillustrated.berkeley.edu/PDFs/handouts/integer-representations-4-comparing-handout.pdf)

## Bài tập
Kiểm tra kiến thức!

### Ôn tập Khái niệm

:::{exercise}
:label: num-01
1. Bit là gì? Có bao nhiêu bits trong một byte? Nibble?
:::

:::{solution} num-01
:label: num-01-sol
:class: dropdown

Bit là đơn vị nhỏ nhất của thông tin số và nó có thể là 0 hoặc 1. Có 4 bits trong một nibble và 8 bits trong một byte.

<!--Xem: [Lecture 2 Slide 13](https://docs.google.com/presentation/d/1dmCk2fZz-P8VedzAXnVmJiYPKszVka5NKmTuLJ6hqZc/edit?slide=id.g2af3b38b3e2_1_154#slide=id.g2af3b38b3e2_1_154)-->
:::

:::{exercise}
:label: num-02
2. Overflow là gì?
:::

:::{solution} num-02
:label: num-02-sol
:class: dropdown

Khi kết quả của phép toán số học nằm ngoài phạm vi có thể biểu diễn bằng số bits cho trước.

<!--Xem: [Lecture 2 Slide 26](https://docs.google.com/presentation/d/1dmCk2fZz-P8VedzAXnVmJiYPKszVka5NKmTuLJ6hqZc/edit?slide=id.g2af3b38b3e2_1_186#slide=id.g2af3b38b3e2_1_186)-->
:::

:::{exercise}
:label: num-03
3. Phạm vi số có thể biểu diễn bằng unsigned, sign-magnitude, one's complement, two's complement, và biased notation $n$-bit là gì?
:::

:::{solution} num-03
:label: num-03-sol
:class: dropdown

* **Unsigned**: $[0, 2^n-1]$
* **Sign-Magnitude**: $[-(2^{n-1} - 1), 2^{n-1} - 1]$
* **One's complement**: $[-(2^{n-1} - 1), 2^{n-1} - 1]$
* **Two's complement**: $[-2^{n-1}, 2^{n-1} - 1]$
* **Bias**: $[0-$bias$, 2^n-1-$bias$]$

<!--Xem:  [Lecture 2](https://docs.google.com/presentation/d/1dmCk2fZz-P8VedzAXnVmJiYPKszVka5NKmTuLJ6hqZc/edit?slide=id.g32e4dda2ba9_0_123#slide=id.g32e4dda2ba9_0_123)-->
:::

:::{exercise}
:label: num-04
4. Các biểu diễn này có bao nhiêu cách biểu diễn số không, unsigned, sign-magnitude, one's complement, two's complement, và biased notation $n$-bit?
:::

:::{solution} num-04
:label: num-04-sol
:class: dropdown
* **Unsigned**: 1
* **Sign-Magnitude**: 2
* **One's complement**: 2
* **Two's complement**: 1
* **Bias**: 1 hoặc 0 (tùy thuộc vào bias)

<!--Xem: [Lecture 2](https://docs.google.com/presentation/d/1dmCk2fZz-P8VedzAXnVmJiYPKszVka5NKmTuLJ6hqZc/edit?slide=id.g32e4dda2ba9_0_123#slide=id.g32e4dda2ba9_0_123)-->
:::

### Bài tập ngắn

:::{exercise}
:label: num-05
1. **Đúng/Sai**: Tùy thuộc vào ngữ cảnh, cùng một dãy bits có thể biểu diễn những thứ khác nhau.
:::

:::{solution} num-05
:label: num-05-sol
:class: dropdown
**Đúng.** Cùng một dãy bits có thể được hiểu theo nhiều cách khác nhau với cùng một dãy bits! Các bits có thể biểu diễn bất cứ thứ gì từ số không dấu đến số có dấu hoặc thậm chí, như chúng ta sẽ đề cập sau, một chương trình. Tất cả phụ thuộc vào cách hiểu đã thỏa thuận.
:::

:::{exercise}
:label: num-06
2. **Đúng/Sai**: Nếu bạn hiểu một số Two's complement $N$-bit như một số không dấu, số âm sẽ nhỏ hơn số dương.
:::

:::{solution} num-06
:label: num-06-sol
:class: dropdown
**Sai.** Trong Two's Complement, MSB luôn là 1 cho số âm. Điều này có nghĩa là MỌI số âm trong Two's Complement, khi chuyển đổi sang không dấu, sẽ lớn hơn các số dương.
:::

:::{exercise}
:label: num-07
3. **Đúng/Sai**: Chúng ta có thể biểu diễn phân số và số thập phân trong các định dạng biểu diễn số cho trước (unsigned, biased, và Two's Complement).
:::

:::{solution} num-07
:label: num-07-sol
:class: dropdown
**Sai.** Các định dạng biểu diễn hiện tại có một hạn chế lớn; chúng ta chỉ có thể biểu diễn và làm số học với số nguyên. Để biểu diễn thành công các giá trị phân số cũng như số có độ lớn cực cao vượt quá giới hạn hiện tại, chúng ta cần một định dạng biểu diễn khác.
:::

:::{exercise}
:label: num-08
4. Có bao nhiêu số có thể được biểu diễn bởi một số không dấu, cơ số 4, $n$-digit.

    **A.** 1

    **B.** $2^n - 1$

    **C.** $4^n$

    **D.** $4^{n-1}$

    **E.** $4^n - 1$
:::

:::{solution} num-08
:label: num-08-sol
:class: dropdown
**C.**
:::

:::{exercise}
:label: num-09
5. Cần bao nhiêu bits để biểu diễn số thập phân 116 trong nhị phân?
:::

:::{solution} num-09
:label: num-09-sol
:class: dropdown
**7 bits**. $(116)_{10} =$ `0b111 0100` hoặc $log{_2}{116} \approx 6.85$ làm tròn thành 7 bits.
:::
