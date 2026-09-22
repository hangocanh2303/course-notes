---
title: "Word, Endianness"
---

(sec-endianness)=
## Mục tiêu học tập

* Có trực giác về cách kích thước word của phần cứng có thể ảnh hưởng đến bố cục bộ nhớ của chương trình C đã biên dịch.
* Phân biệt giữa kiến trúc 32-bit và 64-bit.
* Đọc bố cục bộ nhớ của các chương trình C được biên dịch trên máy little endian.
* Hiểu cách padding và packing có thể ảnh hưởng đến bố cục bộ nhớ của các thành viên trong một struct C.

::::{note} 🎥 Video bài giảng: Endianness
:class: dropdown

:::{iframe} https://www.youtube.com/embed/wXGhuhLKkqg
:width: 100%
:title: "[CS61C FA20] Lecture 07.3 - RISC-V Intro: RISC-V add/sub Instructions"

Video này được lấy từ sau này trong mùa Thu 2020 và tham chiếu đến RISC-V assembly, mà chúng ta sẽ nói đến trong vài bài sau. Bây giờ, vui lòng bắt đầu từ 7:33 trở đi.
:::
::::

(sec-words)=
## Word

Một word là gì? Trong kiến trúc máy tính, một **word** phần cứng là một đơn vị dữ liệu quan trọng. Kích thước word xác định nhiều khía cạnh của cấu trúc và hoạt động của máy tính, từ cách máy tính truy cập bộ nhớ đến cách trình biên dịch dịch một phép toán số học C đơn lẻ thành nhiều lệnh assembly. Kiến trúc 32-bit có kích thước word là 32 bit, hay 4 byte. Kiến trúc 64-bit có kích thước word là 64 bit, hay 8 byte.

Trên hầu hết các kiến trúc hiện đại, kích thước word thường xác định (trong số những thứ khác[^word]) **địa chỉ lớn nhất có thể** và do đó kích thước của con trỏ C (xem [không gian địa chỉ](#sec-address-space)). Kiến trúc 32-bit có con trỏ 4-byte; kiến trúc 64-bit có con trỏ 8-byte. Kích thước word cũng thường xác định **đơn vị bộ nhớ nhỏ nhất có thể truy cập hoặc hiệu quả nhất**. Trên kiến trúc 32-bit, đọc và ghi bộ nhớ thường theo đơn vị 4-byte; trên kiến trúc 64-bit, theo đơn vị 8-byte.

[^word]: Kích thước word phần cứng là đơn vị truy cập tự nhiên trong máy tính và tương ứng với kích thước thanh ghi phần cứng (sẽ thảo luận trong [phần sau](#sec-reg-size)). Tương ứng, kích thước thanh ghi này xác định đơn vị bộ nhớ nhỏ nhất có thể truy cập, kích thước của địa chỉ, v.v.

Chúng ta sẽ đề cập đến word phần cứng chi tiết hơn nhiều khi học về kiến trúc tập lệnh. Hiện tại, chúng ta sử dụng khái niệm word để nhắc nhở rằng các chương trình C đã biên dịch tạo ra bố cục bộ nhớ *phụ thuộc vào kiến trúc*. Chúng ta thảo luận một vài đặc điểm phụ thuộc kiến trúc của các chương trình đã biên dịch bên dưới.

(sec-address-space)=
## Không gian địa chỉ

**Không gian địa chỉ** là phạm vi giả định của các vị trí bộ nhớ có thể đánh địa chỉ trên một máy cụ thể. Ví dụ, kiến trúc 32-bit, một con trỏ có thể đánh địa chỉ $2^{32}$ vị trí trong bộ nhớ[^in-practice]. Vì bộ nhớ được đánh địa chỉ theo byte và liền kề, kích thước không gian địa chỉ của chúng ta cho một chương trình do đó là $2^{32}$ byte (hay 4 GiB, "bốn gibi-byte". Chúng ta sẽ đề cập đến ký hiệu này sau).

[^in-practice]: Về mặt logic, không phải trong thực tế. Một số vùng bộ nhớ được bảo vệ đọc/ghi, ví dụ: truy cập bộ nhớ tại địa chỉ `0` (`NULL`) gây ra lỗi.

:::{tip} Kiểm tra nhanh

Trên kiến trúc 32-bit, `sizeof(int *)` là gì? `sizeof(char *)`?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Một con trỏ trên kiến trúc 32-bit phải đủ lớn để biểu diễn tất cả các địa chỉ có thể trong không gian địa chỉ. Không gian địa chỉ của kiến trúc 32-bit là $2^{32}$ địa chỉ byte từ `0x00000000` đến `0xFFFFFFFF`. Chúng tương ứng với các mẫu bit 32 bit, vì vậy con trỏ phải có khả năng lưu 32 bit thông tin.

Do đó tất cả con trỏ trên kiến trúc 32-bit phải rộng 4 byte[^why-not-larger], vì vậy `sizeof(int *)` là `sizeof(char *)` là `sizeof(int **)` là 4.

[^why-not-larger]: Tại sao chúng ta không làm con trỏ kiến trúc 32-bit lớn hơn 4 byte? Lý do chính là hiệu quả lưu trữ; nếu địa chỉ con trỏ sẽ không bao giờ lớn hơn 4 byte, không lãng phí byte bằng cách cấp phát thêm. Lý do phụ là quy ước; theo định nghĩa, kiến trúc 32-bit định nghĩa [kích thước word](@sec-words), xác định kích thước con trỏ.
:::

## Góc nhìn khác về không gian địa chỉ

Trước khi thảo luận ví dụ, chúng tôi muốn chia sẻ một sơ đồ bộ nhớ mà, mặc dù khó hiểu lúc đầu, sẽ cực kỳ hữu ích trong việc diễn giải bố cục bộ nhớ của bất kỳ chương trình C đã biên dịch nào.

Nhớ lại rằng bộ nhớ trên kiến trúc 32-bit được bố trí như một mảng rất dài gồm $2^{32}$ byte. Một mảng rất dài sẽ không vừa trên bất kỳ trang nào, dù theo chiều ngang hay chiều dọc. Thay vào đó, chúng ta sử dụng một hình ảnh hóa như @tab-mem-layout, hiển thị bộ nhớ như các hàng 4 byte, từ địa chỉ thấp đến cao:

* Trong bốn cột bên phải, các giá trị "xx" đề cập đến dữ liệu (giả định hoặc khác) tại mỗi trong bốn byte bộ nhớ. Bốn byte này có địa chỉ bộ nhớ liền kề.
* Cột trái cùng biểu thị địa chỉ *thấp nhất* của các byte trong hàng đó, tức là địa chỉ của byte phải cùng.
* Trong một hàng nhất định, byte phải cùng có địa chỉ thấp nhất, và byte trái cùng có địa chỉ cao nhất. Điều này được giải thích bởi các tiêu đề +3, +2, +1, và +0.

:::{table} Bộ nhớ là một mảng rất dài các byte, nhưng sơ đồ này "cuộn" mảng dài thành các hàng 4 byte.
:label: tab-mem-layout
:align: center

| địa chỉ | +3 | +2 | +1 | +0 |
| :--- | :--- | :--- | :--- | :--- |
| `0x0` | xx | xx | xx | xx |
| `0x4` | xx | xx | xx | xx |
| `...` | ... | ... | ... | ... |
| `0xFFFFFFF8` | xx | xx | xx | xx |
| `0xFFFFFFFC` | xx | xx | xx | xx |

:::

:::{note} Ví dụ địa chỉ byte
:class: dropdown

* "xx" phía trên bên phải ở địa chỉ `0x0000000`, hay `0b0000 0000 ... 0000 0000`. Đây là địa chỉ thấp nhất có thể trong không gian địa chỉ 32-bit.
* "xx" phía trên bên trái ở địa chỉ `0x00000003`, hay `0b0000 0000 ... 0000 0011`.
* "xx" phía dưới bên phải ở địa chỉ `0xFFFFFFC`, hay `0b1111 1111 ... 1111 1100`.
* "xx" phía dưới bên trái ở địa chỉ `0xFFFFFFFF`, hay `0b1111 1111 ... 1111 1111`. Đây là địa chỉ cao nhất có thể trong không gian địa chỉ 32-bit.

:::

Một thuộc tính đáng chú ý của hình ảnh hóa này là các địa chỉ trong cột trái là bội số của 4. Bố cục này hiệu quả căn chỉnh bố cục bộ nhớ của chúng ta theo **word**, vì kiến trúc 32-bit có word 4-byte.

Một thuộc tính khó hiểu của hình ảnh hóa này là các địa chỉ "thấp hơn" ở các hàng trước, trong khi các địa chỉ "cao hơn" ở các hàng sau. Điều này không tốt cho những người trong chúng ta đánh giá cao các quy ước đặt tên có ý nghĩa. Tuy nhiên, khi hiển thị phạm vi lớn của bộ nhớ sử dụng các debugger như `gdb`, đầu ra dòng lệnh thường hiển thị dữ liệu bắt đầu từ địa chỉ thấp hơn trước, giống như trong hình ảnh hóa này.

## Ví dụ chương trình đã biên dịch

Giả sử rằng chương trình C sau được biên dịch trên kiến trúc 32-bit và tạo ra bố cục bộ nhớ trong @tab-word-program.

(word-program)=
```c
#include <stdio.h>
#include <stdint.h>

int main(int argc, char *argv[]) {
  int32_t value = 0x12345678;
  char str1[] = "hi!";
  char str2[] = "cs61c";
  int16_t short_val = 0xaabb;
  …

  return 0;
}
```

:::{table} Bố cục dữ liệu của chương trình trên máy little endian 32-bit.
:label: tab-word-program
:align: center

| địa chỉ | +3 | +2 | +1 | +0 |
| :--- | :--- | :--- | :--- | :--- |
| `0x0` | xx | xx | xx | xx |
| `0x4` | xx | xx | xx | xx |
| `...` | ... | ... | ... | ... |
| `0x7F...FE164` | `0xaa` | `0xbb` | xx | xx |
| `0x7F...FE168` | `0x12` | `0x34` | `0x56` | `0x78` |
| `0x7F...FE16C` | `'i'` | `'h'` | xx | xx |
| `0x7F...FE170` | `'s'` | `'c'` | `'\0'` | `'!'` |
| `0x7F...FE174` | `'\0'` | `'c'` | `'1'` | `'6'` |
| `0x7F...FE178` | xx | xx | xx | xx |
| `...` | ... | ... | ... | ... |
| `0xFFFFFFFC` | xx | xx | xx | xx |

:::

:::{tip} Kiểm tra nhanh

Xác định địa chỉ của `value` và `str1`.
:::

:::{note} Hiển thị đáp án
:class: dropdown

Nhớ lại rằng địa chỉ của một giá trị được lưu là địa chỉ **thấp nhất** trong số các byte của giá trị đó.

* `value` (số nguyên có dấu 32-bit 305419896 trong hệ thập phân) có bốn byte: `0x12`, `0x34`, `0x56`, và `0x78`. Địa chỉ thấp nhất của bốn byte này là địa chỉ của byte `0x78`, là `0x7FFFE168`.

* `str1` là chuỗi C `"hi!"` có bốn byte bao gồm null terminator: `'h'`, `'i'`, `'!'`, `'\0'`. Địa chỉ thấp nhất của các byte này là địa chỉ của `'h'`, là `0x7FFFE16C` + 2, hay `0x7FFFE16E`.

:::

Có hai khía cạnh phụ thuộc kiến trúc của bố cục bộ nhớ này:

1. Các byte của `value` dường như được lưu theo thứ tự "đảo ngược". Byte ít quan trọng nhất, `0x78`, có địa chỉ thấp nhất!
1. `value` có kích thước word có địa chỉ `0x7FFFE168`, là bội số của bốn.

Cùng nhau, hai quan sát này cho chúng ta biết rằng kiến trúc là **little endian**, và các số nguyên 32-bit (và có lẽ các giá trị có kích thước word khác) được **căn chỉnh theo word**.

## Endianness

Khi dữ liệu chiếm nhiều byte liền kề trong bộ nhớ, máy tính phải xác định byte nào được lưu tại địa chỉ thấp nhất. Quyết định này thường được thông tin bởi kiến trúc phần cứng và thứ tự các byte được đọc từ bộ nhớ.

Thuộc tính này được gọi là **endianness**.[^gulliver] Đối với một word nhất định:

* Máy **Little endian** lưu byte *ít* quan trọng nhất đầu tiên, tại địa chỉ thấp nhất của word.
* Máy **Big endian** lưu byte *quan trọng nhất* đầu tiên, tại địa chỉ thấp nhất của word.

Việc chọn endianness là một quy ước[^endianness]. Hầu như tất cả các kiến trúc máy tính hiện đại là little endian.

:::{tip} Kiểm tra nhanh

Sơ đồ trong @tab-word-program giúp chúng ta đọc các số nguyên 32-bit như thế nào?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Kiến trúc 32-bit là **little endian**. Chúng ta biết điều này vì số nguyên `0x12345678` có byte ít quan trọng nhất, `0x78` được lưu đầu tiên tại địa chỉ thấp nhất.

Trong @tab-word-program, các cột dữ liệu được liệt kê theo thứ tự đảo ngược: +3, +2, +1, +0. Điều này giúp chúng ta đọc các số theo **định dạng con người**, ví dụ: `0x12` ở bên trái, và `0x78` ở bên phải.

:::

Đọc thêm về endianness trên [Wikipedia](https://en.wikipedia.org/wiki/Endianness).

[^gulliver]: Thuật ngữ "little-endian" và "big-endian" bắt nguồn từ _Gulliver's Travels_ (1726) của Jonathan Swift và được đặt ra bởi Danny Cohen. Đọc Internet Experiment Note [Holy Wars and a Plea for Peace](https://www.rfc-editor.org/ien/ien137.txt) để biết thêm thông tin.

[^endianness]: Endianness cũng có thể đề cập đến thứ tự các byte được truyền qua mạng và các phương tiện truyền thông dữ liệu khác; hầu hết các mạng internet hiện đại ưu tiên big endian. Xem thảo luận tương đối thú vị trên [Reddit](https://www.reddit.com/r/learnprogramming/comments/1emdohb/can_someone_explain_to_me_why_therere_big_endian/).

### Chạy Demo

Các hướng dẫn bên dưới chủ yếu để tham khảo. Chúng tôi đề nghị bạn làm Lab 02 trước để có một số kinh nghiệm với `gdb`. Lưu ý rằng để kết nối `gdb` với file nguồn, bạn sẽ cần chạy lệnh `make` bên dưới, biên dịch chương trình với các symbol debug.

:::{note} Các lệnh gdb demo
:class: dropdown

```bash
$ make clean
$ gdb endianness
(gdb) b 9 # đặt breakpoint tại dòng 9
(gdb) r     # chạy, khởi tạo tất cả các biến
(gdb) p/x str  # xem các byte chuỗi
(gdb) p/x str2
(gdb) # in một word, hiển thị dạng hex
(gdb) x/1wx 0x7fffffffe164
(gdb) <enter> # lặp lại hành động cuối
(gdb) … # tiếp tục nhấn <enter>
(gdb) # LSB tại địa chỉ/word thấp nhất
(gdb) # in 4 byte, hiển thị dạng hex
(gdb) x/4bx 0x7fffffffe164
(gdb) <enter> # lặp lại hành động cuối
(gdb) … # tiếp tục nhấn <enter>
(gdb) q
```
:::

## Căn chỉnh

### Căn chỉnh Word

Một phép toán quan trọng mà word phần cứng định nghĩa là truy cập bộ nhớ. Như chúng ta sẽ thấy, nhiều kiến trúc được tối ưu hóa cho truy cập bộ nhớ *căn chỉnh theo word*. Điều này có nghĩa là truy cập toàn bộ word rất nhanh khi word đó nằm tại địa chỉ bộ nhớ là bội số của kích thước word. Đối với kiến trúc 32-bit, điều này có nghĩa là đọc 4 byte, trong đó byte đầu tiên nằm trên ranh giới 4-byte.

Trong bốn biến trong @word-program, chỉ `value` có kích thước của một word. Do đó trình biên dịch đã căn chỉnh `value` theo ranh giới word. Các biến khác `str1`, `str2`, và `short_val` không có giá trị có kích thước word và do đó không có ràng buộc như vậy.

### Căn chỉnh Struct

Hãy xem lại ý tưởng về `struct` và xem xét mỗi `struct` đã khai báo chiếm bao nhiêu không gian.

:::{card} Căn chỉnh cấu trúc dữ liệu
Từ [Wikipedia](https://en.wikipedia.org/wiki/Data_structure_alignment):
^^^
Căn chỉnh cấu trúc dữ liệu là cách dữ liệu được sắp xếp và truy cập trong bộ nhớ máy tính. Nó bao gồm ba vấn đề riêng biệt nhưng liên quan: căn chỉnh dữ liệu, padding cấu trúc dữ liệu, và packing.

_Căn chỉnh dữ liệu_ là việc căn chỉnh các phần tử theo căn chỉnh tự nhiên của chúng. Để đảm bảo căn chỉnh tự nhiên, có thể cần chèn một số _padding_ giữa các phần tử cấu trúc hoặc sau phần tử cuối cùng của cấu trúc. Ví dụ, trên máy 32-bit, một cấu trúc dữ liệu chứa giá trị 16-bit theo sau là giá trị 32-bit có thể có 16 bit padding giữa giá trị 16-bit và giá trị 32-bit để căn chỉnh giá trị 32-bit trên ranh giới 32-bit. Ngoài ra, người ta có thể _pack_ cấu trúc, bỏ qua padding, có thể dẫn đến truy cập chậm hơn, nhưng tiết kiệm 16 bit bộ nhớ.
:::

Xem xét struct `foo`:

```c
struct foo {
    int32_t a;
    char b;
    struct foo *c;
}
```

Tự bản thân, một số nguyên 32-bit, một ký tự, và một con trỏ struct chiếm 9 byte. Tuy nhiên, khi được khai báo cùng nhau như một struct, trình biên dịch C thường có thể chọn đưa **padding** vào chính struct để căn chỉnh các thành viên của struct. Padding một struct cho phép các phép toán trên các thành viên của nó tận dụng cùng các tăng tốc từ căn chỉnh word như thể các thành viên được khai báo riêng biệt.

:::{table} Struct có thể đưa padding byte vào. Trên kiến trúc 32-bit bên dưới, `sizeof(struct foo)` là 12.
:label: tab-struct
:align: center

| +3 | +2 | +1 | +0 |
| :--- | :--- | :--- | :--- |
| AA | AA | AA | AA |
| xx | xx | xx | BB |
| CC | CC | CC | CC |

:::

:::{note} Giải thích
:class: dropdown

Giả sử chúng ta khai báo `struct foo s;` và biên dịch chương trình trên kiến trúc 32-bit. Chúng ta có thể thấy @tab-struct như trên.

- AA biểu thị bốn byte bị chiếm bởi `s.a`. `sizeof(s.a)` là 4.
- BB biểu thị một byte bị chiếm bởi `s.b`. `sizeof(s.b)` là 1. Căn chỉnh chính xác của `s.b` là cụ thể theo triển khai.
- CC biểu thị bốn byte bị chiếm bởi `s.c`. `sizeof(s.c)` là 4.

:::

Cuối cùng, khai báo `struct` là một hướng dẫn về cách sắp xếp một loạt byte trong một thùng chứa. Kích thước chính xác của một struct — và thứ tự trường trong một kiểu struct — phụ thuộc vào trình biên dịch C và liệu nó có tối ưu hóa cho padding hay packing không. Chúng tôi khuyến nghị bạn luôn kiểm tra kích thước với debugger như `gdb`.
