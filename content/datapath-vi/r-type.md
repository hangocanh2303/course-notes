---
title: "Datapath R-Type, ALU"
---

(sec-datapath-r-type-vi)=
## Mục Tiêu Học Tập

* Triển khai một datapath hỗ trợ các lệnh R-Type.
* So sánh các phần tử datapath và logic cần thiết để hỗ trợ các lệnh `add` và `sub`.
* Suy luận về thiết kế cấp cao của phần tử ALU hỗ trợ các lệnh RV32I.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/f1cZGi4mZqo
:width: 100%
:title: "[CS61C FA20] Lecture 18.3 - Single-Cycle CPU Datapath I: R-Type Add Datapath"
:::

Đến 7:40. Timing của add từ 7:40 trở đi được thảo luận trong [phần sau](#sec-instruction-timing).
::::

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/fVjZzX46SU4
:width: 100%
:title: "[CS61C FA20] Lecture 18.4 - Single-Cycle CPU Datapath I: Sub Datapath"
:::

::::

## Xây dựng bộ xử lý thực hiện `add`

Để bắt đầu, hãy xây dựng bộ xử lý đơn giản nhất có thể: một bộ xử lý chỉ có thể xử lý một lệnh: `add`. Các chương trình sẽ chỉ là một chuỗi các `add`:

```bash
add x18 x18 x10
add x18 x18 x18
add ...
```

:::{hint} Ôn lại lệnh `add`

* [Lệnh `add`](#sec-rv-arithmetic)
* [Định dạng lệnh R-Type](#sec-r-type)
* [Bộ Đếm Chương Trình](#sec-rv-pc)
:::

Để hỗ trợ `add` trong datapath của chúng ta, chúng ta xem xét hai phần tử trạng thái được thay đổi bởi các hoạt động của lệnh này:

* RegFile: Chúng ta **đọc** hai thanh ghi `rs1` và `rs2` và ghi một thanh ghi `rd`. Giá trị để ghi là **tổng** giữa hai giá trị thanh ghi đã đọc, `R[rs1] + R[rs2]`.
* PC: Chúng ta **đọc** từ và **ghi** vào thanh ghi PC. Giá trị để ghi là `PC + 4`.

Các phần tử trạng thái khác:

* IMEM: Bộ xử lý phải **đọc** lệnh RV32I từ IMEM chỉ đọc trong giai đoạn `IF` (Fetch Lệnh).
* DMEM: Bộ xử lý không truy cập thêm bộ nhớ qua load hoặc store. Lệnh `add` không tham gia vào giai đoạn `MEM` của [quy trình năm bước](#sec-five-steps-vi).

:::{warning} Mục tiêu: Xây dựng Bộ Xử Lý theo Từng Bước
Trong các ghi chú này, chúng ta sẽ xây dựng bộ xử lý theo từng bước, nghĩa là chúng ta sẽ giới thiệu các khối logic tổ hợp quan trọng khi bộ xử lý hỗ trợ thêm nhiều lệnh. Do đó một số animation sẽ không bao gồm toàn bộ datapath. Để xem các bài tập liên quan đến datapath đầy đủ, xem cuối chương này.
:::

:::{figure} images/add-no-dmem.png
:label: fig-add-no-dmem-vi
:alt: "Biến thể datapath cho việc thực thi chỉ add với IMEM, PC, RegFile, và ALU hoạt động trong khi DMEM bị ngắt kết nối và không sử dụng."
Hiện tại, chúng ta ngắt kết nối DMEM vì nó không được sử dụng cho `add` (@fig-add-no-dmem-vi). Chúng ta sẽ thêm nó lại khi thảo luận về [load và store](#sec-datapath-load-store).
:::

(sec-datapath-add-vi)=
## Theo Dõi Datapath `add`

Với phân tích ở trên, giờ chúng ta có thể kết nối dây giữa các phần tử chính của bộ xử lý.

::::{figure}
:label: anim-datapath-add-vi
:alt: "Các slide nhúng theo dõi fetch thanh ghi, ALU add, và ghi lại trên datapath chu kỳ đơn cho lệnh add R-type."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vTjoG1_5TXfFBOQnHyRBwkrcRobHXJH4WCXyt57uABZuIpE3eBShew6e1ma0qVJ3A/pubembed?start=false&loop=false
:width: 100%
:title: "Slides theo dõi Datapath `add`."
:::
Datapath `add`. Sử dụng thanh menu để theo dõi qua animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/1Md-1g2Cme3ScuGS4jPW23wFhUi8ysedu/edit?usp=sharing).
::::

1. **Fetch Lệnh**:
    * Ở cạnh lên clock, dây `pc` cập nhật thành lệnh để thực thi trong chu kỳ này. Nó đưa vào IMEM, sau một độ trễ, cập nhật tín hiệu đầu ra `inst`.
    * Tăng PC đến lệnh tiếp theo. Dây `pc` cũng đưa vào một bộ cộng nhỏ cộng `4`. Đầu ra của bộ cộng nhỏ này được nối đến đầu vào của thanh ghi `PC`, được thiết lập và sẵn sàng cập nhật ở cạnh lên clock tiếp theo.

1. **Decode Lệnh**: Chúng ta chỉ có một lệnh, nên decoding đơn giản là giải mã các bit cụ thể để xác định các thanh ghi. Chúng ta sử dụng green card và [định dạng R-Type](#tab-rv32i-types) của chúng ta để giới thiệu một bộ tách trên tín hiệu `inst` để "index" vào RegFile như sau:
    * Nối `inst[7:11]` (bit 7 đến 11, bao gồm) đến đầu vào `rd` của RegFile.
    * Nối `inst[15:19]` đến đầu vào `rs1` của RegFile.
    * Nối `inst[20:24]` đến đầu vào `rs2` của RegFile.

    Sau một độ trễ, RegFile cập nhật các tín hiệu `rdata1` và `rdata2` thành các giá trị của `R[rs1]` và `R[rs2]`, trong đó `rs1` và `rs2` được xác định từ lệnh `inst`.

1. **Thực Thi**: ALU của chúng ta (xem [bên dưới](#sec-datapath-alu-vi)) nên thực hiện phép toán *Cộng*. Hiện tại, chúng ta chỉ đánh dấu khối này là Adder. Đưa hai tín hiệu đầu ra RegFile vào các đầu vào `A` và `B` của "ALU" (được đánh dấu là khối Add trong @anim-datapath-add-vi). Sau một độ trễ, tín hiệu đầu ra của khối được cập nhật thành *tổng* của hai giá trị đầu vào.

1. **Bộ Nhớ**: (Chúng ta không truy cập bộ nhớ, nên bỏ qua.)

1. **Ghi Lại**: Kết nối tín hiệu đầu ra của ALU đến tín hiệu đầu vào `wdata` của RegFile. Đặt tín hiệu điều khiển RegFile `RegWEn` là `1` để chỉ ra rằng `wdata` nên được ghi vào `R[rd]` ở cạnh lên clock tiếp theo.

    Xung quanh cạnh lên clock tiếp theo, `wdata`, `RegWEn`, và `rd` nên được giữ ổn định qua thời gian setup và hold của RegFile.


## Xây dựng bộ xử lý thực hiện `add` và `sub`

Tiếp theo, hãy cải thiện bộ xử lý của chúng ta bằng cách hỗ trợ hai lệnh: `add` và `sub`. Chương trình ví dụ:

```bash
add x18 x18 x10
sub x18 x18 x18
sub ...
add ...
```

Hãy xem xét lại các phần tử trạng thái được thay đổi bởi các hoạt động của lệnh này:

* RegFile: Chúng ta vẫn **đọc** hai thanh ghi `rs1` và `rs2` và ghi một thanh ghi `rd`. Nhưng bây giờ giá trị để ghi là **hiệu** giữa hai giá trị thanh ghi đã đọc, `R[rs1] - R[rs2]`.
* PC: Chúng ta **đọc** từ và **ghi** vào thanh ghi PC. Giá trị để ghi là `pc + 4`.

`sub` gần giống như `add`, ngoại trừ bây giờ ALU thực hiện phép trừ. Chúng ta triển khai hỗ trợ cho cả `add` và `sub` bằng cách giả định thêm độ phức tạp trong "khối" Logic Điều khiển (@fig-sub-datapath-vi).

:::{figure} images/sub-datapath.png
:label: fig-sub-datapath-vi
:alt: "Datapath cho add và sub trong đó logic điều khiển decode các bit lệnh và điều khiển ALUSel trong khi các đường thanh ghi và PC vẫn giữ nguyên."
Để triển khai `sub` và `add`, chúng ta cập nhật logic điều khiển.
:::

:::{note} Hiện giải thích

* Logic điều khiển nhận làm đầu vào tất cả 32 bit của lệnh `inst`.
* Xuất tín hiệu `RegWEn`, kết nối với RegFile. Đặt là `1` để đảm bảo chúng ta ghi `wdata` vào `R[rd]`.
* Xuất tín hiệu điều khiển `ALUSel`, kết nối với ALU, chọn phép toán ALU để xuất ra. Xuất tín hiệu "add" hoặc "sub", tùy thuộc vào lệnh.
:::

Làm thế nào để xác định `add` hay `sub`? Nhớ lại thảo luận của chúng ta về [Quyết định Thiết kế cho R-Type](#tab-r-type): `add` và `sub` có cùng các trường `opcode` và `funct3`, nhưng các trường `funct7` khác nhau. Quan trọng, bit `inst[30]` là `1` cho `sub` và `0` cho `add`.

## Xây dựng bộ xử lý R-Type

Chúng ta có thể mở rộng lý luận ở trên để xây dựng một bộ xử lý triển khai tất cả [các lệnh R-Type](#tab-r-type):

* Xây dựng một ALU hỗ trợ tất cả các phép toán số học và logic R-Type trên các toán hạng `R[rs1]` và `R[rs2]`.
* Xây dựng một khối Logic Điều khiển sử dụng các bit lệnh `inst` để chọn các phép toán ALU phù hợp và đặt `ALUSel` tương ứng.

(sec-datapath-alu-vi)=
## Đơn Vị Số Học Logic (ALU)

Chúng tôi khuyến khích xem lại phần này sau khi đọc thêm một vài ví dụ theo dõi datapath.

Trong [chương trước](#sec-alu) chúng ta đã triển khai một ALU bốn phép toán cơ bản. Như trong @fig-element-alu-vi và @tab-alu-signals-vi, ALU RISC-V nhận cùng đầu vào và đầu ra, nhưng tín hiệu điều khiển `ALUSel` rộng hơn nhiều để phù hợp với chức năng cần thiết cho datapath RISC-V đầy đủ.

:::{figure} images/element-alu.png
:label: fig-element-alu-vi
:width: 40%
:alt: "Ký hiệu khối ALU với đầu vào A, B, và ALUSel và đầu ra ALUResult, đại diện cho các phép toán số học và logic được sử dụng qua các loại lệnh."
Khối ALU.

:::

:::{table} Tín hiệu cho Khối ALU
:label: tab-alu-signals-vi
:align: center

| Tên | Hướng | Độ Rộng Bit | Mô Tả |
| :-- | :-- | :-- | :-- |
| `A` | Đầu vào | 32  | Dữ liệu để sử dụng cho Đầu vào A trong phép toán ALU |
| `B` | Đầu vào | 32  | Dữ liệu để sử dụng cho Đầu vào B trong phép toán ALU |
| `ALUSel` | Đầu vào | 4 | Chọn phép toán nào ALU nên thực hiện (xem @tab-alu-operations-vi) |
| `ALUResult` | Đầu ra | 32 | Kết quả của phép toán ALU |

:::

Trong triển khai RISC-V đầy đủ, ALU của chúng ta (@fig-element-alu-vi) phải thực hiện số học cho nhiều tín hiệu:

* Các phép toán số học và logic thanh ghi-thanh ghi cho các lệnh R-Type
* Các phép toán số học và logic thanh ghi-immediate cho các lệnh [I-Type](#sec-datapath-i-type-vi)
* Tính toán địa chỉ Base + Immediate cho [load và store](#sec-datapath-load-store)
* Tính toán địa chỉ tương đối PC (Xem datapath cho [Branch](#sec-datapath-b-type) và [Jump](#sec-datapath-jump))
* Tính toán upper immediate (xem datapath cho [U-Type](#sec-datapath-u-type))

### Chi Tiết Project Khóa Học

Bên dưới, chúng ta chi tiết các phép toán ALU phải được triển khai cho datapath của **project khóa học**.

:::{warning} Project Khóa Học vs. RISC-V ISA

Lưu ý rằng việc triển khai các đơn vị chức năng như ALU **cố ý không được chỉ định** trong ISA. Nói chung, bạn không thể/không nên mong đợi tất cả các kiến trúc RISC-V tuân theo bất kỳ đặc tả nào trong @tab-alu-signals-vi, bao gồm nhưng không giới hạn:

* Các giá trị `ALUSel` được chỉ định (ví dụ: add có `ALUSel=0`)
* Hỗ trợ cho phép nhân tổng quát (không có trong Base RV32I ISA)
* `bsel`, cần thiết cho hỗ trợ của datapath cho [U-Type](#sec-datapath-u-type).
:::

:::{table} Các phép toán cho Khối ALU cho project khóa học
:label: tab-alu-operations-vi

| Giá trị `ALUSel` <br/>(cho Project) | Phép toán | Hàm ALU |
| :-- | :-- | :-- |
| 0  |  add   | `ALUResult = A + B` |
| 1  | sll    | `ALUResult = A << B[4:0]`|
| 2  | slt    | `ALUResult = (A < B (có dấu)) ? 1 : 0` |
| 3  | Không dùng | - |
| 4  | xor    | `ALUResult = A ^ B` |
| 5  | srl    | `ALUResult = (không dấu) A >> B[4:0]` |
| 6  | or     | `ALUResult = A \| B` |
| 7  | and    | `ALUResult = A & B` |
| 8  | mul    | `ALUResult = (có dấu) (A * B)[31:0]` |
| 9  | mulh   | `ALUResult = (có dấu) (A * B)[63:32]` |
| 10 | Không dùng | - |
| 11 | mulhu  | `ALUResult = (A * B)[63:32]` |
| 12 | sub    | `ALUResult = A - B` |
| 13 | sra    | `ALUResult = (có dấu) A >> B[4:0]` |
| 14 | Không dùng | - |
| 15 | bsel   | `ALUResult = B` |
::::

Quan sát/nhắc nhở:

* Khi thực hiện dịch, chỉ cần 5 bit thấp của `B`, vì chỉ hỗ trợ dịch đến 32.
* Thành phần bộ so sánh có thể hữu ích cho việc triển khai các lệnh liên quan đến so sánh đầu vào. Xem [triển khai branch](#sec-datapath-b-type) sau trong chương này.
* Bộ ghép kênh (MUX) có thể hữu ích khi quyết định giữa các đầu ra phép toán (nhớ lại [ALU 4 phép toán cơ bản](#sec-alu) của chúng ta). Xem xét xử lý đầu vào cho **tất cả** các phép toán trước, sau đó xuất ra phép toán mà bạn chọn.
* Xem ghi chú phép nhân tổng quát bên dưới.

(sec-general-multiplication-vi)=
### Phép Nhân Tổng Quát

ALU triển khai các lệnh `mul`, `mulh`, và `mulhu` có thể hỗ trợ các phần của [RISC-V "M" extension](https://docs.riscv.org/reference/isa/unpriv/m-st-ext.html).

| Lệnh | Tên | Mô Tả | Loại | Opcode | Funct3 | Funct7 |
| :--- | :--- | :--- | :---: | :------ | :--- | :--- |
| `mul rd rs1 rs2` | MULtiply | `R[rd] = (R[rs1] * R[rs2])[31:0]` | R | `011 0011` | `000` | `000 0001` |
| `mulh rd rs1 rs2` | MULtiply Higher Bits | `R[rd] = (R[rs1] * R[rs2])[63:32]` (Có dấu) | R | `011 0011` | `001` | `000 0001` |
| `mulhu rd rs1 rs2` | MULtiply Higher Bits (Không dấu) |  `R[rd] = (R[rs1] * R[rs2])[63:32]` (Không dấu)  | R | `011 0011` | `011` | `000 0001` |

Kết quả của việc nhân 2 số 32-bit có thể lên đến 64 bit thông tin, nhưng chúng ta bị giới hạn ở các đường dữ liệu 32-bit, nên `mulh` và `mulhu` được sử dụng để lấy 32 bit cao của tích. Thành phần `Multiplier` có đầu ra `Carry Out` (với mô tả "các bit cao của tích") có thể đặc biệt hữu ích cho một số phép toán nhân.
