---
title: "Thanh ghi Điều khiển và Trạng thái"
subtitle: Nội dung này không được kiểm tra
---

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/WgV4h7NTp5U
:width: 100%
:title: "[CS61C FA20] Lecture 20.1 - Single-Cycle CPU Control: Control and Status Registers"
:::

::::

Cho đến nay chúng ta đã thiết kế datapath và logic điều khiển để hỗ trợ các lệnh trong ISA cơ bản RV32I nhằm chạy bất kỳ chương trình C đã biên dịch nào. Chúng ta vẫn thiếu một vài thành phần cần thiết cho mọi máy tính.

## Thanh ghi Điều khiển và Trạng thái (CSRs)

**Thanh ghi điều khiển và trạng thái** (CSRs) tách biệt với tệp thanh ghi (`x0`-`x31`). CSRs không nằm trong ISA cơ bản, nhưng chúng gần như bắt buộc cho mọi triển khai. Vì ISA là mô-đun, CSRs cần thiết cho các bộ đếm và bộ định thời, và giao tiếp với các thiết bị ngoại vi. Nói cách khác, CSRs giám sát trạng thái và hiệu suất, chẳng hạn như đếm số chu kỳ đã thực thi và giao tiếp với các thiết bị ngoại vi (như máy in) hoặc các đơn vị khác trên cùng chip (như các bộ đồng xử lý dấu phẩy động).

ISA RISC-V cho phép không gian địa chỉ cho đến 4096 CSRs. Giao tiếp thường được thực hiện bằng cách đặt một từ điều khiển vào thanh ghi để thiết bị ngoại vi nhận; khi hoàn thành, thiết bị ngoại vi đặt trạng thái của nó (sẵn sàng, đang chờ, hoặc hoàn thành) trở lại thanh ghi. Đôi khi giao tiếp này chỉ là một bit đơn, tương tự như hộp thư bưu điện nơi việc nâng cờ chỉ ra thư đã sẵn sàng để được lấy. Đây là lý do tại sao các mẩu thông tin bit đơn trong bộ xử lý được gọi là cờ (flags), mà chúng ta "đặt" và "xóa".

Đọc thêm về CSRs trong [RISC-V Privileged ISA Specification, Volume II Chapter 2](https://docs.riscv.org/reference/isa/priv/priv-csrs.html).

### Định dạng Lệnh CSR

Các lệnh CSR tách biệt với ISA cơ bản trong mô-đun extension tiêu chuẩn riêng của chúng. Định dạng lệnh CSR tương tự như [I-Type](#sec-i-type), ngoại trừ trường 12-bit phía trên được dành riêng cho địa chỉ CSR `csr` (thay vì immediate `imm`).

@tab-csr-types-vi hiển thị hai loại định dạng lệnh cho các lệnh CSR:

:::{table} Các Loại Định dạng Lệnh CSR
:label: tab-csr-types-vi
:align: center

<table style="border: 1px solid black; border-collapse: collapse; font-family: monospace; width: 100%;">
  <thead>
    <tr style="background-color: #f2f2f2;">
      <th style="border: 1px solid black; padding: 5px;"><div style="display: flex; justify-content: space-between;"><span>31</span><span>25</span></div></th>
      <th style="border: 1px solid black; padding: 5px;"><div style="display: flex; justify-content: space-between;"><span>24</span><span>20</span></div></th>
      <th style="border: 1px solid black; padding: 5px;"><div style="display: flex; justify-content: space-between;"><span>19</span><span>15</span></div></th>
      <th style="border: 1px solid black; padding: 5px;"><div style="display: flex; justify-content: space-between;"><span>14</span><span>12</span></div></th>
      <th style="border: 1px solid black; padding: 5px;"><div style="display: flex; justify-content: space-between;"><span>11</span><span>7</span></div></th>
      <th style="border: 1px solid black; padding: 5px;"><div style="display: flex; justify-content: space-between;"><span>6</span><span>0</span></div></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td colspan="2" style="border: 1px solid black; text-align: center;">csr</td>
      <td style="border: 1px solid black; text-align: center;">rs</td>
      <td style="border: 1px solid black; text-align: center;">funct3</td>
      <td style="border: 1px solid black; text-align: center;">rd</td>
      <td style="border: 1px solid black; text-align: center;">opcode</td>
    </tr>
    <tr>
      <td colspan="2" style="border: 1px solid black; text-align: center;">csr</td>
      <td style="border: 1px solid black; text-align: center;">uimm</td>
      <td style="border: 1px solid black; text-align: center;">funct3</td>
      <td style="border: 1px solid black; text-align: center;">rd</td>
      <td style="border: 1px solid black; text-align: center;">opcode</td>
    </tr>
  </tbody>
</table>

:::

* **Toán hạng thanh ghi** `rs`: Sử dụng một thanh ghi nguồn.
* **Toán hạng immediate**, `uimm`: Sử dụng giá trị immediate 5-bit luôn được mở rộng zero thành 32 bit. (Không có phép toán số học nào được thực hiện trên các bit trạng thái).

Tất cả các lệnh CSR có trường `opcode` là `1110011` (SYSTEM).

### Các Lệnh CSR

:::{note} Phần này từ ISA RISC-V

Đọc thêm về Các Lệnh CSR trong extension Zicsr trong [RISC-V Unprivileged Specification, Volume I Chapter 6](https://docs.riscv.org/reference/isa/unpriv/zicsr.html).
:::

Các lệnh trong @tab-csr-instructions-reg-vi thường hoạt động bằng cách hoán đổi giá trị giữa CSRs và các thanh ghi đa năng.

:::{table} Các Lệnh CSR: Toán hạng Thanh ghi
:label: tab-csr-instructions-reg-vi

| Lệnh | `rd` là `x0` | `rs1` là `x0` | Đọc CSR | Ghi CSR |
| :--- | :--- | :--- | :--- | :--- |
| CSRRW | Có | - | Không | Có |
| CSRRW | Không | - | Có | Có |
| CSRRS/CSRRC | - | Có | Có | Không |
| CSRRS/CSRRC | - | Không | Có | Có |

:::


* Lệnh **CSRRW (Atomic Read/Write CSR)** sao chép giá trị của một CSR cụ thể vào thanh ghi đích (`rd`) đồng thời sao chép[^atomic-vi] giá trị từ thanh ghi nguồn (`rs1`) vào CSR đó.
* Lệnh **CSRRS (Atomic Read and Set Bits in CSR)** đọc giá trị của một CSR và ghi nó vào đích `rd`, đồng thời đặt[^atomic-vi] các bit trong CSR tương ứng với các bit cao ban đầu trong `rs1` (nếu bit CSR có thể ghi được).
* Lệnh **CSRRC (Atomic Read and Clear Bits in CSR)** hoạt động giống CSRRS, ngoại trừ bây giờ các bit cao của `rs` sẽ khiến các bit tương ứng trong CSR bị xóa đồng thời[^atomic-vi] (nếu bit CSR có thể ghi được).
* Trong tất cả các lệnh, nếu `rd` là `x0`, thì lệnh sẽ không đọc CSR và sẽ không gây ra bất kỳ hiệu ứng phụ nào có thể xảy ra khi đọc CSR.

[^atomic-vi]: Các cặp hoạt động đồng thời này được gọi là **nguyên tử**; chúng ta thảo luận ý tưởng này sau.

Các lệnh toán hạng immediate trong @tab-csr-instructions-imm-vi hoạt động tương tự như các lệnh toán hạng thanh ghi tương ứng. Các lệnh này cập nhật CSR dựa trên trường immediate 5-bit unsigned `uimm` được mở rộng zero thành 32 bit.

:::{table} Các Lệnh CSR: Toán hạng Immediate
:label: tab-csr-instructions-imm-vi

| Lệnh | `rd` là `x0` | `uimm` là `0` | Đọc CSR | Ghi CSR |
| :--- | :--- | :--- | :--- | :--- |
| CSRRWI | Có | - | Không | Có |
| CSRRWI | Không | - | Có | Có |
| CSRRSI/CSRRCI | - | Có | Có | Không |
| CSRRSI/CSRRCI | - | Không | Có | Có |

:::

Có các pseudoinstruction không đọc CSR, chọn đặt `rd=x0`:

* `csrw csr rs1` là `csrrw x0 csr rs1`. Pseudoinstruction này chỉ ghi `rs1` vào CSR được chỉ định.
* `csrwi csr uimm` là `csrrwi x0 csr uimm`. Pseudoinstruction này chỉ ghi `uimm` vào CSR được chỉ định.

### Triển khai

Triển khai CSR và các lệnh CSR không phải là "phép thuật"; giống như RegFile, CSRs chỉ là một khối thanh ghi. Tuy nhiên, xung clock và tín hiệu write enable vẫn cần thiết để tránh vô tình "ghi bậy" lên bất kỳ CSRs nào.

## Các Lệnh Hệ thống

Có một vài lệnh nữa trong tập cơ bản chia sẻ cùng opcode SYSTEM `1110011` mà bạn chắc chắn sẽ gặp:

* `ecall` (I-Type) tạo yêu cầu đến môi trường thực thi hỗ trợ (OS), chẳng hạn như system calls (còn gọi là **syscalls**)
* `ebreak` – (I-Type) được sử dụng (ví dụ bởi debuggers) để chuyển điều khiển đến môi trường debugging.
* `fence` – sắp xếp thứ tự các truy cập bộ nhớ (và I/O) như được nhìn thấy bởi các thread hoặc co-processor khác.

:::{note} Đọc ISA Cơ bản RV32I

* [Section 2.1.8: Environment Call and Breakpoints](https://docs.riscv.org/reference/isa/unpriv/rv32.html#ecall-ebreak): `ecall`, `ebreak`
* [Section 2.1.7: Memory Ordering Instructions](https://docs.riscv.org/reference/isa/unpriv/rv32.html#fence): `fence`

:::
