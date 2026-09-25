---
title: "Các lệnh số học RISC-V I: Phép toán số học"
short_title: "Số học, Immediate, Pseudoinstruction"
---

(sec-rv-arithmetic)=
## Mục tiêu học tập

* Viết hợp ngữ để thực hiện các phép toán số học.
* Viết các lệnh số học liên quan đến immediate.
* Hiểu cách pseudoinstruction và thanh ghi zero giúp cân bằng tập lệnh rút gọn với tính linh hoạt của các phép toán.

::::{note} 🎥 Video bài giảng: `add`/`sub`
:class: dropdown

:::{iframe} https://www.youtube.com/embed/-Vv4UaG9o9k
:width: 100%
:title: "[CS61C FA20] Lecture 07.3 - RISC-V Intro: RISC-V add/sub Instructions"
:::

::::

::::{note} 🎥 Video bài giảng: Immediate
:class: dropdown

:::{iframe} https://www.youtube.com/embed/vGIEgP2ZZP8
:width: 100%
:title: "[CS61C FA20] Lecture 07.4 - RISC-V Intro: RISC-V Immediates"
:::

::::

(sec-rv-add)=
## Các lệnh `add` và `sub`

Nhìn chung, các lệnh hợp ngữ có một định dạng rất cứng nhắc. Xét **các lệnh số học và logic** như phép cộng (`add`), phép AND bit (`and`), v.v., hoạt động trên hai thanh ghi và lưu kết quả trong thanh ghi thứ ba. Các lệnh này luôn tuân theo cùng cú pháp cứng nhắc như trong @fig-r-type-arithmetic:

:::{figure} images/r-type-arithmetic.png
:label: fig-r-type-arithmetic
:width: 50%
:alt: "Mẫu cú pháp cho lệnh R-type: opname rd rs1 rs2 với các dấu ngoặc gắn nhãn phép toán, thanh ghi đích, thanh ghi nguồn thứ nhất, và thanh ghi nguồn thứ hai."

Các lệnh R-Type (số học và logic liên quan đến hai thanh ghi nguồn).
:::

Các trường được phân tách bằng dấu cách[^commas], theo thứ tự:

* `opname`: Tên phép toán
* `rd`: Thanh ghi đích, tức là toán hạng mà chúng ta lưu kết quả của phép toán.
* `rs1`: Thanh ghi "nguồn 1", tức là toán hạng nguồn thứ nhất cho phép toán.
* `rs2`: Thanh ghi "nguồn 2", tức là toán hạng thứ hai cho phép toán.

:::{note} Tại sao cú pháp cứng nhắc?

Cú pháp cứng nhắc của các lệnh hợp ngữ là có chủ đích. Mỗi lệnh hợp ngữ RV32I ánh xạ trực tiếp đến một lệnh máy 32-bit phản ánh thứ tự của các trường lệnh. Điều này giúp phần cứng **giải mã** các lệnh nhanh chóng vì nó có thể tìm các thanh ghi nguồn và đích một cách đáng tin cậy trong cùng các trường bit, bất kể lệnh nào. Thêm sau!
:::

[^commas]: Về mặt phong cách, các lệnh hợp ngữ có thể bao gồm dấu phẩy. Trong RISC-V, dấu phẩy không cần thiết. Chúng ta sử dụng quy ước chỉ dấu cách trong khóa học này.

Tập hợp đầy đủ các lệnh tuân theo định dạng trong @fig-r-type nằm trên [green card RISC-V](#sec-green-card-arithmetic). Bây giờ, chúng ta tập trung vào hai lệnh trong @tab-add-sub:

:::{table} Các lệnh `add` và `sub`.
:label: tab-add-sub
:align: center

| Lệnh | Tên | Mô tả[^verilog] |
| :-- | :-- | :-- |
| `add  rd rs1 rs2` | ADD | `R[rd] = R[rs1] + R[rs2]` |
| `sub rd rs1 rs2` | SUBtract | `R[rd] = R[rs1] - R[rs2]` |

[^verilog]: Chúng ta sẽ không yêu cầu bạn biết Verilog trong khóa học này, nhưng cú pháp rất hữu ích. `R[rs1]` có nghĩa là dữ liệu trong thanh ghi `rs1`.

:::

Lệnh cộng trong hợp ngữ đơn giản là `add`, dễ nhớ. Lệnh RV32I

```
add x1, x2, x3
```

tương đương với câu lệnh C `a = b + c;` cho số nguyên 32-bit[^signed-unsigned] `a`, `b`, và `c`, trong đó mỗi biến tương ứng với một giá trị được lưu trong thanh ghi. Trong @fig-rv32i-add, biến `a` ở trong thanh ghi `x1`, biến `b` ở trong thanh ghi `x2`, và biến `c` ở trong `x3`.

[^signed-unsigned]: Bạn sẽ thấy sau rằng phép cộng và trừ có dấu và không dấu được triển khai với cùng một mạch phần cứng.

:::{figure} images/rv32i-add.png
:label: fig-rv32i-add
:width: 50%
:alt: "Ví dụ phép cộng ghép RISC-V add x1 x2 x3 với C a = b + c, với các mũi tên từ x1 đến a, x2 đến b, và x3 đến c."

Lệnh cộng `add` trong RISC-V và C.
:::

Phép trừ hoạt động _gần như_ theo cách tương tự, sử dụng phép toán sub. Nếu bạn muốn trừ các giá trị trong `x5` và `x6` và lưu kết quả trong `x4`, bạn sẽ viết như bên dưới, tương đương với câu lệnh số học số nguyên C `d = e - f;` (@fig-rv32i-sub).

```
sub x4 x5 x6
```

:::{figure} images/rv32i-sub.png
:label: fig-rv32i-sub
:width: 50%
:alt: "Ví dụ phép trừ ghép RISC-V sub x4 x5 x6 với C d = e - f, với các mũi tên liên kết thanh ghi đích và nguồn với biến và một ghi chú rằng thứ tự toán hạng quan trọng."

Lệnh trừ `sub` trong RISC-V và C.
:::

:::{hint} Thứ tự toán hạng quan trọng

Một khác biệt chính cần nhớ là phép cộng có tính giao hoán, nên không quan trọng thứ tự bạn đặt các toán hạng nguồn cho `add`. Tuy nhiên, phép trừ không có tính giao hoán, nên thứ tự quan trọng. Lệnh `sub` sẽ luôn trừ giá trị của toán hạng nguồn thứ hai từ toán hạng thứ nhất.

:::

(sec-arithmetic-examples)=
### Số học RISC: Ví dụ

Như đã đề cập [trước đó](#tab-hll-vs-assembly), một dòng C đơn lẻ có thể dịch thành nhiều dòng RISC-V. Xét câu lệnh số học số nguyên C:

```c
a = b + c + d - e;
```

Giả sử rằng các biến `a`, `b`, `c`, `d`, và `e` ánh xạ đến thanh ghi `x10`, `x1`, `x2`, `x3`, và `x4`, tương ứng. Chúng ta có thể sử dụng `x10` như một tổng tích lũy để tính giá trị đúng của `a` sau ba lệnh:

```
add x10  x1 x2   # a_temp = b + c
add x10 x10 x3   # a_temp = a_temp + d
sub x10 x10 x4   # a = a_temp - e
```

::::{tip} Kiểm tra nhanh

Đến lượt bạn! Giả sử ánh xạ bên dưới của biến C đến thanh ghi RISC-V trong @fig-example-2 cho câu lệnh số học số nguyên C bên dưới:

```c
f = (g + h) - (i + j);
```

:::{figure} images/example-2.png
:label: fig-example-2
:width: 50%
:alt: "Dòng C f = (g + h) - (i + j) với tên thanh ghi dưới mỗi biến: x19 cho f, x20 đến x23 cho g đến j."

Ánh xạ biến C đến thanh ghi RISC-V.
:::

Bản dịch nào dưới đây hoạt động? Tại sao?

```
# Cách tiếp cận A
add  x5 x20 x21
add  x6 x22 x23
sub x19  x5  x6
```

```
# Cách tiếp cận B
add x19 x20 x21
sub x19 x19 x22
sub x19 x19 x23
```

::::

:::{note} Hiển thị đáp án
:class: dropdown

Cả hai giải pháp đều hợp lệ. Đánh đổi:

* Cách tiếp cận A sử dụng **thanh ghi tạm thời** `x5` và `x6` và tuân theo trực tiếp hơn thứ tự phép toán trong C. Tuy nhiên, bất kỳ dữ liệu hiện có nào trong thanh ghi `x5` và `x6` sẽ bị ghi đè.

* Cách tiếp cận B tận dụng **đại số** để tránh bất kỳ thanh ghi tạm thời nào. Đối với đại số phức tạp hơn (ví dụ: phân phối một phép nhân), chúng ta có thể cần các compiler thông minh hơn.

:::

## Immediate

**Immediate** là các hằng số trong RISC-V. Immediate được gọi như vậy vì mẫu bit của chúng được mã hóa trực tiếp vào lệnh máy—do đó giá trị của chúng "ngay lập tức" có sẵn cho máy tính.

Immediate xuất hiện thường xuyên trong mã; do đó, chúng có các lệnh riêng (@fig-i-type-arithmetic).

:::{figure} images/i-type-arithmetic.png
:label: fig-i-type-arithmetic
:width: 50%
:alt: "Mẫu cú pháp cho lệnh I-type số học hoặc logic: opname rd rs1 imm với các dấu ngoặc gắn nhãn phép toán, đích, thanh ghi nguồn thứ nhất, và toán hạng immediate."

Các lệnh I-Type số học và logic liên quan đến một thanh ghi nguồn và một immediate.
:::

Giống [trước](#fig-r-type), các trường được phân tách bằng dấu cách[^commas], theo thứ tự:

* `opname`: Tên phép toán
* `rd`: Thanh ghi đích, tức là toán hạng mà chúng ta lưu kết quả của phép toán.
* `rs1`: Thanh ghi "nguồn 1", tức là toán hạng nguồn thứ nhất cho phép toán.
* `imm`: Immediate (hằng số).

Tập hợp đầy đủ các lệnh tuân theo định dạng trong @fig-i-type-arithmetic nằm trên [green card RISC-V](#sec-green-card-arithmetic).

### Lệnh `addi`

Hãy thảo luận về lệnh **Add Immediate** (@tab-addi).

:::{table} Lệnh `addi`.
:label: tab-addi
:align: center

| Lệnh | Tên | Mô tả[^verilog] |
| :-- | :-- | :-- |
| `addi rd rs1 imm` | ADD Immediate | `R[rd] = R[rs1] + imm` |

:::

Lệnh RV32I

```
addi x3, x4, 10
```

tương đương với câu lệnh C `f = g + 10;`, trong đó `f` và `g` là số nguyên 32-bit (@fig-rv32i-addi).

:::{figure} images/rv32i-addi.png
:label: fig-rv32i-addi
:width: 50%
:alt: "Ví dụ add-immediate ghép RISC-V addi x3 x4 10 với C f = g + 10, với các mũi tên từ x3 đến f và x4 đến g và nhấn mạnh tương ứng trên hằng số 10."

Lệnh add immediate trong RISC-V và C.
:::

:::{note} Định dạng tương tự

Định dạng của `addi` và `add` rất tương tự và chỉ khác nhau ở toán hạng cuối (thanh ghi vs. immediate).
[^signed-unsigned] Sự tương đồng này là có chủ đích; khi chúng ta triển khai cả hai lệnh này, chúng ta muốn tái sử dụng càng nhiều phần cứng càng tốt.

:::


::::{warning} Không có lệnh `subi`!

Nhớ lại "R" trong RISC là **Rút gọn**. Nếu một phép toán có thể được phân tách thành các phép toán tương đương hoặc đơn giản hơn, đừng đưa nó vào ISA.

Immediate RISC-V là **có dấu**. Do đó **không có lệnh "subtract immediate"**–chỉ có `addi`. Lệnh

```
addi x3 x4 -10
```

tương đương với câu lệnh C `f = g - 10;` trong đó `f` và `g` là số nguyên 32-bit (@fig-rv32i-subi).

:::{figure} images/rv32i-subi.png
:label: fig-rv32i-subi
:width: 50%
:alt: "Ví dụ trừ qua add-immediate ghép RISC-V addi x3 x4 -10 với C f = g - 10, hiển thị immediate âm như hằng số bị trừ."

Lệnh add immediate trong RISC-V và C với giá trị âm.
:::

::::

## Sử dụng `x0` để rút gọn tập lệnh

Chúng ta đã [thảo luận trước đó](#sec-x0) về thanh ghi zero `x0`. Việc cố định thanh ghi `x0` bằng giá trị zero chứng tỏ cực kỳ hữu ích cho việc tái sử dụng `add` và `addi` cho các phép toán C rất phổ biến.
Hai ví dụ đơn giản được hiển thị trong @fig-rv32i-x0-mv và @fig-rv32i-x0-li:

:::{figure} images/rv32i-x0-mv.png
:label: fig-rv32i-x0-mv
:width: 50%
:alt: "Thành ngữ di chuyển thanh ghi hiển thị RISC-V add x3 x4 x0 tương đương với C f = g, với x0 được đánh dấu là zero và các mũi tên từ x3 và x4 đến f và g."

Để gán biến số nguyên C `f` bằng giá trị của biến số nguyên khác `g`, chúng ta _có thể_ sử dụng `add` với `x0` như một toán hạng nguồn (mặc dù trong thực tế chúng ta không làm vậy[^mv]).
:::


:::{figure} images/rv32i-x0-li.png
:label: fig-rv32i-x0-li
:width: 50%
:alt: "Thành ngữ load-immediate hiển thị RISC-V addi x3 x0 0xff tương đương với C f = 0xff, với một mũi tên từ thanh ghi đích x3 đến biến f và x0 như thanh ghi nguồn với giá trị zero."

Để gán biến `f` bằng một hằng số, sử dụng `addi` với `x0` như toán hạng thanh ghi nguồn.
:::

(sec-pseudoinstructions)=
## Pseudoinstruction

Chúng ta vừa thấy nhiều trường hợp trong đó các câu lệnh C phổ biến dịch sang các lệnh khác trong kiến trúc tập lệnh rút gọn của chúng ta. Mặc dù điều này ổn cho việc thiết kế kiến trúc, compiler (và bạn, như một người viết lệnh hợp ngữ) sẽ thấy hữu ích khi sử dụng **pseudoinstruction**.

**Pseudoinstruction** là các lệnh thuận tiện trong RISC-V. Pseudoinstruction giúp compiler dịch mã ngôn ngữ bậc cao trực tiếp hơn thành các lệnh hợp ngữ (thật sự!), nhưng bản thân chúng không phải là lệnh thật.

Xét hai ví dụ bên dưới (và xem tập hợp đầy đủ trong [green card RISC-V](#tab-rv32i-pseudoinstructions)).

:::{table} Một tập con của pseudoinstruction.
:label: tab-mv-li
:align: center

| Pseudoinstruction | Tên | Mô tả | Dịch |
| :--- | :--- | :--- | :--- |
| `mv rd rs1` | MoVe | `R[rd] = R[rs1]` | `addi rd rs1 0`[^mv] |
| `li rd imm` | Load Immediate | `R[rd] = imm` | `addi rd x0 imm`[^lui] |
| `nop` | No OPeration | không làm gì[^nop] | `addi x0 x0 0` |

[^mv]: Tại sao `addi` với immediate `0` và không phải `add` với `x0` (như trong @fig-rv32i-x0-mv)? Xem [ASM manual trên GitHub](https://github.com/riscv-non-isa/riscv-asm-manual/blob/main/src/asm-manual.adoc#pseudoinstructions).
[^lui]: Mô tả này không đầy đủ cho phạm vi của `imm` trong `addi`. Xem [green-card](@tab-rv32i-pseudoinstructions) và [phần sau](#sec-li-lui) để có bản dịch đầy đủ của load immediate.
[^nop]: Chúng ta sẽ thấy sau cách một lệnh "no-op" có thể cải thiện hiệu suất phần cứng (thật sự).
:::

Khi assemble thành lệnh máy, assembler thay thế pseudoinstruction bằng lệnh thật tương ứng của chúng.
