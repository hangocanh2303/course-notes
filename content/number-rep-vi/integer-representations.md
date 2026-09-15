---
title: "Các Biểu diễn Số nguyên"
---

(sec-integer-reps)=
## Mục tiêu học tập

* Hiểu các đánh đổi giữa các biểu diễn số nguyên:
  * Số nguyên (unsigned)
  * Sign-Magnitude (có dấu)
  * Ones' Complement (có dấu)
  * Two's Complement (có dấu)
  * Bias Encoding
* Chuyển đổi giữa số thập phân và các biểu diễn số nguyên khác
* Xác định khi nào và tại sao integer overflow xảy ra
* Thực hiện các phép toán nhị phân đơn giản như cộng


::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/I6USbMvf7vg?si=5QGKCqMvrc4TZLSG
:width: 100%
:title: "[CS61C FA20] Lecture 02.3 - Number Representation: Overflow, Sign and Magnitude, One's Complement"
:::

::::

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/opFCs4m7pW8?si=u3uolJHtdKzZRsBB
:width: 100%
:title: "[CS61C FA20] Lecture 02.4 - Number Representation: Two's Complement"
:::

::::

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/KC_QTRW0dY0?si=qbTGX8iqN5FRjXDz
:width: 100%
:title: "[CS61C FA26] Lecture 02 - Number Representation: Bias Encoding"
:::

::::

## Giới thiệu

Trong lịch sử, máy tính được sử dụng như máy tính khoa học. Do đó chúng ta dành phần lớn thời gian để hiểu cách một chuỗi bits (một **bit string**) có thể biểu diễn một số.

> Làm thế nào chúng ta có thể sử dụng $N$ bits để biểu diễn một tập các số nguyên?

Có nhiều hệ thống chúng ta có thể sử dụng. Không phải tất cả các hệ thống đều giúp chúng ta biểu diễn $2^N$ số nguyên duy nhất với $N$ bits! Chúng ta sẽ tập trung vào biểu diễn các số nguyên *có dấu* và *không dấu*.

* Số nguyên có dấu chỉ các số nguyên dương, số nguyên âm, và số không.
* Số nguyên không dấu chỉ các số nguyên không âm, tức là, không hoặc lớn hơn.

## Biểu diễn Số nguyên Không dấu $N$-bit

Hãy giải quyết biểu diễn số nguyên không dấu chuẩn của chúng ta trước. Sơ đồ toán học đã thảo luận [trước đó](#bin-dec-hex) là đủ để biểu diễn $2^N$ **số nguyên không dấu** với $N$ bits.

* `0b0...0` ($N$ số không) biểu diễn số không
* `0b1...1` ($N$ số một) biểu diễn $2^N - 1$ (tại sao?)
* Mọi thứ khác: Giả sử bitstring là biểu diễn cơ số 2 của một số. Chuyển đổi.

Biểu diễn này được hỗ trợ trong C (thảo luận thêm sau). Các kiểu dữ liệu tích hợp như `unsigned int` có thể gây nhập nhằng vì nó không chỉ định độ rộng của `int`. Header `inttypes.h` cung cấp các typedefs như `uint8_t`, `uint16_t`, `uint32_t`, v.v. để chỉ định các biểu diễn số nguyên không dấu 8-bit, 16-bit, 32-bit, v.v.

:::{caution} Cần bao nhiêu bits cho hệ thống hỗ trợ $10 + 7$?

* 10 trong nhị phân là $1010_2$, có thể lưu như số nguyên không dấu 4-bit: `1010`
* 7 trong nhị phân là $111_2$, cũng có thể lưu như số nguyên không dấu 4-bit với đệm không: `0111`.
* 17 trong nhị phân là $10001_2$, tối thiểu cần **5 bits** lưu trữ.

Nếu chúng ta dùng biểu diễn số nguyên không dấu 4-bit, chúng ta sẽ không có đủ chỗ để biểu diễn số 17. Thay vào đó, "đồng hồ đo nhị phân" của chúng ta sẽ cắt kết quả, cắt bỏ `1` ngoài cùng bên trái và lưu `0001`. Vì vậy phép cộng nhị phân với số nguyên không dấu 4-bit sẽ ngụ ý rằng $10 + 7 = 1$...?!

Đây là khái niệm **integer overflow** ([xem thêm sau](#sec-integer-overflow)).
:::

## Các Cân nhắc Thiết kế

Chúng ta sẽ ưu tiên một số hệ thống hơn các hệ thống khác, tùy thuộc vào loại phép toán số nguyên nào chúng ta muốn hỗ trợ.

### Các phép toán số thông thường

Trong thập phân, chúng ta thích cộng, trừ, nhân, chia, và so sánh số. Vì vậy chúng ta phải có khả năng làm điều tương tự với biểu diễn bit của số.

Hóa ra nhiều phép toán số học chuyển đổi khá hợp lý giữa thập phân và nhị phân.

Ví dụ, hãy cộng $10$ và $7$, là `1010` và `0111`, tương ứng, trong nhị phân. Kết quả là số 17, hoặc `10001` trong nhị phân. 

$$
\begin{array}{rrl}
  & \texttt{ 11  }\hspace*{0.5em} & \text{carry bits} \\
  & \texttt{ 1010} \\
+ & \texttt{ 0111} \\
\hline
  & \texttt{10001} &
\end{array}
$$

:::{note} Giải thích
:class: dropdown

Làm việc từ phải sang trái:

1. `0+1=1`
1. `1+1=0` nhớ `1`
1. `1+0+1=0` nhớ `1`
1. `1+0+1=0` nhớ `1`
1. `1`

(Kiểm tra lại rằng toán nhị phân khớp với toán thập phân: $10 + 7 = 17$)
:::

Phép trừ trong nhị phân hoạt động tương tự. Chúng tôi để thảo luận chuyên sâu về triển khai so sánh nhị phân (ví dụ, $X < Y$) cho một dự án trong tương lai.


### Phép tương tự Đồng hồ đo

Về lý thuyết, số có vô số chữ số (thường là các số không đứng đầu), nhưng trong tính toán, chúng ta phải phân bổ một số _hữu hạn_ bits. Phần cứng có giới hạn! Các mẫu bit nhị phân do đó là **trừu tượng**: chúng chỉ đơn giản là đại diện của số. 

Trong việc chọn biểu diễn số nguyên, chúng ta phải xem xét liệu các phép toán được hỗ trợ sẽ vẫn hoạt động trong **độ rộng bit** cho trước của biểu diễn số nguyên. Một phép tương tự hữu ích để xem xét các trường hợp biên đến từ ý tưởng về đồng hồ đo:

:::{figure} images/odometer.jpg
:label: fig-odometer
:alt: "Ảnh cận cảnh đồng hồ đo cơ khí trên xe hiển thị số đọc 999999 ngay trước khi nó quay vòng. Hình ảnh minh họa một thiết bị đếm hữu hạn sẽ quay lại 000000 sau một lần tích tiếp theo."
:width: 70%

Đồng hồ đo ô tô đo quãng đường. Nó bắt đầu từ 0 và từ từ tăng lên, sau đó quay vòng lại. Tại một thời điểm nào đó, đồng hồ đo ở trên sẽ đạt 999999; số tiếp theo lại là 0.
:::

Hai tập giá trị cần xem xét:

* Các giá trị nhị phân `000...000`, `000...001`, `000...010`, ..., `111...111`
* Các số nguyên tăng thêm một, khi được đặt trên trục số

Như chúng ta sẽ thấy, có những hệ thống trong đó "hướng" của các giá trị này có thể khác nhau.

(sec-integer-overflow)=
### Integer Overflow

> *Integer overflow*: Kết quả số học nằm ngoài phạm vi có thể biểu diễn của số nguyên.

Giả sử chúng ta dùng $N$ bits để biểu diễn số nguyên trong phần cứng. Nếu kết quả của một phép toán số nguyên ($+, -, \times, \div, <, =, \leq$, v.v.) không thể được biểu diễn chính xác trong $N$ bits, chúng ta nói rằng *integer overflow* đã xảy ra.

:::{figure} images/overflow.png
:label: fig-overflow
:alt: "Một đường thẳng nằm ngang màu xanh được đánh dấu với các giá trị nhị phân 4-bit từ 0000 đến 1111 minh họa một hệ thống số hữu hạn. Một đường cong màu vàng nối giá trị tối đa trở lại giá trị tối thiểu để trực quan biểu diễn khái niệm arithmetic overflow trong các hệ thống số."
:align: center

Với số nguyên không dấu, "đồng hồ đo nhị phân" quay vòng.
:::

Số nguyên không dấu 4-bit: Sử dụng 4 bits, bạn có thể biểu diễn 0 đến 15.

* *Positive Overflow*: Nếu bạn đang ở 15 (`0b1111`) và cộng 1, giá trị quay vòng về 0 (`0b0000`).
* *Negative Overflow*: Nếu bạn đang ở 0 (`0b0000`) và trừ 1, nó quay vòng về 15 (`0b1111`).

:::{caution} Không có thứ gọi là integer underflow

Mọi người thường nhầm gọi negative overflow là "underflow," nhưng underflow là một khái niệm khác chúng ta sẽ thảo luận sau khi chúng ta xem xét biểu diễn phân số.
:::

### Các Biểu diễn Số nguyên Có dấu $N$-bit

Làm thế nào bạn biểu diễn số âm? Tổng quát hơn, làm thế nào bạn biểu diễn _cả_ số dương _và_ số âm (và _không_) với cùng $N$ bits?

Ngoại lề: Có một vị vua yêu cầu các nhà tư tưởng thông thái của ông dạy ông kinh tế học. Họ cứ mang đến cho ông những cuốn sách dài, và ông cứ gửi họ đi để làm ngắn hơn. Cuối cùng, họ quay lại và nói, "Thưa Bệ hạ, chúng tôi có lý thuyết kinh tế học trong bốn từ: '[Ain't No Free Lunch](http://en.wikipedia.org/wiki/No_such_thing_as_a_free_lunch).' " Nó có nghĩa là bạn không thể có thứ gì mà không đánh đổi.

Nếu chúng ta muốn biểu diễn *số âm*, bạn phải từ bỏ thứ gì đó; bạn mất một số số dương mà bạn từng có. Nếu chúng ta mượn một bit, chúng ta không thể đi cao bằng trong phạm vi dương, nhưng bây giờ chúng ta có thể làm số âm.

Tiếp theo, chúng ta thảo luận một vài cách hợp lý và xem xét các đánh đổi. Trong [phần tiếp theo](#sec-twos-complement), chúng ta sẽ tiết lộ biểu diễn chuẩn được sử dụng trong các kiến trúc hiện đại và được hỗ trợ bởi chuẩn C23.

## Sign-Magnitude

> * Bit **sign** ngoài cùng bên trái: **dấu** của số nguyên. `0` là dương; `1` là âm.
> * Các bit còn lại: **độ lớn** của số nguyên trong nhị phân

:::{card}
Sign-Magnitude 4-bit
^^^

* Số dương: $1$ (`0b0001`) đến $7$ (`0b0111`)
* Số âm: $-1$ (`0b1001`) đến $-7$ (`0b1111`)
* Hai số không: $+0$ (`0b0000`) và $-0$ (`0b1000`)

:::

Chúng ta đã thấy một vấn đề! Với số không dương và số không âm, chúng ta phải kiểm tra hai mẫu khác nhau trong code, mỗi khi chúng ta muốn so sánh giá trị với không.

:::{figure} images/sign-magnitude-two-zeros.png
:label: fig-sign-magnitude-two-zeros
:width: 50%
:alt: "Hai phương trình hiển thị các giá trị thập lục phân 0x00000000 và 0x80000000 bằng với số không dương và số không âm, tương ứng. Các đường cong ánh xạ các chữ số thập lục phân sang mở rộng nhị phân, minh họa rằng bit đứng đầu xác định dấu trong khi các bit còn lại biểu diễn độ lớn của số không."
:align: center
Sign-Magnitude có hai biểu diễn cho số không: "số không dương" và "số không âm."
:::

Hãy xem xét một vấn đề tinh tế hơn, được tiết lộ qua đồng hồ đo nhị phân:

:::{figure} images/sign-magnitude-number-line.png
:label: fig-sign-magnitude-number-line
:width: 100%
:alt: "Một trục số nằm ngang màu xanh hiển thị các giá trị nhị phân 4-bit để minh họa biểu diễn sign-magnitude, với 0000 ở trung tâm. Hai mũi tên màu vàng chỉ theo hướng ngược nhau từ trung tâm để chỉ ra cách giá trị tăng về độ lớn cho cả dãy nhị phân dương và âm."
:align: center

"Đồng hồ đo nhị phân" cho sign-magnitude 4-bit.

:::

Vấn đề với Sign and Magnitude là khi đồng hồ đo đi lên, nó đi sai hướng: bạn đi dương, dương, và sau đó đột nhiên bạn rơi vào phạm vi âm. Tăng đồng hồ đo nhị phân từ `0000` đến `1111` bắt đầu từ $0$, rồi $1$, đến $7$, rồi quay về $0$ một lần nữa, rồi $-1$, rồi $-7$. Nói cách khác, đôi khi cộng số nguyên tương ứng với cộng bits, và đôi khi cộng số nguyên tương ứng với trừ bits. Điều này sẽ trở nên phức tạp rất nhanh!

:::{note} Giải thích thêm
:class: dropdown

Xem xét cộng $5+(-5)$ với số nguyên sign-magnitude 4-bit.

* $+5$: `0101`
* $-5$: `1101`

Nếu cả hai số có cùng dấu, chúng ta giữ dấu và thực hiện cộng trên các độ lớn, tính toán overflow khi cần. Trong trường hợp này, các số có dấu khác nhau, nên chúng ta phải thực hiện trừ trên các độ lớn. Phép cộng số học có điều kiện trong biểu diễn này–nó có thể là cộng nhị phân hoặc trừ–và mạch sẽ phức tạp hơn.
:::

Cuối cùng, Sign-Magnitude được coi là cách tiếp cận straw man[^strawman] để hỗ trợ tính toán mục đích chung với số nguyên. Tuy nhiên, nó có một số ứng dụng hợp lý trong, chẳng hạn, xử lý tín hiệu, nơi người dùng thường muốn tách dấu khỏi độ lớn, ít hơn là cộng số với nhau. Hỏi chúng tôi để biết thêm.

[^strawman]: Wikipedia: [Straw Man](https://en.wikipedia.org/wiki/Straw_man)

## Ones' Complement

> Để biểu diễn số âm, bù các bit của biểu diễn dương của nó.

Ở đây, "bù" có nghĩa là nếu bit là `0` thì đổi thành `1`, và ngược lại. Tương đương, để đổi dấu của một số, **đảo** tất cả bits của biểu diễn nhị phân của nó.

::::{card}
$-7$ với Ones' Complement 8-bit
^^^
:::{figure} images/ones-complement-number-line.png
:label: fig-ones-complement-number-line
:width: 100%
:align: center
:alt: "Một trục số nằm ngang màu xanh hiển thị các giá trị nhị phân 4-bit để minh họa biểu diễn ones' complement, tập trung xung quanh các giá trị 0000 và 1111. Hai mũi tên màu vàng chỉ sang phải để chỉ ra rằng cả dãy nhị phân dương và âm đều tăng giá trị khi đồng hồ đo tăng từ trái sang phải."

"Đồng hồ đo nhị phân" cho ones' complement 4-bit.
:::

:::{note} Giải thích thêm
:class: dropdown

Xem xét cộng $5+(-5)$ với số nguyên one's complement 4-bit.

* $+5$: `0101`
* $-5$: `1010`

Phép cộng: `0101` + `1010` = `1111`, hoặc $-0$. Phép cộng số học có thể được triển khai với cộng nhị phân, bất kể dấu của toán hạng.
:::

::::


:::{tip} Kiểm tra nhanh

Giả sử bạn hiểu một mẫu N-bit như một số nguyên Ones' Complement. Làm thế nào bạn xác định nếu mẫu bit biểu diễn số dương? số âm?
:::

:::{note} Hiển thị đáp án
:class: dropdown

* Số dương: các 0 đứng đầu
* Số âm: các 1 đứng đầu
:::

:::{tip} Kiểm tra nhanh

Giả sử bạn dùng $N$ bits để biểu diễn số nguyên với ones' complement. Có bao nhiêu số dương? số âm? số không?
:::

:::{note} Hiển thị đáp án
:class: dropdown

* Số không: 2
* Số dương: $2^{N-1} - 1$
* Số âm: giống số dương
:::

:::{card}
Một lợi ích thêm của Ones' Complement
^^^
Bit ngoài cùng bên trái (còn gọi là **most significant bit**) vẫn hiệu quả là **sign bit**.
:::

...Nhưng chúng ta vẫn có vấn đề hai số không! Trong lịch sử, one's complement đã được sử dụng một thời gian, nhưng cuối cùng bị bỏ để chuyển sang [two's complement](#sec-twos-complement).


(sec-twos-complement)=
## Two's Complement

Ones' complement:
* Vấn đề: Các ánh xạ âm "chồng lấn" với các ánh xạ dương, tạo ra hai số 0.
* Giải pháp: **Dịch các ánh xạ âm sang trái một đơn vị.**

:::{figure} images/twos-complement-number-line.png
:label: fig-twos-complement-number-line
:width: 100%
:align: center
:alt: "Một trục số nằm ngang màu xanh hiển thị các giá trị nhị phân 4-bit và các tương đương thập phân tương ứng từ -8 đến 7 để minh họa biểu diễn two's complement. Hai mũi tên màu vàng chỉ sang phải trên toàn bộ thang, chỉ ra rằng cộng vào giá trị nhị phân liên tục tăng giá trị số trên cả phạm vi âm và dương."

"Đồng hồ đo nhị phân" cho twos' complement 4-bit.
:::

Những điểm đáng chú ý:
* Giống như trong ones' complement, tăng đồng hồ đo nhị phân tương ứng với cộng số nguyên thêm một.
* `0b0000` vẫn là $0$.
* Số dương giống như ones' complement.
* Số âm được dịch sang! Ví dụ, `0b1111` bây giờ ánh xạ đến $-1$. Điều này cho chúng ta thêm một số âm.
* Bit có ý nghĩa nhất (bit ngoài cùng bên trái) vẫn có thể được hiểu như **sign bit**.

> Trong Two's Complement, một mẫu bit toàn một là $-1$.

:::{tip} Kiểm tra nhanh

Giả sử bạn dùng $N$ bits để biểu diễn số nguyên với two's complement. Có bao nhiêu số dương? số âm? số không?
:::

:::{note} Hiển thị đáp án
:class: dropdown

* Số không: 1
* Số dương: $2^{N-1} - 1$
* Số âm: $2^{N-1}$
:::

### Số học và chuyển đổi

Phần cứng cho two's complement giờ đơn giản.

> Phép cộng hoàn toàn giống như với số không dấu.

Các số $5$ và $-5$ được biểu diễn trong two's complement 4-bit với `0b0101` và `0b1011`, tương ứng. Cộng chúng lại nên cho kết quả $0$, hoặc `0b0000`.

:::{figure} images/twos-complement-addition.png
:label: fig-twos-complement-addition
:width: 100%
:align: center
:alt: "Một sơ đồ so sánh phép cộng thập phân của năm dương và năm âm với phép toán tương đương trong nhị phân two's complement, cộng 0101 và 1011. Phép tính nhị phân hiển thị các bit nhớ màu đỏ và kết quả trong tổng năm-bit trong đó bit đứng đầu bị loại bỏ để đạt được giá trị bốn-bit đúng là 0000, tức là số không."

Phép cộng trong two's complement tuân theo trực giác thập phân.
:::

:::{note} Giải thích
:class: dropdown

Làm việc từ phải sang trái:

1. `1+1=0` nhớ `1`
1. `1+0+1=0` nhớ `1`
1. `1+1+0=0` nhớ `1`
1. `1+0+1=0` nhớ `1`
1. `1` (bị cắt bỏ và loại bỏ trong biểu diễn 4-bit)

(Kiểm tra lại rằng toán nhị phân khớp với toán thập phân: $5 + -5 = 0$)
:::


### Định nghĩa chính thức

Chúng ta có thể viết giá trị của một số two's complement $n$-digit là

$$
- 2^{n-1} d_{n-1} + \sum_{i=0}^{n-2} 2^i d_i
$$

Số dương và số âm có thể được tính bằng cùng một công thức. Ở trên, dấu được tính bằng cách nhân bit cao nhất với $(-2^{n-1})$.

:::{card}
Ví dụ: $5$ và $-5$ trong two's complement 4-bit
^^^

$$
\begin{align}
\texttt{0b1011}
&= (1 \times -2^3)  + (0 \times 2^2)  + (1 \times 2^1)  + (1 \times 2^0) \\
&=  -8    +  0          +  2            +  1 \\
&=  -5 \\
\end{align}
$$

$$
\begin{align}
\texttt{0b0101}
&= (0 \times -2^3)  + (1 \times 2^2)  + (0 \times 2^1)  + (1 \times 2^0) \\
&=  0           +  4            +  0            +  1 \\
&=  5 \\
\end{align}
$$

:::

<!--TODO: có cách nghĩ đơn giản hơn về mặt sư phạm. Chúng ta thảo luận sau (tức là, sao chép công thức từ precheck summary).-->

### Two's Complement: Đảo dấu

Phần cứng để chuyển dương sang âm (& ngược lại) đơn giản.

1. Bù tất cả bits
1. Sau đó cộng 1

:::{figure} images/twos-complement-flip-shift.png
:label: fig-twos-complement-flip-shift
:width: 70%
:align: center
:alt: "Hai quy trình minh họa quá trình đổi dấu two's complement bằng cách chuyển năm dương sang năm âm và ngược lại. Mỗi ví dụ minh họa việc đảo bits của giá trị nhị phân bắt đầu và sau đó cộng một để có kết quả cuối cùng với dấu ngược lại."

Two's Complement: Để đổi dấu, đảo bits và cộng một.
:::

Ở nhà: Chứng minh thuật toán tương đương với công thức!

Trực giác đến từ biểu diễn "bánh xe số" của đồng hồ đo nhị phân (Hình @fig-twos-complement-number-wheel). Bánh xe này cũng giúp chúng ta xác định nơi integer overflow xảy ra:

:::{figure} images/twos-complement-number-wheel.png
:label: fig-twos-complement-number-wheel
:width: 100%
:align: center
:alt: "Một trục số nằm ngang và một bánh xe số tròn trực quan biểu diễn tính liên tục và các điểm overflow của số nguyên two's complement 4-bit. Các tam giác cảnh báo màu vàng đánh dấu ranh giới quan trọng giữa giá trị dương tối đa 0111 và giá trị âm tối thiểu 1000 để chỉ ra nơi arithmetic overflow xảy ra."

Trên: Một trục số chỉ ra nơi integer overflow xảy ra. Dưới: Một "bánh xe" số chỉ ra cùng vị trí integer overflow.
:::

Trong @fig-twos-complement-number-wheel, 0 đến 7 vẫn giữ nguyên như mọi biểu diễn. Nhưng sau đó nó nhảy đến $-8$. Số hạng cấp cao thú vị đó ($-8$) kéo tất cả số âm xuống **một** để không có chồng lấn tại số không.


### Two's Complement: Chuẩn C (tính đến 2025)

Two's complement là biểu diễn số chuẩn C23 cho số nguyên có dấu. Một lần nữa, `int` tích hợp là nhập nhằng vì nó không chỉ định bitwidth. Và một lần nữa, header `stdint.h` cung cấp các typedefs như `int8_t`, `int16_t`, `int32_t`, v.v., cho các biểu diễn số nguyên có dấu.

## Bias Encoding

**Bias Encoding**:

> * Theo dõi một **bias**.
> * Để hiểu nhị phân lưu trữ: Đọc dữ liệu như một số nguyên không dấu, sau đó **trừ** bias
> * Để lưu một số nguyên như dữ liệu: **Cộng** bias, sau đó lưu số kết quả như một số nguyên không dấu.

Tưởng tượng bạn đang ghi lại tín hiệu điện dao động giữa 0 và 31 volts. Sẽ tuyệt không nếu lấy biểu đồ đó và kéo nó xuống để nó dao động quanh số không? Đó là bias encoding.

Chúng ta có thể dịch sang bất kỳ bias tùy ý nào chúng ta muốn để phù hợp với nhu cầu. Để biểu diễn (gần như) nhiều số âm như số dương, một **bias thường dùng** cho $N$-bits là $(2^{N-1} - 1)$.

:::{figure} images/bias-encoding-shift.png
:label: fig-bias-encoding-shift
:width: 50%
:align: center
:alt: "Một sơ đồ trình bày hai trục số nằm ngang song song. Các đường thẳng đứng nối các điểm cụ thể trên trục trên với các giá trị tương ứng trên trục dưới để chỉ ra ánh xạ giữa hai hệ thống."

Một biểu diễn bias-encoded hiệu quả dịch trục số sang biểu diễn không dấu.
:::

:::{card}
Ví dụ: $N = 4$ với bias $(2^{N-1} - 1)$
^^^

* Biểu diễn số nguyên 4-bit
* Bias: $(2^{4-1} - 1) = 7$
:::

Đây là một số sơ đồ trong trường hợp chúng hữu ích. @fig-bias-encoding-number-line biểu diễn một bias encoding trong đó $N = 4$ và bias $ = 7$. Đồng hồ đo chỉ làm đúng; nó đếm lên qua số không mà không có gì lạ xảy ra.

:::{figure} images/bias-encoding-number-line.png
:label: fig-bias-encoding-number-line
:width: 100%
:align: center
:alt: "Một trục số nằm ngang màu xanh hiển thị các giá trị nhị phân 4-bit và các tương đương thập phân tương ứng từ -7 (cho 0000) đến 8 (cho 1111) để minh họa bias encoding. Một mũi tên màu vàng duy nhất chỉ sang phải để chỉ ra rằng các giá trị thập phân tăng đơn điệu khi dãy nhị phân tăng từ 0000 đến 1111."

"Đồng hồ đo nhị phân" cho số nguyên bias-encoded 4-bit, với bias 7.
:::

Bạn cũng có thể thấy **bánh xe số** hữu ích để xem overflow xảy ra ở đâu, và số nguyên tăng như thế nào so với việc tăng nhị phân. Xem @fig-bias-encoding-number-wheel.

:::{figure} images/bias-encoding-number-wheel.png
:label: fig-bias-encoding-number-wheel
:width: 70%
:align: center
:alt: "Một bánh xe số tròn trực quan biểu diễn một số nguyên bias-encoded 4-bit. Các giá trị bên trong và bên ngoài bánh xe biểu diễn các số và biểu diễn bit, tương ứng; bánh xe có các vạch đi từ -7 (0000) đến 1 (1000) đến 8 (1111)."

Bánh xe số cho bias encoding.
:::

Chúng tôi thực sự thích biased encoding cho một số ứng dụng cụ thể chúng ta sẽ thấy sau trong khóa học.
