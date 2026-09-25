---
title: "Các lệnh truyền dữ liệu"
subtitle: "Các lệnh Load và các lệnh Store"
---

(sec-data-transfer)=
## Mục tiêu học tập

* Sử dụng các lệnh hợp ngữ load và store và tính toán địa chỉ bộ nhớ như thanh ghi cơ sở cộng offset immediate.
* Mở rộng dấu cho các partial load có dấu với `lb` và `lh`.
* Giải thích tại sao partial store không cần mở rộng dấu hoặc mở rộng zero.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/Vo2WL9acC5M
:width: 100%
:title: "[CS61C FA20] Lecture 08.2 - RISC-V lw, sw, Decisions I: Data Transfer Instructions"
:::

Từ 5:00 trở đi

::::

## Cơ bản về truyền dữ liệu RISC-V

Xét cú pháp truy cập bộ nhớ để load và store từ được hiển thị trong @tab-lw-sw:

:::{table} Các lệnh RV32I: Load word (`lw`), Store word (`sw`).
:label: tab-lw-sw
:align: center

| Lệnh | Tên | Mô tả |
| :--- | :--- | :--- |
| `lw rd imm(rs1)` | Load Word | `R[rd] = M[R[rs1] + imm][31:0]`[^verilog] |
| `sw rs2 imm(rs1)` | Store Word | `M[R[rs1] + imm][31:0] = R[rs2][31:0]`[^verilog] |

[^verilog]: Giống trước, chúng ta sử dụng cú pháp Verilog để định nghĩa phép toán chính xác hơn, nhưng bạn không cần phải học Verilog cho khóa học này.

:::

(sec-load-word)=
### Load Word

Lệnh **load word**:

* **Tính toán địa chỉ bộ nhớ** `R[rs1]+imm`
* **Load một từ** từ địa chỉ này trong bộ nhớ, `M[R[rs1] + imm][31:0]`...
* ...vào một **thanh ghi đích**, `rd`.

Địa chỉ bộ nhớ được tính toán với phép số học thanh ghi và immediate. `rs1` được gọi là **thanh ghi cơ sở**. Immediate `imm` được gọi là **offset** và là một hằng số phải được biết tại thời điểm assemble.

:::{hint} Truy cập bộ nhớ phản ánh truy cập mảng

Bạn có thể đang thắc mắc tại sao ISA RISC-V quyết định định dạng các lệnh bộ nhớ để liên quan đến một phép tính số học của địa chỉ (thanh ghi cơ sở cộng offset), sau đó một số truy cập bộ nhớ. Trực giác đến từ truy cập mảng trong ngôn ngữ bậc cao.

Nhớ mảng C? Trong C, [lập chỉ mục ngoặc vuông](#sec-array-indexing) là đường cú pháp cho **số học con trỏ**, sau đó dereference (tức là truy cập bộ nhớ). Với thiết kế của `lw`, chúng ta có thể dịch truy cập mảng với một lệnh: thanh ghi cơ sở `rs1` hoạt động như con trỏ đến đầu mảng, và offset là bước (theo byte). Wow!!!
:::

Hãy xem một ví dụ:

```
lw x10 12(x5)
```

Trong @fig-lw-example, việc thực thi lệnh `lw` này load từ `0x00564253` từ bộ nhớ (tại địa chỉ `0x10C`) vào thanh ghi `x10`.

:::{figure} images/lw-example.png
:label: fig-lw-example
:width: 100%
:alt: "Ví dụ load-word: x5 chứa cơ sở 0x100; từ được assemble theo little-endian tại 0x10C trong bộ nhớ được đánh dấu và load vào thanh ghi x10 như 0x00564253, với mũi tên tím load-từ từ bộ nhớ đến x10."

Minh họa `lw x10 12(x5)`.
:::

:::{note} Hiển thị giải thích
:class: dropdown

Các trường `lw x10 12(x5)`:

* Phép toán: Load word (4 byte trên RV32I)
* Thanh ghi cơ sở: `x5`, trong đó giá trị tại thanh ghi này là `R[x5]` hay `0x100`.
* Offset: immediate `12`
* Thanh ghi đích: `x10`.

Đọc [phần trước](#sec-address-space) để biết cách đọc bố cục bộ nhớ little endian.

1. Tính địa chỉ: `0x100 + 12` là `0x10C`.
2. Đọc từ tại địa chỉ `0x10C`. Bắt đầu từ địa chỉ này, các byte là `0x53`, `0x42`, `0x56`, và `0x00`.
3. Nếu chúng ta giả định bố cục bộ nhớ đang hiển thị kiến trúc **little endian**, chúng ta xây dựng giá trị 32-bit `0x00564253`. Đặt `x10` bằng từ 32-bit này.
:::

(sec-store-word)=
### Store Word

Lệnh **store word**:

* **Tính toán địa chỉ bộ nhớ** `R[rs1]+imm` từ thanh ghi cơ sở `rs1` và offset `imm`.
* **Store từ** trong **thanh ghi nguồn** `rs2` ...
* ...đến từ trong bộ nhớ, `M[R[rs1] + imm][31:0]`.

Hãy xem một ví dụ:

```
sw x10 0(x5)
```

Trong @fig-sw-example, việc thực thi lệnh `sw` này store từ `0x12345678` trong thanh ghi `x10` vào bộ nhớ (tại địa chỉ `0x100`).

:::{figure} images/sw-example.png
:label: fig-sw-example
:width: 100%
:alt: "Ví dụ store-word: x10 chứa 0x12345678 và x5 chứa 0x100; một mũi tên xanh store-vào hiển thị từ trong thanh ghi x10 được ghi theo little-endian vào bốn byte bộ nhớ bắt đầu tại địa chỉ 0x100."

Minh họa `sw x10 0(x5)`.
:::

:::{note} Hiển thị giải thích
:class: dropdown

Các trường `sw x10 0(x5)`:

* Phép toán: Store word (4 byte trên RV32I)
* Thanh ghi cơ sở: `x5`, trong đó giá trị tại thanh ghi này là `R[x5]` hay `0x100`.
* Thanh ghi nguồn: `x10`.
* Offset: immediate `0`

Đọc [phần trước](#sec-address-space) để biết cách đọc bố cục bộ nhớ little endian.

1. Tính địa chỉ: `0x100 + 0` là `0x100`.
2. Từ trong thanh ghi `x10` là `0x12345678`.
3. Store từ này tại địa chỉ `0x100`. Bắt đầu từ địa chỉ này, các byte nên là `0x78`, `0x56`, `0x34`, và `0x12` (một lần nữa, giả định kiến trúc little endian).
:::

(sec-rv-aligned)=
### Endianness và Alignment

Phần này là thời điểm tốt để lùi lại và nhận ra rằng bây giờ chúng ta đã học đủ để diễn giải _đặc tả RISC-V thực tế_!

Từ [Đặc tả RV32I](https://docs.riscv.org/reference/isa/unpriv/rv32.html#ldst):

>  Trong RISC-V, endianness là bất biến địa chỉ byte.
> 
> Trong một hệ thống mà endianness là bất biến địa chỉ byte, thuộc tính sau đây được giữ: nếu một byte được lưu vào bộ nhớ tại một địa chỉ nào đó với một endianness nào đó, thì một load kích thước byte từ địa chỉ đó với bất kỳ endianness nào đều trả về giá trị đã lưu.

Do đó RISC-V hỗ trợ **cả kiến trúc little-endian và big-endian**, và các load và store nhất quán với endianness của kiến trúc. Như một ví dụ:

> Trong cấu hình little-endian, các store nhiều byte ghi byte thanh ghi ít quan trọng nhất tại địa chỉ byte bộ nhớ thấp nhất, theo sau là các byte thanh ghi khác theo thứ tự tăng dần của mức độ quan trọng của chúng. Load tương tự chuyển nội dung của các địa chỉ byte bộ nhớ thấp hơn đến các byte thanh ghi ít quan trọng hơn.

Từ [Đặc tả RV32I](https://docs.riscv.org/reference/isa/unpriv/rv32.html#ldst):

> Bất kể EEI[^eei], các load và store mà địa chỉ hiệu quả của chúng được căn chỉnh tự nhiên sẽ không tạo ra exception địa chỉ không căn chỉnh. Các load và store mà địa chỉ hiệu quả không được căn chỉnh tự nhiên với kiểu dữ liệu được tham chiếu (tức là địa chỉ hiệu quả không chia hết cho kích thước truy cập tính bằng byte) có hành vi phụ thuộc vào EEI.
> ...
> Các truy cập không căn chỉnh đôi khi cần thiết khi port mã cũ...

Theo tiêu chuẩn RISC-V, các load và store từ **nên chỉ định địa chỉ căn chỉnh theo từ**. Đối với RV32I, điều này có nghĩa là `R[rs1] + imm` nên là bội của 4. Các địa chỉ không căn chỉnh theo từ tạo ra hành vi phụ thuộc vào triển khai. Nói cách khác, trong khi RISC-V về mặt kỹ thuật cho phép truy cập không căn chỉnh để hỗ trợ mã cũ, nó rất chậm và lộn xộn. Bạn nên coi "nên" như "phải" để tránh "viết bậy" khắp bộ nhớ. :-)

[^eei]: Giao diện môi trường thực thi (EEI). Chúng ta không thảo luận EEI trong khóa học này.

(sec-partial-load-store)=
## Partial Load và Store

Chúng ta thường làm việc với các kiểu dữ liệu nhỏ hơn 32 bit, như ký tự 8-bit[^colors]. Sẽ lãng phí khi sử dụng một từ đầy đủ cho những thứ này, nên RISC-V hỗ trợ các lệnh để truyền dữ liệu **theo byte**.

[^colors]: Ngoài ra, màu RGB là giá trị 24-bit; mỗi trong ba kênh màu rộng 8-bit.

Nhớ rằng bản thân các thanh ghi có **kích thước từ**. Do đó đặc tả RV32I định nghĩa phải làm gì với các byte còn lại khi load hoặc store dữ liệu **kích thước byte** hoặc **kích thước nửa từ**.

### Store

:::{table} Các lệnh store RV32I.
:label: tab-rv32i-store
:align: center

| Lệnh | Tên | Mô tả |
| :--- | :--- | :--- |
| `sb rs2 imm(rs1)` | Store Byte | `M[R[rs1] + imm][7:0] = R[rs2][7:0]` |
| `sh rs2 imm(rs1)` | Store Half-word | `M[R[rs1] + imm][15:0] = R[rs2][15:0]` |
| `sw rs2 imm(rs1)` | Store Word | `M[R[rs1] + imm][31:0] = R[rs2][31:0]` |
:::

@tab-rv32i-store minh họa [Đặc tả RV32I](https://docs.riscv.org/reference/isa/unpriv/rv32.html#ldst):

> Các lệnh SW, SH, và SB lưu các giá trị 32-bit, 16-bit, và 8-bit từ các bit thấp của thanh ghi rs2 vào bộ nhớ.

Xét lệnh:

```
sb x10 0(x5)
```

Như hiển thị trong @fig-rv-storebyte, lệnh store byte này bỏ qua các byte cao hơn trong thanh ghi nguồn `x10` và chỉ xét **byte ít quan trọng nhất** `R[x10][7:0]` (giá trị `0xEF`). Byte đơn này sau đó được lưu vào bộ nhớ tại địa chỉ `0x100`.

:::{figure} images/storebyte.png
:label: fig-rv-storebyte
:width: 100%
:alt: "Ví dụ store-byte cho sb x10 0(x5): chỉ byte ít quan trọng nhất 0xEF của nội dung thanh ghi x10 được ghi vào địa chỉ 0x100 (được giữ trong thanh ghi x5), để các byte khác trong từ không thay đổi."

Ví dụ lệnh store byte trong bộ nhớ.
:::

### Load

Lệnh Load Byte `lb` lấy một byte đơn từ bộ nhớ và (tương tự như `sb`) đặt byte vào `R[rd][7:0]`, vị trí byte thấp nhất của thanh ghi đích `rd`. Tuy nhiên, không giống `store`, `load` với độ rộng dưới từ phải xét những gì đặt vào các bit cao hơn, hay `R[rd][31:8]` (xem `x10` trong @fig-rv-loadbyte).

:::{figure} images/loadbyte.png
:label: fig-rv-loadbyte
:width: 100%
:alt: "Ví dụ load-byte cho lb x10 0(x5): byte EF tại địa chỉ 0x100 được đặt vào byte thấp của x10 trong khi ba byte cao hơn được điền bằng mở rộng dấu, hiển thị như dấu hỏi trước 0xEF trong thanh ghi x10 được cập nhật."

Ví dụ lệnh load byte trong bộ nhớ. `lb x10 0(x5)` load `0xEF` nhưng _cũng_ phải xác định cách điền 24 bit cao hơn của `x10`.
:::

Vì các **phép toán** hợp ngữ xác định cách diễn giải toán hạng, chúng ta do đó định nghĩa **hai** phép toán "load byte": Load Byte `lb` và Load Byte Unsigned `lbu`.

* `lb`: Nếu giá trị đích nên là số bù hai có dấu, **mở rộng dấu**. Bit quan trọng nhất của byte được load từ bộ nhớ xác định nếu số là âm. Trong @fig-rv-loadbyte, `0xEF` (`0b11101111`) có bit dấu `1` cho kết quả `R[x10]` là `0xFFFFFFEF`. Nếu byte được load là, nói, `0x73` (`0b01110011`), chúng ta điền các bit cao hơn bằng `0` để cho kết quả `R[x10]` là `0x00000073`.[^avocado]
* `lbu`: Nếu giá trị đích nên là số không dấu, đơn giản là mở rộng zero. Nếu lệnh trong @fig-rv-loadbyte thay vào đó là `lbu x10 0(x5)`, thì kết quả `R[x10]` sẽ là `0x000000EF`, bất kể các bit của `0xEF`.

[^avocado]: Trong video bài giảng, Giáo sư Nikolic đưa ra phép so sánh rằng mở rộng dấu giống như đặt một ít bơ trên một bên bánh mì nướng, sau đó phết bit trên cùng khắp các bit cao hơn.

:::{table} Các lệnh load RV32I.
:label: tab-rv32i-load
:align: center

| Lệnh | Tên | Mô tả |
| :--- | :--- | :--- |
| `lb rd imm(rs1)` | Load Byte | `R[rd] = M[R[rs1] + imm][7:0]` (Mở rộng dấu) |
| `lbu rd imm(rs1)` | Load Byte (Unsigned) | `R[rd] = M[R[rs1] + imm][7:0]` (Mở rộng zero) |
| `lh rd imm(rs1)` | Load Half-word | `R[rd] = M[R[rs1] + imm][15:0]` (Mở rộng dấu)|
| `lhu rd imm(rs1)` | Load Half-word (Unsigned)| `R[rd] = M[R[rs1] + imm][15:0]` (Mở rộng zero)|
| `lw rd imm(rs1)` | Load Word | `R[rd] = M[R[rs1] + imm][31:0]` |


:::{warning} Tại sao không có "Store Byte Unsigned"?

Khi bạn store một byte vào bộ nhớ, bạn chỉ lấy byte và đặt nó tại một vị trí cụ thể trong _bộ nhớ_. Không cần điền hoặc mở rộng, cũng không được ưu tiên (ví dụ: có thể bạn đang cập nhật một ký tự đơn trong chuỗi C). Ngược lại, `load` đặt dữ liệu vào _thanh ghi_.

Nhớ trong hợp ngữ, tất cả giá trị thanh ghi chỉ được xử lý như bit. Do đó không có đảm bảo về cách các lệnh tương lai sẽ sử dụng thanh ghi đích, nên các partial load của chúng ta nên ghi tất cả 32 bit của thanh ghi đích.

:::
