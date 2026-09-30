---
title: "B-Type"
subtitle: "TODO. Ghi chú này chưa hoàn chỉnh; xem bài giảng Spring 2026"
---

(sec-b-type-vi)=
## Mục Tiêu Học Tập

* Chuyển đổi qua lại giữa lệnh assembly B-Type và lệnh máy.
* Xác định lệnh nào sử dụng định địa chỉ tương đối PC và lệnh nào sử dụng định địa chỉ tuyệt đối.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/cH5MqwqX_Kg
:width: 100%
:title: "[CS61C FA20] Lecture 12.1 - RISC-V Instruction Formats II: B-Format"
:::

::::

:::{warning} Ghi chú này chưa hoàn chỉnh, nhưng bài giảng vẫn nằm trong phạm vi

Vui lòng tham khảo [slide bài giảng Spring 2026](https://docs.google.com/presentation/d/18YSyN37XjHEjfkWigPHJWXNJB17AsSiAKW8f9XebE-8/edit?usp=sharing).

Sẽ có thêm nội dung sớm!

:::

(sec-implicit-zero-b-type-vi)=
### Mở Rộng RISC-V: Lệnh 16-bit

RISC-V Base ISA cho RV32, RV64, RV128 đều có lệnh rộng 32 bit. ISA "Cơ sở" được **mở rộng** bởi các **phần mở rộng** lệnh thực hiện nhiều việc: phép nhân tổng quát, hỗ trợ kiến trúc khác nhau, v.v.

Một phần mở rộng như vậy là **phần mở rộng nén 16-bit**, cho phép các lệnh _độ dài thay đổi_ là bội số của 16 bit. Để chủ động hỗ trợ điều này và các phần mở rộng khác, RISC-V Base ISA mã hóa **offset nhánh half-word**, ngay cả khi không có lệnh 16 bit trong tập cơ sở.

Trong khóa học này, chúng ta chỉ tập trung vào bộ xử lý RISC-V với lệnh 32 bit. Hàm ý của offset nhánh half-word này:

* Một nửa các đích nhánh có thể sẽ là lỗi
* Nhánh có điều kiện RISC-V chỉ có thể nhánh đến $\pm 2^{10}$ lệnh cách PC.

:::{tip} Kiểm Tra Nhanh

**Đúng/Sai**: Nếu chương trình chỉ có lệnh 32 bit, bit vị trí 8 của tất cả lệnh B-Type sẽ luôn là `0`.
:::

:::{note} Hiện Đáp Án
**Đúng**. Trong lệnh B-Type, offset nhánh được mã hóa như offset half-word, nghĩa là `imm[0]` là `0`. Nếu chương trình chỉ có lệnh 32 bit, thì nhánh có điều kiện sẽ chỉ là bội số của word, ví dụ 4 byte, và chúng ta sẽ chỉ nhảy theo bội số hai half-word. Trong trường hợp này, ràng buộc `imm[1]` và `imm[0]` là `0`.
:::

(sec-imm-swirl-vi)=
## So sánh B-Type vs. I-Type, S-Type: Định Dạng Immediate

Nhớ lại thành phần cốt lõi của thiết kế RISC-V là giữ các trường nhất quán nhất có thể giữa các định dạng lệnh. Chúng ta đã thấy các trường thanh ghi nguồn/đích `rs1`, `rs2`, và `rd` nhất quán giữa các định dạng, cho phép sự rõ ràng nhất quán về thanh ghi nào để **đọc** và thanh ghi nào để **ghi**.

RISC-V cũng cố gắng giữ vị trí bit của immediate nhất quán. Sự "**xoáy**" của các bit immediate trong @fig-ISB-type-comparison thực sự đơn giản hóa thiết kế phần cứng!


:::{figure} images/ISB-type-comparison.png
:label: fig-ISB-type-comparison-vi
:width: 100%
:alt: "So sánh song song các định dạng lệnh I-type, S-type, và B-type, cho thấy cách các bit immediate giống nhau được đóng gói vào mỗi định dạng và cách lệnh B-type sắp xếp lại bit 11 và 12 so với I-type và S-type trong khi giữ vị trí bit trường rs1, rs2, funct3, và opcode thẳng hàng."

So sánh định dạng lệnh I-Type, S-Type, và B-Type.
:::

Quan sát:

* Trên I-Type, S-Type, B-Type, bit lệnh `inst[31]` luôn là bit dấu của `imm`.
* Immediate 13 bit của B-type có `imm[0] = 0`, vì vậy B-Type cố gắng giữ `imm[10:5]`, `imm[4:1]` ở cùng vị trí như trong I-Type và S-Type, ví dụ: bit lệnh `inst[30:25]` và `inst[11:8]`.
* Định dạng lệnh S-Type, B-Type có **chỉ hai bit thay đổi ý nghĩa**:
  * Bit lệnh `inst[31]` là bit immediate `imm[11]` trong S-Type và `imm[12]` trong B-Type.
  * Bit lệnh `inst[7]` là bit immediate `imm[0]` trong S-Type và `imm[11]` trong B-Type.

<!--

:::{tip} Kiểm Tra Nhanh

RV32I: Với lệnh 32 bit, imm[1] = imm[0] = 0.

:::

## Hình Ảnh

:::{figure} images/branch-isa-register.png
:label: fig-branch-isa-register-vi
:width: 100%
:alt: "Bố cục định dạng lệnh B-type: branchop rs1 rs2 Label với các trường imm[12|10:5], rs2, rs1, funct3, imm[4:1|11], và opcode; các ngoặc gắn nhãn rs1 và rs2 là hai thanh ghi nguồn được so sánh bởi nhánh."

Định dạng lệnh branch với thanh ghi.
:::

:::{figure} images/branch-isa-offset.png
:label: fig-branch-isa-offset-vi
:width: 100%
:alt: "Mã hóa B-type cộng với thanh thứ hai xây dựng lại offset nhánh có dấu 13 bit: bit lệnh ánh xạ đến bit offset 12:1, với số 0 ngầm định ở vị trí bit ít quan trọng nhất 0 cho căn chỉnh lệnh."

Định dạng lệnh branch với offset được gắn nhãn.
:::

:::{figure} images/practice-beq.png
:label: fig-practice-beq-vi
:width: 100%
:alt: "Bảng trường B-type nhỏ gọn với giá trị nhị phân: imm[12|10:5] là 0b0000000, rs2 là 0b01010, rs1 là 0b10011, funct3 là 0b000, imm[4:1|11] là 0b10000, và opcode là 0b1100011, nhất quán với lệnh beq."

Ví dụ lệnh branch.
:::


:::{figure} images/branch-example.png
:label: fig-branch-example-vi
:width: 100%
:alt: "Mã hóa chú thích của beq x19 x10 End: ngoặc ánh xạ rs1 đến x19 và rs2 đến x10, mũi tên gắn beq với funct3 000, trường opcode được gắn nhãn B-type, và offset nhánh được chia qua imm[12|10:5] và imm[4:1|11] với bit zero ngầm định cho imm[0]."

Ví dụ lệnh branch.
:::

:::{figure} images/branch-example-sol.png
:label: fig-branch-example-sol-vi
:width: 100%
:alt: "Cùng từ máy beq như trên với sơ đồ thứ hai lắp ráp immediate thành offset byte có dấu 13 bit là 16 (bốn lệnh), khớp với nhánh tiến đến nhãn End."

Giải pháp ví dụ lệnh branch.
:::


|   |   |   | funct3 |   | opcode |   |
| :-- | -- | -- | -- | -- | -- | --: |
| `imm[12\|10:5]` | `rs2` | `rs1` | `000` | `imm[4:1\|11]` | `1100011` | `beq` |
| `imm[12\|10:5]` | `rs2` | `rs1` | `001` | `imm[4:1\|11]` | `1100011` | `bne` |
| `imm[12\|10:5]` | `rs2` | `rs1` | `100` | `imm[4:1\|11]` | `1100011` | `blt` |
| `imm[12\|10:5]` | `rs2` | `rs1` | `110` | `imm[4:1\|11]` | `1100011` | `bltu` |
| `imm[12\|10:5]` | `rs2` | `rs1` | `101` | `imm[4:1\|11]` | `1100011` | `bge` |
| `imm[12\|10:5]` | `rs2` | `rs1` | `111` | `imm[4:1\|11]` | `1100011` | `bgeu` |

-->
