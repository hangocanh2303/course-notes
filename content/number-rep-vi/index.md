---
title: "Giới thiệu"
subtitle: "Làm thế nào để biểu diễn dữ liệu số?"
---

## Mục tiêu học tập

* Biết các thuật ngữ: bits, bytes, bitstrings
* Tính được cần bao nhiêu bit để biểu diễn $k$ vật thể.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/mGgOK9ShS6g?si=OJZv82-TwMgnC-KY
:width: 100%
:title: "[CS61C FA20] Lecture 02.0 - Number Representation: Intro, Bits can be anything"
:::

::::

## Dữ liệu số

Dữ liệu tồn tại xung quanh chúng ta.

### Ví dụ: Lưu trữ dữ liệu dưới dạng số

Thế giới thực là analog—mọi thứ bạn nghe, thấy và ngửi đều là analog. Ví dụ, số thực là cách tuyệt vời để biểu diễn thế giới, nhưng để sử dụng máy tính làm việc với những con số này, chúng ta thường cần chuyển đổi hoặc tìm các số tương đương có thể biểu diễn được dưới dạng số.

:::{figure} images/a2d-signal.png
:label: fig-signal
:width: 50%
:alt: "Bốn biểu đồ đường liên tiếp minh họa quá trình chuyển đổi sóng analog gốc thành biểu diễn số thông qua rời rạc hóa thời gian và lượng tử hóa biên độ. Các sơ đồ minh họa cách tín hiệu liên tục được lấy mẫu tại các khoảng thời gian cụ thể và ánh xạ thành các giá trị rời rạc để tạo ra tập dữ liệu số cuối cùng."

Chuyển đổi tín hiệu analog sang biểu diễn số.
:::

Để chuyển đổi dữ liệu analog sang dữ liệu số, chúng ta phải làm hai việc:

1. **Lấy mẫu (Sample)**: Chúng ta hỏi tín hiệu tại mỗi bước thời gian: "Giá trị của bạn là gì?" Điều này thường xảy ra theo khoảng đều đặn. Ví dụ, với nhạc trên CD, chúng ta hỏi 44.100 lần mỗi giây xem độ cao của nó là bao nhiêu.
2. **Lượng tử hóa (Quantize)**: Vì độ cao có thể ra một số lẻ nào đó, chúng ta cần chia biên độ bằng một "thước đo." Chúng ta chia thành số 16-bit, tức là $2^{16} = 65.536$ vạch có thể. Sau đó, mẫu "bắt" vào vạch gần nhất.

Khi hoàn thành, chúng ta có một tập các mẫu 16-bit để làm việc. Có *rất nhiều* kỹ thuật đi vào quá trình này. Trong các lớp khác, bạn sẽ học cách lấy mẫu tín hiệu, xây dựng bộ chuyển đổi analog-sang-số, và nhiều hơn nữa. Trong lớp này, chúng ta tập trung vào thiết kế hệ thống để biểu diễn số thực với số bit hữu hạn.

### Ví dụ: Dữ liệu số thuần túy

Không phải tất cả dữ liệu số đều nhất thiết là analog nhàm chán; đôi khi bạn có thể tạo nghệ thuật, âm nhạc, hoặc video hoàn toàn không có tham chiếu analog nào. Ví dụ, phần mềm [POV-Ray](https://www.povray.org/) là phần mềm render tạo ra những hình ảnh số đẹp mà chỉ tồn tại trong đầu của nghệ sĩ. Ngày nay, có cả những lĩnh vực trí tuệ nhân tạo về tạo hình ảnh và video số, thường hoàn toàn từ các nguồn dữ liệu số.

:::::{tab-set}

::::{tab-item}   The Last Guardian, Johnny Yip

  :::{image} images/guardian.jpg
  :label: fig-art1
  :alt: "Một minh họa số mô tả cảnh dưới nước với một loài bò sát biển lớn bơi qua xác tàu đắm trên rạn san hô. Các tia nắng mặt trời xuyên qua nước biển xanh đậm, chiếu sáng một đàn cá khổng lồ phía trên xác tàu."
  :width: 40%

  :::

::::

::::{tab-item}  My First CGSphere, Robert McGregor

  :::{image} images/rwmcgsphere2_final.jpg
  :label: fig-art2
  :alt: "Một hình ảnh render số minh họa quả cầu kim loại vàng có các lỗ tròn lồng trong một lưới lớn hơn màu hổ phách trong suốt. Cấu trúc được đặt trên bề mặt gạch phản chiếu trong môi trường lưới cong màu xám và được bao quanh bởi các làn hơi nước trắng."
  :width: 40%

  :::

::::

:::::

## Bits, Bytes, và Nibbles

Một **bit** là một *chữ số nhị phân*. Nó có giá trị `0` hoặc `1`. 
Chúng ta dùng các cụm từ **chuỗi nhị phân**, **bitstring**, **dãy bit**, v.v. để chỉ các dãy chữ số nhị phân. Ví dụ, tập các chuỗi nhị phân độ dài 4 chỉ $2^4 = 16$ bitstrings `0000`, `0001`, `0010`, ..., `1111`.

Một **byte** là một bitstring độ dài 8. Chúng ta sẽ thấy rằng việc có một nhóm bit chuẩn là hữu ích, để các nhóm bit có thể biểu diễn nhiều thông tin hơn. Một byte có thể biểu diễn $2^8 = 256$ thứ.

Làm sao chúng ta nên thảo luận về bytes một cách thông thường? Thay vì luôn viết ra tám bit (và phải nói, "không không một không một một một một" cho `00101111`), chúng ta có thể viết hai chữ số thập lục phân để viết tắt (và chỉ cần nói `2F`). Đọc [phần tiếp theo](#bin-dec-hex) để học cách chuyển đổi giữa giá trị thập lục phân và nhị phân, và tại sao có cách viết tắt thập lục phân là hữu ích.

Nếu bạn tò mò, 4 bits được gọi là "nibble" (hoặc "nybble") và có thể biểu diễn $2^4 = 16$ thứ. Điều này tương đương với một chữ số thập lục phân.

## Ý TƯỞNG LỚN: Bits có thể biểu diễn bất cứ thứ gì!

Ý tưởng lớn trong bài giảng đầu tiên này là:

> Bits có thể biểu diễn _bất cứ thứ gì_.

**Giá trị logic**: Thông thường, `0` là false và `1` là true.

**Ký tự**: Chúng ta có 26 ký tự (A-Z). Nếu dùng 5 bits, $2^5 = 32$, nên chúng ta có thể có một mẫu bit cho mỗi ký tự, với sáu bit còn lại cho thông tin khác.

* Chuẩn [ASCII](https://en.wikipedia.org/wiki/ASCII) là biểu diễn 8-bit mở rộng có thể biểu diễn chữ hoa, chữ thường, và dấu câu được sử dụng trong tiếng Anh Mỹ chuẩn.
* Chuẩn [Unicode](http://www.unicode.com) biểu diễn tất cả ký hiệu và ngôn ngữ trên thế giới, bao gồm emoji. Có các phiên bản 8-bit, 16-bit, và 32-bit của Unicode.

**Màu sắc**: Mã màu HTML là biểu diễn 24-bit (3-byte). @fig-hex-colors cho thấy mã màu HTML cho [California Gold](https://brand.berkeley.edu/visual-identity/colors/), `0xFDB515`. Bạn sẽ đọc thêm về thập lục phân và nhị phân trong [phần tiếp theo](#bin-dec-hex).

:::{figure} images/hex-colors.png
:label: fig-hex-colors
:width: 70%
:alt: "Một sơ đồ với đường trừu tượng màu vàng nằm ngang tách từ Numeral ở trên khỏi từ Number ở dưới. Bố cục trực quan này củng cố chú thích bằng cách đặt các chữ số như biểu diễn ký hiệu phía trên đường và số như khái niệm trừu tượng cơ bản phía dưới nó."
:align: center

Mã màu HTML
:::

:::{tip} Giải thích mã màu
:class: dropdown

Xem lại giải thích này sau khi bạn đọc thêm về thập lục phân và nhị phân trong [phần tiếp theo](#bin-dec-hex). Bạn có thể dùng ví dụ này làm bài tập chuyển đổi giữa thập lục phân, nhị phân, và thập phân.

* `FDB515` là viết tắt thập lục phân (được biểu thị bằng tiền tố `0x`) cho bitstring `0b111111011011010100010101` (được biểu thị bằng tiền tố `0b`). Chúng ta chèn khoảng trắng bên dưới để dễ đọc, nhóm bit theo nibbles:

  `1111 1101 1011 0101 0001 0101`

* 32 bits này sau đó được nhóm lại thành ba nhóm tám bit để biểu diễn lượng đỏ, xanh lá, và xanh dương, tương ứng, trong California Gold. Các giá trị _RGB_ này mỗi cái trên thang từ 0 đến 255.
  * Đỏ: Byte "ngoài cùng bên trái", `0xFD` hoặc `0b11111101` hoặc 253.
  * Xanh lá: Byte "ở giữa", `0xB5` hoặc `0b10110101` hoặc 181.
  * Xanh dương: Byte "ngoài cùng bên phải", `0x15` hoặc `0b00010101` hoặc 21.
  
:::

**Vị trí/Địa chỉ**: IPv4 và IPv6 là biểu diễn 32-bit và 64-bit của địa chỉ thiết bị trên Internet, còn được gọi là địa chỉ Internet Protocol. Đọc thêm về [Địa chỉ IP](https://en.wikipedia.org/wiki/IP_address) nếu bạn tò mò.

**Nhiều loại dữ liệu** Bạn thậm chí có thể biểu diễn cảm xúc, như "vui" là `00` hoặc "cáu" là `01`. Chúng ta lưu ý rằng biểu diễn 2-bit có lẽ không đủ để biểu diễn phạm vi đa dạng của cảm xúc con người. Thực tế, các nỗ lực lượng tử hóa cảm xúc con người (thường nhằm mục đích xử lý dữ liệu qua máy tính) là một lĩnh vực nghiên cứu lớn. Các hàm ý của việc sử dụng máy tính để lấy mẫu và rời rạc hóa trải nghiệm con người là gì? Để biết thêm, chúng tôi khuyên bạn xem các khóa học kỹ thuật xã hội khám phá bối cảnh con người và đạo đức của dữ liệu.

## Bất cứ thứ gì bạn có thể liệt kê, bạn có thể số hóa

Ý tưởng lớn của bài giảng này cần ghi nhớ:

> Với N bits, bạn có thể biểu diễn tối đa $2^N$ thứ.

Nói cách khác, bạn có thể biểu diễn $k$ thứ với tối thiểu $N$ bits, trong đó $N = \lceil \log_2 k \rceil$.

:::{card}
**Cần bao nhiêu bits để biểu diễn chữ cái viết thường trong tiếng Anh?**
^^^
Có 26 chữ cái viết thường trong tiếng Anh: `a`, `b`, ..., `z`.

$\log_2 (26) = \log_{10}(26)/\log_{10}(2) \approx 4.7$

Do đó chúng ta cần tối thiểu **5 bits**.

_Kiểm tra lại_: 5 bits biểu diễn $2^5 = 32$ thứ, nên chúng ta chắc chắn có thể biểu diễn 26 chữ cái (và sáu thứ khác, nếu bạn muốn). 32 là lũy thừa nhỏ nhất của 2 lớn hơn số thứ chúng ta muốn lưu trữ.
:::

:::{tip} Kiểm tra nhanh
Có bao nhiêu "thứ" có thể được biểu diễn bằng 4 bits?

* 4
* 8
* 16
* 64
* Thứ gì khác
:::

:::{note} Đáp án
:class: dropdown

$2^4 = 16$
:::

:::{tip} Kiểm tra nhanh
Cần bao nhiêu bits để biểu diễn $\pi$ (pi)?

* 1
* 9 ($\pi=3.14$, nên `0.011 "." 001100`)
* 64 (Macs là máy 64-bit)
* Mọi bit máy có
* $\infty$

:::

:::{note} Đáp án
:class: dropdown

Câu hỏi mẹo (xin lỗi). Chúng ta dùng bits để biểu diễn *tập hợp* các thứ, không chỉ một thứ đơn lẻ. Tất cả đáp án đều có thể, tùy thuộc vào bạn đang muốn biểu diễn bao nhiêu thứ ngoài $\pi$.

Để dùng 1 bit, xem xét biểu diễn hai thứ:

* $\pi$
* không phải $\pi$
:::
