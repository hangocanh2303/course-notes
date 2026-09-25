---
title: "Branch có điều kiện"
---

(sec-branches)=
## Mục tiêu học tập

* Viết các câu lệnh điều kiện trong RISC-V.
* Sử dụng các lệnh `bne` cho các điều kiện if so sánh bằng nhau.
* Phân biệt giữa các lệnh branch có điều kiện và các lệnh jump vô điều kiện.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/OWxcAqFNkpo
:width: 100%
:title: "[CS61C FA20] Lecture 08.3 - RISC-V lw, sw, Decisions I: Decision Making"
:::

Đến 8:50
::::

Trong vài chương trước, chúng ta đã học cách sử dụng RISC-V như một máy tính. Chúng ta đã học các phép toán [số học](#sec-rv-arithmetic) và [bit](#sec-rv-bitwise) và [các lệnh truyền dữ liệu](#sec-data-transfer) để truy cập bộ nhớ.
Tuy nhiên, để hỗ trợ các ngôn ngữ lập trình hiện đại, chúng ta cần hỗ trợ **ra quyết định**–nghĩa là, máy tính nên có khả năng thực thi _có điều kiện_ một số lệnh dựa trên kết quả của các lệnh khác.

## Ôn tập: Luồng điều khiển

Trong C và ngôn ngữ bậc cao, chúng ta chỉ định ra quyết định với cú pháp **luồng điều khiển**. Nhớ lại rằng trong C, thông thường chúng ta thực thi các câu lệnh tuần tự. Cú pháp luồng điều khiển tạo ra các cấu trúc "nhảy" đến các dòng mã khác:

* Câu lệnh `if` (và `if-else`, `if-else-elif`, v.v.)
* Vòng lặp `for` và `while`
* Gọi hàm[^fn-call]

[^fn-call]: Thêm sau!

Nếu chúng ta coi bộ xử lý như một thành phần trung tâm, chúng ta **chuyển** điều khiển đến các câu lệnh tiếp theo để thực thi chúng. Luồng điều khiển có thể chỉ định chuyển **có điều kiện** hoặc **vô điều kiện** đến các câu lệnh không theo thứ tự. Xét mã C trong @fig-c-control-flow:

:::{figure} images/c-control-flow.png
:label: fig-c-control-flow
:width: 60%
:alt: "Đoạn C đánh số dòng với foo và main: các mũi tên xanh và đen từ if hiển thị các đường có điều kiện đến thân then hoặc qua nó nếu không, trong khi các mũi tên xanh hiển thị gọi vô điều kiện vào foo và trả về dòng sau cuộc gọi."

Minh họa chuyển luồng điều khiển vô điều kiện và có điều kiện.
:::

Câu lệnh điều kiện `if` thực thi có điều kiện Dòng 4 chỉ nếu `n > 5`; nếu không, nó thực thi câu lệnh tiếp theo, trên Dòng 6. Ngược lại, cuộc gọi đến `foo` trong Dòng 11 chuyển điều khiển vô điều kiện đến câu lệnh đầu tiên trong `foo` trên Dòng 2; sau đó, khi `foo` trả về trên Dòng 6, điều khiển chuyển vô điều kiện trở lại Dòng 12.

(sec-branches-jumps)=
## Branch và Jump

RISC-V triển khai luồng điều khiển bằng cách thay đổi thứ tự thực thi trong mã. Cụ thể, các lệnh như vậy **chuyển điều khiển** bằng cách cập nhật [bộ đếm chương trình](#sec-program-counter) không phải đến lệnh tiếp theo như mặc định, mà đến _một_ lệnh khác. Có hai loại[^philosophy] lệnh như vậy trong RV32I: **jump vô điều kiện** và **branch có điều kiện**.

[^philosophy]: Nếu jump được định nghĩa là vô điều kiện, có jump _có điều kiện_ không? Còn branch _vô điều kiện_ thì sao? Cả hai có thể được định nghĩa bằng cách sử dụng cái kia, và bạn sẽ thấy sự chồng chéo thuật ngữ này trong bài giảng, trong cuộc trò chuyện, và trong tài liệu. [RISC-V ISA](https://docs.riscv.org/reference/isa/unpriv/rv32.html#2-6-control-transfer-instructions) cẩn thận đề cập đến cả "lệnh chuyển điều khiển" và, trong hầu hết các trường hợp, liên kết các tính từ "vô điều kiện" và "có điều kiện" duy nhất với "jump" và "branch", tương ứng.

**Jump vô điều kiện**. Luôn đặt PC thành một lệnh khác. Chúng ta giới thiệu lệnh jump chính ở đây:

```bash
j Label
```

Khi chạy, lệnh này sẽ **J**ump vô điều kiện đến lệnh được gắn nhãn `Label`.

**Branch có điều kiện**. Điều kiện dựa trên so sánh hai giá trị thanh ghi. Nếu điều kiện được đáp ứng, đặt PC thành một lệnh khác. Nếu không, điều kiện không được đáp ứng, nên đặt PC thành lệnh tiếp theo. Định dạng chung cho các lệnh branch là `bxx rs1 rs2 Label`, trong đó "`xx`" chỉ định loại so sánh cần thực hiện.

(sec-labels)=
:::{hint} Nhãn là gì?
Nhãn là các định danh cho các dòng mã cụ thể (trong C) hoặc lệnh hợp ngữ (trong RISC-V). Nhãn không tự chúng định nghĩa một dòng mã, mặc dù chúng phải có **tên duy nhất**. Trong hợp ngữ, nhãn được liên kết với các địa chỉ lệnh cụ thể. Chúng sau đó được assembler sử dụng để dịch các lệnh branch và jump thành các đối tác mã máy của chúng.[^labels]
[^labels]: Thêm sau.
:::

### Ví dụ Branch

Làm thế nào chúng ta sử dụng các lệnh branch? Hãy xem `beq` và `bne` trong @tab-rv-beq-bne:

:::{table} Hai lệnh branch có điều kiện RV32I.
:label: tab-rv-beq-bne
:align: center
| Lệnh | `beq rs1 rs2 Label` | `bne rs1 rs2 Label` |
| :--: | :--- | :--- |
| Gợi nhớ | **B**ranch if **eq**ual (Branch nếu bằng) | **B**ranch if **n**ot **e**qual (Branch nếu không bằng) |
| Điều kiện so sánh | Giá trị thanh ghi bằng nhau:<br/>`R[rs1] == R[rs2]` | Giá trị thanh ghi không bằng nhau:<br/>`R[rs1] != R[rs2]` |
| Điều kiện được đáp ứng | `PC = <địa chỉ của Label>` |  `PC = <địa chỉ của Label>` |
| Điều kiện không được đáp ứng | `PC = PC + 4` | `PC = PC + 4` |

:::

:::{hint}
Để có chương trình RISC-V dễ đọc, hiệu quả, bạn có thể cần **phủ định điều kiện branch**.
:::

Xét hai ví dụ bên dưới, giả định ánh xạ sau của biến số nguyên C đến thanh ghi:

```bash
x    y    z    i    j
x10  x11  x12  x13  x14
```

(sec-branch-ex1)=
:::{tip} Ví dụ 1

```{code} c
:linenos:
if (i == j) {
  x = y + z;
} else {
  x = y - z;
}
// …
```

Hai bản dịch hợp lý của mã này sang hợp ngữ RISC-V ở bên dưới và tận dụng hai nhãn lệnh, `If` và `End`.

```bash
# Lựa chọn A
       beq x13 x14 If
       sub x10 x11 x12
       j End
If:    add x10 x11 x12
End:   # …
```

Lựa chọn A sử dụng `beq`. Nếu `i` và `j` bằng nhau, nhảy đến nhãn `If` để thực thi lệnh `add` có điều kiện. Nếu không, bộ xử lý tự nhiên di chuyển đến điều kiện "else", đó là lệnh `sub`, sau đó nó nhảy vô điều kiện với `j End` để bỏ qua lệnh được gắn nhãn `If`.

```bash
# Lựa chọn B
       bne x13 x14 Else
       add x10 x11 x12
       j End
Else:  sub x10 x11 x12
End:   # …
```

Lựa chọn B sử dụng `bne` để _đảo ngược_ điều kiện bất đẳng thức trong Dòng 1 của mã C. Nếu `i` và `j` không bằng nhau, _bỏ qua_ lệnh `add` có điều kiện, và đi đến nhãn `Else` để thực thi lệnh `sub`. Nếu branch _không_ được lấy (`i` và `j` bằng nhau), thì bộ xử lý tự nhiên di chuyển vào lệnh tiếp theo—`add`, chính xác là những gì chúng ta muốn–và nhảy vô điều kiện với `j End` để bỏ qua lệnh được gắn nhãn `Else`.
:::

(sec-branch-ex2)=
:::{tip} Ví dụ 2

```{code} c
:linenos:
if (i == j)	{
	x = y + z;
}
// …
```

```bash
# Lựa chọn A
       beq x13 x14 If
       j End
If:    add x10 x11 x12
End:   # …
```

```bash
# Lựa chọn B
       bne x13 x14 End
       add x10 x11 x12
End:   # …
```

Một lần nữa, cả hai lựa chọn đều hợp lệ. Một lần nữa, Lựa chọn B sử dụng `bne` và biểu diễn gần hơn với cấu trúc C gốc. Lựa chọn B cũng tạo ra ít lệnh hơn và sẽ tốt hơn cho hiệu suất về lâu dài.
:::

## Tóm tắt các lệnh Branch

@tab-rv-branch liệt kê các lệnh branch có điều kiện từ [bảng Control green card RISC-V](#tab-rv32i-control):

:::{table} Các lệnh branch có điều kiện RV32I.
:label: tab-rv-branch
:align: center

| Lệnh | Tên/Mô tả |
|:--- |:---|
| `beq rs1 rs2 Label` | Branch if Equal (Branch nếu bằng) |
| `bne rs1 rs2 Label` | Branch if Not Equal (Branch nếu không bằng) |
| `blt rs1 rs2 Label` | Branch if Less Than (signed) (Branch nếu nhỏ hơn, có dấu) (rs1 < rs2) |
| `bge rs1 rs2 Label` | Branch if Greater or Equal (signed) (Branch nếu lớn hơn hoặc bằng, có dấu) (rs1 >= rs2) |
| `bltu rs1 rs2 Label` | Branch if Less Than (unsigned) (Branch nếu nhỏ hơn, không dấu) |
| `bgeu rs1 rs2 Label` | Branch if Greater or Equal (unsigned) (Branch nếu lớn hơn hoặc bằng, không dấu) |
:::

Tập này[^mnemonic] đủ để mô tả các so sánh C: `==`, `!=`, `>`, `<`, `>=`, `<=` cho số nguyên có dấu và không dấu. Từ [Đặc tả RV32I](https://docs.riscv.org/reference/isa/unpriv/rv32.html#2-6-2-conditional-branches):

[^mnemonic]: Đây là mẹo của Giáo sư Bora Nikolic để nhớ các lệnh branch nào được hỗ trợ: "Có tồn tại bánh mì BLT (Bacon, Lettuce, Tomato), nhưng tôi chưa bao giờ thấy bánh mì BGT."

> Lưu ý, BGT, BGTU, BLE, và BLEU có thể được tổng hợp bằng cách đảo ngược các toán hạng thành BLT, BLTU, BGE, và BGEU, tương ứng.

Nói cách khác, `bgt`, `bgtu`, `ble`, `bleu` là pseudoinstruction! Chúng ta để bản dịch của chúng như một bài tập cho bạn :-)

Chúng ta cũng đã thảo luận một **pseudo**instruction jump trong @tab-rv-jump. Chúng ta sẽ giải thích pseudoinstruction này chi tiết hơn trong [phần tương lai](#sec-jumps).

:::{table} Pseudoinstruction jump vô điều kiện RV32I
:label: tab-rv-jump
:align: center

| Lệnh | Tên/Mô tả |
|:--- |:---|
| `j Label` | Unconditional jump (Jump vô điều kiện) |

:::
