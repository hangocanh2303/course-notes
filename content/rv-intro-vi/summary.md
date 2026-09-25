---
title: "Tóm tắt"
---

## Và để kết luận$\dots$

### Các lệnh RISC-V

RISC-V là một ngôn ngữ hợp ngữ bao gồm các lệnh đơn giản mà mỗi lệnh thực hiện một tác vụ duy nhất như cộng hai số hoặc lưu dữ liệu vào bộ nhớ. Dưới đây là so sánh giữa mã RISC-V và mã C tương đương:

```
//x trong s0, &y trong s1

addi s0, x0, 5  // int x = 5;
sw s0, 0(s1)    // y[0] = x;
mul t0, s0, s0
sw t0, 4(s1)    // y[1] = x * x;
```

Để tham khảo, các bảng bên dưới hiển thị một số lệnh số học/bit cơ bản cũng có thể được tìm thấy trên [thẻ tham khảo 61C](https://cs61c.org/sp26/pdfs/resources/reference-card.pdf).

Dưới đây là các chữ viết tắt sẽ được sử dụng trong bảng:
* `rs1`: Thanh ghi đối số 1
* `rs2`: Thanh ghi đối số 2
* `rd`: Thanh ghi đích
* `imm`: Giá trị immediate (hằng số nguyên)
* `R[register]`: Giá trị chứa trong thanh ghi
* `inst`: Một trong các lệnh trong bảng

:::{figure} #tab-add-sub
:alt: "In lại bảng lệnh số học R-type và I-type cơ bản RV32I bao gồm add, sub, và các biến thể immediate từ phần số học RISC-V."
Các lệnh số học cơ bản (in lại từ @tab-add-sub từ [phần này](#sec-rv-arithmetic)).
:::

:::{figure} #tab-rv-bitwise
:alt: "In lại bảng lệnh logic bit và dịch RV32I với các dạng thanh ghi và immediate từ phần bit."
Các lệnh bit cơ bản (in lại từ @tab-rv-bitwise từ [phần này](#sec-rv-bitwise)).
:::

"Immediate" RISC-V là bất kỳ hằng số nào. Ví dụ, `addi t0, t0, 20`, `sw a4, -8(sp)`, và `lw a1, 0x44(t2)` có các immediate `20`, `-8`, và `0x44` tương ứng. Lưu ý rằng có giới hạn về kích thước (số bit) của immediate trong bất kỳ lệnh nào (phụ thuộc vào loại lệnh, sẽ nói thêm sớm!). Bạn cũng có thể thấy rằng có một "i" ở cuối một số lệnh, như `addi`, `slli`, v.v. Điều này có nghĩa là `rs2` trở thành một "immediate" hoặc một số nguyên thay vì thanh ghi. Có các immediate trong các lệnh sử dụng offset như `sw` và `lw`. Khi viết mã RISC-V, luôn sử dụng [thẻ tham khảo 61C](https://cs61c.org/sp26/pdfs/resources/reference-card.pdf) để biết chi tiết của mỗi lệnh (thẻ tham khảo là bạn của bạn)!

## Bài đọc giáo trình

P&H 2.1-2.3

## Tài liệu tham khảo bổ sung

Xem các liên kết hướng dẫn RISC-V trên [trang green card RISC-V](#sec-green-card) của chúng ta.

## Bài tập
Kiểm tra kiến thức của bạn!

### Ôn tập khái niệm

:::{exercise}
:label: rv-01
1. **Đúng hay Sai**: Kiểu được liên kết với khai báo trong C (thông thường), nhưng được liên kết với các lệnh (toán tử) trong RISC-V.
:::

:::{solution} rv-01
:label: rv-01-sol
:class: dropdown

Đúng. Xem @tab-hll-vs-assembly.

:::

:::{exercise}
:label: rv-02
2. **Đúng hay Sai**: Vì chỉ có 32 thanh ghi, chúng ta không thể viết RISC-V cho các biểu thức C chứa > 32 biến.
:::

:::{solution} rv-02
:label: rv-02-sol
:class: dropdown

Sai. Chúng ta đã thấy [một số ví dụ](#sec-arithmetic-examples) về cách chia nhỏ các phương trình dài thành các phương trình nhỏ hơn.
:::

:::{exercise}
:label: rv-03
3. **Đúng hay Sai**: Nếu `p` (được lưu trong `x9`) là con trỏ đến mảng `int`, thì `p++;` sẽ được dịch thành `addi x9,x9, 1`.
:::

:::{solution} rv-03
:label: rv-03-sol
:class: dropdown

Sai. Đừng quên rằng `int` có kích thước `4` byte trên RV32I, nên lệnh sẽ là `addi x9, x9, 4`.
:::
