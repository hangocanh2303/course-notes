---
title: "Ví dụ: Mảng trên Stack"
---

(sec-example-sp)=
## Mục tiêu học tập

* Sử dụng con trỏ stack để đẩy lượng lớn dữ liệu lên stack.
* Dịch truy cập mảng trong mã C thành lệnh hợp ngữ.
* Xác định một quy ước thanh ghi theo tên: con trỏ stack `sp` là thanh ghi `x2`.

Không có video. Chúng tôi khuyến nghị mở [phần bộ nhớ](#tab-rv32i-memory) của [Green Card RISC-V](#sec-green-card).

## Biến cục bộ trên stack

Cho đến nay, chúng ta đã có thể dịch lượng rất nhỏ mã C để vừa vào thanh ghi. Nếu chúng ta có quá nhiều (dữ liệu) từ không vừa vào thanh ghi, chúng ta phải sử dụng bộ nhớ.

(sec-register-conventions)=
## Quy ước thanh ghi và tên thanh ghi

Như đã đề cập [trước đó](#sec-register-names), một thanh ghi cũng có thể được tham chiếu bằng **tên thanh ghi** của nó. Tên thanh ghi định nghĩa **quy ước**—tức là, chỉ định cách các lệnh hợp ngữ nên sử dụng các thanh ghi cụ thể cho các chức năng phổ biến cụ thể. Các hạn chế này giúp xây dựng "thỏa thuận" về cách dịch các thành phần riêng biệt của chương trình để các lệnh hợp ngữ khớp với nhau.

:::{warning} Con trỏ stack `sp`

Một tên thanh ghi như vậy là con trỏ stack `sp`, tương ứng với thanh ghi `x2`. Giống như trong C, mô hình bộ nhớ RV32 đẩy và bật các stack frame trong quá trình thực thi chương trình; mỗi stack frame tương ứng với dữ liệu cục bộ của phạm vi hiện tại. Tất cả các lệnh hợp ngữ có thể sử dụng `sp` để truy cập stack frame hiện tại và có thể đẩy hoặc bật dữ liệu lên và xuống stack.

Một **quy ước thanh ghi** có nghĩa là tất cả các lệnh hợp ngữ nên tôn trọng quy ước. Trong một lệnh hợp ngữ, viết `sp` _tương đương_ với viết `x2`! Do đó quan trọng là không có lệnh nào sử dụng `x2` như, nói, bộ nhớ tạm thời và vô tình ghi đè `sp`. Làm như vậy có thể khiến các lệnh khác sử dụng `sp` vô tình đọc và ghi bộ nhớ tại các địa chỉ khác ngoài stack!

:::

Green card RISC-V liệt kê tất cả [tên thanh ghi](#tab-calling-convention); chúng ta bắt đầu giới thiệu và sử dụng chúng trong ví dụ này.

## Mô tả bài toán

Dịch chương trình C bên dưới bằng cách **chỉ** sử dụng các thanh ghi tạm thời `t0`, `t1`, `t2`, và con trỏ stack `sp`. Bạn có thể truy cập bộ nhớ khi cần.

(code-sp-example)=
```{code} c
:linenos:

int a = 5;
char b[] = "string"; // Mảng sẽ được lưu trên stack
int c[10];
uint8_t d = b[3];
c[4] = a+d;
c[a] = 20;
```

(sec-sp-example-setup)=
## Thiết lập

Chúng ta sẽ sử dụng thanh ghi tạm thời để lưu địa chỉ, dữ liệu số học, v.v. Chúng ta sẽ lưu biến cục bộ trên stack bằng cách gán mỗi biến một offset nào đó từ con trỏ stack `sp`.

Địa chỉ chính xác của các biến cục bộ này không quan trọng, miễn là chúng ta nhất quán. Giả sử chúng ta sử dụng phép gán sau:

| Biến | Địa chỉ tương đối với con trỏ stack |
| :-- | :--: |
| `int a` | `0(sp)` |
| `char b[7]` | `4(sp)` | 
| `int c[10]` | `12(sp)` | 
| `uint8_t d` | `52(sp)` |

## Giải pháp

Đây là bản dịch hợp ngữ đầy đủ của [mã C gốc](#code-sp-example):

(code-sp-example-rv)=
```{code} bash
:linenos:
li t0 5            # R[t0] = 5
sw t0 0(sp)        # store int a trên stack
li t0 0x69727473   # load "stri"
sw t0 4(sp)        # store phần đầu của chuỗi
li t1 0x0000676E   # load phần còn lại của chuỗi
sw t1 8(sp)        # store phần còn lại của chuỗi
lb t0 7(sp)        # 4(sp) từ b, 3(sp) từ [3] 
sb t0 52(sp)       # store vào d
lw t0 0(sp)        # load a 
lbu t1 52(sp)      # load d
add t2 t0 t1       # R[t2] = a+d
sw t2 28(sp)       # 12(sp) từ c, 16(sp) từ [4] 
li t0 20           # R[t0] = 20
lw t1 0(sp)        # R[t1] = a
slli t1 t1 2       # 5*sizeof(int) = 5*4 = 5<<2
addi t1 t1 12      # t1 từ [a], 12 từ c  
add t1 t1 sp       # tính &c[a]
sw t0 0(t1)        # c[a] = 20
```

Chúng ta thảo luận bản dịch từng dòng bên dưới. Với mỗi phần, cố gắng sử dụng hình ảnh để diễn giải tại sao chúng ta chỉ định các lệnh hợp ngữ tương ứng. Sau đó, kiểm tra lập luận của bạn.

### Dòng 2: `int a = 5;`

```{code} bash
:linenos:
li t0 5            # R[t0] = 5
sw t0 0(sp)        # store int a trên stack
```

:::{figure} images/sp-example-1.png
:label: fig-sp-example-1
:width: 100%
:alt: "Slide cho int a = 5: hợp ngữ li và sw lưu 0x00000005 tại 0(sp) trên stack; các từ stack cao hơn hiển thị rác giữ chỗ với một bản đồ liên kết a, b, c, và d với các offset sp."

Dòng 1 của [Ví dụ con trỏ stack](#code-sp-example).
:::

:::{note} Hiển thị giải thích
:class: dropdown

Để khởi tạo `int a = 5`, store một từ 4B lên stack. Vì `sw` yêu cầu dữ liệu từ của chúng ta phải ở trong thanh ghi trước, thực thi `li` (load immediate) và sau đó `sw`.

:::

### Dòng 3: Khởi tạo chuỗi `char b[] = "string";`

Chúng ta muốn store các byte của `"string"` lên stack, bắt đầu tại vị trí `4(sp)`. Có một cách đơn giản nhưng dài dòng, và một cách ngắn gọn cần một chút khéo léo.

Một cách tiếp cận (đơn giản nhưng dài dòng):

```{code} bash
:linenos:
li t0 0x73
sb t0 4(sp)
li t0 0x74
sb t0 5(sp)
li t0 0x72
sb t0 6(sp)
li t0 0x69
sb t0 7(sp)
li t0 0x6E
sb t0 8(sp)
li t0 0x67
sb t0 9(sp)
sb x0 10(sp)
```

:::{figure} images/sp-example-2-costly.png
:label: fig-sp-example-2-costly
:width: 100%
:alt: "Thiết lập chuỗi từng byte: các lệnh li và sb lặp lại đặt ASCII cho string cộng null tại 4(sp) đến 10(sp); sơ đồ stack liên quan đánh dấu các ký tự đã lưu màu đỏ với rác ở những nơi khác."

Dòng 2 của [Ví dụ con trỏ stack](#code-sp-example).
:::

:::{note} Hiển thị giải thích
:class: dropdown
Các byte char của `"string"` là `{0x73, 0x74, 0x72, 0x69, 0x6E, 0x67, 0x00}` theo ASCII. Sử dụng `li` và `sb` lặp lại để store các byte này tại địa chỉ `4(sp)`, `5(sp)`, v.v. Chi tiết: Chúng ta không `li` cho ký tự kết thúc null và thay vào đó trực tiếp store `x0`, `0` cố định, cho ký tự kết thúc null `'\0'` (có giá trị ASCII `0x00`).
:::

Cách tiếp cận thay thế với ít lệnh hơn nhiều:

```{code} bash
:linenos:
li t0 0x69727473   # load "stri"
sw t0 4(sp)        # store phần đầu của chuỗi
li t1 0x0000676E   # load phần còn lại của chuỗi
sw t1 8(sp)        # store phần còn lại của chuỗi
```

:::{figure} images/sp-example-2-concise.png
:label: fig-sp-example-2-concise
:width: 100%
:alt: "Thiết lập chuỗi ngắn gọn: hai immediate đóng gói stri và ng với padding, sau đó sw tại 4(sp) và 8(sp) bố trí các từ little-endian; thanh ghi t0 và t1 giữ các hằng số đóng gói."

Dòng 2 của [Ví dụ con trỏ stack](#code-sp-example), phiên bản ngắn gọn.
:::

:::{note} Hiển thị giải thích
:class: dropdown

Chúng ta có thể cắt giảm 13 lệnh load immediate/store byte xuống còn 4 lệnh: load immediate, store word.

Điều này yêu cầu chúng ta nghĩ về các ký tự không phải như các byte riêng lẻ, mà như các nhóm 4 ký tự. Nhớ rằng, các ký tự được lưu như bit, và những bit đó cũng có thể đại diện cho số nguyên. Miễn là chúng ta lưu giá trị 32-bit đúng vào dữ liệu, thì 32 bit đó có thể được diễn giải như ký tự khi chúng ta cần.

Thứ hai, lưu ý rằng các từ được lưu theo little endian. Nhưng mảng ký tự nên được lưu theo thứ tự địa chỉ tăng dần. Nếu chúng ta đại diện mỗi nhóm 4 ký tự như một từ 4B, chúng ta sẽ cần đảo ngược các byte sao cho ký tự đầu tiên được lưu tại địa chỉ sớm nhất, và ký tự thứ tư tại địa chỉ cao nhất. Nếu không làm điều này, chuỗi ký tự của chúng ta sẽ không được lưu đúng thứ tự.

* Từ 4B đầu tiên: `{'s', 't', 'r', 'i'}` là `{0x73, 0x74, 0x72, 0x69}`. Trên kiến trúc little endian này, chúng ta nên lưu immediate 32-bit `0x69727473`. Little endian có nghĩa là byte ít quan trọng nhất được lưu tại địa chỉ thấp nhất. `0x73` (`'s'`) được lưu tại `4(sp)`, `0x74` (`'t'`) tại `5(sp)`, v.v.

* Từ 4B thứ hai: `{'n', 'g', '\0'}` là `{0x6E, 0x67, 0x00}`, tức là chỉ ba byte. Nhưng chúng ta cần tạo một immediate 4-byte, nên một lựa chọn hợp lý là thêm zero như byte thứ tư. Trên kiến trúc little endian này, chúng ta nên lưu 32-bit `0x0000676E` (trong đó "đệm zero" là từ lựa chọn của tôi để chèn một byte zero sau ký tự kết thúc null). Little endian có nghĩa là byte ít quan trọng nhất được lưu tại địa chỉ thấp nhất, nên `sw t1 8(sp)` chỉ ra `0x6E` (`'n'`) được lưu tại `8(sp)`, `0x67` (`'g'`) tại `9(sp)`, `0x00` (`'\0'`) tại `10(sp)`, và byte thừa dummy của chúng ta tại `11(sp)`.

Hai câu hỏi chi tiết được nêu ra trong lớp cách đây một thời gian:

* Tại sao sử dụng `t0` và `t1`? Tại sao không chỉ `t0`? Đúng, tôi có thể đã làm điều này (giải pháp gốc cũng tối thiểu hóa thanh ghi sử dụng). Nhưng tôi muốn vẽ hai immediate được lưu vào các thanh ghi riêng biệt trên slide.

* `li` là pseudoinstruction thường phân giải thành `addi`. `addi` không chỉ cho phép immediate 12b, không phải immediate 32b được chỉ định? Đúng, bạn hoàn toàn đúng. `li` như một pseudoinstruction đôi khi phân giải thành cặp lệnh, `lui`/`addi`, trong đó lệnh đầu là "load upper immediate." `lui` cho phép chúng ta load 20 bit trên, và addi cho phép chúng ta load 12 bit dưới. Thêm sau; hóa ra có một ghi chú phức tạp về phép cộng có dấu cần xem xét :-)
:::

### Dòng 4: Mảng chưa khởi tạo `int c[10];`

Không cần lệnh.

:::{figure} images/sp-example-3.png
:label: fig-sp-example-3
:width: 100%
:alt: "int c[10] chưa khởi tạo: slide ghi chú không có lệnh chạy; stack hiển thị int a và dữ liệu chuỗi không thay đổi trong khi 12(sp) trở đi vẫn được gắn nhãn rác ngẫu nhiên cho không gian dành riêng của c."

Dòng 3 của [Ví dụ con trỏ stack](#code-sp-example).
:::

:::{note} Hiển thị giải thích
:class: dropdown

Chúng ta thực sự không lưu bất kỳ dữ liệu nào vào bộ nhớ, nên không cần thực thi lệnh nào. Lúc này tốt để nhớ rằng Thiết lập của chúng ta là quyết định thiết kế (mà chúng ta, như compiler con người, đã thực hiện) không thực sự phân giải thành bất kỳ lệnh nào.
:::

### Dòng 4: Đọc phần tử mảng `uint8_t d = b[3];`

```{code} bash
:linenos:
lb t0 7(sp)        # 4(sp) từ b, 3(sp) từ [3] 
sb t0 52(sp)       # store vào d
```

:::{figure} images/sp-example-4.png
:label: fig-sp-example-4
:width: 100%
:alt: "uint8_t d = b[3]: lb từ 7(sp) load byte 0x69 vào t0, sau đó sb ghi byte đó vào 52(sp); bộ nhớ đánh dấu ký tự được lập chỉ mục và byte thấp của d."

Dòng 4 của [Ví dụ con trỏ stack](#code-sp-example).
:::

:::{note} Hiển thị giải thích
:class: dropdown

Bước này thực hành cách lập chỉ mục mảng, do đó tiết lộ lý do thực sự tại sao lb và sw sử dụng quy ước thanh ghi cơ sở + offset cho địa chỉ. Load `b[3]` (nằm tại `(3 + (4 + R[sp]))`) và store byte đến vị trí của `d` (nằm tại `52 + R[sp]`).
:::

### Dòng 5: Store phần tử mảng `c[4] = a + d;`

```{code} bash
:linenos:
lw t0 0(sp)        # load a 
lbu t1 52(sp)      # load d
add t2 t0 t1       # R[t2] = a+d
sw t2 28(sp)       # 12(sp) từ c, 16(sp) từ [4] 
```

:::{figure} images/sp-example-5.png
:label: fig-sp-example-5
:width: 100%
:alt: "Ví dụ c[4] = a + d: lw và lbu mang int a và uint8_t d vào t0 và t1, add đặt tổng vào t2, và sw lưu nó tại 28(sp); một sơ đồ liệt kê các offset từ 12 đến 32 cho c[0] đến c[5]."

Dòng 5 của [Ví dụ con trỏ stack](#code-sp-example).
:::

:::{note} Hiển thị giải thích
:class: dropdown

Bước này thực hành (1) thêm lập chỉ mục mảng, và (2) cách cộng hai biến ban đầu được lưu trong bộ nhớ.

1. Để cộng hai biến lưu trong bộ nhớ, chúng ta cần load cả hai biến từ bộ nhớ vào thanh ghi (L9-10), sau đó cộng chúng (L11).

2. Dòng cuối tính `&c[4]`. Vì `c` bắt đầu tại `12(sp)`, và `sizeof(int)` là `4`, `c[4]` nằm cách đầu `c` 16 byte, hay (12 + 16) = 28 byte từ `sp`.

:::

(sec-sp-example-line-6)=
### Dòng 6: Lập chỉ mục mảng biến `c[a] = 20;`

Dòng này là thử thách nhất trong cả đống, nên chúng tôi khuyến nghị làm việc qua Venus khi bạn học cách sử dụng Venus trong lab.

```{code} bash
:linenos:
li t0 20           # R[t0] = 20
lw t1 0(sp)        # R[t1] = a
slli t1 t1 2       # 5*sizeof(int) = 5*4 = 5<<2
addi t1 t1 12      # t1 từ [a], 12 từ c  
add t1 t1 sp       # tính &c[a]
sw t0 0(t1)        # c[a] = 20
```

:::{note} Hiển thị giải thích
:class: dropdown

Bước này khác với bước trước vì chúng ta phải tính địa chỉ mảng sử dụng một biến khác, `a`. Vì vậy chúng ta không thể chỉ tính địa chỉ tương đối. Thay vào đó, chúng ta phải tính địa chỉ tuyệt đối trong thanh ghi trước, sau đó store tại địa chỉ đó.

* **Dòng 1.** Thanh ghi `t0` chứa dữ liệu chúng ta muốn store, `20`.

* **Dòng 2-5.** Tính địa chỉ `&c[a]`, theo byte là `4*a + &c` (vì `sizeof(int)` là `4`), tương đối với con trỏ stack là `4*a + 12 + sp`.

  * **Dòng 2.** Load `a` vào `t1`.

  * **Dòng 3.** Nhân `a` với `4`. Điều này tương đương với dịch trái 2 bit. Chúng ta chưa đề cập `slli` rõ ràng trong bài giảng, nhưng tra cứu trong [refcard](#tab-rv32i-arithmetic) chúng ta thấy `slli` là shift left logical immediate.
  
    ```
    0x5 = 0b101     # a
    0b10100         # a << 2 
    = 20 = 0x14     # a << 2 theo thập phân, hex
    ```

  * **Dòng 4.** Cộng `12`. Đây là offset của `c` so với `sp`.

    ```
    20 + 12 = 32   # theo thập phân
    = 0x20         # theo hex 
    ```

  * **Dòng 5.** Cộng sp. Điều này hoàn thành địa chỉ.

* **Dòng 6**: Store dữ liệu mong muốn (thanh ghi `t0`) tại địa chỉ mong muốn (thanh ghi `t1`, với offset zero, vì chúng ta đã tính địa chỉ tuyệt đối).
:::

## Kiểm tra nhanh

:::{tip} Kiểm tra nhanh 1

Xét Dòng 18 từ [bản dịch hợp ngữ](#code-sp-example-rv). Như đã thảo luận trong giải thích của chúng ta về [dòng C 6](#sec-sp-example-line-6), đây là:

```bash
sw t0 0(t1)        # c[a] = 20
```

Khi lệnh này được thực thi, biểu thức C nào là biểu diễn gần nhất của giá trị trong thanh ghi `t1`?

* **A.** `&c`
* **B.** `&a`
* **C.** `c[20]`
* **D.** `c[a]`
* **E.** `&c[a]`
* **F.** Cái gì đó khác

:::

:::{note} Hiển thị đáp án
:class: dropdown

**E.** `&c[a]`. Thanh ghi `t1` ban đầu giữ giá trị của `a` (được load trong [Dòng 14](#code-sp-example-rv)), sau đó bị ghi đè trong Dòng 15-17 để tính `(a * 4) + 12 + R[sp]`. Giá trị thanh ghi `R[t1]` được sử dụng như một địa chỉ–nó là vị trí bộ nhớ mà chúng ta store từ `R[t0]`.

Ghi chú: Giá trị trong thanh ghi `t1` được viết theo ký hiệu (với cú pháp Verilog) là `R[t1]`.

:::


:::{tip} Kiểm tra nhanh 2

Xét chính dòng C 6.

```c
c[a] = 20;
```

Tại sao dịch thành sáu dòng được thảo luận trong giải thích của chúng ta về [dòng C 6](#sec-sp-example-line-6)? Tại sao compiler có thể KHÔNG đơn giản dịch câu lệnh C này thành một lệnh `sw` đơn lẻ như `sw t0 20(sp)`?

:::

:::{note} Hiển thị đáp án
:class: dropdown

`a` là một biến. Giá trị của nó tình cờ là `5` tại runtime, nhưng compiler không nhất thiết có thể mô phỏng runtime chương trình!
:::
