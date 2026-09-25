---
title: "Ví dụ ngắn"
---

## Mục tiêu học tập

* Thực hành load và store.
* Dịch truy cập mảng trong mã C thành lệnh hợp ngữ (xem Ví dụ dài).

Không có video. Chúng tôi khuyến nghị mở [phần bộ nhớ](#tab-rv32i-memory) của [Green Card RISC-V](#sec-green-card).

## Ví dụ 1

Xét mã hợp ngữ:

(code-data-ex1)=
```
li x11 0x93F5 
sw x11 0(x5)
lb x12 1(x5)
```

Giả sử rằng bố cục bộ nhớ bắt đầu như trong @fig-rv-example-x12. Sau khi thực thi các lệnh này, giá trị trong `x12` là gì?

:::{figure} images/examplex12.png
:label: fig-rv-example-x12
:width: 100%
:alt: "Trạng thái ban đầu: các thanh ghi x5, x11, và x12 giữ 0x100, 0xABCDEFAB, và 0xCDEFABCD bên cạnh lưới bộ nhớ little-endian từ 0x100 đến 0x10C với các giá trị byte tương ứng."

Bố cục bộ nhớ bắt đầu cho [Ví dụ 1](#code-data-ex1).
:::

_Gợi ý_: Xem phần về [pseudoinstruction](#sec-pseudoinstructions) như `li`.

:::{note} Hiển thị đáp án
:class: dropdown

`R[x12]` là `0xFFFF FF93`.

:::

::::{note} Giải thích `li x11 0x93F5`
:class: dropdown

Load Immediate (`li rd imm`) là `addi rd x0 0x93F5`. Do đó pseudoinstruction ánh xạ đến `addi x11 x0 0x93F5`, nên đặt `R[x11]` (giá trị của thanh ghi `x11`) bằng các bit `0x000093F5`.

:::{figure} images/examplex12-sol1.png
:label: fig-rv-example-x12-sol1
:width: 100%
:alt: "Sau load immediate: x11 cập nhật thành 0x000093F5 trong khi x5 và x12 không thay đổi và bộ nhớ vẫn hiển thị lưới byte gốc."

Giải pháp (1/3) cho [Ví dụ 1](#code-data-ex1).
:::

::::

::::{note} Giải thích `sw x11 0(x5)`
:class: dropdown

Tính địa chỉ bộ nhớ như thanh ghi cơ sở + offset, hay `R[x5] + 0` = `0x100 + 0` = `0x100`. Store `R[x11]` (giá trị của `x11`) vào bộ nhớ tại địa chỉ `0x100`. Trên kiến trúc little endian (giả định) này, `0xF5` được lưu tại byte thấp nhất `0x100`.

:::{figure} images/examplex12-sol2.png
:label: fig-rv-example-x12-sol2
:width: 100%
:alt: "Sau store word: từ tại 0x100 được đánh dấu màu xanh là 0x000093F5 theo little-endian, khớp với giá trị cập nhật của x11, với các địa chỉ khác của bộ nhớ không thay đổi."

Giải pháp (2/3) cho [Ví dụ 1](#code-data-ex1).
:::

::::

::::{note} Giải thích `lb x12 1(x5)`
:class: dropdown

Tính địa chỉ bộ nhớ như thanh ghi cơ sở + offset, hay `R[x5] + 1` = `0x100 + 1` = `0x101`. Load byte `0x93` từ bộ nhớ tại địa chỉ `0x101` vào byte thấp nhất của thanh ghi `x12`.

`lb` có nghĩa là chúng ta phải mở rộng dấu. Bit trên cùng của 0x93 là 1, nên điền 24 bit trên cùng bằng `1`:

```
0x93 = 0b1001 0011
--> 0b1…1 1001 0011
--> 0xFFFF FF93
```

:::{figure} images/examplex12-sol3.png
:label: fig-rv-example-x12-sol3
:width: 100%
:alt: "Sau load byte: byte 0x93 tại offset cộng một từ 0x100 được đánh dấu và x12 giữ giá trị mở rộng dấu cập nhật 0xFFFFFF93 trong khi x5 và x11 giữ các giá trị trước."

Giải pháp (3/3) cho [Ví dụ 1](#code-data-ex1).
:::

**Giải pháp**: `R[x11]` (giá trị trong `x11`) là `0xFFFFFF93`.
::::

### Ví dụ 2

Giả sử rằng `x` và `y` là con trỏ `int *` mà giá trị của chúng ở trong thanh ghi `x3` và `x5`.

Làm thế nào chúng ta dịch câu lệnh `*x = *y;` thành hợp ngữ?

Xét các lệnh sau:

1. add x3 x5 zero
2. add x5 x3 zero
3. lw x3 0(x5)
4. lw x5 0(x3)
5. lw x8 0(x5)
6. sw x8 0(x3)
7. lw x5 0(x8)
8. sw x3 0(x8)

Và xét các lựa chọn sau:

* A. 1
* B. 2
* C. 3
* D. 4
* E. 5 → 6
* F. 6 -> 5
* G. 7 → 8
* H. Cái gì đó khác

:::{note} Đáp án (không có gì ở đây)

Chúng ta để giải pháp cho bạn, bây giờ!
:::
