---
title: "Hỗ Trợ Jump"
---

(sec-datapath-jump-vi)=
## Mục Tiêu Học Tập

* Triển khai một datapath hỗ trợ các jump vô điều kiện (`jal` và `jalr`).
* So sánh các tín hiệu điều khiển được đặt cho `jal` và `jalr`.
* Giải thích tại sao các jump vô điều kiện không sử dụng bộ so sánh branch.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/op--CudioaA
:width: 100%
:title: "[CS61C FA20] Lecture 19.5 - Single-Cycle CPU Datapath II: Adding JAL"
:::

::::

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/l5ML8v9R8w8
:width: 100%
:title: "[CS61C FA20] Lecture 19.4 - Single-Cycle CPU Datapath II: Adding JALR to Datapath"
:::

::::

:::{note} Ôn lại các lệnh jump

RISC-V có hai lệnh jump vô điều kiện.
Để có tổng quan chung về jump vô điều kiện, xem lại [phần này](#sec-jumps).

* `jal` (Jump and link): Lệnh J-Type duy nhất; xem lại [chi tiết J-Type](#sec-j-type).
* `jalr` (Jump and link register): Một lệnh I-Type; xem lại [chi tiết định dạng lệnh `jalr`](#sec-jalr-itype).

:::

## Cập Nhật Datapath cho `jal`

Để hỗ trợ `jal`:

* RegFile: Chúng ta không đọc thanh ghi nào nhưng ghi một thanh ghi `rd`. Giá trị để ghi là `pc + 4`, tức là "link" đến lệnh tiếp theo.
* PC: Chúng ta **đọc** từ và **ghi** vào PC. Giá trị để ghi là `pc + imm`.

Chúng ta có thể tái sử dụng nhiều thành phần của datapath R-, I-, S-, và B-Type. Chúng ta sẽ cần **cập nhật** hai khối, được hiển thị trong @fig-jal-new-blocks-vi.

::::{figure} images/jal-new-blocks.png
:label: fig-jal-new-blocks-vi
:width: 100%
:alt: "Mô tả cập nhật datapath jump: bộ tạo immediate mở rộng cho J-type và mux writeback mở rộng để bao gồm giá trị link PC cộng bốn."

Cập nhật khối Bộ Tạo Immediate và mux `WBSel`.
::::

**Bộ Tạo Immediate**: Nâng cấp [Bộ Tạo Immediate](#sec-datapath-immgen-vi) để hỗ trợ immediate trong các lệnh J-Type.

**Mux**: Mux `WBSel` bây giờ phải chọn giữa **ba giá trị** cho `wdata` (dữ liệu để ghi vào `R[rd]`):

* Các lệnh R-Type hoặc I-Type số học và logic: Đầu ra của ALU (`alu`), bây giờ được nối cả vào `addr` và vào mux mới.
* Các lệnh load: Đầu ra của DMEM (`mem`).
* Các lệnh jump: `pc + 4`.

## Theo Dõi Datapath `jal`

::::{figure}
:label: anim-datapath-jal-vi
:alt: "Các slide nhúng theo dõi jump và link tương đối PC trên datapath chu kỳ đơn cho jal, bao gồm bắt địa chỉ link và mux đích branch."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vRG-3tt1Kr0wyvIhKtNd4_QUT38Kn7gL8ulf_jXpKi8sUlQJ4Vk9cRW0woMIXuXbQ/pubembed?start=false&loop=false
:width: 100%
:title: "Slides theo dõi Datapath `jal`."
:::
Datapath `jal`. Sử dụng thanh menu để theo dõi qua animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/1NTWJNNbw4Pk-2SlUlo8JLNw8fOSHKXvQ/edit?usp=sharing).
::::

1. **Fetch Lệnh**: Ở đầu chu kỳ clock, đọc PC và fetch lệnh hiện tại từ IMEM. Đưa `pc` và `pc + 4` đến các khối.

    Trước cạnh lên clock tiếp theo, đặt đầu ra của ALU vào đầu vào của PC.

1. **Decode Lệnh**: Xây dựng immediate `imm` cho các lệnh J-Type. Cũng cấu hình logic điều khiển (xem bên dưới).

1. **Thực Thi**: Tính `pc + imm` sử dụng ALU. Vì các tín hiệu điều khiển là `ASel=1` và `BSel=1`, hai mux trước ALU sẽ chọn `pc` và `imm`, tương ứng. Vì `ALUSel=Add`, hai giá trị này sẽ được cộng với nhau sử dụng ALU để tạo ra `pc + imm`.

1. **Bộ Nhớ**: (Chúng ta không truy cập DMEM, nên bỏ qua.)

1. **Ghi Lại**: Ghi đầu ra `pc + 4` vào thanh ghi đích bằng cách kết nối đầu ra của mux `WBSel` đến đầu vào `wdata` của RegFile.

    Xung quanh cạnh lên clock tiếp theo, `wdata`, `RegWEn`, và `rd` nên được giữ ổn định qua thời gian setup và hold của RegFile.


:::{note} Tín Hiệu Điều Khiển cho `jal`

* Cấu hình `ImmSel` thành immediate loại `J`.
* Đặt `RegWEn` là `1`, tức là ghi lại vào RegFile.
* Đặt `BrUn` là _don't care_ (*)[^dont-care-vi]. Lệnh `jal` không sử dụng bộ so sánh branch.
* Đặt `ASel` là `1`.
* Đặt `BSel` là `1`.
* Đặt `ALUSel` là `Add`.
* Đặt `MemRW` là `Read`. Điều này có nghĩa là **không** ghi vào DMEM.
* Đặt `WBSel` là `2` (để chọn `pc + 4` trong @anim-datapath-jal-vi).
* Đặt `PCSel` là `taken`.

[^dont-care-vi]: Xem lại [Tín Hiệu Điều Khiển cho Store](#sec-control-store-vi) để giải thích "don't care."

:::

## Cập Nhật Datapath cho `jalr`

Để hỗ trợ `jalr`:

* RegFile: Chúng ta **đọc** một thanh ghi `rs1` và ghi một thanh ghi `rd`. Giá trị để ghi là `pc + 4`, tức là "link" đến lệnh tiếp theo.
* PC: Chúng ta **đọc** từ và **ghi** vào `PC`. Giá trị để ghi là `R[rs1] + imm`.

Chúng ta **không cần cập nhật nào** cho datapath để hỗ trợ `jalr`! Vì `jalr` là I-Type, khối bộ tạo immediate đã hỗ trợ nó. Thay vào đó, cho `jalr`, chúng ta cần đặt các tín hiệu điều khiển tương ứng.

## Theo Dõi Datapath `jalr`

::::{figure}
:label: anim-datapath-jalr-vi
:alt: "Các slide nhúng theo dõi jump và link thanh ghi cộng immediate trên datapath chu kỳ đơn cho jalr."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vTeMjM-GcVH1ueYM4pQwmgUPEBfKJ4xf3PgkGmcv3eYiSOGlLuUZ4ty02YeXq1gqw/pubembed?start=false&loop=false
:width: 100%
:title: "Slides theo dõi Datapath `jalr`."
:::
Datapath `jalr`. Sử dụng thanh menu để theo dõi qua animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/1E4qsF1FVE6fhnl1wTnp824rWql7BIc6r/edit?usp=sharing).
::::

1. **Fetch Lệnh**: Ở đầu chu kỳ clock, đọc PC và fetch lệnh hiện tại từ `IMEM`. Đưa `pc + 4` đến các khối.

    Trước cạnh lên clock tiếp theo, đặt đầu ra của ALU vào đầu vào của PC.

1. **Decode Lệnh**: Fetch `R[rs1]` từ RegFile và xây dựng immediate `imm` cho các lệnh I-Type. Cũng cấu hình logic điều khiển (xem bên dưới).

1. **Thực Thi**: Tính `R[rs1] + imm` sử dụng ALU. Vì các tín hiệu điều khiển là `ASel=0` và `BSel=1`, hai mux trước ALU sẽ chọn `R[rs1]` và `imm`, tương ứng. Vì `ALUSel=Add`, ALU sẽ cộng hai giá trị này với nhau.

1. **Bộ Nhớ**: (Chúng ta không truy cập DMEM, nên bỏ qua.)

1. **Ghi Lại**: Ghi đầu ra `pc + 4` vào thanh ghi đích và kết nối đầu ra của mux `WBSel` đến đầu vào `wdata` của RegFile.

    Xung quanh cạnh lên clock tiếp theo, `wdata`, `RegWEn`, và `rd` nên được giữ ổn định qua thời gian setup và hold của RegFile.


:::{note} Tín Hiệu Điều Khiển cho `jalr`

* Cấu hình `ImmSel` thành immediate loại `I`.
* Đặt `RegWEn` là `1`, tức là ghi lại vào RegFile.
* Đặt `BrUn` là _don't care_ (*)[^dont-care-vi]. Lệnh `jalr` không sử dụng bộ so sánh branch.
* Đặt `ASel` là `0`.
* Đặt `BSel` là `1`.
* Đặt `ALUSel` là `Add`.
* Đặt `MemRW` là `Read`. Điều này có nghĩa là **không** ghi vào DMEM.
* Đặt `WBSel` là `2` (để chọn `pc + 4` trong @anim-datapath-jalr-vi).
* Đặt `PCSel` là `taken`.

:::
