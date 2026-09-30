---
title: "I-Type"
---

(sec-i-type-vi)=
## Mục tiêu học tập

* Dịch qua lại giữa các lệnh hợp ngữ I-type và các lệnh máy.
* Giải thích tại sao load và `jalr` là các lệnh I-Type.
* Dịch các pseudoinstruction `jr` và `ret` thành các lệnh máy.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/EtLGVuSFwkU
:width: 100%
:title: "[CS61C FA20] Lecture 11.3 - RISC-V Instruction Formats I: I-Format Layout"
:::

::::

Bây giờ chúng ta chuyển sang định dạng lệnh tiếp theo: I-Type. Mặc dù "I" là viết tắt của Immediate, định dạng lệnh I-Type được sử dụng cho nhiều loại lệnh:

* Các lệnh số học **thanh ghi-immediate**
* Các lệnh load
* Lệnh `jalr`
* (ngoài phạm vi) Các cuộc gọi môi trường và điểm dừng

Định dạng lệnh I-Type là hàng thứ hai và thứ ba của [bảng định dạng lệnh](#tab-rv32i-types).

## I-Type: Các trường

Trước tiên chúng ta thảo luận các lệnh số học có dạng `opname rd rs1 imm` như `addi`, `xori`, v.v. Định dạng được hiển thị trong @fig-i-type:

:::{figure} images/i-type.png
:label: fig-i-type
:width: 100%
:alt: "Bố cục I-type RISC-V: hợp ngữ opname rd rs1 imm phía trên một thanh 32-bit với imm[11:0] (bit 31–20), rs1 (bit 19–15), funct3 (bit 14–12), rd (bit 11–7), và opcode (bit 6–0), gắn nhãn immediate 12-bit, thanh ghi nguồn, và các trường thanh ghi đích."

Định dạng lệnh I-Type.
:::

Các trường I-Type:

* **Toán hạng thanh ghi**: Lưu ý rằng các trường thanh ghi `rs1` và `rd` ở cùng vị trí trong I-Type và [R-type](#fig-r-type); thiết kế có chủ đích này giảm độ phức tạp phần cứng.
* **Toán hạng hằng số**: **Trường immediate**[^imm-reminder] `imm` chỉ định một hằng số 12-bit. I-Type không có thanh ghi nguồn thứ hai `rs2`, nên `imm` tái sử dụng các vị trí bit đó để mã hóa một hằng số lớn hơn qua các bit 15 đến 31.

    Immediate 12-bit là một số nguyên bù hai với phạm vi $-2^{11} = -2048$ đến $2^{11} - 1 = 2047$. Trước khi sử dụng giá trị 12-bit này trong một phép toán, phần cứng **mở rộng dấu** nó thành giá trị 32-bit.
* **Các trường phép toán**: `opcode`, `funct3`

[^imm-reminder]: Một lần nữa, immediate được gọi như vậy vì mẫu bit của chúng được mã hóa trực tiếp vào lệnh máy.

## Lệnh hợp ngữ $\rightarrow$ Lệnh máy

Xét @fig-itype-addi-example, dịch `addi x15 x1 -50` thành một lệnh máy.

:::{figure} images/itype-addi-example.png
:label: fig-itype-addi-example
:width: 100%
:alt: "Mã hóa của addi x15 x1 -50: imm[11:0] chứa bù hai 0b111111001110 cho -50, rs1 là 0b00001 cho x1, funct3 là 0b000 cho add, rd là 0b01111 cho x15, và opcode là 0b0010011 cho số học I-type."

Lệnh I-Type `addi x15 x1 -50`.
:::

Chúng ta tuân theo [các bước dịch hợp ngữ thành mã máy](#sec-assembly-to-machine) từ trước:

1. **Xác định loại định dạng lệnh**. `addi` là I-type vì nó thực hiện số học giữa một toán hạng thanh ghi và một toán hạng hằng số. Chúng ta sử dụng [bảng các lệnh số học](#tab-rv32i-arithmetic) trên green card RISC-V.

1. **Xác định mã trường phép toán**.

    * `opcode`: `0010011` cho tất cả các lệnh số học thanh ghi-immediate
    * `funct3`: `000` cho `add`

1. **Dịch các thanh ghi, immediate, v.v.**

    * `rs1`: Thanh ghi `x1`. Dịch $1$ thành biểu diễn số nguyên không dấu 5-bit `00001`.
    * `rd`: Thanh ghi `x15`. Dịch $15$ thành biểu diễn số nguyên không dấu 5-bit `01111`.
    * `imm`: $-50$ dưới dạng bù hai 12-bit:
        * $+50$ là `0000 0011 0010`.
        * Đảo bit: `1111 1100 1101`.
        * Cộng một để có $-50$: `1111 1100 1110`.

1. (nếu cần) Chuyển đổi sang thập lục phân.

    * Chúng tôi để đây như một bài tập cho bạn!

## I-Type vs. R-Type

@fig-comparison-itype-rtype so sánh hai định dạng lệnh chúng ta đã thấy cho đến nay:

:::{figure} images/comparison-itype-rtype.png
:label: fig-comparison-itype-rtype
:width: 100%
:alt: "So sánh định dạng lệnh R-type và I-type cạnh nhau: cả hai định dạng chia sẻ vị trí bit cho opcode, rd, funct3, và rs1; R-type sử dụng bit 31–20 cho funct7 cộng rs2, trong khi I-type thay thế vùng đó bằng immediate 12-bit imm[11:0]."

So sánh tập lệnh I-Type với tập lệnh R-Type.
:::

Một lần nữa, thiết kế tốt đòi hỏi những thỏa hiệp tốt. Nếu chúng ta chỉ có định dạng R-Type đơn lẻ và chỉ định `imm` như một trường 5-bit thay thế `rs2`, thì chúng ta chỉ có thể biểu diễn 32 giá trị immediate. Trường I-Type do đó mở rộng trường `imm` qua các trường `rs2` và `funct7` của R-type để ít nhất đảm bảo chúng ta có thể biểu diễn một phạm vi rộng hơn gồm $2^{12}$ giá trị immediate.

Sự nhất quán của thiết kế I-type với R-type đơn giản hóa cách phần cứng xử lý hai định dạng lệnh này. Các hằng số thường ngắn và có thể vừa trong trường `imm` 12-bit. Đối với các hằng số lớn hơn, chúng ta phải sử dụng các lệnh bổ sung như `lui` (load upper-immediate), mà chúng ta thảo luận sau.

Lưu ý trường `funct3` 3-bit không đủ để chỉ định 9 lệnh số học thanh ghi-immediate, chưa kể bất kỳ lệnh load nào. I-Type do đó vẫn có một vài chi tiết. Tiếp tục!

## Dịch số học ("I*-Type")

Mặc dù không có lệnh "I*-Type" chính thức, [RV32I Unprivileged Manual](https://docs.riscv.org/reference/isa/unpriv/rv32.html#2-5-1-integer-register-immediate-instructions) nói:

> Dịch theo một hằng số được mã hóa như một chuyên biệt hóa của định dạng I-type.

Trong khóa học này chúng ta sẽ gọi những cái này là "I*-Type" (trong đó dấu hoa thị là "hầu hết là I-Type, trừ một số chi tiết"). Xét @tab-istar-type và đoạn trích sau từ manual:

> Toán hạng cần dịch nằm trong rs1, và lượng dịch được mã hóa trong 5 bit thấp hơn của trường I-immediate. Loại dịch phải được mã hóa trong bit 30.

:::{table} Các lệnh dịch theo immediate.
:label: tab-istar-type
:align: center

| Lệnh | imm[11:5] | imm[4:0] | rs1 | funct3 | rd | opcode |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `slli` | `0000000` | `imm[4:0]` | `rs1` | `001` | `rd` | `0010011` |
| `srli` | `0000000` | `imm[4:0]` | `rs1` | `101` | `rd` | `0010011` |
| `srai` | `0100000` | `imm[4:0]` | `rs1` | `101` | `rd` | `0010011` |
:::

Quan sát:

* Các phép dịch bit số học chỉ cần một immediate **không dấu** 5-bit, trong `imm[4:0]`. Dịch tối đa là 31; bất kỳ số nào lớn hơn sẽ dịch tất cả dữ liệu ra khỏi thanh ghi, có độ rộng 32 bit.

* Bảy bit cao hơn[^itype-funct7] **không** phải là một phần của immediate và được sử dụng rất giống với trường `funct7` cho các lệnh R-Type `sll`, `srl`, và `sra`.

    * Giống với R-type, Bit 30 là một cờ chỉ ra khi nào cần mở rộng dấu.
    * Nếu Bit 30 bật, chèn các `1` đầu (cho `srai`).
    * Nếu Bit 30 tắt, chỉ mở rộng zero cho `srli` (và `slli`, trong đó dịch trái chỉ luôn là logic).

[^itype-funct7]: [Green card RISC-V](#sec-green-card) của khóa học gắn nhãn 7 bit cao hơn này là "funct7"–một cách gọi sai, vì I-Type không có trường `funct7`. Theo [RISC-V Unprivileged Manual](https://docs.riscv.org/reference/isa/unpriv/rv32.html#2-5-1-integer-register-immediate-instructions) 7 bit cao hơn này thực sự nên được gắn nhãn lại `imm[11:5]`–một phần của trường `imm[11:0]`, nhưng không được coi là một phần của hằng số.

(sec-rv-load)=
## Các lệnh Load

Nhớ rằng, các định dạng lệnh đơn giản chỉ là các định dạng. Phép toán chỉ định phần cứng thực sự làm gì. Tuy nhiên, giữ cùng định dạng lệnh cho phép chúng ta đơn giản hóa và tái sử dụng phần cứng nhất định.

**Các lệnh load** là một ví dụ như vậy.
Nhớ lại từ [phần trước](#sec-load-word):

> Lệnh **load word**:
> 
> * **Tính toán địa chỉ bộ nhớ** `R[rs1]+imm`
> * **Load một từ** từ địa chỉ này trong bộ nhớ, `M[R[rs1] + imm][31:0]` vào **thanh ghi đích**, `rd`.

Do đó các load có thể sử dụng định dạng lệnh I-Type (@fig-load-operation-isa):

* Các trường `imm`, `rs1`, `rd`
* `opcode` (như tất cả các lệnh). Các load sử dụng opcode `000 0011`.
* `funct3` chỉ định partial load và load có dấu/không dấu.

:::{figure} images/load-operation-isa.png
:label: fig-load-operation-isa
:width: 100%
:alt: "Bố cục I-type cho load: cú pháp loadop rd imm(rs1) với immediate imm[11:0] như một byte offset được cộng vào cơ sở rs1, funct3, rd như đích của giá trị được load, và trường opcode load; mã màu phân biệt immediate, thanh ghi nguồn, và thanh ghi đích."

Các lệnh load sử dụng định dạng lệnh I-Type.
:::

Quan sát:

* Thực tế là load thực hiện truy cập bộ nhớ không liên quan đến cách chúng ta chỉ định lệnh. Các bit lệnh đơn giản cung cấp đủ thông tin cho phần cứng để giải mã và thực thi đúng lệnh.
* Các load _có_ chia sẻ một số điểm tương đồng với các lệnh I-Type khác. Đáng chú ý, load cũng thực hiện **phép cộng thanh ghi-immediate** để tính địa chỉ bộ nhớ như `R[rs1] + imm`. Do đó các load có thể tái sử dụng bất kỳ phần cứng nào cần thiết cho các lệnh số học thanh ghi-immediate.

Chúng tôi khuyến nghị xem lại [chương trước](#sec-data-transfer) để có mô tả về mỗi lệnh load trong @tab-i-type-loads.

:::{table} Các lệnh Load (nhớ rằng không có `lwu`).
:label: tab-i-type-loads
:align: center

| Lệnh | imm[11:0] | rs1 | funct3 | rd | opcode |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `lb` | `imm[11:0]` | `rs1` | `000` | `rd` | `0000011` |
| `lbu` | `imm[11:0]` | `rs1` | `100` | `rd` | `0000011` |
| `lh` | `imm[11:0]` | `rs1` | `001` | `rd` | `0000011` |
| `lhu` | `imm[11:0]` | `rs1` | `101` | `rd` | `0000011` |
| `lw` | `imm[11:0]` | `rs1` | `010` | `rd` | `0000011` |
:::

### Ví dụ Load

:::{tip} Kiểm tra nhanh

Dịch `lw x14 8(x2)` thành một lệnh máy.

:::

::::{note} Hiển thị giải thích
:class: dropdown

:::{figure} images/practice-lw.png
:label: fig-practice-lw
:width: 100%
:alt: "Mã hóa của lw x14 8(x2): imm[11:0] 0b000000001000 cho offset 8, rs1 là 0b00010 cho thanh ghi x2, funct3 là 0b010, rd là 0b01110 cho x14, và opcode là 0b0000011 cho load."

Lệnh I-Type `lw x14 8(x2)`.
:::

Chúng ta tuân theo [các bước dịch hợp ngữ thành mã máy](#sec-assembly-to-machine) từ trước:

1. **Xác định loại định dạng lệnh**. `lw` là I-type. Chúng ta sử dụng [bảng bộ nhớ](#tab-rv32i-memory) trên green card RISC-V.

1. **Xác định mã trường phép toán**.

    * `opcode`: `0000011` cho các lệnh load
    * `funct3`: `010` cho load **word**

1. **Dịch các thanh ghi, immediate, v.v.**

    * `rs1`: Thanh ghi cơ sở `x2`. Dịch $2$ thành biểu diễn số nguyên không dấu 5-bit `00010`.
    * `rd`: Thanh ghi `x14`. Dịch $14$ thành biểu diễn số nguyên không dấu 5-bit `01110`.
    * `imm`: address offset $+8$ dưới dạng bù hai 12-bit: `0000 0000 1000`.
::::

(sec-jalr-itype)=
## `jalr`: I-Type

Chúng tôi khuyến nghị xem lại [các lệnh jump](#sec-jumps) trước khi tiếp tục:

> **J**ump **a**nd **L**ink **R**egister (`jalr rd rs1 imm`). Link "địa chỉ trả về" (`PC + 4`) vào một thanh ghi `rd`. Sau đó thực hiện một jump vô điều kiện bằng cách đặt `PC` thành `R[rs1] + imm`.

Lệnh `jalr` cũng có thể được hỗ trợ với định dạng I-Type (@fig-jalr-isa):

* Các trường `imm`, `rs1`, `rd`
* `opcode` (như tất cả các lệnh). `jalr` sử dụng opcode `110 0111`.
* `funct3` không đặc biệt cần thiết, vì chỉ có một lệnh `jalr`. Tuy nhiên, vì định dạng I-Type yêu cầu nó, `jalr` sử dụng `000`.

:::{figure} images/jalr-isa.png
:label: fig-jalr-isa
:width: 100%
:alt: "Bố cục I-type jalr bao gồm trường immediate imm[11:0] như một offset được cộng vào thanh ghi nguồn rs1 để tạo thành đích jump, funct3, rd nhận giá trị link PC cộng bốn, và opcode; các chú thích phân biệt offset, thanh ghi cơ sở, và đích link."

Định dạng lệnh jalr. Bộ đếm chương trình được cập nhật thành thanh ghi cơ sở cộng một hằng số, ví dụ: `PC = R[rs1] + imm`.
:::

Quan sát:

* Giống với load, thực tế là `jalr` truy cập PC không liên quan đến cách chúng ta chỉ định lệnh.
* Giống với load, `jalr` cũng thực hiện **phép cộng thanh ghi-immediate**, do đó `jalr` có thể tái sử dụng phần cứng số học thanh ghi-immediate để tính địa chỉ jump `R[rs1] + imm`.

:::{warning} Còn `jr` và `ret` thì sao?

Pseudoinstruction không được cấp phát `opcode` hay `funct3` bổ sung, vì chúng không phải là lệnh RISC-V thực.

Các pseudoinstruction `jr rs1` và `ret` là `jalr x0 rs1 0` và `jalr x0 ra 0`, tương ứng.
:::

## Các quyết định thiết kế cho I-Type

Phần này nhằm giúp bạn phát triển trực giác cho các lệnh I-Type bằng những gì bạn đã học trong phần này.

Xét các lệnh I-Type[^ebreak-ecall] được hiển thị trong @tab-i-type, là một định dạng lại của các cột ngoài cùng bên phải của các bảng liên quan trên green card RISC-V.

[^ebreak-ecall]: `ecall`, `ebreak` ngoài phạm vi trong thảo luận của chúng ta, nhưng xem thêm chi tiết trong [Green Card RISC-V](#sec-green-card) và [RISC-V Unprivileged Manual](https://docs.riscv.org/reference/isa/unpriv/rv32.html#ecall-ebreak).

:::{table} Các lệnh RV32I: (a) I-Type; (b) "I*-Type", một bản in lại của @tab-istar-type.
:label: tab-i-type
:align: center

| Lệnh | imm[11:0] | rs1 | funct3 | rd | opcode |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `addi` | `imm[11:0]` | `rs1` | `000` | `rd` | `0010011` |
| `andi` | `imm[11:0]` | `rs1` | `111` | `rd` | `0010011` |
| `ori` | `imm[11:0]` | `rs1` | `110` | `rd` | `0010011` |
| `xori` | `imm[11:0]` | `rs1` | `100` | `rd` | `0010011` |
| `slti` | `imm[11:0]` | `rs1` | `010` | `rd` | `0010011` |
| `sltiu` | `imm[11:0]` | `rs1` | `011` | `rd` | `0010011` |
| `lb` | `imm[11:0]` | `rs1` | `000` | `rd` | `0000011` |
| `lbu` | `imm[11:0]` | `rs1` | `100` | `rd` | `0000011` |
| `lh` | `imm[11:0]` | `rs1` | `001` | `rd` | `0000011` |
| `lhu` | `imm[11:0]` | `rs1` | `101` | `rd` | `0000011` |
| `lw` | `imm[11:0]` | `rs1` | `010` | `rd` | `0000011` |
| `jalr` | `imm[11:0]` | `rs1` | `000` | `rd` | `1100111` |
| `ecall` | `000000000000` | `00000` | `000` | `00000` | `1110011` |
| `ebreak` | `000000000001` | `00000` | `000` | `00000` | `1110011` |

| Lệnh | imm[11:5] | imm[4:0] | rs1 | funct3 | rd | opcode |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `slli` | `0000000` | `imm[4:0]` | `rs1` | `001` | `rd` | `0010011` |
| `srli` | `0000000` | `imm[4:0]` | `rs1` | `101` | `rd` | `0010011` |
| `srai` | `0100000` | `imm[4:0]` | `rs1` | `101` | `rd` | `0010011` |
:::

:::{tip} Câu hỏi thảo luận 1
I-Type: Có bao nhiêu `opcode` duy nhất?
:::

:::{note} Hiển thị giải thích
:class: dropdown
1. Số học: `0010011`
1. Load: `0000011`
1. jalr: `1100111`
1. Khác (`ecall`, `ebreak`): `1110011`
:::

:::{tip} Câu hỏi thảo luận 2
So sánh các trường `funct3` của I-Type với các trường `funct3` của R-Type ([bảng](#tab-r-type) R-Type từ phần trước).
:::

:::{note} Hiển thị giải thích
:class: dropdown

Cùng tám trường `funct3` duy nhất như phép toán R-format tương ứng (nhớ rằng, không có `subi`)

| funct3 | R-Type  | I-Type |
| :--: | :--: | :--: |
| `000` | `add` | `addi` |
| `000` | `sub` | |
| `111` | `and` | `andi` |
| `110` | `or` | `ori` |
| `100` | `xor` | `xori` |
| `001` | `sll` | `slli` |
| `101` | `srl` | `srli` |
| `101` | `sra` | `srai` |
| `010` | `slt` | `slti` |
| `011` | `sltu` | `sltiu` |

:::

:::{tip} Câu hỏi thảo luận 3
Trong [@tab-i-type(b)](#tab-istar-type), các lệnh "I*-Type" là gì, tức là loại phép toán số học nào được chỉ định? Tại sao chúng ta chỉ cần 5 bit cho `imm`? 7 bit cao hơn được sử dụng cho gì? Có những điểm tương đồng gì với R-type?
:::

Xem giải thích [ở trên].

:::{tip} Câu hỏi thảo luận 4
Tại sao load cũng sử dụng định dạng lệnh I-Type?
Còn `jalr` thì sao?
:::

Câu hỏi đầu tiên: xem giải thích [ở trên]. Câu hỏi thứ hai: Xem phần sau.
