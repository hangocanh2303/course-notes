---
title: "Hỗ Trợ Branch"
---

(sec-datapath-b-type-vi)=
## Mục Tiêu Học Tập

* Triển khai một datapath hỗ trợ branch có điều kiện (B-Type).
* Giải thích cách các tín hiệu đầu vào/đầu ra của bộ so sánh branch giúp xác định (cho tất cả các lệnh B-Type) liệu branch có được "taken" (thực hiện).
* Giải thích cách tín hiệu điều khiển PCSel xác định lệnh cần thực thi trong chu kỳ clock tiếp theo.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/XTgyEsFRvWg
:width: 100%
:title: "[CS61C FA20] Lecture 19.3 - Single-Cycle CPU Datapath II: Implementing Branches"
:::

::::

Để hỗ trợ [các lệnh branch](#sec-b-type) như `beq` chúng ta phải xem xét **cập nhật phần tử trạng thái**, **các phép toán số học**, và **bộ chọn dữ liệu**.

_Cập nhật phần tử trạng thái_:

* RegFile: Chúng ta **đọc** hai thanh ghi `rs1` và `rs2` và so sánh các giá trị `R[rs1]` và `R[rs2]`. Chúng ta không ghi vào bất kỳ thanh ghi nào.
* PC: Chúng ta **đọc** từ và **ghi** vào PC. Giá trị để ghi bây giờ **có điều kiện** phụ thuộc vào kết quả của so sánh hai thanh ghi, xác định liệu branch có được *taken*:
  * **taken**: `pc + imm`
  * **not taken**: `pc + 4`
* DMEM: Không đọc cũng không ghi.

Như trước, chúng ta **tái sử dụng** những gì đã tồn tại trong datapath R-, I-, và S-Type. Ngay cả với điều này, chúng ta sẽ cần thêm ba khối mới và một số logic điều khiển bổ sung.

(sec-intro-branch-comparator-vi)=
### Bộ So Sánh Branch (Branch Comparator)

Có **ba** phép toán số học mà các lệnh branch phải (chủ động) thực hiện.

1. `pc + 4`. Phần cứng này đã có trong datapath của chúng ta.
1. `pc + imm`.
1. So sánh `R[rs1]` và `R[rs2]`.

Chúng ta chỉ có một [ALU](#fig-element-alu-vi) đa năng sẵn có trong giai đoạn `EX` của datapath chu kỳ đơn. Chúng ta sử dụng ALU này để tính `pc + imm`. Chúng ta thảo luận chi tiết [bên dưới](#sec-datapath-asel-vi)

Vì ALU này bây giờ bận, chúng ta phải giới thiệu logic tổ hợp bổ sung để tính so sánh `R[rs1]` và `R[rs2]` trong cùng một chu kỳ clock. Chúng ta gọi khối logic tổ hợp mới này là [**bộ so sánh branch**](#sec-datapath-branch-comparator-vi).

Chúng ta thảo luận chi tiết về bộ so sánh branch ở [cuối phần này](#sec-datapath-branch-comparator-vi).

### MUX cho Đầu Vào PC

Để cập nhật có điều kiện đầu vào cho phần tử `PC`, chúng ta giới thiệu một **mux** mới chọn giữa `pc + imm` và `pc + 4` để đưa vào phần tử `PC`. Do đó chúng ta cũng giới thiệu một **tín hiệu điều khiển** mới `PCSel` để đưa vào mux này.

Hai khối mới này được hiển thị trong @fig-branch-new-blocks-vi. Bộ so sánh branch thực hiện phép toán logic để so sánh `R[rs1]` và `R[rs2]` và đưa hai tín hiệu 1-bit rộng, `BrEq` và `BrLT`, vào logic điều khiển. Mux mới sử dụng `PCSel` để cập nhật `PC` dựa trên kết quả branch.

:::{figure} images/branch-new-blocks.png
:label: fig-branch-new-blocks-vi
:alt: "Bổ sung datapath branch: đầu ra bộ so sánh branch vào logic điều khiển và mux PCSel chọn giữa PC cộng bốn và đích branch."

Khối bộ so sánh branch và mux `PCSel`, với tín hiệu điều khiển `PCSel`.
:::

(sec-datapath-asel-vi)=
### MUX cho Đầu Vào ALU

Chúng ta cần **thêm một mux nữa** trong datapath để tính `PC + imm` với ALU hiện có. Mux mới trong @fig-branch-new-blocks-asel-vi chọn đầu vào ALU `A` dựa trên tín hiệu điều khiển mới, `ASel`.

* Đặt `ASel` là `1` để truyền giá trị bộ đếm chương trình `pc`.
* Đặt `ASel` là `0` để truyền giá trị thanh ghi `R[rs1]`.

:::{figure} images/branch-new-blocks-asel.png
:label: fig-branch-new-blocks-asel-vi
:alt: "Datapath hiển thị mux ASel bổ sung trước ALU chọn dữ liệu thanh ghi hoặc PC để branch có thể tính PC cộng immediate."

Branch yêu cầu hai mux với hai tín hiệu điều khiển: `PCSel` và `ASel`. Cái sau xác định một trong các đầu vào cho ALU.
:::

### Khối Bộ Tạo Immediate

Chúng ta cũng phải cập nhật khối Bộ Tạo Immediate, `ImmGen`. Immediate trong B-Type [khác với I-Type và S-Type](#sec-imm-swirl) và có [trailing zero ngầm định](#sec-implicit-zero-b-type). Đọc thêm trong [phần Bộ Tạo Immediate](#sec-immgen-b-type-vi).

## Theo Dõi Datapath Branch

Hãy đi qua datapath cập nhật cho các lệnh branch (B-Type):

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vTmG8BNrBej6JPZP0ojYqkOD7YGviC9zxdYq340Lr6970db52IUXk4_5-v8tt4Cpw/pubembed?start=false&loop=false" frameborder="0" width="100%" height="600px" title="Theo dõi slide datapath cho các lệnh branch."></iframe>

1. **Fetch Lệnh**: Ở đầu chu kỳ clock, đọc PC và fetch lệnh hiện tại từ IMEM.

    Trước cạnh lên clock tiếp theo, thiết lập `PCSel`, và đảm bảo rằng đầu vào cho PC ổn định. Nếu `PCSel=taken`, cập nhật PC thành đầu ra của ALU. Nếu không, cập nhật thành lệnh tiếp theo `pc + 4`.

1. **Decode Lệnh**: Fetch `R[rs1]` và `R[rs2]` từ RegFile, xây dựng immediate `imm` cho các lệnh B-Type. Cũng cấu hình logic điều khiển (xem bên dưới).

1. **Thực Thi**: Tính `pc + imm` sử dụng ALU. Vì các tín hiệu điều khiển là `ASel=1` và `BSel=1`, hai mux trước ALU sẽ chọn `pc` và `imm`, tương ứng. Vì `ALUSel=Add`, ALU sẽ cộng hai giá trị này với nhau.

    Bộ so sánh branch so sánh `R[rs1]` và `R[rs2]` (thực hiện so sánh không dấu nếu `BrUn=1`) và xuất hai tín hiệu, `BrEq` và `BrLT`, đến logic điều khiển.

    Sau một độ trễ, đầu ra của ALU ổn định tại mux được điều khiển bởi `PCSel`. Ngoài ra, tín hiệu điều khiển `PCSel` ổn định sau khi logic điều khiển sử dụng các tín hiệu `BrEq` và `BrLT` để xác định liệu có taken branch (xem chi tiết [bên dưới](#sec-control-pcsel-vi)).

1. **Bộ Nhớ**: (Chúng ta không truy cập DMEM, nên bỏ qua.)

1. **Ghi Lại**: (Chúng ta không ghi vào RegFile, nên bỏ qua.)


:::{note} Tín Hiệu Điều Khiển cho Branch

* Cấu hình `ImmSel` thành immediate loại `UB`.
* Đặt `RegWEn` là `0`. Điều này có nghĩa là **không** ghi lại vào RegFile.
* Đặt `BrUn` là `1` nếu bộ so sánh branch nên thực hiện so sánh không dấu. Nếu bộ so sánh branch nên thực hiện so sánh có dấu, đặt là `0`.
* Đặt `ASel` là `1`.
* Đặt `BSel` là `1`.
* Đặt `ALUSel` là `Add`.
* Đặt `MemRW` là `Read`. Điều này có nghĩa là **không** ghi vào `DMEM`.
* Đặt `WBSel` là _don't care_ (*)[^dont-care-vi]
* Đặt `PCSel` là `taken` nếu các tín hiệu `BrEq` và `BrLT` chỉ ra branch nên được taken. Đặt `not taken` nếu không. Xem [bên dưới](#sec-control-pcsel-vi).

[^dont-care-vi]: Xem lại [Tín Hiệu Điều Khiển cho Store](#sec-control-store-vi) để giải thích "don't care."

:::

(sec-datapath-branch-comparator-vi)=
## Khối Bộ So Sánh Branch

Khối Bộ So Sánh Branch trong @fig-element-branch-comparator-vi nhận hai đầu vào dữ liệu và một đầu vào điều khiển, sau đó xuất kết quả so sánh hai đầu vào.

:::{figure} images/element-branch-comparator.png
:label: fig-element-branch-comparator-vi
:width: 30%
:alt: "Khối bộ so sánh branch với đầu vào A và B, tín hiệu chọn BrUn, và đầu ra BrEq và BrLT cho quyết định branch có dấu hoặc không dấu."

Khối Bộ So Sánh Branch
:::

:::{table} Tín hiệu cho Bộ So Sánh Branch. Tên tín hiệu project khóa học, nếu khác, được đặt trong ngoặc đơn.
:label: tab-branch-comparator-signals-vi
:align: center

| Tên | Hướng | Độ Rộng Bit | Mô Tả |
| :-- | :-- | :-- | :-- |
| `A` (`BrData1`) | Đầu vào | 32 | Giá trị đầu tiên để so sánh |
| `B` (`BrData2`) | Đầu vào | 32 | Giá trị thứ hai để so sánh |
| `BrUn` | Đầu vào | 1 | `1` khi muốn so sánh **không dấu**, và `0` khi muốn so sánh **có dấu**. Tín hiệu điều khiển. |
| `BrEq` | Đầu ra | 1 | Đặt là `1` nếu `A == B`, tức là hai giá trị bằng nhau. |
| `BrLT` (`BrLt`) | Đầu ra | 1 | Đặt là `1` nếu `A < B`. Thực hiện so sánh không dấu nếu `BrUn=1`, và có dấu nếu không. |

:::

:::{note} Bộ so sánh branch này tính "taken" như thế nào?

Xem lại [các lệnh B-Type](#tab-rv32i-control): `beq`, `bne`, `blt`, `bltu`, `bge`, `bgeu`. Nhớ rằng tất cả các so sánh branch khác là pseudoinstruction.
:::

Khối branch này được sử dụng để triển khai branch trên datapath với logic trong @fig-branch-branch-comparator-vi:

:::{figure} images/branch-branch-comparator.png
:label: fig-branch-branch-comparator-vi
:width: 60%
:alt: "Sơ đồ datapath với bộ so sánh branch nhận giá trị từ hai thanh ghi, sử dụng cờ BrUn làm đầu vào chọn, và xuất tín hiệu BrEq và BrLT đến khối logic điều khiển. Khối logic điều khiển cũng xác định PCSel cho liệu branch có taken, và nhận inst[31:0]."
:::

Logic điều khiển đặt hai tín hiệu điều khiển:

1. Đặt `BrUn` dựa trên lệnh hiện tại, tức là đặt `BrUn=1` nếu lệnh là `bltu` hoặc `bgeu`.
2. Đặt `PCSel` dựa trên các cờ branch `BrLT` và `BrEq`.

Nói cách khác, subcircuit logic điều khiển đưa đầu vào vào bộ so sánh branch _và_ sử dụng đầu ra của bộ so sánh branch để tính các tín hiệu điều khiển bổ sung.[^control-logic-vi].

(sec-control-pcsel-vi)=
### Đặt `PCSel`

Lưu ý rằng hai tín hiệu đầu ra `BrLT` và `BrEq` là đủ để xác định kết quả của tất cả các so sánh branch:

* `beq`, `bne`: Kiểm tra `BrEq` và đặt `PCSel` tương ứng.
* `blt` (và `bltu`): Nếu `BrLT=1`, thì `PCSel=taken`.
* `bge` (và `bgeu`): Nếu `BrLT=0`, thì `PCSel=taken`. Với các số nguyên $A$ và $B$, kiểm tra $A \geq B$ tương đương với kiểm tra $A \nless B$.

[^control-logic-vi]: Xem [phần sau](#sec-datapath-control) để biết thêm chi tiết.
