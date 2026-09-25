---
title: "Các lệnh số học RISC-V II: Phép toán bit"
short_title: "Phép toán bit"
---

(sec-rv-bitwise)=
## Mục tiêu học tập

* Viết hợp ngữ để thực hiện các phép toán bit.
* Hiểu tại sao chỉ có ba phép dịch bit RISC-V: `sll`, `srl`, và `sra` (và các phiên bản immediate tương ứng `slli`, `srli`, `srai`).

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/WQQlcC3btmM
:width: 100%
:title: "[CS61C FA20] Lecture 09.1 - RISC-V Decisions II: Logical Instructions"
:::

::::

Chúng tôi khuyến nghị xem lại [Phép toán bit trong C](#sec-c-bitwise-ops) trước khi tiếp tục.

Chúng ta đã [thảo luận trước đó](#sec-hll-vs-assembly) rằng trong RISC-V, các phép toán xác định "kiểu", tức là cách nội dung thanh ghi được xử lý (xem [bảng này](#tab-hll-vs-assembly)). Tiếp theo, chúng ta sẽ thấy cách khái niệm này áp dụng cho tập lệnh RISC-V cho các phép toán bit.

## Phép toán bit

Như [trước đây](#sec-bitwise-ops-defined), **phép toán bit** được thực hiện trên các toán hạng n-bit **từng bit một**.

ISA RV32I cung cấp các lệnh cho các phép toán bit phổ biến.[^green-card] @tab-bitwise cho thấy hầu hết các phép toán bit tương ứng với hai lệnh:

* **RISC-V: Thanh ghi**. Thực hiện phép toán bit trên hai toán hạng thanh ghi `rs1` và `rs2`, và lưu kết quả trong thanh ghi đích `rd`.
* **RISC-V: Immediate**. Thực hiện phép toán bit trên một toán hạng thanh ghi `rs1` và một immediate `imm`, và lưu kết quả trong thanh ghi đích `rd`.

[^green-card]: Xem tập hợp đầy đủ các lệnh số học trên [green card RISC-V](#sec-green-card).

Trong @tab-rv-bitwise bên dưới, di chuột qua mỗi chú thích để nhảy đến phần tương ứng trên trang này.

:::{table} Các lệnh số học bit RISC-V.
:label: tab-rv-bitwise
:align: center

| Phép toán bit | RISC-V: Thanh ghi | RISC-V: Immediate |
| :--- | :--- | :--- |
| **AND** | `and rd rs1 rs2` | `andi rd rs1 imm` |
| **OR** | `or rd rs1 rs2` | `ori rd rs1 imm` |
| **XOR** | `xor rd rs1 rs2` | `xori rd rs1 imm` |
| **NOT**[^not] | `not rd rs1` (pseudo) | |
| **Dịch trái**[^sll] | `sll rd rs1 rs2` | `slli rd rs1 imm` |
| **Dịch phải**[^srl-sra] | `srl rd rs1 rs2` <br> `sra rd rs1 rs2` | `srli rd rs1 imm` <br> `srai rd rs1 imm` |

[^not]: Xem [pseudoinstruction `not`](#sec-rv32i-not).

[^sll]: Xem [dịch trái](#sec-rv32i-sll).

[^srl-sra]: Xem [dịch phải](#sec-rv32i-srl-sra).

:::

(sec-rv32i-not)=
### Pseudoinstruction `not`

Trong RISC-V, NOT bit là một [pseudoinstruction](#sec-pseudoinstructions) và tương ứng với XOR bit với immediate `-1`:

:::{table} NOT là XOR với -1.
:align: center

| Pseudoinstruction | Tên | Mô tả | Dịch |
| :--- | :--- | :--- | :--- |
| `not rd rs1` | NOT bit | `R[rd] = ~(R[rs1])` | `xori rd rs1 -1` |
:::

Ghi chú:

* NOT bit của giá trị `R[rs1]` được định nghĩa là đảo ngược bit của tất cả 32 bit của thanh ghi `rs1`.
* Nhớ lại từ thảo luận về [tính chất XOR](#tab-bitwise-props) rằng với một bit đơn `x`, biểu thức `x XOR 1` (`x ^ 1`) **đảo ngược** `x`.
* Immediate `-1` có biểu diễn số nguyên có dấu bù hai 32-bit là `0b 1111 1111 1111 1111 1111 1111 1111 1111`.

Ba ghi chú này cùng nhau giải thích @fig-rv32i-not bên dưới.

:::{figure} images/rv32i-not.png
:label: fig-rv32i-not
:width: 50%
:alt: "Ba mẫu 32-bit được căn chỉnh có nhãn rs1, trừ một, và rd hiển thị cách phép XOR với immediate toàn một đảo ngược mọi bit của rs1, biến một giá trị kết thúc bằng 0111 thành kết quả kết thúc bằng 1000 trong thanh ghi đích rd."

NOT bit tương đương với XOR với immediate toàn một.
:::

:::{note} Hiển thị giải thích
:class: dropdown

* Thanh ghi nguồn `rs1` có giá trị `0b 1111 1111 1111 1111 1111 1111 1111 0111`.
* Thanh ghi đích `rd` có giá trị `0b 0000 0000 0000 0000 0000 0000 0000 1000`.
* Theo định nghĩa, phép toán này vừa là NOT (đảo ngược bit) và XOR với biểu diễn bù hai 32-bit của `-1`.

:::

:::{warning} Tại sao không có lệnh `noti`?

Giả định, một lệnh `noti` sẽ đảo ngược các bit của một immediate và lưu nó vào thanh ghi. Điều này tương đương với việc chỉ định immediate đã đảo ngược ngay từ đầu, nên RISC-V không "dành" một lệnh cho phép toán này.

:::

(sec-rv32i-sll)=
### Dịch trái

Giống như tất cả các lệnh số học RISC-V, phép dịch trái `sll` phải ghi tất cả 32 bit của thanh ghi đích. Nhớ lại thảo luận về [phép dịch trái](#sec-left-shift): biểu thức `x << n` dịch các bit của `x` sang trái `n` bit, điền `n` bit thấp hơn bằng zero. Do đó phép toán `sll` điền các bit mới này bằng `0`.

:::{warning} Tại sao không có lệnh `sla`?

`sll` là viết tắt của **S**hift **L**eft **L**ogical (Dịch trái logic). Xem [ghi chú này](#sec-why-shift-left) để biết tại sao dịch trái logic và dịch trái số học là tương đương.

:::

(sec-rv32i-srl-sra)=
### Dịch phải

Nhớ lại thảo luận về [phép dịch phải](#sec-right-shift): biểu thức `x >> n` dịch các bit của `x` sang phải `n` bit, điền `n` bit thấp hơn bằng zero hoặc một. Trong C, điều này được xác định bởi **kiểu** của `x`. Trong RISC-V, **lệnh** xác định các bit thấp hơn được điền bằng gì.

* `srl`, hoặc **S**hift **R**ight **L**ogical (Dịch phải logic) (`srli` cho immediate). "Mở rộng zero" và điền các bit cao hơn bằng `0`. Lệnh này hiệu quả giải thích nội dung thanh ghi `rs1` như một số nguyên không dấu. Đọc thêm trong [phần trước](#sec-right-shift-logical).
* `sra`, hoặc **S**hift **R**ight **A**rithmetic (Dịch phải số học) (`srai` cho immediate). Điền các bit cao hơn bằng bit dấu của thanh ghi `rs1`. Lệnh này hiệu quả giải thích nội dung thanh ghi `rs1` như một số nguyên có dấu. Đọc thêm trong [phần trước](#sec-right-shift-arithmetic).

## Các lệnh số học RISC-V khác

Phép nhân tổng quát không được bao gồm trong ISA RISC-V cơ bản nhưng được chỉ định như một phần của các extension RISC-V phổ biến. Xem lệnh `mul` trên [green card RISC-V](#tab-rv32i-extension).

Mạch cho phép nhân tổng quát phức tạp hơn đáng kể so với các phép dịch trái và phải bit được thảo luận ở trên. Vì lý do tương tự, chúng ta không thảo luận về phép chia, modulo, và các phép toán số dấu phẩy động.[^curious]

[^curious]: Chúng tôi khuyến khích bạn đọc RISC-V unprivileged ISA cho [M Extension](https://docs.riscv.org/reference/isa/unpriv/m-st-ext.html) và [F extension](https://docs.riscv.org/reference/isa/unpriv/f-st-ext.html).

## Thực hành

:::{tip} Thực hành
Sau khi các lệnh bên dưới được thực thi, giá trị trong thanh ghi `x12` là gì?

```{code} bash
:linenos:
li    x10 0x34FF
slli  x12 x10 0x10
srli  x12 x12 0x08
and   x12 x12 x10
```

* **A.** `0x0`
* **B.** `0x3400`
* **C.** `0x4F0`
* **D.** `0xFF00`
* **E.** `0x34FF`
* **F.** Cái gì đó khác

:::

:::{note} Hiển thị đáp án
:class: dropdown

**B.** `0x3400`.

Các lệnh này là bản dịch RISC-V của [ví dụ thực hành C](#sec-c-bitwise-practice) khi chúng ta thảo luận về các phép toán bit trong C. Chúng tôi khuyến nghị xem lại điều đó trước.

Mỗi dòng, giải thích:

1. Ghi giá trị `0x000034FF` vào thanh ghi `x10`.

2. `R[x10] << 0x10` là `0x34FF0000`. Ghi giá trị này vào thanh ghi `x12`.

3. `R[x12] >> 0x8` là `0x0034FF00` vì bit dấu của `R[x12]` ban đầu là `0`.

4. `R[x12] & R[x10]` là `0x000034FF & 0x0034FF00` là `0x00003400`. Xem @fig-bit-shift từ [ví dụ thực hành C](#sec-c-bitwise-practice).

:::
