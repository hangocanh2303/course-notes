---
title: "Thiết kế Logic Điều khiển"
---

(sec-datapath-control-vi)=
## Mục tiêu Học tập

* Cho một lệnh, xác định các tín hiệu điều khiển.
* Giải thích cách ROM có thể được sử dụng trong logic điều khiển để chuyển đổi các bit lệnh ("địa chỉ đầu vào") thành các tín hiệu điều khiển ("từ đầu ra").
* Triển khai logic điều khiển với ROM và các khối logic tổ hợp.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/VZHbfvhIEq8
:width: 100%
:title: "[CS61C FA20] Lecture 20.4 - Single-Cycle CPU Control: Control Logic Design"
:::

::::

Trong phần này, chúng ta thảo luận cách triển khai **bộ điều khiển**, tức là khối logic điều khiển. Đây là @fig-five-step-single-cycle-control từ [phần giới thiệu chương](#sec-single-cycle-vi) để nhắc lại.

:::{embed} #fig-five-step-single-cycle-control
:::

## Ôn tập Tín hiệu Điều khiển

Trước khi tiếp tục, xem video sau để ôn tập việc xác định và thiết lập các tín hiệu điều khiển cho hoạt động datapath của hai lệnh khác nhau: `sw` và `beq`.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/wM85LGWD54U
:width: 100%
:title: "[CS61C FA20] Lecture 20.2 - Single-Cycle CPU Control: Datapath Control"
:::
::::

Sau đó, xác nhận các tín hiệu điều khiển cho tập con các lệnh được hiển thị trong @tab-control-truth-table-vi.

:::{table} Bảng chân trị được điền một phần cho Logic Điều khiển RV32I, dựa trên lệnh đầu vào `inst`. Dấu hoa thị (*) nghĩa là "không quan tâm". Hoàn thành bảng đầy đủ trong Project 3!
:label: tab-control-truth-table-vi

| Lệnh | PCSel | ImmSel | BrUn | ASel | BSel | ALUSel | MemRW | RegWEn | WBSel |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `add` | +4 | * | * | Reg | Reg | Add | Read | 1 | ALU |
| `sub` | +4 | * | * | Reg | Reg | Sub | Read | 1 | ALU |
| `addi` | +4 | I | * | Reg | Imm | Add | Read | 1 | ALU |
| `lw` | +4 | I | * | Reg | Imm | Add | Read | 1 | Mem |
| `sw` | +4 | S | * | Reg | Imm | Add | Write | 0 | * |
| `beq` | (@tab-branch-truth-table-vi) | B | * | PC | Imm | Add | Read | 0 | * |
| `bne` | (@tab-branch-truth-table-vi) | B | * | PC | Imm | Add | Read | 0 | * |
| `blt` | (@tab-branch-truth-table-vi) | B | 0 | PC | Imm | Add | Read | 0 | * |
| `bltu` | (@tab-branch-truth-table-vi) | B | 1 | PC | Imm | Add | Read | 0 | * |
| `jalr` | ALU | I | * | Reg | Imm | Add | Read | 1 | PC+4 |
| `jal` | ALU | J | * | PC | Imm | Add | Read | 1 | PC+4 |
| `auipc` | +4 | U | * | PC | Imm | Add | Read | 1 | ALU |

:::

Nhớ rằng các branch cập nhật PC có điều kiện dựa trên đầu ra của [khối branch comparator](#sec-datapath-branch-comparator-vi). Các tín hiệu đầu ra của khối `BrEq` và `BrLT` được đưa vào logic điều khiển, sau đó thiết lập bộ chọn `PCSel` nối vào mux `PCSel`. Xem bảng chân trị một phần bên dưới (@tab-branch-truth-table-vi).

:::{table} Bảng chân trị được điền một phần để xác định PCSel cho các lệnh branch.
:label: tab-branch-truth-table-vi

| Lệnh | Branch? | BrEq | BrLT | PCSel |
| :--- | :--: | :--: | :--: | :--: |
| `beq` | không nhảy | 0 | * | +4 |
| `beq` | nhảy | 1 | * | ALU |
| `bne` | không nhảy | 1 | * | +4 |
| `bne` | nhảy | 0 | * | ALU |
| `blt` | nhảy | * | 1 | ALU |
| `bltu` | nhảy | * | 1 | ALU |
:::

## Logic Điều khiển / Bộ điều khiển

**Mạch con logic điều khiển** (tức là **bộ điều khiển**) nhận các bit lệnh (và `BrEq`, `BrLT`) và xuất ra tất cả các tín hiệu điều khiển cần thiết để thực thi lệnh đó. Các tín hiệu điều khiển được liệt kê trong @tab-controller-signals-vi.

:::{table} Các tín hiệu cho logic điều khiển. Tên tín hiệu trong project khóa học, nếu khác, được ghi trong ngoặc đơn.
:label: tab-controller-signals-vi

| Tên | Độ rộng Bit | Mục đích |
| :--- | :--- | :--- |
| `PCSel` | 1 | Chọn đầu vào ALU cho tất cả các lệnh B-type mà branch được thực hiện (theo đầu ra branch comparator) và tất cả các jump. Chọn đầu vào PC+4 cho tất cả các lệnh khác. |
| `ImmSel` | 3 | Chọn định dạng lệnh để immediate generator có thể trích xuất immediate đúng cách. Xem @tab-immgen-operations trong [phần Immediate Generator](#sec-datapath-immgen-vi) |
| `RegWEn` | 1 | 1 nếu lệnh ghi vào thanh ghi, và 0 nếu không. |
| `BrUn` | 1 | 1 nếu lệnh branch là unsigned, và 0 nếu lệnh branch là signed. Không quan tâm cho tất cả các lệnh khác. |
| `ASel` | 1 | Chọn gửi dữ liệu trong `rdata1` (`RegReadData1`) hay PC đến ALU. |
| `BSel` | 1 | Chọn gửi dữ liệu trong `rdata2` (`RegReadData2`) hay immediate đến ALU. |
| `ALUSel` | 4 | Chọn phép toán đúng cho ALU. Xem @tab-alu-operations trong [phần ALU](#sec-datapath-alu-vi). |
| `MemRW` | 1 | 1 nếu lệnh ghi vào bộ nhớ, và 0 nếu không. |
| `WBSel` | 2 | Chọn ghi dữ liệu đọc từ DMEM, đầu ra ALU, hay PC+4 vào `rd`. |

:::

Nói chung, có hai cách tiếp cận để triển khai logic điều khiển:

1. **Bộ nhớ Chỉ đọc (ROM)**: ROM đọc ra các từ tại địa chỉ đầu vào cho trước. Vì ROM chỉ đọc, nó được điền sẵn các giá trị một và không tại thời điểm thiết kế.

    Cấu trúc đều đặn của ROM nghĩa là nó có thể dễ dàng được thiết kế và lập trình lại để sửa lỗi (ví dụ, trong quá trình tạo mẫu) hoặc khi thêm lệnh (như các extension cho compressed instructions). ROM cũng phổ biến khi thiết kế logic điều khiển thủ công, như trong khóa học này.

1. **Logic Tổ hợp** sử dụng các cổng AND, OR, và NOT. Trong thiết kế chip thực tế, control thường được thiết kế với các cổng logic vì nó nhỏ gọn hơn và nhanh hơn nhiều so với các phương án ROM. Ngày nay, các nhà thiết kế chip sử dụng công cụ tổng hợp logic để chuyển đổi bảng chân trị thành mạng các cổng. Xem cách tiếp cận này [bên dưới](#sec-control-cl-vi).

## Cách tiếp cận ROM

Trong khóa học này, chúng ta triển khai thiết kế bộ điều khiển trong @fig-control-design, bao gồm hai mạch con:

1. **Bộ nhớ Chỉ đọc (ROM)**: ROM nhận địa chỉ 9-bit được xây dựng từ các bit lệnh, sau đó xuất ra "từ" 14-bit có thể được giải mã thành các tín hiệu điều khiển cần thiết.

2. Khối logic tổ hợp **Take branch?** nhận đầu ra Branch Comparator và xuất ra tín hiệu điều khiển `PCSel`.

:::{figure} images/control-design.png
:label: fig-control-design
:alt: "Thiết kế bộ điều khiển với khối ROM và ký hiệu decoder tạo hầu hết các tín hiệu điều khiển từ các bit lệnh và khối logic take-branch tạo PCSel từ BrEq và BrLT."

Thiết kế control được đề xuất có hai phần.

:::

:::{tip} Các Bit Lệnh

Tại sao chúng ta chỉ cần 9 bit lệnh `inst[30, 14:12, 6:2]`?

:::

:::{note} Hiển thị Đáp án
:class: dropdown

Bằng cách giảm địa chỉ đầu vào, chúng ta có thể giảm kích thước ROM.

Trong RV32I, loại lệnh được định nghĩa bởi các trường `opcode`, `funct3`, và `funct7`, trong đó hai trường sau có mặt dựa trên loại định dạng lệnh. Điều này có nghĩa là địa chỉ 17-bit cho ROM.

Tuy nhiên, khi xem xét kỹ hơn, chúng ta nhận ra rằng một số trường bit này là dư thừa, dẫn đến "ISA 9-bit" cho RV32I:

* `inst[6:2]`: Năm bit duy nhất của trường `opcode` rộng 7-bit. Trong RV32I, chỉ năm bit trên của `opcode` là duy nhất. Hai bit dưới `inst[1:0]` cố định trong RV32I vì chúng được dành cho các extension (ví dụ, compressed instructions).
* `inst[14:12]`: Ba bit duy nhất của trường `funct3`.
* `inst[30]`: Bit duy nhất của trường `funct7` rộng 7-bit. Bit đơn này phân biệt `add` với `sub` và `sll` với `srl` (để biết thêm chi tiết, xem [phần này](#sec-funct7-explanation)).

Nói cách khác, trong RV32I, loại lệnh được mã hóa chỉ sử dụng **9 bit**.
:::

:::{tip} Decoder

Decoder làm gì?

:::

:::{note} Hiển thị Đáp án
:class: dropdown

Decoder nhận từ đầu ra 14-bit từ ROM và chia nó thành các tín hiệu điều khiển trong @tab-controller-signals-vi, *ngoại trừ* `PCSel`.
:::

:::{tip} Tín hiệu điều khiển `PCSel`

Tại sao chúng ta cần "Phần 2"? Tại sao `PCSel` không được mã hóa trực tiếp trong ROM?
:::

:::{note} Hiển thị Đáp án
:class: dropdown

Tín hiệu điều khiển `PCSel` không thể được mã hóa trong ROM vì nó phụ thuộc vào `BrEq` và `BrLT` cho các lệnh B-Type/branch. Các tín hiệu này được tính toán với branch comparator (từ datapath), phụ thuộc vào đầu ra của các phần tử datapath được điều khiển bởi các tín hiệu điều khiển từ đơn vị ROM.
Để hoàn thành logic điều khiển, triển khai khối "Take branch?" như logic tổ hợp để điều khiển đầu ra `PCSel`.
:::

:::{tip} Dạng sóng ổn định

Làm thế nào chúng ta có thể đảm bảo `PCSel` ổn định?
:::

:::{note} Hiển thị Đáp án
:class: dropdown

Vì hai "phần" của logic điều khiển sử dụng các tín hiệu dữ liệu khác nhau ổn định tại các phần khác nhau của chu kỳ xung clock, có thể sớm trong chu kỳ, `PCSel` có thể giữ giá trị không đúng trước khi `BrEq` và `BrLT` được tính toán sử dụng các đầu vào ổn định `R[rs1]` và `R[rs2]` của [branch comparator](#fig-branch-branch-comparator). Gần cuối chu kỳ, `BrEq` và `BrLT` sẽ ổn định, do đó thiết lập `PCSel` về giá trị đúng.

Chu kỳ xung clock phải (ít nhất) đủ dài để chứa:

* `ID`: Đọc giá trị thanh ghi `R[rs1]` và `R[rs2]`
* `EX`: Tính `PC + imm` với ALU
* Control: Thiết lập `BrUn` (và tất cả các tín hiệu dựa trên `inst` khác)
* `EX`: So sánh sử dụng branch comparator
* Control: Thiết lập `PCSel` dựa trên `BrEq` và `BrLT`

:::

(sec-control-cl-vi)=
## Cách tiếp cận Logic Tổ hợp

Vì ROM thực chất thực hiện tra cứu trên một [bảng chân trị](#tab-control-truth-table-vi) khổng lồ, chúng ta có thể triển khai mạch con logic điều khiển hoàn toàn sử dụng các cổng AND, OR, và NOT. Chúng ta có thể sử dụng phương pháp [Sum of Products](#sec-sum-product) quen thuộc để viết dạng chính tắc, sau đó rút gọn biểu thức sử dụng các Định luật Đại số Boolean.

**Ví dụ 1**: Giả sử chúng ta muốn triển khai một tín hiệu có giá trị cao nếu chúng ta có lệnh R-Type. Opcode cho tất cả các lệnh R-Type là `0110011` (các bit lệnh `inst[7:0]`), vì vậy biểu thức boolean tương ứng là:

```{math}
\text{R-Type} = (\overline{\texttt{inst}[6]} \cdot \texttt{inst}[5] \cdot \texttt{inst}[4] \cdot \overline{\texttt{inst}[3]} \cdot \overline{\texttt{inst}[2]})
```

**Ví dụ 2**: Giả sử chúng ta muốn triển khai tín hiệu `BrUn`. Tín hiệu `BrUn` có giá trị cao nếu chúng ta có lệnh B-Type VÀ nếu `funct3` của lệnh đó chỉ ra so sánh unsigned.

:::{table} Các trường `opcode` và `funct3` của B-Type được dẫn xuất từ [RISC-V green card](#tab-rv32i-control).
:label: tab-b-type-opcode-funct3-vi

| `Lệnh` | Opcode <br/>(`inst[6:0]`) | Funct3<br/>(`inst[14:12]`) |
| :--- | :--- | :--- |
| `beq rs1 rs2 label` | `110 0011` | `000` |
| `bne rs1 rs2 label` | `110 0011` | `001` |
| `blt rs1 rs2 label` | `110 0011` | `100` |
| `bltu rs1 rs2 label` | `110 0011` | `110` |
| `bge rs1 rs2 label` | `110 0011` | `101` |
| `bgeu rs1 rs2 label` | `110 0011` | `111` |

:::

Xem xét các trường của các lệnh liên quan ở trên, chúng ta có thể viết biểu thức cho `BrUn`:

```{math}
\begin{aligned}
\texttt{BrUn} &= (\text{B-Type}) \cdot (\text{So sánh Unsigned}) \\
&= (\texttt{inst}[6] \cdot \texttt{inst}[5] \cdot \overline{\texttt{inst}[4]} \cdot \overline{\texttt{inst}[3]} \cdot \overline{\texttt{inst}[2]}) \cdot (\texttt{inst}[13])
\end{aligned}
```
