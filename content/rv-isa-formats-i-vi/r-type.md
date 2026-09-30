---
title: "Định dạng lệnh R-Type"
short_title: "R-Type"
---

(sec-r-type-vi)=
## Mục tiêu học tập

* Xác định loại định dạng lệnh bằng trường `opcode`.
* Dịch qua lại giữa các lệnh hợp ngữ R-type và các lệnh máy.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/MpFv2_nuXv4
:width: 100%
:title: "[CS61C FA20] Lecture 11.2 - RISC-V Instruction Formats I: R-Format Layout"
:::

::::

Giống như với [số dấu phẩy động IEEE 754](#fig-float), 32 bit của một lệnh máy được chia thành các trường. Giống như dấu phẩy động, mỗi trường có một **tên** và chiếm một tập hợp vị trí bit nhất định (và do đó có độ rộng cố định). Không giống dấu phẩy động, tên và vị trí các trường phụ thuộc vào **loại lệnh**.

Ví dụ, một lệnh R-type được hiển thị trong @fig-opcode-field. Trường có tên `opcode` chiếm các bit ít quan trọng nhất (vị trí bit 0 đến 6); trường `opcode` rộng 7 bit. Cũng lưu ý rằng lệnh hợp ngữ `opname rd rs1 rs2` chia sẻ _một số_ thuật ngữ với các trường lệnh, _nhưng không phải tất cả_. Ví dụ, `rs1`, `rs2`, và `rd` là tên trường, nhưng `opname` thì không.

:::{figure} images/opcode-field.png
:label: fig-opcode-field
:width: 100%
:alt: "Mẫu để đọc một từ lệnh RISC-V 32-bit: các chú thích hiển thị vị trí bit, tên trường (ví dụ opcode), và độ rộng trường; thanh được phân chia thành funct7, rs2, rs1, funct3, rd, và opcode từ bit 31 xuống đến 0."

Tên và vị trí trường phụ thuộc vào định dạng lệnh.
:::


:::{warning} Các định dạng lệnh chỉ định trường nào ở đâu!
Các định dạng lệnh khác nhau có thể có các trường khác nhau. Đôi khi, các định dạng khác nhau có thể chỉ định _vị trí khác nhau_ cho cùng tên trường!
:::

:::{hint} Sử dụng `opcode` để xác định định dạng lệnh!

May mắn thay, tất cả các [định dạng lệnh](#tab-rv32i-types) RISC-V dành riêng 7 bit thấp nhất cho trường `opcode`. Mỗi định dạng lệnh được gán một tập hợp giá trị opcode **riêng biệt**.

Khi dịch một lệnh máy sang hợp ngữ, phần cứng nhìn vào `opcode` 7-bit **trước tiên** để xác định loại định dạng lệnh. **Một khi loại định dạng được xác định**, phần cứng sau đó biết cách xử lý phần còn lại của lệnh.
:::

## R-Type: Các trường

Định dạng lệnh R-Type là hàng đầu tiên của [bảng định dạng lệnh](#tab-rv32i-types) của [green card RISC-V](#sec-green-card). Tất cả **các lệnh số học thanh ghi-thanh ghi sử dụng R-Type** ("R" là viết tắt của Register). Bây giờ chúng ta sử dụng "số học" để bao gồm các phép toán số học và bit: `add`, `xor`, `sll`, v.v.
Chúng tôi khuyến nghị bạn tham chiếu [bảng các lệnh số học](#tab-rv32i-arithmetic) khi bạn khám phá định dạng lệnh R-Type bên dưới.

@fig-r-type (@fig-opcode-field với ít chú thích hơn) hiển thị định dạng R-Type. Lưu ý rằng tất cả các lệnh số học thanh ghi-thanh ghi tuân theo cùng cú pháp lệnh hợp ngữ `opname rd rs1 rs2`.

:::{figure} images/r-type.png
:label: fig-r-type
:width: 100%
:alt: "Bố cục R-type RISC-V mô tả định dạng hợp ngữ: opname rd rs1 rs2 phía trên một thanh 32-bit với các trường funct7 (bit 31–25), rs2 (bit 24–20), rs1 (bit 19–15), funct3 (bit 14–12), rd (bit 11–7), và opcode (bit 6–0), với rs1 và rs2 là thanh ghi nguồn và rd là thanh ghi đích."

Định dạng lệnh R-Type.
:::

**Toán hạng thanh ghi**: Ba trường R-Type có tên `rs1`, `rs2`, `rd` ánh xạ đến các toán hạng lệnh hợp ngữ tương đương của chúng. Mỗi trường là một số nguyên không dấu 5-bit ($0$ đến $31$) tương ứng với số thanh ghi (`x0-x31`). Tên thanh ghi (ví dụ: `a0`) được dịch trước thành số thanh ghi của chúng (ví dụ: `x10`), sau đó đến mẫu bit của chúng (ví dụ: `01010`).

* `rs1`: Thanh ghi "Nguồn", toán hạng thứ nhất
* `rs2`: Thanh ghi "Nguồn", toán hạng thứ hai
* `rd`: Thanh ghi "Đích" nhận kết quả của phép tính số học.

**Các trường khác**: Phép toán lệnh hợp ngữ `opname` được ánh xạ qua ba trường:

* `opcode`: Tất cả các lệnh R-type có cùng opcode 7-bit: `0110011`.
* `funct3`, `funct7`: Phép toán số học cần thực hiện. Trường `funct3` rộng 3 bit; `funct7` rộng 7 bit.

:::{warning} Tại sao chúng ta sử dụng 17 bit để chỉ định phép toán/opcode?
Trong toàn bộ tập lệnh cơ bản RV32I, chắc chắn có ít lệnh hơn $2^{32}$ thứ có thể biểu diễn trong một từ 32-bit, nên một số dư thừa là không thể tránh khỏi. Thiết kế tốt đòi hỏi các tiền đề tốt.

Các định dạng lệnh khác nhau **tái sử dụng cùng vị trí bit cho cùng trường bất cứ khi nào có thể**. Giữ các định dạng lệnh càng giống nhau càng tốt giảm độ phức tạp phần cứng. Các trường thanh ghi `rs1`, `rs2`, và `rd` do đó được ưu tiên, và các trường opcode như `funct3` và `funct7` chiếm các bit còn lại.

:::

## Lệnh hợp ngữ $\rightarrow$ Lệnh máy

Xét @fig-rtype-add-example, dịch `add x18 x19 x10` thành một lệnh máy.

:::{figure} images/rtype-add-example.png
:label: fig-rtype-add-example
:width: 100%
:alt: "Mã hóa của add x18 x19 x10: funct7 là 0b0000000, rs2 là 0b01010 cho thanh ghi x10, rs1 là 0b10011 cho thanh ghi x19, funct3 là 0b000, rd là 0b10010 cho thanh ghi x18, và opcode là 0b0110011, với các mũi tên liên kết add với funct7 và funct3 và tên thanh ghi với các trường năm bit của chúng."

Lệnh R-Type `add x18 x19 x10`.
:::

(sec-assembly-to-machine)=
:::{note} Dịch các lệnh từ hợp ngữ sang mã máy

1. **Xác định loại định dạng lệnh**. Sử dụng [green card RISC-V](#sec-green-card).

1. **Xác định mã trường phép toán**. Sử dụng [green card RISC-V](#sec-green-card) để xác định `opcode` và (nếu áp dụng) `funct3` và `funct7`.

1. **Dịch các thanh ghi, immediate, v.v.**

1. (nếu cần cho khả năng đọc của con người) **Chuyển đổi sang thập lục phân**.

:::

1. **Xác định loại định dạng lệnh**. `add` là R-type vì nó thực hiện số học trên hai toán hạng thanh ghi. Chúng ta sử dụng [bảng các lệnh số học](#tab-rv32i-arithmetic) trên green card RISC-V.

1. **Xác định mã trường phép toán**.

    * `opcode`: `0110011` (cho tất cả các lệnh R-Type).
    * `funct3`: `000` cho `add`
    * `funct7`: `0000000` cho `add`

1. **Dịch các thanh ghi, immediate, v.v.**

    * `rs1`: Thanh ghi `x19`. Dịch $19$ thành biểu diễn số nguyên không dấu 5-bit `10011`.
    * `rs2`: Thanh ghi `x10`. Dịch $10$ thành biểu diễn số nguyên không dấu 5-bit `01010`.
    * `rd`: Thanh ghi `x18`. Dịch $18$ thành biểu diễn số nguyên không dấu 5-bit `10010`.

1. (nếu cần) Chuyển đổi sang thập lục phân.

    * Chúng tôi để đây như một bài tập cho bạn!

## Lệnh máy $\rightarrow$ Lệnh hợp ngữ

Các bước để dịch mã máy thành hợp ngữ:

1. Nếu cần, chuyển đổi sang nhị phân. Sau đó, tìm opcode và sử dụng nó để xác định loại định dạng lệnh.
1. Chia thành các trường theo loại định dạng lệnh.
1. Đối với R-Type, xác định `opname` lệnh hợp ngữ với các trường `funct3`, `funct7`.
1. Đối với R-Type, xác định các toán hạng thanh ghi và (nếu cần) tên thanh ghi.

Hai bước cuối cùng là **đặc thù cho R-Type**. Nói chung, bạn luôn nên làm hai bước đầu tiên, bất kể loại định dạng lệnh. Sau đó, thực hiện các chuyển đổi đặc thù theo loại để tái tạo lệnh hợp ngữ gốc.

:::{tip} Kiểm tra nhanh

Dịch từ này thành một lệnh RISC-V:
`0x01B342B3`.

Dạng nhị phân: `0b0000 0001 1011 0011 0100 0010 1011 0011`.

* **A.** `slt  x5  x6 x27`
* **B.** `slt x27  x5  x6`
* **C.** `slt  x5 x27  x6`
* **D.** `xor s11  t0  t1`
* **E.** `xor  t0  t1 s11`
* **F.** `xor  t0 s11  t1`
* **G.** Cái gì đó khác
:::

:::{note} Hiển thị đáp án
:class: dropdown

**E.** `xor  t0  t1 s11`

:::

Giải thích:

1. Chúng ta đã chuyển đổi `0x01B342B3` sang nhị phân: `0b0000 0001 1011 0011 0100 0010 1011 0011`.

    **Sau đó, tìm opcode và sử dụng nó để xác định loại định dạng lệnh.** Trường `opcode` luôn là 7 bit thấp nhất của lệnh, bất kể định dạng. `0110011` là opcode cho các lệnh R-Type.

2. **Chia thành các trường theo loại định dạng lệnh.**. Để dịch, không đặc biệt hữu ích khi phân cách các nybble theo không gian trực quan, như trên. Trong @fig-practice-hex-xor, mẫu 32-bit ở trên được chia trực quan thành sáu trường của định dạng lệnh R-Type.

:::{figure} images/practice-hex-xor.png
:label: fig-practice-hex-xor
:width: 100%
:alt: "Ví dụ R-type đã giải: funct7 là 0b0000000, rs2 là 0b11011, rs1 là 0b00110, funct3 là 0b100, rd là 0b00101, và opcode là 0b0110011, tương ứng với xor x5 x6 x27 một khi các trường được dịch."

Một khi loại lệnh được biết, các bit lệnh có thể được ánh xạ đến các trường.
:::

3. **Xác định `opname` lệnh hợp ngữ**. Trường `funct3` là `100`; trường `funct7` là `0000000`. Chúng ta tra cứu các trường này trên [green card](#tab-rv32i-arithmetic) của chúng ta và phát hiện lệnh là `xor`.

4. **Xác định các thanh ghi**. Sử dụng [bảng quy ước thanh ghi](#tab-calling-convention) cho tên thanh ghi.

    * `rd`: `00101` là $5$, nên là thanh ghi `x5`, hay `t0`.
    * `rs1`: `00110` là $6$, nên là thanh ghi `x6`, hay `t1`
    * `rs2`: `11011` là $27$, nên là thanh ghi `x27`, hay `s11`.

Với những điều trên, lệnh là **E.** `xor  t0  t1 s11`.

## Các quyết định thiết kế cho R-Type

Xét tất cả 10 lệnh R-Type được hiển thị trong @tab-r-type, là một định dạng lại của các cột ngoài cùng bên phải của [bảng tương đương](#tab-rv32i-arithmetic) trên green card RISC-V.

Phân tích bảng này. Bạn nhận thấy gì? Bạn thắc mắc gì? Các câu hỏi thảo luận bên dưới thực hành những điều sau:

1. Đọc và diễn giải các trường lệnh
1. Phát triển trực giác thiết kế cho kiến trúc.

:::{table} Các lệnh RV32I: R-Type
:label: tab-r-type
:align: center

| Lệnh | funct7 | rs2 | rs1 | funct3 | rd | opcode |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `add` | `0000000` | `rs2` | `rs1` | `000` | `rd` | `0110011` |
| `sub` | `0100000` | `rs2` | `rs1` | `000` | `rd` | `0110011` |
| `and` | `0000000` | `rs2` | `rs1` | `111` | `rd` | `0110011` |
| `or` | `0000000` | `rs2` | `rs1` | `110` | `rd` | `0110011` |
| `xor` | `0000000` | `rs2` | `rs1` | `100` | `rd` | `0110011` |
| `sll` | `0000000` | `rs2` | `rs1` | `001` | `rd` | `0110011` |
| `srl` | `0000000` | `rs2` | `rs1` | `101` | `rd` | `0110011` |
| `sra` | `0100000` | `rs2` | `rs1` | `101` | `rd` | `0110011` |
| `slt` | `0000000` | `rs2` | `rs1` | `010` | `rd` | `0110011` |
| `sltu` | `0000000` | `rs2` | `rs1` | `011` | `rd` | `0110011` |

:::


:::{tip} Câu hỏi thảo luận 1
R-Type: Có bao nhiêu `opcode` duy nhất?
:::

:::{note} Hiển thị giải thích
:class: dropdown
Một `opcode` cho tất cả R-Type: `0110011`.
:::

:::{tip} Câu hỏi thảo luận 2
R-Type: Có bao nhiêu `funct3` duy nhất? `funct7` duy nhất?
:::

:::{note} Hiển thị giải thích
:class: dropdown

* `funct3`: 8 duy nhất, mặc dù có 10 lệnh.
* `funct7`: 2 duy nhất. Hai lệnh còn lại khác nhau ở trường `funct7`.
:::

:::{tip} Câu hỏi thảo luận 3
Các lệnh nào chia sẻ trường `funct3`? Các trường `funct7` tương ứng của chúng là gì?
:::

:::{note} Hiển thị giải thích
:class: dropdown

(sec-funct7-explanation)=
* Trường `funct3` `000`: `add` và `sub`. Để phân biệt, nhìn vào trường `funct7`.
    * `sub` có `1` ở bit 30, trong khi `add` thì không.
* Trường `funct3` `101`: `srl`, `sra`. Để phân biệt, nhìn vào trường `funct7`.
    * `sra` có `1` ở bit 30, trong khi `srl` thì không.

:::

:::{tip} Câu hỏi thảo luận 4
Đối với các lệnh bạn tìm thấy ở phần trước, chúng _còn_ tương tự như thế nào? Tại sao điều này có thể hữu ích?
:::

:::{note} Hiển thị giải thích

Chúng tôi đã mở rộng trước hộp này vì nó rất quan trọng.

* `add` và `sub` có thể chia sẻ phần cứng. Để thực thi `sub rd rs1 rs2` với một `add`, phần cứng nên phủ định giá trị thanh ghi `rs2`. Bit 30 do đó là một **cờ**, hay chỉ báo boolean, chỉ ra khi nào cần thực hiện **phủ định** bù hai của các bit của `rs2` trước khi cộng (cho `sub`). Nếu Bit 30 tắt, chỉ `add` như bình thường.
* `srl` và `sra` đều là dịch phải và có thể chia sẻ phần cứng. Sự khác biệt là **mở rộng dấu**. Bit 30 do đó là một cờ chỉ ra khi nào cần mở rộng dấu và chèn các `1` đầu (cho `sra`). Nếu Bit 30 tắt, chỉ mở rộng zero cho `srl`.
:::
