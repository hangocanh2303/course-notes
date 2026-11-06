---
title: "Hỗ Trợ U-Type"
---

(sec-datapath-u-type-vi)=
## Mục Tiêu Học Tập

* Triển khai một datapath hỗ trợ các lệnh U-Type (`lui` và `auipc`).
* Giải thích tại sao ALU (trong datapath khóa học của chúng ta) cần hỗ trợ nối đầu vào `B` trực tiếp đến đầu ra của nó.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/8q8l0tXs_Lc
:width: 100%
:title: "[CS61C FA20] Lecture 19.6 - Single-Cycle CPU Datapath II: Adding U-Types"
:::

::::

Đến thời điểm này, chúng ta có một datapath có thể hỗ trợ các lệnh R-, I-, S-, B-, và J-Type. Chỉ còn một định dạng lệnh: lệnh **U-Type** (**upper immediate**). Xem lại [các lệnh U-Type](#sec-u-type) trước khi tiếp tục.

## Cập Nhật Cần Thiết cho U-Type

Để hỗ trợ U-Type:

* RegFile: Chúng ta không đọc thanh ghi nào nhưng ghi một thanh ghi `rd`.
* PC: Chúng ta **đọc** từ và **ghi** vào PC. Giá trị để ghi là `pc + 4`.

Có **hai** cập nhật chúng ta cần thực hiện:

* [**Bộ tạo immediate**](#sec-datapath-immgen-vi) bây giờ phải hỗ trợ immediate trong các lệnh U-Type.
* [ALU](#sec-datapath-alu-vi) phải hỗ trợ truyền immediate qua (tức là chỉ chọn đầu vào ALU `B` và bỏ qua đầu vào `A`). Phép toán này cần thiết cho lệnh `lui`.

## Theo Dõi Datapath `lui`

::::{figure}
:label: anim-datapath-lui-vi
:alt: "Các slide nhúng theo dõi load-upper-immediate trên datapath chu kỳ đơn, nối immediate U-type vào đường bypass ALU và ghi lại thanh ghi."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vQaJ2vmNAHWYtsHOqepOlL_mOnOM3HPvfvhTAO_CLlNLoEo17GiOuslYVycWHUb_w/pubembed?start=false&loop=false
:width: 100%
:title: "Slides theo dõi Datapath `lui`."
:::
Datapath `lui`. Sử dụng thanh menu để theo dõi qua animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/11CKXsYN9z-0RRWkwVz-Ae6ZvZIcirluV/edit?usp=sharing).
::::

1. **Fetch Lệnh**: Ở đầu chu kỳ clock, đọc PC và fetch lệnh hiện tại từ IMEM.

    Trước cạnh lên clock tiếp theo, đặt `pc + 4` vào đầu vào của PC.

1. **Decode Lệnh**: Xây dựng immediate `imm` cho các lệnh U-Type. Cũng cấu hình logic điều khiển (xem bên dưới).

1. **Thực Thi**: Lấy giá trị `imm` và đặt làm đầu ra ALU.

1. **Bộ Nhớ**: (Chúng ta không truy cập DMEM, nên bỏ qua.)

1. **Ghi Lại**: Ghi đầu ra ALU vào thanh ghi đích và kết nối đầu ra của mux `WBSel` đến đầu vào `wdata` của RegFile.

    Xung quanh cạnh lên clock tiếp theo, `wdata`, `RegWEn`, và `rd` nên được giữ ổn định qua thời gian setup và hold của RegFile.

:::{note} Tín Hiệu Điều Khiển cho `lui`

* Cấu hình `ImmSel` thành immediate loại `U`.
* Đặt `RegWEn` là `1`, tức là ghi lại vào RegFile
* Đặt `BrUn` là _don't care_ (*)[^dont-care-vi].
* Đặt `ASel` là _don't care_ (*)[^dont-care-vi].
* Đặt `BSel` là `1`.
* Đặt `ALUSel` để kết nối đầu vào `B` trực tiếp đến đầu ra (tên project: `bsel`).
* Đặt `MemRW` là `Read`. Điều này có nghĩa là **không** ghi vào `DMEM`.
* Đặt `WBSel` là `1` (để chọn đầu ra `alu` trong @anim-datapath-lui-vi).
* Đặt `PCSel` là `not taken`.

[^dont-care-vi]: Xem lại [Tín Hiệu Điều Khiển cho Store](#sec-control-store-vi) để giải thích "don't care."

:::

## Theo Dõi Datapath `auipc`

::::{figure}
:label: anim-datapath-auipc-vi
:alt: "Các slide nhúng theo dõi add-upper-immediate-to-PC trên datapath chu kỳ đơn, kết hợp PC với upper immediate để thiết lập địa chỉ tương đối PC."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vQTNKmHKQSo2p_zRGfFjMYdnQ5QoIcEeFI57fdjlvmkOQJKbXAZ42_Y6x5iT8TNGw/pubembed?start=false&loop=false
:width: 100%
:title: "Slides theo dõi Datapath `auipc`."
:::
Datapath `auipc`. Sử dụng thanh menu để theo dõi qua animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/117GS6W-vKD25dAOlAd9ycbFZpMBSAgp1/edit?usp=sharing).
::::

1. **Fetch Lệnh**: Ở đầu chu kỳ clock, đọc PC và fetch lệnh hiện tại từ IMEM. Đưa `pc` đến giai đoạn `EX`.

    Trước cạnh lên clock tiếp theo, đặt `pc + 4` vào đầu vào của PC.

1. **Decode Lệnh**: Xây dựng immediate `imm` cho các lệnh U-Type. Cũng cấu hình logic điều khiển (xem bên dưới).

1. **Thực Thi**: Tính `pc + imm` sử dụng ALU. Vì các tín hiệu điều khiển là `ASel=1` và `BSel=1`, hai mux trước ALU sẽ chọn `pc` và `imm`, tương ứng. Vì `ALUSel=Add`, ALU sẽ cộng hai giá trị này với nhau.

1. **Bộ Nhớ**: (Chúng ta không truy cập DMEM, nên bỏ qua.)

1. **Ghi Lại**: Ghi đầu ra ALU vào thanh ghi đích và kết nối đầu ra của mux `WBSel` đến đầu vào `wdata` của RegFile.

    Xung quanh cạnh lên clock tiếp theo, `wdata`, `RegWEn`, và `rd` nên được giữ ổn định qua thời gian setup và hold của RegFile.

:::{note} Tín Hiệu Điều Khiển cho `auipc`

* Cấu hình `ImmSel` thành immediate loại `U`.
* Đặt `RegWEn` là `1`, tức là ghi lại vào RegFile
* Đặt `BrUn` là _don't care_ (*)[^dont-care-vi].
* Đặt `ASel` là `1`
* Đặt `BSel` là `1`.
* Đặt `ALUSel` là `Add`.
* Đặt `MemRW` là `Read`. Điều này có nghĩa là **không** ghi vào `DMEM`.
* Đặt `WBSel` là `1` (để chọn đầu ra `alu` trong @anim-datapath-auipc-vi).
* Đặt `PCSel` là `not taken`.

:::
