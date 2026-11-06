---
title: "Hỗ Trợ Immediate"
---

(sec-datapath-i-type-vi)=
## Mục Tiêu Học Tập

* Triển khai một datapath hỗ trợ các lệnh số học/logic R-Type và I-Type bằng cách tái sử dụng ALU và thêm một khối tạo immediate.
* Giải thích tại sao cần MUX ở đầu vào `B` của ALU.
* Thiết kế khối tạo immediate xuất ra giá trị immediate 32-bit từ lệnh 32-bit.
* Mở rộng khối tạo immediate để hỗ trợ nhiều định dạng immediate dựa trên định dạng lệnh: I-, S-, B-, J-, và U-Type.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/vtZ6UAboFRk
:width: 100%
:title: "[CS61C FA20] Lecture 18.5 - Single-Cycle CPU Datapath I: Datapath with Immediates"
:::

::::

## Xây dựng bộ xử lý R-Type + `addi`

Hãy mở rộng datapath R-Type để hỗ trợ các lệnh số học và logic I-Type, bắt đầu với `addi`. Để hỗ trợ `addi`:

* RegFile: Chúng ta **đọc** _một_ thanh ghi `rs1` và ghi một thanh ghi `rd`. Giá trị để ghi là `R[rs1] + imm`, **tổng** của giá trị thanh ghi đã đọc và một immediate.
* PC: Chúng ta **đọc** từ và **ghi** vào PC. Giá trị để ghi là `pc + 4`.

Lệnh `addi` cập nhật cùng hai trạng thái như các lệnh R-Type. Nhưng bây giờ chúng ta cần xây dựng immediate `imm`!

Để làm điều này, chúng ta **tái sử dụng** những gì đã tồn tại trong datapath R-Type, sau đó xem xét những khối bổ sung nào chúng ta cần thêm. Trong @fig-addi-cloud-vi, chúng ta nhận thấy:

* Chúng ta có thể giữ nguyên phần `PC = PC + 4` của datapath.
* Chúng ta có thể tái sử dụng nhiều phần đọc/ghi của phần RegFile trong datapath. Chúng ta vẫn muốn đọc `R[rs1]` và ghi `R[rd]`.
* Chúng ta muốn cộng hai giá trị 32-bit, nên chúng ta có lẽ nên tái sử dụng `ALU` để "cộng."
  * Chúng ta có thể giữ dây đến tín hiệu đầu vào `A` của ALU không đổi.
  * Chúng ta muốn thay đổi tín hiệu đầu vào `B` để được đặt thành immediate `imm` để ALU tính `alu = R[rs1] + imm`.

::::{figure} images/addi-cloud.png
:label: fig-addi-cloud-vi
:width: 100%
:alt: "Overlay datapath cho addi hiển thị các đường R-type được tái sử dụng và làm nổi bật nhu cầu tạo immediate và chọn đầu vào ALU."

`addi`: Tái sử dụng vòng lặp `PC = PC + 4` và lý tưởng là phép toán "add" trong ALU.
::::

Do đó chúng ta cần **logic bổ sung** mà, đối với các lệnh I-Type, đưa immediate `imm` vào đầu vào ALU `B` (thay vì `R[rs2]`, được sử dụng cho R-Type). @fig-addi-new-blocks-vi giới thiệu hai khối mới và nối chúng vào datapath hiện có:

1. Một **mux** mới chọn đầu vào ALU `B` dựa trên tín hiệu điều khiển mới `BSel`. Đọc về mux/bộ ghép kênh trong [phần trước](#sec-mux).
    * Đặt `BSel` là `1` để truyền immediate `imm`.
    * Đặt `Bsel` là `0` để truyền giá trị thanh ghi `R[rs2]`.
1. Một khối mới, **bộ tạo immediate**, tạo giá trị 32-bit `imm` từ các bit lệnh `inst`.

::::{figure} images/addi-new-blocks.png
:label: fig-addi-new-blocks-vi
:width: 100%
:alt: "Datapath cập nhật giới thiệu khối ImmGen và mux BSel đưa vào đầu vào ALU B với dữ liệu thanh ghi hoặc immediate."

`addi`: Thêm mux `BSel` và khối `ImmGen`.
::::

(sec-datapath-addi-vi)=
## Theo Dõi Datapath `addi`

Hãy đi qua datapath `addi` với kiến thức mới này.

::::{figure}
:label: anim-datapath-addi-vi
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vTd4As46cq50NW45y6rUi3eNLajyd-yaaiO2hky-fUgX5G5i5vQCgvdo0hQ78c2Bg/pubembed?start=false&loop=false
:width: 100%
:title: "Slides theo dõi Datapath `addi`."
:::
Datapath `addi`. Sử dụng thanh menu để theo dõi qua animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/1P6cAJaZCHFy5jj-MFHh4L2hy7XHpCH47/edit?usp=sharing). 
::::

1. **Fetch Lệnh**: Tăng PC đến lệnh tiếp theo (xem [datapath R-Type](#sec-datapath-add-vi)). Đọc lệnh `inst` từ IMEM.

1. **Decode Lệnh**:
    * Đọc `R[rs1]` từ RegFile (xem [R-Type](#sec-datapath-add-vi)).
    * Thiết lập thanh ghi đích `rd` để ghi.
    * Xây dựng immediate `imm`. Đối với [các lệnh I-Type](#tab-i-type), nối 12 bit cao của lệnh `inst[31:20]` đến đầu vào của khối Bộ Tạo Immediate.
    * Cấu hình logic điều khiển.
        * Cấu hình `ImmSel` thành immediate loại `I` (hiện tại, chúng ta chỉ có một loại immediate).
        * Đặt `RegWEn` là `1`.
        * Đặt `BSel` là `1`.
        * Đặt `ALUSel` là `Add`.

    Sau một độ trễ, tín hiệu đầu ra `imm` của khối bộ tạo immediate được cập nhật thành giá trị immediate 32-bit mở rộng dấu phù hợp, giá trị thanh ghi `R[rs1]` được đọc, và các tín hiệu điều khiển được đặt.

1. **Thực Thi**: Vì đường điều khiển `BSel=1` chọn immediate đã tạo `imm` cho đầu vào ALU `B`, ALU của chúng ta tính `R[rs1] + imm`.

1. **Bộ Nhớ**: (Chúng ta không truy cập DMEM, nên bỏ qua.)

1. **Ghi Lại**: Ghi đầu ra ALU vào thanh ghi đích bằng cách kết nối `alu` đến đầu vào `wdata` của RegFile (xem [R-Type](#sec-datapath-add-vi)).

    Xung quanh cạnh lên clock tiếp theo, `wdata`, `RegWEn`, và `rd` nên được giữ ổn định qua thời gian setup và hold của RegFile.


:::{note} RegFile vẫn xuất hai tín hiệu!

Trong datapath của chúng ta, các hoạt động đọc giống như logic tổ hợp, nghĩa là RegFile, IMEM, và DMEM thực hiện đọc dựa trên các tín hiệu đầu vào. Không có cách nào để "tắt" các hoạt động đọc.

Dữ liệu `inst[24:20]` vẫn đưa vào RegFile, vẫn xuất `R[rs2]`.
Tuy nhiên, điều khiển `Bsel=1` có nghĩa là đường dữ liệu `R[rs2]` bị bỏ qua.

Nói chung, các sơ đồ của chúng ta sẽ bỏ qua các đường dữ liệu không được chọn khỏi việc sáng.

:::

(sec-datapath-immgen-vi)=
## Khối Bộ Tạo Immediate

Chúng tôi khuyến khích xem lại phần này sau khi đọc thêm một vài ví dụ theo dõi datapath.

:::{note} Bộ Tạo Immediate là logic tổ hợp

Trong triển khai RISC-V đầy đủ, Bộ Tạo Immediate của chúng ta (@fig-element-immgen-vi) tạo các giá trị 32-bit cho tất cả các định dạng lệnh có immediate. Đầu ra 32-bit chính xác `imm` phụ thuộc vào lệnh `inst`, chỉ định:

* "Loại" immediate, dựa trên loại định dạng lệnh: I-, S-, B-, J-, và U-Type.
* Các bit immediate trong lệnh, phụ thuộc vào loại lệnh.

Chúng tôi khuyến nghị xem lại "bit swirling" của immediate từ [phần trước](#sec-imm-swirl).

:::

:::{figure} images/element-immgen.png
:label: fig-element-immgen-vi
:alt: "Khối ImmGen cho immediate I-type: đầu vào là inst[31:20] và đầu ra là imm[31:0] sử dụng mở rộng dấu. Khối được điều khiển bởi tín hiệu điều khiển ImmSel."

Khối Bộ Tạo Immediate
:::

:::{table} Tín hiệu cho Bộ Tạo Immediate. Tên tín hiệu project khóa học, nếu khác, được đặt trong ngoặc đơn.
:label: tab-immgen-signals-vi
:align: center

| Tên | Hướng | Độ Rộng Bit | Mô Tả |
| :-- | :-- | :-- | :-- |
| Đầu vào | `inst` (`Instruction`) | 32 | Lệnh đang được thực thi |
| Đầu vào | `ImmSel` | 3  | Giá trị xác định cách tái tạo immediate |
| Đầu ra | `imm` (`Immediate`) | 32 | Giá trị của immediate trong lệnh |

:::

Nhớ rằng các bit của immediate được lưu trữ ở các bit khác nhau của lệnh, tùy thuộc vào loại lệnh. Tín hiệu `ImmSel`, được triển khai trong logic điều khiển, sẽ xác định loại immediate nào subcircuit này nên tạo.

Các định dạng lưu trữ immediate được liệt kê bên dưới trong @tab-immgen-types-vi.

:::{table} Các Định dạng Lưu trữ Immediate từ [RISC-V Green Card](#sec-green-card).
:label: tab-immgen-types-vi
<table style="border: 1px solid black; border-collapse: collapse; font-family: monospace; width: 100%;">
  <thead>
    <tr style="background-color: #f2f2f2;">
      <th style="border: 1px solid black; padding: 5px; text-align: left;">Loại</th>
      <th style="border: 1px solid black; padding: 5px; text-align: left;">ImmSel (mặc định)</th>
      <th style="border: 1px solid black; padding: 5px; text-align: center;">Bit 31-20</th>
      <th style="border: 1px solid black; padding: 5px; text-align: center;">Bit 19-12</th>
      <th style="border: 1px solid black; padding: 5px; text-align: center;">Bit 11</th>
      <th style="border: 1px solid black; padding: 5px; text-align: center;">Bit 10-5</th>
      <th style="border: 1px solid black; padding: 5px; text-align: center;">Bit 4-1</th>
      <th style="border: 1px solid black; padding: 5px; text-align: center;">Bit 0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid black; padding: 5px;">I</td>
      <td style="border: 1px solid black; padding: 5px;"><code>0b000</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;" colspan="3"><code>inst[31]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;" colspan="3"><code>inst[30:20]</code></td>
    </tr>
    <tr>
      <td style="border: 1px solid black; padding: 5px;">S</td>
      <td style="border: 1px solid black; padding: 5px;"><code>0b001</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;" colspan="3"><code>inst[31]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;"><code>inst[30:25]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;" colspan="2"><code>inst[11:7]</code></td>
    </tr>
    <tr>
      <td style="border: 1px solid black; padding: 5px;">B</td>
      <td style="border: 1px solid black; padding: 5px;"><code>0b010</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;" colspan="2"><code>inst[31]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;"><code>inst[7]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;"><code>inst[30:25]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;"><code>inst[11:8]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;"><code>0</code></td>
    </tr>
    <tr>
      <td style="border: 1px solid black; padding: 5px;">U</td>
      <td style="border: 1px solid black; padding: 5px;"><code>0b011</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;" colspan="2"><code>inst[31:12]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;" colspan="4"><code>0</code></td>
    </tr>
    <tr>
      <td style="border: 1px solid black; padding: 5px;">J</td>
      <td style="border: 1px solid black; padding: 5px;"><code>0b100</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;"><code>inst[31]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;"><code>inst[19:12]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;"><code>inst[20]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;" colspan="2"><code>inst[30:21]</code></td>
      <td style="border: 1px solid black; padding: 5px; text-align: center;"><code>0</code></td>
    </tr>
  </tbody>
</table>

:::

Quan sát/nhắc nhở:

* Bạn nên xử lý immediate I*-type như immediate I-type, vì ALU chỉ nên sử dụng 5 bit thấp nhất của đầu vào B khi tính toán dịch.
* Nhớ rằng tất cả immediate là 32 bit và **mở rộng dấu**. Mở rộng dấu được hiển thị trong @tab-immgen-types-vi là `inst[31]` được lặp lại ở các bit cao.
* Các lệnh U-type yêu cầu dịch trái immediate 12 bit (ví dụ: `lui` được viết là `rd = imm << 12` trên thẻ tham chiếu). Điều này nên được thực hiện trong bộ tạo immediate để datapath không cần thực hiện dịch bổ sung.

Trong các tiểu mục sau, chúng ta "lặp lại" xây dựng bộ tạo immediate để hỗ trợ I-Type, sau đó S-Type, sau đó B-Type. Chúng tôi để việc triển khai immediate J-Type và U-Type cho bạn và project khóa học.

### I-Type

Đầu tiên, giả sử datapath của chúng ta chỉ hỗ trợ immediate từ các lệnh I-Type. Trong trường hợp này, bộ tạo immediate sẽ thực hiện hai hoạt động như trong @fig-immgen-i-type-vi.
:::{figure} images/immgen-i-type.png
:label: fig-immgen-i-type-vi
:alt: "Mô tả ImmGen cho I-type và S-type sử dụng mux để chọn các bit immediate thấp trong khi chia sẻ mở rộng dấu và nối các bit giữa."

Khối Bộ Tạo Immediate: I-Type
:::

:::{note} Hiện Giải Thích
:class: dropdown

1. `imm[31:12]`: **Mở rộng dấu**. Sao chép **bit dấu** `inst[31]` vào 20 bit cao của immediate, `imm[31:12]`.
1. `imm[11:0]`: **Nối dây**. Kết nối trực tiếp 12 bit cao của lệnh `inst[31:20]` đến 12 bit thấp của đầu ra `imm[11:0]`.

:::


### I-Type, S-Type

Tiếp theo, giả sử datapath của chúng ta hỗ trợ immediate từ cả lệnh I-Type và S-Type. Bộ tạo immediate phải đặt các bit thấp `imm[4:0]` dựa trên loại immediate. Trong @fig-immgen-i-s-type-vi, chúng ta triển khai việc chọn này với một MUX.

:::{figure} images/immgen-i-s-type.png
:label: fig-immgen-i-s-type-vi
:alt: "ImmGen cho I-type và S-type với mux định tuyến các bit lệnh phân tán và mở rộng dấu chia sẻ."

Khối Bộ Tạo Immediate: I-Type, S-Type
:::

:::{note} Hiện Giải Thích
:class: dropdown

1. `imm[31:12]`: **Mở rộng dấu**. Trong cả hai định dạng lệnh, `inst[31]` là bit dấu của immediate. Sao chép **bit dấu** `inst[31]` vào 20 bit cao của immediate, `imm[31:12]`.
1. `imm[11:5]`: **Nối dây**. Kết nối trực tiếp các bit lệnh `inst[31:25]` đến đầu ra `imm[11:5]`.
1. `imm[4:0]`: **Chọn**. Chọn năm bit để điền `imm[4:0]`: `inst[24:20]` nếu I-Type, và `inst[11:7]` nếu S-Type.
:::

(sec-immgen-b-type-vi)=
### I-Type, S-Type, B-Type

Cuối cùng, giả sử datapath của chúng ta hỗ trợ các lệnh I-Type, S-Type, và B-Type. Thiết kế bộ tạo immediate được hiển thị trong @fig-immgen-i-s-b-type-vi, bây giờ với nhiều MUX hơn.

:::{figure} images/immgen-i-s-b-type.png
:label: fig-immgen-i-s-b-type-vi
:alt: "ImmGen cho I-type, S-type, và B-type với nhiều mux định tuyến các bit lệnh phân tán, mở rộng dấu, và vị trí zero ngầm định."

Khối Bộ Tạo Immediate: I-Type, S-Type, B-Type
:::

:::{note} Hiện Giải Thích
:class: dropdown

1. `imm[31:12]`: **Mở rộng dấu**. Trong cả ba định dạng lệnh, `inst[31]` là bit dấu của immediate. Sao chép bit dấu `inst[31]` vào 20 bit cao của immediate, `imm[31:12]`.
1. `imm[11]`: **Chọn** `inst[31]` nếu I-Type hoặc S-Type, và `inst[7]` nếu B-Type.
1. `imm[10:5]`: **Nối dây**. Kết nối trực tiếp các bit lệnh `inst[30:25]` đến đầu ra `imm[10:5]`.
1. `imm[4:1]`: **Chọn**. `inst[24:21]` nếu I-Type, và `inst[11:8]` nếu S- hoặc B-Type.
1. `imm[0]`: **Chọn**. `inst[20]` nếu I-Type, `inst[7]` nếu S-Type, và `0` ([ngầm định](#sec-implicit-zero-b-type)) nếu B-Type.

:::

### Chi Tiết Project Khóa Học

:::{table} Mã hóa `ImmSel` mặc định cho project khóa học. Bạn có thể chọn mã hóa riêng của mình.
:label: tab-immgen-operations-vi

| Giá trị `ImmGen` <br/>(cho Project) | Loại Immediate |
| :-- | :-- |
| `0b000` | I |
| `0b001` | S |
| `0b010` | B |
| `0b011` | U |
| `0b100` | J |

:::
