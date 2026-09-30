---
title: "U-Type"
subtitle: "lui và auipc"
---

(sec-u-type-vi)=
## Mục Tiêu Học Tập

* Giải thích tại sao U-Type cần thiết cho việc xây dựng immediate rộng.
* Chuyển đổi giả lệnh `li` (Load Immediate) thành mã máy.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/e6WlS-QsNZc
:width: 100%
:title: "[CS61C FA20] Lecture 12.2 - RISC-V Instruction Formats II: Upper Immediates"
:::

::::

Nhớ lại rằng các lệnh số học [I-Type](#sec-i-type-vi) mã hóa immediate 12 bit như số nguyên bù hai (tức là có dấu); immediate này sau đó được mở rộng dấu thành giá trị 32 bit trước khi thực hiện thao tác (ví dụ: cộng với thanh ghi). Trong khi các hằng số thường ngắn và vừa với trường 12 bit I-Type, chúng ta không thể tránh khỏi việc cần mã hóa **immediate rộng** để xây dựng hằng số số lớn hơn `imm` 12 bit của I-type.

RISC-V đạt được immediate rộng như sau:

* Chỉ định hằng số 32 bit sử dụng **hai lệnh**: một lệnh số học I-Type cho **immediate thấp** 12 bit, tức là bit 0 đến 11; và một lệnh khác cho **immediate cao** 20 bit, tức là bit 12 đến 31. (Một lần nữa, tập lệnh **rút gọn**!)
* Định nghĩa một định dạng lệnh, **U-Type** ("U" cho Upper immediate), có thể được sử dụng cho các lệnh chỉ định immediate cao.

Có hai lệnh U-Type: `lui` và `auipc`. Chúng nằm trong phần "Other" của [green card RISC-V](#tab-rv32i-other).

## Load Upper Immediate (`lui`)

Từ sách giáo khoa P&H (Chương 2.10), một lệnh như vậy, `lui`, là **L**oad **U**pper **I**mmediate:

> ...nạp một hằng số 20 bit vào các bit 12 đến 31 của thanh ghi. 12 bit bên phải được điền bằng số không.

Trong @tab-lui, chúng ta gọi immediate cao này là `immu` và immediate 32 bit là `imm`.

:::{table} Lệnh `lui`.
:label: tab-lui-vi
:align: center

| Lệnh | Tên | Mô Tả |
| :--- | :--- | :--- |
| `lui rd immu` | Load Upper Immediate | `imm = immu << 12`<br/>`R[rd] = imm` |
:::

Như được hiển thị trong @fig-u-type-immu, `imm = immu << 12`; hằng số số 32 bit `imm` này sau đó được ghi vào thanh ghi `rd`.

:::{figure} images/u-type-immu.png
:label: fig-u-type-immu-vi
:width: 50%
:alt: "Định dạng lệnh U-Type 32 bit: trường 20 bit immu nằm ở bit 31–12, bit 11–0 là zero, và immediate đầy đủ imm được tạo thành bằng cách nối immu với mười hai zero ở vị trí bit ít quan trọng nhất."

Với lệnh U-Type, `immu` là 20 bit cao của hằng số số rộng 32 bit `imm`.
:::

(sec-li-lui-vi)=
### Giả Lệnh Load Immediate

Với lệnh mới này, chúng ta có thể quay lại chuyển đổi giả lệnh `li`, Load Immediate. Khi chúng ta [giới thiệu](#sec-pseudoinstructions) load immediate, bạn có thể đã nhận thấy chú thích trong @tab-mv-li:

> Mô tả này chưa đầy đủ với phạm vi của `imm` trong `addi`. Xem [green-card](@tab-rv32i-pseudoinstructions) và [phần sau](#sec-li-lui-vi) để có bản dịch đầy đủ của load immediate.

Nói cách khác, khi `li` phải sử dụng hằng số số rộng hơn 12 bit được chứa bởi trường `imm` của `addi` I-Type, nó được chuyển đổi thành **hai** lệnh: `lui` và `addi`.

| Giả Lệnh | Tên | Mô Tả | Chuyển Đổi |
| :--- | :--- | :--- | :--- |
| `li rd imm` | Load Immediate | `R[rd] = imm` | `lui` (nếu cần), `addi` |

Nói cách khác, `li` chuyển đổi thành `addi` khi immediate nằm trong khoảng -2048 đến 2047. Với bất cứ gì lớn hơn, `li` chuyển đổi thành **hai** lệnh: `lui` và `addi`.

Nói chung, trình biên dịch hoặc assembler sẽ chia các hằng số lớn qua các lệnh `lui` và `addi`, khi cần. Sau cùng, có một số **lưu ý** quan trọng cho quá trình này. Đưa chuột qua hai chú thích bên dưới trước khi tiếp tục.

* `lui` để đặt 20 bit cao[^lui-caveat]
* `addi` để "đặt" 12 bit thấp[^addi-caveat]

[^lui-caveat]: Và đặt 12 bit thấp thành zero
[^addi-caveat]: "Đặt" nghĩa là, cộng immediate 12 bit **được mở rộng dấu**

:::{warning} Chuyển đổi `li` thành lệnh thực

`addi` mở rộng dấu của immediate 12 bit của nó. Xem Ví dụ 2 bên dưới. Hãy cẩn thận!
:::

:::{tip} Ví Dụ 1
Chuyển đổi `li x10 0x87654321` thành lệnh thực.

Giải pháp:

```bash
lui  x10 0x87654
addi x10 x10 0x321
```

:::

:::{note} Hiện Giải Thích
:class: dropdown

1. `lui` đặt `x10` thành `0x87654000`. Chú ý `0x87654` là 20 bit.
2. `addi` đặt `x10` thành `0x87654321 = 0x87654000 + 0x00000321`, trong đó toán hạng sau là phiên bản mở rộng dấu của immediate 12 bit `0x321`.

:::

:::{tip} Ví Dụ 2: Trường Hợp Đặc Biệt
Chuyển đổi `li x10 0xB0BACAFE` thành lệnh thực.

Không hoạt động:

```bash
lui  x10 0xB0BAC
addi x10 x10 0xAFE
```

Giải pháp:

```bash
lui  x10 0xB0BAD
addi x10 x10 0xAFE
```
:::

::::{note} Hiện Giải Thích
:class: dropdown

Hãy xem xét strawman[^strawman] trước, **không hoạt động**:

[^strawman]: Wikipedia: [Straw Man](https://en.wikipedia.org/wiki/Straw_man)

```bash
lui  x10 0xB0BAC
addi x10 x10 0xAFE
```

Nhớ rằng, lệnh `addi` **mở rộng dấu** immediate 12 bit `imm`. Nếu bit dấu của `imm` được đặt, thì `imm` là số âm, **trừ 1** từ 20 bit cao được đặt bởi lệnh `lui` trước đó.

1. `lui` đặt `x10` thành `0xB0BAC000`.
2. `addi` đặt `x10` thành `0xB0BAC000 + 0xFFFFFAFE`, vì immediate 12 bit `0xAFE` là số âm bù hai. `x10` sau đó được đặt thành `0xB0BABAFE`.

@fig-li-lui-strawman "dễ thương"[^pain] minh họa "trừ một" này sử dụng số học thập lục phân và các tính chất đại số.

[^pain]: Với một số định nghĩa của "dễ thương"; ở đây là "toán thú vị"

:::{figure} images/li-lui-strawman.png
:label: fig-li-lui-strawman-vi
:width: 40%
:alt: "Ví dụ số học cho thấy phép cộng giá trị thập lục phân 0xB0BAC000 với 0xFFFFFAFE để tạo ra từ kết quả 0xB0BABAFE. Phép tính được tách thành phép cộng của ba nibble ít quan trọng nhất với ba số 0, và 5 nibble quan trọng nhất cộng với -1, được biểu diễn bằng năm F thập lục phân."

Khi immediate `imm` của `addi` có dấu, immediate `immu` của `lui` ngây thơ sẽ cho hằng số số không chính xác.
:::

Để giải quyết điều này, biết rằng `addi` sẽ luôn theo sau `lui` khi giải quyết giả lệnh `li`, và `addi` sẽ luôn mở rộng dấu immediate `imm` của nó. Vì vậy nếu trường `imm` 12 bit của `addi` là số âm, **chủ động cộng 1** vào trường `immu` 20 bit của `lui`:

```bash
lui  x10 0xB0BAD
addi x10 x10 0xAFE
```

1. `lui` đặt `x10` thành `0xB0BAD000`.
2. `addi` đặt `x10` thành `0xB0BAD000 + 0xFFFFFAFE`, vì immediate 12 bit `0xAFE` là số âm bù hai. `x10` sau đó được đặt thành `0xB0BACAFE`.

::::

## Add Upper Immediate to PC

Sẽ có sớm. Được sử dụng cho [lệnh J-Type](#sec-j-type-vi).

:::{table} Lệnh `auipc`.
:label: tab-auipc-vi
:align: center

| Lệnh | Tên | Mô Tả |
| :--- | :--- | :--- |
| `auipc rd immu` | Add Upper Imm to PC | `imm = immu << 12`<br/>`R[rd] = PC + imm` |
:::

## U-Type: Các Trường

Để chuyển đổi `lui` và `auipc` thành mã máy, chúng ta cần định dạng lệnh hỗ trợ immediate 20 bit.
Định dạng lệnh U-Type chỉ được sử dụng bởi `lui` và `auipc` và là một trong các hàng cuối của [bảng định dạng lệnh](#tab-rv32i-types) của [green card RISC-V](#sec-green-card).

:::{figure} images/u-type.png
:label: fig-u-type-vi
:width: 100%
:alt: "Bố cục định dạng lệnh U-type với cú pháp opname rd immu và các trường imm[31:12] (bit 31–12), rd (bit 11–7), và opcode (bit 6–0); các ghi chú bổ sung chỉ định rằng immu là 20 bit cao của immediate 32 bit được tạo ra bằng cách dịch trái trường đó 12 bit."

Định Dạng Lệnh U-Type.
:::

U-Type chỉ có ba trường (@fig-u-type):

* **Toán hạng thanh ghi**: Trường U-Type `rd` là thanh ghi đích để đặt immediate.

* **Opcode**: Trường U-Type `opcode` chỉ định thao tác: `auipc` hoặc `lui` (@tab-u-type).

* **Toán hạng hằng số**: **Trường upper immediate** (chúng ta gọi là `immu`) chỉ định giá trị immediate 20 bit. `immu` được **dịch trái** 12 bit để tạo thành 20 bit cao của hằng số số 32 bit `imm` (@fig-u-type-immu).

:::{table} Các Lệnh RV32I: U-Type
:label: tab-u-type-vi
:align: center

| Lệnh | imm[31:12] ("immu") | rd | opcode |
| :-- | :-- | :-- | :-- |
| `auipc` | `imm[31:12]` | `rd` | `0010111` |
| `lui` | `imm[31:12]` | `rd` | `0110111` |
:::
