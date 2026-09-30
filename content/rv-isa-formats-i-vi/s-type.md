---
title: "S-Type"
---

(sec-s-type-vi)=
## Mục Tiêu Học Tập

* Chuyển đổi qua lại giữa lệnh assembly S-type và lệnh máy.
* Giải thích tại sao các lệnh store `sb`, `sh`, và `sw` có định dạng lệnh riêng.
* Giải thích tại sao trường immediate `imm` được chia thành hai phần bit trong định dạng S-Type.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/JmxA8jF-LME
:width: 100%
:title: "[CS61C FA20] Lecture 11.5 - RISC-V Instruction Formats I: S-Format"
:::

::::

Định dạng lệnh tiếp theo là S-Type, định dạng được sử dụng cho các lệnh **S**tore (lưu trữ) `sb`, `sh`, và `sw`. Trong khi các lệnh load phù hợp với định dạng [I-Type](#sec-i-type-vi), các lệnh store sử dụng thanh ghi khác với các lệnh chúng ta đã thấy trước đó. Chúng ta cần một định dạng lệnh đặc biệt để chuyển đổi các lệnh store thành mã máy.

Nhớ lại từ [phần trước](#sec-store-word):

> Lệnh **store word** (lưu từ):
>
> * **Tính toán địa chỉ bộ nhớ** `R[rs1]+imm`
> * **Lưu một từ** từ thanh ghi `rs2`, `R[rs2][31:0]`, vào địa chỉ này trong bộ nhớ, `M[R[rs1] + imm][31:0]`.

Các lệnh [Store](#tab-rv32i-memory) do đó cần các trường sau:

* `rs1`: thanh ghi "cơ sở" chứa địa chỉ bộ nhớ cơ sở
* `rs2`: thanh ghi "nguồn" chứa dữ liệu cần lưu vào bộ nhớ
* `imm`: "offset" dạng bù 2 cộng với thanh ghi "cơ sở" để tạo địa chỉ bộ nhớ sử dụng trong store (địa chỉ store = R\[thanh ghi cơ sở\]) + (offset immediate)
* `opcode` (như tất cả các lệnh). Các lệnh Store dùng opcode `0100011`.
* `funct3` chỉ định các store từng phần.

Các lệnh store có hai thanh ghi toán hạng, giống như trong các lệnh [R-type](#sec-r-type-vi), nhưng không có thanh ghi đích `rd`. Thay vào đó, các lệnh S-type có giá trị immediate, giống như trong các lệnh [I-type](#sec-i-type-vi). Do đó, chúng ta sử dụng định dạng lệnh mới: S-type cho "Store"-type.

## S-Type: Các Trường

Định dạng lệnh S-Type là hàng thứ tư trong [bảng định dạng lệnh](#tab-rv32i-types) của [green card RISC-V](#sec-green-card).

Định dạng cho các lệnh `storeop rs2 imm(rs1)` được hiển thị trong @fig-s-type:

:::{figure} images/s-type.png
:label: fig-s-type-vi
:width: 100%
:alt: "RISC-V S-type cho stores: cú pháp là storeop rs2 imm(rs1) với imm được chia thành imm[11:5] và imm[4:0]. rs2 là thanh ghi nguồn dữ liệu, rs1 là thanh ghi nguồn địa chỉ cơ sở, và funct3 cùng opcode chỉ định loại store; một dấu ngoặc cho thấy hai phần immediate nối lại tạo thành imm[11:0] cộng với nội dung thanh ghi cơ sở để có địa chỉ bộ nhớ đích."

Định dạng Lệnh S-Type.
:::

**Toán hạng thanh ghi**: Chú ý rằng các trường thanh ghi `rs1` và `rs2` ở cùng vị trí trong S-Type và R-type; thiết kế có chủ đích này giúp giảm độ phức tạp phần cứng.

**Toán hạng hằng số**: Tương tự như I-Type, **trường immediate** `imm` chỉ định một giá trị immediate rộng 12 bit. Tuy nhiên, trong S-type, trường immediate được chia thành hai vị trí bit khác nhau. 5 bit thấp của immediate `imm[4:0]` nằm ở các bit `[11:7]` và 7 bit cao `imm[11:5]` nằm ở các bit `[31:25]` trong lệnh mã máy.

Immediate 12 bit vẫn là số nguyên bù hai với phạm vi $-2^{11} = -2048$ đến $2^{11} - 1 = 2047$, giống như trong I-type. Điểm khác biệt là bây giờ chúng ta phải chia tách các bit hoặc ghép chúng lại để khôi phục immediate, tùy thuộc vào việc chúng ta đang chuyển đổi sang mã máy hay sang mã assembly.

## Lệnh Assembly $\rightarrow$ Lệnh Máy

Xem xét việc chuyển đổi `sw x14 36(x2)` thành lệnh máy được hiển thị trong @fig-practice-sw.

:::{figure} images/practice-sw.png
:label: fig-practice-sw-vi
:width: 100%
:alt: "Mã hóa của sw x14 36(x2): imm[11:5] là 0b0000001, rs2 là 0b01110 cho thanh ghi x14, rs1 là 0b00010 cho thanh ghi x2, funct3 là 0b010, imm[4:0] là 0b00100, và opcode là 0b0100011, nối immediate đã chia và chuyển đổi các trường store cho word store."

Lệnh S-Type `sw x14 36(x2)`.
:::

Chúng ta tuân theo [các bước chuyển đổi assembly thành mã máy](#sec-assembly-to-machine) từ trước:

1. **Xác định mã trường thao tác**.

    * `opcode`: `0100011` cho tất cả các lệnh S-Type
    * `funct3`: `010` cho `sw`

1. **Chuyển đổi thanh ghi và immediate.**

    * `rs1`: Thanh ghi `x2`. Chuyển $2$ thành biểu diễn số nguyên không dấu 5 bit `0b00010`.
    * `rs2`: Thanh ghi `x14`. Chuyển $14$ thành biểu diễn số nguyên không dấu 5 bit `0b01110`.
    * `imm`: $36$ dạng bù hai 12 bit, chia thành 7 bit cao và 5 bit thấp:
        * $+36$ là `0b0000 0010 0100`.
        * Chia các bit offset thành vị trí bit cao và thấp như trong @fig-practice-sw-imm: bit cao `imm[11:5] = 0b0000001`, bit thấp `imm[4:0] = 0b00100`

1. (nếu cần) Chuyển đổi sang hệ thập lục phân.

    * Chúng tôi để đây như bài tập cho bạn!

:::{figure} images/practice-sw-imm.png
:label: fig-practice-sw-imm-vi
:width: 35%
:alt: "Bảng hai ô cho thấy cách offset 36 được đóng gói vào các trường immediate S-type: imm[11:5] là 0b0000001 và imm[4:0] là 0b00100, nối lại thành offset 12 bit."

Giá trị Immediate (offset) cho lệnh S-Type `sw x14 36(x2)`.
:::

## So sánh S-Type vs. I-Type vs. R-Type

@fig-R-I-S-type-comparison so sánh ba định dạng lệnh chúng ta đã thấy cho đến nay:

:::{figure} images/R-I-S-type-comparison.png
:label: fig-R-I-S-type-comparison-vi
:width: 100%
:alt: "Ba định dạng lệnh 32 bit so sánh R-type (funct7, rs2, rs1, funct3, rd, opcode), I-type (imm[11:0], rs1, funct3, rd, opcode), và S-type (imm[11:5], rs2, rs1, funct3, imm[4:0], opcode), với các vị trí chung cho opcode, funct3, và rs1 và mã màu biểu thị các trường thanh ghi và immediate khác nhau."

So sánh định dạng lệnh S-Type với định dạng lệnh I-type và R-Type.
:::

Một lần nữa, giống như trong I-Type, các lệnh S-Type mở rộng trường `imm` qua trường `funct7` của R-Type. Tuy nhiên, khác với I-Type, các lệnh store sử dụng thanh ghi toán hạng nguồn thứ hai `rs2` và cần sử dụng các bit `[24:20]` để mã hóa thanh ghi nào dùng để truy cập dữ liệu cần lưu vào bộ nhớ.

Tuy nhiên, nếu chúng ta chỉ có trường 7 bit thay thế `funct7`, thì chúng ta chỉ có thể biểu diễn 128 giá trị immediate. Vì các lệnh S-Type không ghi vào thanh ghi đích `rd`, chúng ta có thêm trường 5 bit để sử dụng biểu diễn trường `imm` 12 bit. Immediate trong các lệnh store đại diện cho một offset được cộng vào địa chỉ cơ sở. Các hằng số offset này thường ngắn và có thể vừa với trường `imm` 12 bit.

:::{note} Tại sao chia tách trường immediate trong S-Type?

Thiết kế ISA RISC-V ưu tiên giữ các trường **thanh ghi** ở cùng vị trí trong mã máy để các giá trị này dễ tìm khi xử lý lệnh. Chúng ta giữ các vị trí bit được sử dụng cho thanh ghi nguồn `rs1` và `rs2` (thanh ghi chúng ta đọc từ) và thanh ghi đích `rd` (thanh ghi chúng ta ghi vào) cố định.

Bộ xử lý biết:

* Nếu chúng ta đang đọc giá trị của một thanh ghi, thanh ghi nguồn `rs1` sẽ ở các bit `[19:15]`.
* Nếu chúng ta có thanh ghi nguồn thứ hai để đọc, `rs2` sẽ ở các bit `[24:20]` trong lệnh.
* Nếu chúng ta đang ghi giá trị vào thanh ghi đích, `rd` sẽ ở các bit `[11:7]`.

Các giá trị immediate ít quan trọng hơn và có thể di chuyển trong mã máy. Vì các lệnh S-Type không sử dụng trường `rd`, chúng ta có thể sử dụng các bit này để mở rộng không gian biểu diễn `imm`, nhưng chia thành hai vị trí bit `[31:25]` và `[11:7]`.

:::

Sự nhất quán giữa ba định dạng loại lệnh cho phép thiết kế phần cứng đơn giản!

## Tham Khảo cho Các Lệnh S-Type

Phần này dành làm tài liệu tham khảo cho các lệnh S-Type dựa trên những gì bạn đã học trong phần này.

Xem xét các lệnh S-Type trong @tab-s-type từ green card RISC-V.

:::{table} Các Lệnh RV32I: S-Type
:label: tab-s-type-vi
:align: center

| Lệnh | imm[11:5] | rs2 | rs1 | funct3 | imm[4:0] | opcode |
| :-- | -- | -- | -- | -- | -- | --: |
| `sb` | `imm[11:5]` | `rs2` | `rs1` | `000` | `imm[4:0]` | `0100011` |
| `sh` | `imm[11:5]` | `rs2` | `rs1` | `001` | `imm[4:0]` | `0100011` |
| `sw` | `imm[11:5]` | `rs2` | `rs1` | `010` | `imm[4:0]` | `0100011` |
:::

Nhớ rằng các lệnh store ghi vào bộ nhớ với chỉ các byte được chỉ định. Chúng ta không cần mở rộng dấu/logic.

:::{note} Trường `funct3`

Chú ý rằng trường `funct3` trong các lệnh S-Type khớp chính xác với các lệnh load sử dụng định dạng I-Type.

* Store/load **byte** (8 bit): `sb` và `lb` đều có trường `funct3` là `000`.
* Store/load **half word** (16 bit): `sh` và `lh` đều có trường `funct3` là `001`.
* Store/load **word** (32 bit): `sw` và `lw` đều có trường `funct3` là `010`
:::


<!--

### TODO: đặt cái này ở đâu đó

:::{warning} Tại sao chúng ta dùng 17 bit để chỉ định thao tác?

Nhớ rằng chỉ có 10 lệnh R-Type, 17 lệnh I/I*-Type, và 3 lệnh S-Type! Trên toàn bộ tập lệnh cơ sở RV32I, chắc chắn có ít lệnh hơn $2^32$ thứ có thể biểu diễn được trong một từ 32 bit, vì vậy một số dư thừa là không thể tránh khỏi.

Quan trọng hơn, chúng ta sẽ thấy rằng để đơn giản hóa thiết kế kiến trúc, các định dạng lệnh khác nhau **tái sử dụng cùng vị trí bit cho cùng các trường khi có thể**. Các trường thanh ghi `rs1`, `rs2`, và `rd` do đó được ưu tiên, và các trường như `funct3` và `funct7` chiếm các bit còn lại.

:::

-->
