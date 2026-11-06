---
title: "Hỗ Trợ Load và Store"
---

(sec-datapath-load-store-vi)=
## Mục Tiêu Học Tập

* Triển khai một datapath hỗ trợ load (I-Type) và store (S-Type).
* Giải thích tại sao một số tín hiệu điều khiển có thể được đánh dấu là "don't care" (*).
* Triển khai partial load và store cho project khóa học, và hiểu cơ bản về các yêu cầu tối thiểu cho khối DMEM.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/TnDpXz75SGA
:width: 100%
:title: "[CS61C FA20] Lecture 19.1 - Single-Cycle CPU Datapath II: Supporting Loads"
:::

::::

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/Sra2m_9mgCU
:width: 100%
:title: "[CS61C FA20] Lecture 19.2 - Single-Cycle CPU Datapath II: Datapath for Stores"
:::

::::

## Xây dựng Bộ Xử Lý với Truy Cập DMEM

Load và store tham gia vào giai đoạn `MEM` của [quy trình năm bước](#sec-five-steps-vi). Do đó chúng ta giới thiệu logic bổ sung kết nối DMEM với ALU và RegFile. @fig-lw-new-blocks-vi mô tả truy cập DMEM cho các lệnh load.

::::{figure} images/lw-new-blocks.png
:label: fig-lw-new-blocks-vi
:width: 100%
:alt: "Bổ sung datapath load: DMEM kết nối với địa chỉ tính từ ALU và mux writeback chọn giữa kết quả ALU và dữ liệu bộ nhớ."

Cho giai đoạn `MEM` của lệnh load, kết nối DMEM với ALU và sử dụng mux trước giai đoạn `WB` (Ghi Lại).
::::

**DMEM**: Để đọc bộ nhớ tại một địa chỉ, chúng ta sử dụng ALU để tính địa chỉ là `alu = R[rs1] + imm`. Điều này dễ dàng tái sử dụng mạch cho các lệnh số học và logic I-Type.

**Mux**: Bây giờ chúng ta bao gồm một mux sau ALU và DMEM sử dụng tín hiệu điều khiển `WBSel` để chọn giữa hai giá trị cho `wdata` (dữ liệu để ghi vào `R[rd]`):

* Các lệnh R-Type hoặc I-Type số học và logic: Đầu ra của ALU (`alu`), bây giờ được nối cả vào `addr` và vào mux mới.
* Các lệnh load: Đầu ra của DMEM (`mem`).

(sec-datapath-load-vi)=
## Theo Dõi Datapath Load

Nhớ rằng các lệnh [load](#sec-load-word) là [I-Type](#sec-rv-load) vì chúng đọc một thanh ghi, có immediate, và ghi vào thanh ghi một giá trị 32-bit đọc từ bộ nhớ.

Cho `lw`, chúng ta sử dụng datapath tương tự như `addi`, nhưng chúng ta sử dụng ALU để tính địa chỉ truyền vào DMEM. Cập nhật trạng thái:

* RegFile: Chúng ta **đọc** _một_ thanh ghi `rs1` và ghi một thanh ghi `rd`. Giá trị để ghi là một **word** (nhắc lại, `lw`) được đọc từ bộ nhớ.
* PC: Chúng ta **đọc** từ và **ghi** vào PC. Giá trị để ghi là `pc + 4`.
* DMEM: Chúng ta **đọc** word bộ nhớ tại địa chỉ `R[rs1] + imm`.

::::{figure}
:label: anim-datapath-lw-vi
:alt: "Các slide nhúng theo dõi tính địa chỉ, đọc bộ nhớ, và ghi lại thanh ghi trên datapath chu kỳ đơn cho lệnh load-word."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vRxeC98vPTgADlY5t3P_seJfptV4jyHqG7xxR6X7tcr52EXVEiCtcOQBg_0cgacHA/pubembed?start=false&loop=false
:width: 100%
:title: "Slides theo dõi Datapath `lw`."
:::
Datapath `lw`. Sử dụng thanh menu để theo dõi qua animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/1UahZFjwYvnpvTWXY9JrPKPvzLzPE00Jg/edit?usp=sharing). 
::::

1. **Fetch Lệnh**: Tăng PC đến lệnh tiếp theo (xem [datapath R-Type](#sec-datapath-add-vi)). Đọc lệnh `inst` từ IMEM.

1. **Decode Lệnh**: Fetch `R[rs1]` từ RegFile, xây dựng immediate `imm` cho [các lệnh I-Type](#tab-i-type) (xem [datapath I-Type](#sec-datapath-i-type-vi)). Cũng cấu hình logic điều khiển:
    * Cấu hình `ImmSel` thành immediate loại `I`.
    * Đặt `RegWEn` là `1`.
    * Đặt `BSel` là `1`.
    * Đặt `ALUSel` là `Add`.
    * Đặt `MemRW` là `Read`.
    * Đặt `WBSel` là `0`.

    Sau một độ trễ, khối bộ tạo immediate cập nhật tín hiệu đầu ra `imm` thành giá trị immediate 32-bit mở rộng dấu phù hợp, giá trị thanh ghi `R[rs1]` được đọc, và các tín hiệu điều khiển được đặt.

1. **Thực Thi**: Vì đường điều khiển `BSel=1` chọn immediate đã tạo `imm` cho đầu vào ALU `B`, ALU của chúng ta tính `R[rs1] + imm`.

1. **Bộ Nhớ**: Đọc bộ nhớ tại địa chỉ `alu = R[rs1] + imm`. Sau một độ trễ, tín hiệu đầu ra `mem` có giá trị `Mem[R[rs1] + imm]`.

1. **Ghi Lại**: Ghi đầu ra DMEM vào thanh ghi đích và kết nối đầu ra của mux `WBSel` đến đầu vào `wdata` của RegFile.

    Xung quanh cạnh lên clock tiếp theo, `wdata`, `RegWEn`, và `rd` nên được giữ ổn định qua thời gian setup và hold của RegFile.

(sec-datapath-store-vi)=
## Theo Dõi Datapath Store

Ngược lại, các lệnh [store](#sec-store-word), là [S-Type](#sec-s-type) vì chúng đọc hai thanh ghi, có immediate, và ghi vào bộ nhớ. Store không ghi dữ liệu vào thanh ghi.

Chúng ta **không cần** thêm các khối bổ sung cho store, nhưng chúng ta sẽ cần:

* Nâng cấp Bộ Tạo Immediate để hỗ trợ immediate trong các lệnh S-Type; chúng tôi khuyến khích bạn đọc [phần đó](#sec-datapath-immgen-vi) sau.
* Nối `R[rs2]` đến `wdata` (tín hiệu đầu vào DMEM).

::::{figure}
:label: anim-datapath-sw-vi
:alt: "Các slide nhúng theo dõi tính địa chỉ và ghi bộ nhớ trên datapath chu kỳ đơn cho lệnh store-word."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vRiHxdtVBLJ-3cMkMSg7t09c_DM0b4oXInmUggpSThdg0P3Cp4O7gh4ojtZhFfd8g/pubembed?start=false&loop=false
:width: 100%
:title: "Slides theo dõi Datapath `sw`."
:::
Datapath `sw`. Sử dụng thanh menu để theo dõi qua animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/1QO1RMuVSP-5Y_18RVc8Oy_QMXSK0JYqN/edit?usp=sharing).
::::

1. **Fetch Lệnh**: Tăng PC đến lệnh tiếp theo (xem [datapath R-Type](#sec-datapath-add-vi)).

1. **Decode Lệnh**: Fetch `R[rs1]` và `R[rs2]` từ RegFile, và xây dựng immediate `imm` cho các lệnh S-Type. Cũng cấu hình logic điều khiển (xem bên dưới).

1. **Thực Thi**: Vì đường điều khiển `BSel=1` chọn immediate đã tạo `imm` cho đầu vào ALU `B`, ALU của chúng ta tính `R[rs1] + imm`.

1. **Bộ Nhớ**: Ghi bộ nhớ tại địa chỉ `alu = R[rs1] + imm` bằng cách giữ `addr` và `wdata` (đầu vào DMEM).

    Xung quanh cạnh lên clock tiếp theo, `wdata` (đầu vào DMEM), `MemRW`, và `addr` nên được giữ ổn định qua thời gian setup và hold của DMEM.

1. **Ghi Lại**: (Chúng ta không ghi vào RegFile, nên bỏ qua.)

(sec-control-store-vi)=
:::{note} Tín Hiệu Điều Khiển cho Store

* Cấu hình `ImmSel` thành immediate loại `S`.
* Đặt `RegWEn` là `0`. Điều này có nghĩa là **không** ghi lại vào RegFile.
* Đặt `BSel` là `1`.
* Đặt `ALUSel` là `Add`.
* Đặt `MemRW` là `Write`. Điều này có nghĩa là ghi vào `DMEM` ở cạnh lên clock tiếp theo.
* Đặt `WBSel` là _tùy ý_.

Chúng ta **không quan tâm** đến tín hiệu `WBSel` vì `RegWEn` được đặt là `0`. Chúng ta viết "don't care" là `*`, ngụ ý rằng `WBSel` có thể là `0` hoặc `1` cho các lệnh store.

:::

(sec-datapath-partial-load-store-vi)=
## Partial Load và Store: Chi Tiết Project Khóa Học

Phần này thảo luận cách triển khai partial load và store. Các chi tiết trong phần này dành riêng cho project khóa học. Chúng ta bắt đầu bằng cách giới thiệu các chức năng bổ sung của khối DMEM trong project khóa học.

:::{note} Ôn lại Partial Load và Store

Chúng tôi khuyên bạn ôn lại [phần trước](#sec-partial-load-store) về partial load và store trước khi tiếp tục.
:::

Nhớ lại từ [phần trước](#sec-rv-aligned) rằng theo tiêu chuẩn RISC-V, load và store của word nên chỉ định địa chỉ căn chỉnh word. Truy cập bộ nhớ DMEM trong khóa này (và trong project khóa học) do đó tuân theo các quy ước sau:

* Tất cả truy cập bộ nhớ DMEM là truy cập **word**. Địa chỉ phải rộng 32-bit và là **bội của 4**.
* Load đọc một word (4 byte) mỗi lần.
* Store ghi tối đa 4 byte mỗi lần, được chỉ định bởi [bitmask](#sec-bitmasks).

Xem xét @fig-aligned-memory-vi, hiển thị mỗi byte trong bộ nhớ, được tham chiếu bằng địa chỉ 32-bit của nó.

* Trong một lần truy cập, DMEM có thể truy cập một tập 4 byte trong một trong các hộp đỏ trong sơ đồ, ví dụ: byte tại địa chỉ bộ nhớ `0-1-2-3` hoặc `12-13-14-15`.
* Trong một lần truy cập, DMEM không thể truy cập một tập 4 byte qua hai hộp đỏ trong sơ đồ, ví dụ: byte tại địa chỉ bộ nhớ `13-14-15-16`. Điều này sẽ cần hai lần truy cập.

:::{figure} images/aligned-memory.svg
:label: fig-aligned-memory-vi
:width: 30%
:alt: "Bộ nhớ địa chỉ theo byte được vẽ theo hàng với các hộp đỏ xung quanh mỗi ranh giới word 4-byte căn chỉnh để minh họa các truy cập word hợp lệ."
Sơ đồ bộ nhớ căn chỉnh (Hộp đỏ xung quanh mỗi 4 byte).
:::

:::{note} Đừng nghĩ về endianness
Endianness liên quan khi bạn đang cố gắng diễn giải một tập 4 byte như một word (ví dụ: số nguyên, địa chỉ) để thực hiện tính toán trên nó. Ở mức trừu tượng này, chúng ta chỉ cần lo lắng về bit nào được load và store, không phải chúng có nghĩa gì.
:::

(sec-datapath-dmem-details-vi)=
### DMEM Căn Chỉnh Word

* **DMEM buộc truy cập bộ nhớ phải với địa chỉ là bội của 4 byte.** DMEM luôn đặt 2 bit thấp của bất kỳ địa chỉ nào bạn cung cấp về không (tức là, làm tròn xuống bội gần nhất của 4), và sau đó truy cập 4 byte bắt đầu từ địa chỉ đã sửa đổi này.

    Ví dụ, nếu bạn cho DMEM địa chỉ `19` = `0b010011`, DMEM sẽ đặt 2 bit thấp về không để có `16` = `0b010000`, và sau đó bắt đầu từ địa chỉ đã sửa đổi này để truy cập 4 byte (`16-17-18-19`).

    Bạn không cần đặt 2 bit thấp về không—triển khai DMEM được cung cấp sẽ tự động làm điều này cho bất kỳ địa chỉ nào bạn cung cấp.

* **Giả sử truy cập bộ nhớ trong các lệnh không vượt qua ranh giới word**. Tất cả các lệnh nên tuân theo quy ước tiêu chuẩn RV32I và sẽ không vượt qua ranh giới word trong bộ nhớ (tức là, một lần truy cập đơn sẽ không bao giờ vượt qua đường đỏ trong @fig-aligned-memory-vi).
  * Tất cả các lệnh `lw` và `sw` sẽ sử dụng địa chỉ bộ nhớ kết thúc bằng `0b00` (tức là bội của 4).
  * Tất cả các lệnh `lh`, `lhu`, `sh` sẽ sử dụng địa chỉ bộ nhớ kết thúc bằng `0b00`, `0b01`, hoặc `0b10` (không bao giờ địa chỉ kết thúc bằng `0b11`[^why-vi]).
  * Tất cả các lệnh `lb`, `lbu`, `sb` sử dụng bất kỳ địa chỉ bộ nhớ nào.[^why-vi]

* **DMEM chỉ sử dụng 16 bit thấp**. Do giới hạn kích thước Logisim, đơn vị bộ nhớ chỉ sử dụng 16 bit thấp của địa chỉ được cung cấp, loại bỏ 16 bit cao. Điều này có nghĩa là bộ nhớ chỉ có thể lưu trữ $2^{16}$ byte dữ liệu.

    Các test được cung cấp sẽ luôn đặt 16 bit cao của địa chỉ là 0, và bất kỳ test nào bạn viết nên tránh sử dụng 16 bit cao khi tương tác với bộ nhớ.

* **Partial Store**: Xem phần bên dưới để biết chi tiết dành riêng cho partial store, ví dụ: unit test.

[^why-vi]: Chúng tôi để bạn xác định tại sao các trường hợp này sẽ/sẽ không vượt qua ranh giới word 32-bit.

### Partial Load

Mạch `partial_load.circ` trong project khóa học được thiết kế để nhận 32 bit dữ liệu đọc từ DMEM, trích xuất dữ liệu liên quan, sau đó xử lý dữ liệu để đưa vào thanh ghi 32-bit. Các tín hiệu cho subcircuit này được liệt kê trong @tab-partial-load-vi.

:::{table} Tín hiệu cho subcircuit partial load của project khóa học.
:label: tab-partial-load-vi

| **Tên** | **Hướng** | **Độ Rộng Bit** | **Mô Tả** |
| :--- | :--- | :--- | :--- |
| `Instruction` | Đầu vào | 32 | Lệnh load đang được thực thi. |
| `MemAddress` | Đầu vào | 32 | Địa chỉ bộ nhớ để đọc từ.[^addr-vs-access-vi] |
| `DataFromMem` | Đầu vào | 32 | Dữ liệu đọc từ DMEM. |
| `DataToReg` | Đầu ra | 32 | Dữ liệu để đưa vào thanh ghi. |
:::

[^addr-vs-access-vi]: Quan trọng: Hai bit thấp của địa chỉ `MemAddress` (ví dụ: tính từ lệnh) KHÔNG được đặt về không. Tuy nhiên, DMEM sẽ chuyển đổi địa chỉ thành bội của 4 trước khi truy cập bộ nhớ.

**Hành vi**: Cho load, DMEM sẽ luôn đọc 32 bit bộ nhớ bắt đầu từ địa chỉ là bội của 4 byte. Subcircuit partial load sau đó sử dụng chính lệnh để xác định byte nào cần trích xuất.

**Ví dụ 1**. Giả sử chúng ta có lệnh `lb` trên địa chỉ `6` = `0b000110`.

* DMEM sẽ đọc 4 byte tại địa chỉ `4-5-6-7`.
* Chúng ta muốn chỉ byte tại địa chỉ `6` (nhắc lại, lệnh `lb`).
* 2 bit thấp của địa chỉ `6` là `0b10`, nên chúng ta muốn byte thứ 2 (đánh số từ 0, tức là bit 16-23) của dữ liệu đọc từ DMEM.

**Ví dụ 2**. Giả sử chúng ta có lệnh `lh` trên địa chỉ `9` = `0b001001`.

* DMEM sẽ đọc 4 byte tại địa chỉ `8-9-10-11`.
* Chúng ta muốn hai byte tại địa chỉ `9-10` (nhắc lại, lệnh `lh`).
* 2 bit thấp của địa chỉ `9` là `0b01`, nên chúng ta muốn bắt đầu trích xuất tại byte thứ 1 (đánh số từ 0) và trích xuất hai byte, tức là bit 8-23 của dữ liệu đọc từ DMEM.

:::{table} Nếu hữu ích, tất cả các kịch bản bạn nên xử lý trong subcircuit partial load cho project khóa học. `SignExt` có nghĩa là mở rộng dấu.
:label: tab-partial-load-details-vi

| Lệnh | `MemAddress[1:0]`[^addr-vs-access-vi] | Giá trị để đưa vào `DataToReg` |
| :--- | :--- | :--- |
| `lb` | `00` | `SignExt(DataFromMem[7:0])` |
| `lb` | `01` | `SignExt(DataFromMem[15:8])` |
| `lb` | `10` | `SignExt(DataFromMem[23:16])` |
| `lb` | `11` | `SignExt(DataFromMem[31:24])` |
| `lh` | `00` | `SignExt(DataFromMem[15:0])` |
| `lh` | `01` | `SignExt(DataFromMem[23:8])` |
| `lh` | `10` | `SignExt(DataFromMem[31:16])` |
| `lw` | `00` | `DataFromMem` |

:::

### Partial Store

Mạch `partial_store.circ` trong project khóa học được thiết kế để nhận dữ liệu từ thanh ghi, xử lý nó, và lưu các byte liên quan vào bộ nhớ. Các tín hiệu cho subcircuit này được liệt kê trong @tab-partial-store-vi.

:::{table} Tín hiệu cho subcircuit partial store của project khóa học.
:label: tab-partial-store-vi

| **Tên Tín Hiệu** | **Hướng** | **Độ Rộng Bit** | **Mô Tả** |
| :--- | :--- | :--- | :--- |
| `Instruction` | Đầu vào | 32 | Lệnh store đang được thực thi. |
| `MemAddress` | Đầu vào | 32 | Địa chỉ bộ nhớ để store vào.[^addr-vs-access-vi] |
| `DataFromReg` | Đầu vào | 32 | Dữ liệu từ thanh ghi. |
| `MemWEn` | Đầu vào | 1 | Tín hiệu điều khiển chỉ ra liệu ghi vào bộ nhớ có được bật cho lệnh này. |
| `DataToMem` | Đầu ra | 32 | Dữ liệu để store vào bộ nhớ. |
| `MemWriteMask` | Đầu ra | 4 | Mặt nạ ghi chỉ ra liệu mỗi byte của `DataToMem` sẽ được ghi vào bộ nhớ. |
:::

**Hành vi**: Cho store, DMEM sẽ ghi dữ liệu được bitmask vào bộ nhớ tại địa chỉ là bội của 4 byte. Mỗi bit trong **mặt nạ ghi** 4-bit `MemWriteMask` tương ứng với 4 byte của word; nếu một bit trong `MemWriteMask` là 0, DMEM sẽ không store byte tương ứng của `DataToMem` vào bộ nhớ.

**Ví dụ 1**: Giả sử chúng ta có lệnh `sb` trên địa chỉ `3` = `0b000011`.

* Trong các địa chỉ byte của word `0-1-2-3`, chúng ta thực sự muốn chỉ ghi một byte tại địa chỉ `3`, vì 2 bit thấp của địa chỉ `3` là `0b11` (và đây là lệnh `sb`).
* Tạo giá trị 32-bit trong đó bit 24-31 là 8 bit chúng ta muốn store vào bộ nhớ.[^doesnt-matter-vi]
* Tạo mặt nạ ghi 4-bit `0b1000`, nói rằng chỉ ghi byte thứ ba vào bộ nhớ, để các byte khác trong word bộ nhớ không thay đổi.

**Ví dụ 2**: Giả sử chúng ta có lệnh `sh` trên địa chỉ `2` = `0b000010`.

* Trong các địa chỉ byte của word `0-1-2-3`, chúng ta thực sự muốn ghi hai byte tại địa chỉ `2` và `3`, vì 2 bit thấp của địa chỉ `2` là `0b10` (và đây là lệnh `sh`).
* Tạo giá trị 32-bit trong đó bit 16-31 là 16 bit chúng ta muốn store vào bộ nhớ.[^doesnt-matter-vi]
* Tạo mặt nạ ghi 4-bit `0b1100`, nói rằng chỉ ghi byte thứ hai và thứ ba vào bộ nhớ, để các byte khác trong word bộ nhớ không thay đổi.

[^doesnt-matter-vi]: Các bit thấp 0-23 có thể là tất cả số không, mặc dù trong thực tế các bit này không quan trọng vì `MemWriteMask`.

Lưu ý rằng các lệnh `sh` và `sb` chỉ định dữ liệu là các bit thấp của giá trị 32-bit, tức là 16 bit dưới và 8 bit dưới, tương ứng. Xem @tab-partial-store-details-vi.

:::{table} Nếu hữu ích, tất cả các kịch bản bạn nên xử lý trong subcircuit partial store cho project khóa học. (Cũng nhớ lại [không mở rộng dấu cho store](#sec-partial-load-store).)
:label: tab-partial-store-details-vi

| Lệnh | `MemAddress[1:0]`[^addr-vs-access-vi]  | `DataToMem[31:24]` | `DataToMem[23:16]` | `DataToMem[15:8]` | `DataToMem[7:0]` | `MemWriteMask` |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| `sb` | `00` | `0` | `0` | `0` | `DataFromReg[7:0]` | `0001` |
| `sb` | `01` | `0` | `0` | `DataFromReg[7:0]` | `0` | `0010` |
| `sb` | `10` | `0` | `DataFromReg[7:0]` | `0` | `0` | `0100` |
| `sb` | `11` | `DataFromReg[7:0]` | `0` | `0` | `0` | `1000` |
| `sh` | `00` | `0` | `0` | `DataFromReg[15:8]` | `DataFromReg[7:0]` | `0011` |
| `sh` | `10` | `DataFromReg[15:8]` | `DataFromReg[7:0]` | `0` | `0` | `1100` |
| `sw` | `0b00` | `DataFromReg[31:24]` | `DataFromReg[23:16]` | `DataFromReg[15:8]` | `DataFromReg[7:0]` | `1111` |

:::

Mẹo/nhắc nhở cho project khóa học:

* Nhớ rằng **chúng ta không nên ghi** khi lệnh của chúng ta không phải là store. Khi tín hiệu điều khiển `MemWEn` là 0, `MemWriteMask` nên được đặt là `0b0000`.
* Các byte trong `DataToMem` không được ghi vào file (tức là nơi `MemWriteMask` là 0) về mặt kỹ thuật có thể là bất kỳ giá trị nào[^doesnt-matter-vi], nhưng @tab-partial-store-details-vi liệt kê chúng là 0 trong bảng. Unit test cho phần này cũng giả định rằng các byte đó sẽ là 0.
