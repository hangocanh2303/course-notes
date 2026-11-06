---
title: "Các Phần Tử Trạng Thái trên Datapath"
---

(sec-state-elements-vi)=
## Mục Tiêu Học Tập

* Mô tả các phần tử trạng thái chính trên datapath RISC-V chu kỳ đơn.
* Xác định khi nào dữ liệu được ghi trên các phần tử trạng thái đồng bộ trên datapath RISC-V.
* So sánh và đối chiếu hành vi đọc và ghi của các phần tử trạng thái RISC-V.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/8Dv725H2-Os
:width: 100%
:title: "[CS61C FA20] Lecture 18.2 - Single-Cycle CPU Datapath I: Building a RISC-V Processor"
:::

::::

Như đã đề cập trong phần trước, CPU có hai loại phần tử, phản ánh thiết kế của nhiều hệ thống logic số.

* [Phần tử trạng thái](#sec-state-elements-vi) chứa trạng thái: thanh ghi và bộ nhớ
* Các Khối Logic Tổ hợp hoạt động trên các giá trị dữ liệu: [ALU](#sec-alu), logic tổ hợp khác, v.v.

Trong phần này, chúng ta thảo luận về **các phần tử trạng thái** cần thiết trong bộ xử lý RISC-V. Chúng ta sẽ thảo luận và giới thiệu các khối logic tổ hợp khi chúng ta xây dựng datapath đầy đủ.

(sec-element-pc-vi)=
### Bộ Đếm Chương Trình (Program Counter)

Bộ Đếm Chương Trình (PC) là một thanh ghi 32-bit trong @fig-element-pc-vi và giữ giá trị của **lệnh hiện tại**, tức là lệnh cần thực thi trong chu kỳ clock hiện tại.

:::{figure} images/element-pc.png
:label: fig-element-pc-vi
:width: 50%
:alt: "Khối bộ đếm chương trình: một thanh ghi N-bit với data-in, write-enable, đầu vào clock, và đầu ra dữ liệu N-bit."

Bộ Đếm Chương Trình, PC, là một thanh ghi 32-bit đơn trong CPU.
:::

:::{note} Tín Hiệu PC
:class: dropdown

**Đầu vào**:

* _Dữ liệu_: Bus đầu vào dữ liệu N-bit
* _Điều khiển_: Bit Write Enable. 1: asserted/high, 0: deasserted/low.
* Tín hiệu Clock.

**Đầu ra**:

* _Dữ liệu_: Bus đầu ra dữ liệu N-bit

:::

**Hành vi**:

* _Đọc_: Vào tất cả các thời điểm khác, Data Out sẽ không thay đổi; nó sẽ xuất giá trị hiện tại của nó.
* _Ghi_: **Kích hoạt bởi cạnh lên**. Ở cạnh lên clock, nếu Write Enable là 1, đặt Data Out thành Data In (độ trễ của clk-to-q).

(sec-element-regfile-vi)=
### Register File (Regfile)

**Register File** (hay RegFile) có 32 thanh ghi: số thanh ghi `x0` đến `x31`.

:::{figure} images/element-regfile.png
:label: fig-element-regfile-vi
:width: 50%
:alt: "Khối register file nhận đầu vào wdata 32-bit, rd 5-bit, giá trị thanh ghi rs1 5-bit, và giá trị thanh ghi rs2 5-bit. Khối register xuất hai giá trị đầu ra rdata1 và rdata2 32-bit và được điều khiển bởi tín hiệu RegWEn và clk."

RegFile được viết ký hiệu là RegFile và bao gồm các thanh ghi `x0` đến `x31`.
:::

::::{table} Các tín hiệu RegFile. Tên tín hiệu của project khóa học, nếu khác, được đặt trong ngoặc đơn.
:label: tab-regfile-signals-vi
:align: center

| Tên | Hướng | Độ Rộng Bit | Mô Tả |
| :-- | :-- | :-- | :-- |
| `rs1` (`ReadIndex1`) | Đầu vào | 5 | Xác định giá trị thanh ghi nào được gửi đến đầu ra `rdata1` (`ReadData1`) |
| `rs2` (`ReadIndex2`) | Đầu vào | 5 | Xác định giá trị thanh ghi nào được gửi đến đầu ra `rdata2` (`ReadData2`) |
| `rd` (`WriteIndex`) | Đầu vào | 5 | Thanh ghi để ghi vào ở cạnh lên tiếp theo của clock (nếu `RegWEn` là 1) |
| `wdata` (`WriteData`) | Đầu vào | 32 | Dữ liệu để ghi vào `rd` ở cạnh lên tiếp theo của clock (nếu `RegWEn` là 1) |
| `RegWEn` | Đầu vào | 1 | Xác định liệu dữ liệu có được ghi vào register file ở cạnh lên tiếp theo của clock |
| `clk` | Đầu vào | 1 | Đầu vào Clock |
| `rdata1` (`ReadData1`) | Đầu ra | 32 | Giá trị của thanh ghi được xác định bởi `rs1` (`ReadIndex1`) |
| `rdata2` (`ReadData2`) | Đầu ra | 32 | Giá trị của thanh ghi được xác định bởi `rs2`(`ReadIndex2`) |
::::

**Hành vi**:

* Các thanh ghi được truy cập qua số thanh ghi 5-bit của chúng:
  * `R[rs1]`: `rs1` chọn thanh ghi để đặt lên bus `rdata1` ra.
  * `R[rs2]`: `rs2` chọn thanh ghi để đặt lên bus `rdata2` ra.
  * `R[rd]`: `rd` chọn thanh ghi để được ghi qua `wdata` khi `RegWEn` được đặt là 1.
* _Đọc_: Miễn là `rs1` và `rs2` hợp lệ, thì `rdata1` và `rdata2` hợp lệ sau thời gian truy cập, _bất kể `RegWEn` được đặt như thế nào_.
* _Ghi_: **Ghi kích hoạt bởi cạnh lên**. Ở cạnh lên clock, nếu `RegWEn` được đặt là 1, ghi `wdata` vào `R[rd]`.

:::{note} Tín hiệu clock chỉ là yếu tố khi ghi!

Đối với các hoạt động _đọc_, RegFile hoạt động như **một khối logic tổ hợp**. Nói cách khác, hoạt động đọc _không_ được định thời theo cạnh lên clock và liên tục xảy ra, cập nhật đầu ra dữ liệu dựa trên đầu vào (sau một độ trễ thời gian truy cập). Đây là giả định cần thiết để đảm bảo datapath chu kỳ đơn của chúng ta thực thi tất cả các giai đoạn của một lệnh trong một chu kỳ.
:::

(sec-element-dmem-vi)=
### DMEM: Bộ Nhớ Dữ Liệu (Data Memory)

Đối với lớp này, bộ nhớ là "phép thuật." Giả sử không gian bộ nhớ địa chỉ theo byte 32-bit, và truy cập bộ nhớ xảy ra với các word 32-bit. Chúng ta đi vào chi tiết hơn với các project khóa học.

Đối với datapath chu kỳ đơn của chúng ta, chúng ta phải truy cập bộ nhớ **hai lần**: một lần trong `IF` (Fetch Lệnh) để đọc lệnh từ bộ nhớ, và một lần trong `MEM` (Truy Cập Bộ Nhớ) nếu chúng ta load/store dữ liệu từ/vào bộ nhớ. Do đó chúng ta cần hai khối bộ nhớ: IMEM và DMEM cho bộ nhớ lệnh và bộ nhớ dữ liệu, tương ứng.[^imem-dmem-cache-vi]

[^imem-dmem-cache-vi]: Bên trong, IMEM và DMEM là placeholders cho các cache L1: `L1i`, `L1d`. Xem [phần sau](#sec-multi-level-caches).

Khối Bộ Nhớ Dữ Liệu DMEM có ghi kích hoạt bởi cạnh, giống như RegFile.

:::{figure} images/element-dmem.png
:label: fig-dmem-block-vi
:width: 50%
:alt: "Khối bộ nhớ dữ liệu với đầu vào địa chỉ 32-bit và dữ liệu ghi 32-bit, tín hiệu điều khiển MemRW và clk, và đầu ra rdata 32-bit."

Khối Bộ Nhớ Dữ Liệu DMEM. Các hoạt động đọc hoạt động như logic tổ hợp, trong khi các hoạt động ghi xảy ra ở cạnh lên clock.
:::


::::{table} Tín hiệu DMEM. Xem phần [Load và Store Một Phần](#sec-datapath-partial-load-store) để biết tên tín hiệu project khóa học.
:label: tab-dmem-signals-vi
:align: center

| Tên | Hướng | Độ Rộng Bit | Mô Tả |
| :-- | :-- | :-- | :-- |
| `addr`  | Đầu vào | 32 | Địa chỉ trong bộ nhớ để đọc từ hoặc ghi vào |
| `wdata` | Đầu vào | 32 | Dữ liệu để ghi vào bộ nhớ |
| `MemRW` | Đầu vào | 1 | Xác định liệu dữ liệu có được ghi vào bộ nhớ ở cạnh lên tiếp theo của clock |
| `clk` | Đầu vào | 1 | Đầu vào Clock |
| `rdata` (`MemReadData`) | Đầu ra | 32 | Dữ liệu tại `addr` (`MemAddress`) từ bộ nhớ |

::::

**Hành vi**: Đọc/ghi DMEM hoạt động tương tự như Regfile, mặc dù bây giờ chúng ta cung cấp địa chỉ bộ nhớ làm đầu vào, không phải số thanh ghi.

* Đọc: Địa chỉ `addr` chọn word để đặt lên bus `rdata`. Nếu `MemRW` là 0 và `addr` hợp lệ, thì `rdata` hợp lệ sau thời gian truy cập.
* Ghi: **Ghi kích hoạt bởi cạnh lên**. Ở cạnh lên clock, nếu `MemRW` được đặt là 1, ghi `wdata` vào địa chỉ `addr`.

:::{warning} Load và Store Một Phần

Chúng ta triển khai một khối DMEM phức tạp hơn trong project của chúng ta; xem [phần Load và Store Một Phần](#sec-datapath-partial-load-store).
:::

(sec-element-imem-vi)=
### IMEM: Bộ Nhớ Lệnh (Instruction Memory)

Khối Bộ Nhớ Lệnh IMEM là **bộ nhớ chỉ đọc** fetch các lệnh.[^imem-foot-vi]

[^imem-foot-vi]: Chúng ta sẽ cần ghi bộ nhớ lệnh khi chúng ta load chương trình, điều mà chúng ta bỏ qua để đơn giản.

:::{figure} images/element-imem.png
:label: fig-element-imem-vi
:width: 50%
:alt: "Khối bộ nhớ lệnh với đầu vào địa chỉ 32-bit và đầu ra lệnh 32-bit, được mô hình hóa như logic fetch tổ hợp chỉ đọc. Bên phải, tín hiệu khối IMEM."

Trong CPU của chúng ta, Khối Bộ Nhớ Lệnh IMEM là chỉ đọc và hoạt động như logic tổ hợp.
:::

::::{table} Tín hiệu IMEM. Tên tín hiệu project khóa học, nếu khác, được đặt trong ngoặc đơn.
:label: tab-imem-signals-vi
:align: center

| Tên | Hướng | Độ Rộng Bit | Mô Tả |
| :-- | :-- | :-- | :-- |
| `addr` (N/A) | Đầu vào | 32 | Địa chỉ trong bộ nhớ để đọc từ |
| `inst` (`Instruction`) | Đầu ra | 32 | Lệnh tại địa chỉ bộ nhớ `addr`(`ProgramCounter`) |

::::

**Hành vi**:

* Đọc: Địa chỉ `addr` chọn word để đặt lên bus `inst`. Nếu `addr` hợp lệ, thì `inst` hợp lệ sau thời gian truy cập.

:::{warning} IMEM không có clock!

Trong CPU của chúng ta, chúng ta giả định IMEM là phần tử trạng thái chỉ đọc và **không có clock**. Do đó nó _luôn luôn_ hoạt động như một khối logic tổ hợp.

:::
