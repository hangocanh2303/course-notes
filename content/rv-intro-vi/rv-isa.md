---
title: "RISC-V ISA"
---

## Mục tiêu học tập

* Biết rằng ISA RISC-V 32I chỉ định 32 thanh ghi rộng 32-bit. Trong số này, thanh ghi zero `x0` được cố định bằng zero.
* So sánh và đối chiếu hợp ngữ với ngôn ngữ bậc cao.
* Biết rằng ISA RISC-V 32I chỉ định các lệnh hợp ngữ và cách chúng dịch sang lệnh máy.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/mIDxHr5_sxo
:width: 100%
:title: "[CS61C FA20] Lecture 07.2 - RISC-V Intro: Elements of Architecture: Registers"
:::

::::

Nhớ lại từ [trước đó](#sec-isa-note) rằng một ISA chỉ định các lệnh ngôn ngữ hợp ngữ, lệnh máy, và các tính năng kiến trúc cơ bản.

## Định nghĩa tập lệnh RISC-V

Một kiến trúc nhất định (ví dụ: RISC-V) hỗ trợ một **tập lệnh** nhất định, tức là tập hợp các lệnh chúng ta có thể sử dụng cho kiến trúc đó.

(sec-assembly-language)=
### Ngôn ngữ hợp ngữ

Các lệnh máy tính có thể được chỉ định chính xác bằng **ngôn ngữ hợp ngữ**. Thực thi một dòng mã hợp ngữ có nghĩa là thực thi một lệnh trên máy tính.

Trong RISC-V, một **lệnh hợp ngữ** có một **tên phép toán** (**opname**) và **toán hạng** chỉ định thanh ghi nguồn và đích, giá trị, vị trí bộ nhớ, các lệnh khác, v.v.

```
add x1 x2 x3
```

Lệnh này là phép toán `add`. Khi được thực thi, bộ xử lý sẽ cộng các giá trị của thanh ghi `x2` và `x3` với nhau và lưu kết quả trong thanh ghi `x1`. Chúng ta thảo luận quy ước này và _nhiều lệnh khác nữa_ trong vài bài giảng tiếp theo!

:::{hint} Comment mã hợp ngữ của bạn!

Luôn quan trọng phải có comment để làm cho mã có thể đọc được. Comment trong hợp ngữ được cho là quan trọng hơn comment trong ngôn ngữ bậc cao. Rốt cuộc, trong C và Java, chúng ta có thể chỉ định tên biến để tạo mã có ý nghĩa cú pháp. Ngược lại, không có tên biến trong hợp ngữ, nên mã RISC-V không có comment **thực tế không thể debug** đúng cách.

Để viết comment trong RISC-V, sử dụng dấu thăng (`#`)[^apollo]. Mọi thứ bên phải dấu thăng bị assembler bỏ qua. Không giống C, không có comment nhiều dòng (như `/* ... */` trong C); bạn phải sử dụng kiểu dấu thăng cho mỗi dòng được comment.

:::

[^apollo]: Kiểu comment `#` có lịch sử lâu dài. Ví dụ, mã máy tính Hướng dẫn Apollo năm 1966 (được viết bởi lập trình viên trưởng [Margaret Hamilton](https://www.smithsonianmag.com/smithsonian-institution/margaret-hamilton-led-nasa-software-team-landed-astronauts-moon-180971575/)) sử dụng các quy ước comment tương tự. Bản in của mã cho tàu đổ bộ mặt trăng nổi tiếng cao hơn chính Hamilton. Bạn thực sự có thể tìm thấy mã hạ cánh Apollo trên GitHub. Như được báo cáo trên [ABC News](https://abcnews.go.com/Technology/apollo-11s-source-code-tons-easter-eggs-including/story?id=40515222), [mã](https://github.com/chrislgarry/Apollo-11/blob/247dd7d0d1b0e7f9f270750ec08983e0a72e73e1/Luminary099/THE_LUNAR_LANDING.agc#L245) có [rất nhiều](https://github.com/chrislgarry/Apollo-11/blob/247dd7d0d1b0e7f9f270750ec08983e0a72e73e1/Luminary099/LUNAR_LANDING_GUIDANCE_EQUATIONS.agc#L179) [easter egg](https://github.com/chrislgarry/Apollo-11/blob/247dd7d0d1b0e7f9f270750ec08983e0a72e73e1/Luminary099/BURN_BABY_BURN--MASTER_IGNITION_ROUTINE.agc#L61) cho thấy bản chất của các nhà khoa học máy tính thời kỳ đầu.

(sec-machine-language)=
### Ngôn ngữ máy

Hãy nhớ rằng máy tính chỉ hiểu `1` và `0`; văn bản như `add x1 x2 x3` là khá vô nghĩa với phần cứng. Do đó, một ISA thực sự định nghĩa các lệnh cho máy tính xuống đến _mức bit_; nó chỉ định cả ngôn ngữ hợp ngữ _và_ **ngôn ngữ máy**.

Một **lệnh máy** là biểu diễn bit của một lệnh hợp ngữ. ISA RISC-V chỉ định rằng lệnh `add x1 x2 x3` dịch sang 32 bit mã máy sau:

```
00000000001100010000000010110011
```

Các bit mã máy này chỉ định cách kiến trúc máy tính nên thực hiện số học, đọc/ghi thanh ghi, truy cập bộ nhớ, chuyển điều khiển, v.v. Chúng ta thảo luận về các đặc tả lệnh máy trong một [phần sau](#sec-machine-instructions).

(sec-rv32i-registers)=
## Thanh ghi RV32I

Điều gì định nghĩa "các tính năng kiến trúc cơ bản"? Bây giờ, chúng ta giới thiệu **thanh ghi**; chúng ta để [truy cập bộ nhớ](#sec-load-store) và [thực thi lệnh](#sec-rv-pc) cho các phần sau.

ISA RISC-V chỉ định **32 thanh ghi**.

:::{note} Tại sao 32 thanh ghi?

Sự lựa chọn này có vẻ tùy ý nhưng tuân theo nguyên tắc "Goldilocks".[^goldilocks] Quá nhiều thanh ghi sẽ làm chậm máy và cực kỳ đắt tiền. Tuy nhiên, quá ít thanh ghi sẽ yêu cầu (trong số những thứ khác) logic compiler cực kỳ phức tạp. 32 thanh ghi, ít nhất đối với các kiến trúc sư RISC-V, được coi là "vừa phải".
:::

(sec-reg-size)=
### Kích thước thanh ghi

Nhớ một khái niệm chúng ta đã thảo luận trước đó trong khóa học: [từ phần cứng](#sec-words). Kích thước từ cũng xác định **kích thước thanh ghi**. Trong ISA RV32I, kích thước từ là 32 bit, nghĩa là mỗi thanh ghi RV32I **rộng 32 bit**.

[^goldilocks]: Từ [truyện cổ tích Anh](https://en.wikipedia.org/wiki/Goldilocks_and_the_Three_Bears): "Cháo này quá nóng; Cháo này quá lạnh; cháo này vừa phải."

(sec-register-names)=
### Tên và số thanh ghi

Không giống ngôn ngữ bậc cao như C hoặc Java, không có biến trong hợp ngữ. Thay vào đó, để thao tác trên dữ liệu, các lệnh hợp ngữ sử dụng **tên thanh ghi** hoặc **số thanh ghi**.

Trong RV32I, thanh ghi được đánh số từ 0 đến 31. Để tham chiếu đến thanh ghi theo số, một lệnh hợp ngữ có thể chỉ định `x0`, `x1`, ..., `x31`. Thanh ghi cũng có thể được tham chiếu bằng tên. Các tên thanh ghi này được ISA chỉ định và tạo điều kiện cho các quy ước khi viết hợp ngữ. Chúng ta thảo luận thêm sau.

(sec-x0)=
#### Thanh ghi Zero

Bây giờ, chúng ta định nghĩa một thanh ghi rất quan trọng: thanh ghi `x0`, có tên thanh ghi `zero`. [Thanh ghi zero](https://en.wikipedia.org/wiki/Zero_register) đặc biệt này được cố định bằng zero, và bạn không thể thay đổi giá trị của nó. Trái với trực giác, việc có sẵn biểu diễn kích thước thanh ghi của zero cho vô số phép toán cực kỳ hữu ích. Các kiến trúc sư RISC-V cũng nghĩ vậy, và sẵn sàng hy sinh một thanh ghi dữ liệu ít hơn để chỉ định zero trực tiếp trên bộ xử lý.

(sec-hll-vs-assembly)=
## Ngôn ngữ hợp ngữ vs. Ngôn ngữ bậc cao

Bạn có thể thấy phần này hữu ích hơn sau khi xem một số ví dụ về hợp ngữ [trong phần tiếp theo](#sec-rv-arithmetic).

Compiler dịch từ ngôn ngữ bậc cao như C và Java xuống hợp ngữ, vì vậy các lệnh RISC-V thường **liên quan chặt chẽ** với các phép toán ngôn ngữ bậc cao phổ biến. Tuy nhiên, ngôn ngữ bậc cao trừu tượng hóa các thành phần của kiến trúc. @tab-hll-vs-assembly ghi nhận các khác biệt chính.

:::{table} Ngôn ngữ bậc cao vs. Hợp ngữ
:label: tab-hll-vs-assembly
:align: center

| Điểm | C, Java | RISC-V |
|:-: | :--- | :--- |
| 1 | Biến phải được khai báo và có kiểu. | Nội dung thanh ghi chỉ là bit. |
| 2 | Kiểu biến xác định phép toán. | Phép toán xác định cách sử dụng nội dung thanh ghi. |
| 3 | Mỗi toán tử có thể biểu thị nhiều phép toán. | Tên phép toán và phép toán là một-một. |
| 4 | Mỗi câu lệnh có thể chứa nhiều phép toán. | Mỗi dòng là một lệnh duy nhất. |

:::

**Điểm 1**. Trong C (và hầu hết ngôn ngữ bậc cao), chúng ta khai báo biến của một kiểu cụ thể; các tên này CHỈ có thể đại diện cho một giá trị của kiểu nó được khai báo. Ngược lại, thanh ghi không có kiểu, cũng không được khai báo. Chúng đơn giản là các bộ chứa lưu trữ–một tập nhỏ các vị trí dữ liệu đặc biệt được xây dựng trực tiếp vào phần cứng bộ xử lý. Trong RV32, thanh ghi lưu trữ 32 bit.

**Điểm 2 và 3**. Trong ngôn ngữ bậc cao, kiểu biến xác định phép toán. Trong mã C bên dưới, Dòng 1 sử dụng toán tử `+` như số học con trỏ, trong khi Dòng 4 sử dụng `+` như số học số nguyên. Dòng 4 cũng có hai cách sử dụng toán tử `*`, cho phép nhân số nguyên và dereference con trỏ.

```{code} c
:linenos:
int *p = …; 
p = p + 2;
int x = 42;
x = 3 * x + *p + 4;
```

Ngược lại, trong hợp ngữ phép toán được xác định chính xác bởi opname. Ví dụ, chỉ có một lệnh `add` trong RV32I, và `add` luôn có nghĩa là phép cộng số nguyên có dấu. Opname cũng xác định cách nội dung thanh ghi được xử lý. Ví dụ, trong RV32I `add` có nghĩa là nội dung của các toán hạng thanh ghi là số nguyên có dấu. Các lệnh khác xử lý toán hạng thanh ghi như số nguyên không dấu, hoặc thậm chí là địa chỉ bộ nhớ.

**Điểm 4**. Mỗi dòng C (hoặc chính xác hơn, mỗi _câu lệnh_) có thể chứa nhiều phép toán, ví dụ: `a = b * 2 - (arr[2] + *p);` Ngược lại, mỗi dòng trong hợp ngữ chính xác là một lệnh máy tính. Compiler sẽ dịch mã C phức tạp ở trên thành nửa tá dòng hợp ngữ. Chúng ta để đây như một bài tập cho bạn khi bạn học thêm RISC-V.
