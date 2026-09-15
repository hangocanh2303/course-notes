---
title: "Nhị phân, Thập phân, Thập lục phân"
---

(bin-dec-hex)=
## Mục tiêu học tập

* Chuyển đổi giữa các biểu diễn số nhị phân, thập phân, và thập lục phân
* Sử dụng thập lục phân như cách viết tắt cho nhị phân
* Biết khi nào mỗi biểu diễn hữu ích

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/5rmB4SvfDPo?si=7YF8BXMnDpFCQiCH
:width: 100%
:title: "[CS61C FA20] Lecture 02.2 - Number Representation: Conversions"
:::

::::

## Giới thiệu

Trong phần này, chúng ta thảo luận cách các kiến trúc sư máy tính và nhà khoa học máy tính chuyển đổi giữa thế giới phong phú mà con người thấy và thông tin mà máy tính lưu trữ. Cái trước được định khung bởi cách con người nghĩ—sau cùng, chúng ta có mười ngón tay, còn được gọi là "digits" (chữ số). Cái sau là trong bits.


## Chữ số như là biểu diễn của Số

Hãy thảo luận ý tưởng biểu diễn chính thức một **số** bằng nhiều **chữ số** có thể, tức là, các ký hiệu.

* **Chữ số (Numeral)**: Một ký hiệu (hoặc chuỗi ký hiệu) hoặc tên đại diện cho một số, ví dụ, 4, four, quatro, IV, IIII, …. Chữ số được tạo thành từ nhiều ký hiệu gọi là **digits**.
* **Số (Number)**: "Ý tưởng" trong đầu chúng ta, ví dụ, khái niệm "4". Chỉ có MỘT khái niệm về một số, nhưng có thể có nhiều biểu diễn số có thể qua nhiều chữ số có thể.

:::{figure} images/numeral-number.png
:label: fig-numeral-number
:alt: "Một sơ đồ với đường trừu tượng màu vàng nằm ngang tách từ Numeral ở trên khỏi từ Number ở dưới. Bố cục trực quan này củng cố chú thích bằng cách đặt các chữ số như biểu diễn ký hiệu phía trên đường và số như khái niệm trừu tượng cơ bản phía dưới nó."
:align: center

Chữ số (và do đó các digits) là biểu diễn của số.
:::

@fig-every-base-is-base-10 là một ví dụ động lực (và hài hước). Một người ngoài hành tinh và một phi hành gia đang thảo luận cách biểu diễn số đá trong một đống.

:::{figure} images/every-base-is-base-10.png
:label: fig-every-base-is-base-10
:alt: "Một phi hành gia nói chuyện với người ngoài hành tinh có 2 ngón tay mỗi bàn tay và có 4 viên đá trên mặt đất. Người ngoài hành tinh nói 'Có 10 viên đá.' Phi hành gia nói 'Ồ, bạn chắc đang dùng cơ số 4. Thấy chưa, tôi dùng cơ số 10.' Người ngoài hành tinh nói 'Không. Tôi dùng cơ số 10. Cơ số 4 là gì?' Chú thích đọc 'Mọi cơ số đều là cơ số 10'"
:align: center

Mọi cơ số đều là cơ số 10 ([web.archive.org](https://web.archive.org/web/20160505151914/http://cowbirdsinlove.com/43))
:::

Người ngoài hành tinh, phi hành gia, và đống đá sử dụng ba biểu diễn khác nhau của số *bốn*. Phi hành gia dùng chữ số 4 để biểu diễn bốn như một số nguyên cơ số 10. Người ngoài hành tinh dùng chữ số 10 để biểu diễn bốn như một số nguyên cơ số 4. Đống đá dùng bốn viên đá để biểu diễn bốn như, à, một đống đá.

## Biểu diễn Nhị phân, Thập phân, và Thập lục phân

Mặc dù có vô số cơ số để biểu diễn số, chúng ta thảo luận ba cơ số hữu ích nhất cho chúng ta, như các nhà khoa học máy tính: biểu diễn **thập phân**, **nhị phân**, và **thập lục phân**.

@tab-dec-hex-bin có lẽ chưa có nhiều ý nghĩa với bạn lúc này, nhưng chúng tôi trình bày nó trước để bạn có thể đoán được một số điều.

:::{table} Các chữ số thập phân, nhị phân, và thập lục phân.
:label: tab-dec-hex-bin
:align: center

| Hệ thống | # Chữ số | Các chữ số |
| :--- | :-: | :--- |
| Thập phân | 10 | `0, 1, 2, 3, 4, 5, 6, 7, 8, 9` |
| Nhị phân | 2 | `0, 1` |
| Thập lục phân | 16 | `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E, F` |

:::


### Thập phân: Số Cơ số 10 (Mười)

Đầu tiên, xem xét chữ số **thập phân**. Chữ số thập phân $3271$ được viết theo thứ tự đó, vì nó mô tả cách đếm các lũy thừa của **mười** tương ứng với phương trình bên dưới (lưu ý chỉ số trên $10$ biểu thị chữ số cơ số 10):

$$
\begin{align}
3271
&= 3271_{10} \\
&=  (3 \times 10^3) + (2 \times 10^2) + (7 \times 10^1) + (1 \times 10^0)
\end{align}
$$

Mỗi trong bốn chữ số chỉ định một *số đếm* của một lũy thừa của mười. Chúng ta cộng những thứ này lại để có số ba nghìn hai trăm bảy mươi mốt.

:::{note} Giải thích
:class: dropdown

* Chữ số **ngoài cùng bên phải**, *1*, tương ứng với lũy thừa **nhỏ nhất** của 10 tạo thành một số nguyên không âm. Đây là lũy thừa thứ không, hoặc $10^0 = 1$. Chúng ta bao gồm *một*.
* Chữ số tiếp theo, $7$, tương ứng với lũy thừa nhỏ nhất tiếp theo của 10, là $10^1 = 10$, hoặc mười. Chúng ta bao gồm *bảy* mười.
* Chữ số tiếp theo, $2$, tương ứng với lũy thừa nhỏ nhất tiếp theo của 10, là $10^2 = 100$, hoặc một trăm. Chúng ta bao gồm *hai* trăm.
* Chữ số cuối cùng **ngoài cùng bên trái**, tương ứng với lũy thừa **lớn nhất** của 10 cho số này, là $10^3 = 1000$, hoặc một nghìn. Chúng ta bao gồm *ba* nghìn.
:::

Quá trình này sẽ có vẻ tự nhiên với bạn—vì con người chúng ta nghĩ theo cơ số mười—nhưng chúng ta sẽ thấy rằng chúng ta có thể áp dụng hiểu biết này để biểu diễn số trong các cơ số khác. Tuy nhiên, chúng tôi làm nổi bật một vài giả định ngầm:

* Hiện tại, chúng ta chỉ xem xét các lũy thừa của 10 cần để tạo thành số nguyên không âm; chúng ta sẽ thảo luận cách biểu diễn phân số sau. Lũy thừa nhỏ nhất như vậy của 10 là $10^0 = 1$.
* Lũy thừa "lớn nhất" của 10 cho số này có thể được định nghĩa chính xác hơn là lũy thừa lớn nhất của 10 tương ứng với chữ số khác không. Nói cách khác, số thập phân $0 \dots 03271$ với các số không đứng đầu (bên trái) và số thập phân $3271$ biểu diễn cùng một số.
* Cơ số mười sử dụng mười chữ số `0` đến `9` để tạo ra các biểu diễn thập phân duy nhất. Nói cách khác, để tạo số thập phân $10$, thay vì dùng mười của $10^0$, chúng ta dùng một $10^1$ và không $10^0$. [Wikipedia](https://en.wikipedia.org/wiki/Radix) cung cấp thảo luận chính thức hơn về tính duy nhất.

### Số Cơ số 2 (Hai), Nhị phân

Số nhị phân như `1101` được viết theo thứ tự đó để mô tả cách đếm các lũy thừa của **hai**. Đáng chú ý, hai chữ số **nhị** phân, `0` và `1`, là nguồn gốc của **b**it (lấy hai giá trị đó).

> Chữ số nhị phân `1101` biểu diễn gì? 

Vì con người nghĩ theo thập phân, chúng ta chuyển đổi giá trị nhị phân này sang thập phân với quá trình tương tự như trên: 

$$
\begin{align}
\texttt{0b1101}
&= 1101_{2}	\\
&= (1 \times 2^3) + (1 \times 2^2) + (0 \times 2^1) + (1 \times 2^0) \\
&=  8         +  4        +  0         +  1 \\
&=  13
\end{align}
$$

:::{note} Giải thích
:class: dropdown

* Chữ số **ngoài cùng bên phải**, *1*, tương ứng với lũy thừa **nhỏ nhất** của **2** tạo thành một số nguyên không âm. Đây *vẫn* là lũy thừa thứ không, hoặc $2^0 = 1$. Chúng ta bao gồm *một* một.
* Chữ số tiếp theo, *0*, tương ứng với $2^1 = 2$, hoặc hai. Chúng ta bao gồm *không* hai.
* Chữ số tiếp theo, *1*, tương ứng với $2^2 = 4$, hoặc bốn. Chúng ta bao gồm *một* bốn.
* Chữ số cuối cùng **ngoài cùng bên trái**, *1* tương ứng với $2^3 = 8$, hoặc tám. Chúng ta bao gồm *một* tám.

Cộng một một, không hai, một bốn, và một tám cho ra mười ba, hoặc số thập phân $13$.
:::

Các lưu ý khác:

* Chúng ta thêm tiền tố `0b` để biểu thị rằng chữ số `1101` nên được hiểu theo cơ số 2; **viết tắt** `0b1101` tương đương với ký hiệu toán học $1101_2$ nhưng có thể được viết bằng bàn phím chuẩn.
* Như trước, `0b0...01101` và `0b1101` biểu diễn cùng một số, mười ba.
* Vì chỉ có hai chữ số nhị phân `0` và `1`, trong nhị phân chúng ta luôn hoặc bao gồm một giá trị (ở đây, một lũy thừa cụ thể của hai), hoặc không bao gồm nó. `1` hoặc `0`, `True` hoặc `False`. Ý tưởng về nhị phân biểu diễn "bao gồm" hoặc "loại trừ" này sẽ xuất hiện lặp đi lặp lại trong khóa học này.

### Số Cơ số 16 (Mười sáu), Thập lục phân

Cuối cùng, chúng ta xem xét số thập lục phân.

> Chữ số thập lục phân `A5` biểu diễn gì? 

Chuyển đổi sang thập phân:

$$
\begin{align}
\texttt{0xA5}
&= A5_{16} \\
& = (10 \times 16^1) + (5 \times 16^0) \\
& =  160        +  5 \\
& =  165
\end{align}
$$

Chúng ta thêm tiền tố `0x` để biểu thị rằng chữ số `A5` nên được hiểu theo cơ số 16. Như trước, viết tắt `0xA5` tương đương với $A5_{16}$ nhưng có thể được viết bằng bàn phím chuẩn.

Các chữ số thập lục phân hữu ích như cách viết tắt để biểu diễn các nhóm bốn chữ số nhị phân. Chúng ta thảo luận thêm [ở cuối phần này](#which-base).

## Chuyển đổi giữa các biểu diễn

:::{tip}
Ghi nhớ @tbl-dec-hex-bin-16. Như bạn sẽ sớm thấy, việc nhanh chóng chuyển đổi giữa các biểu diễn thập phân, nhị phân, và thập lục phân sẽ rất hữu ích.

:::

:::{table} Mười sáu số đầu tiên dưới dạng biểu diễn thập phân, thập lục phân, và nhị phân.
:label: tbl-dec-hex-bin-16
:align: center

| Số | Thập lục phân | Nhị phân |
| :--- | :- | :--- | 
| 0 | `0` | `0000` |
| 1 | `1` | `0001` |
| 2 | `2` | `0010` |
| 3 | `3` | `0011` |
| 4 | `4` | `0100` |
| 5 | `5` | `0101` |
| 6 | `6` | `0110` |
| 7 | `7` | `0111` |
| 8 | `8` | `1000` |
| 9 | `9` | `1001` |
| 10 | `A` | `1010` |
| 11 | `B` | `1011` |
| 12 | `C` | `1100` |
| 13 | `D` | `1101` |
| 14 | `E` | `1110` |
| 15 | `F` | `1111` |

:::

Hãy thảo luận về chuyển đổi chi tiết hơn. Chúng ta chỉ xem xét các chữ số "unsigned", tức là, số không âm.

Nếu chúng ta có một chữ số unsigned $n$-digit $d_{n-1}$ $d_{n-2}$...$d_0$ trong radix (hoặc cơ số) $r$, thì giá trị của chữ số đó là

(eq-unsigned-rep)=
$$
\sum_{i=0}^{n-1} r^i d_i,
$$

đó chỉ là ký hiệu đẹp để nói rằng thay vì hàng 10 hoặc hàng 100 chúng ta có hàng $r$ hoặc hàng $r^2$. Với ba radix nhị phân, thập phân, và hex, chúng ta chỉ cho $r$ là 2, 10, và 16, tương ứng. 

### Thập phân $\rightarrow$ Nhị phân

Slidedeck bên dưới cho thấy cách chúng ta có thể chuyển đổi số thập phân $13$ thành biểu diễn nhị phân, `0b1101`.

:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vSRRi1DDwigsxmr5R_fPwZ1uAOKKJ-fblPQg6GFNICf9he20UUYX_gZLwdrMG4HRvrtcD3e9nkBwk29/pubembed?start=false&loop=false
:width: 100%
:title: "Animation hướng dẫn từng bước cách chuyển đổi giá trị thập phân 13 thành tương đương nhị phân, như được trình bày trong phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1aihcZDiAEMCarSs-QIzRE_ixW5mACYYTykdmRyqIZUc/edit?usp=sharing)"
:::

Cho `val` là $13$ trong giải thích bên dưới. Nhấp để hiển thị.

:::{note} Giải thích
:class: dropdown

Tạo các cột 1, 2, 4, 8, tương ứng với các lũy thừa nguyên tăng dần của hai. Các cột này cũng tương ứng với một số nhị phân 4-digit, ví dụ, `0b _ _ _ _`.

1. Lấy lũy thừa lớn nhất của hai: $2^3 = 8$, hoặc tám. Điều này "vừa" vào `val`, nên "dùng" nó. "Dùng" nghĩa là chúng ta cần chữ số nhị phân `1`. Đặt ô thứ 4-từ-phải thành `1`, tức là, điền `0b 1 _ _ _`. Vì chúng ta đã "dùng hết" tám, trừ đi và cập nhật `val` thành 5. Đây là phần còn lại chúng ta cần biểu diễn.
1. Lấy lũy thừa tiếp theo của hai: $2^2 = 4$, hoặc bốn. "Dùng" nó bằng cách đặt ô thứ 3-từ-phải thành `1`, tức là, điền `0b 1 1 _ _`. Cập nhật `val` thành 1.
1. Lấy lũy thừa tiếp theo của hai: $2^1 = 2$, hoặc hai. Chúng ta không thể "dùng" nó vì nó quá lớn. "Không dùng" nghĩa là chúng ta cần chữ số nhị phân `0`. Đặt ô thứ hai-từ-phải thành `0`, tức là, điền `0b 1 1 0 _`. `val` không đổi và vẫn là 1.
1. Lấy lũy thừa tiếp theo của hai, cũng là nhỏ nhất: $2^0 = 1$, hoặc một. "Dùng" nó bằng cách đặt ô ngoài cùng bên phải thành `1`, tức là, điền `0b 1 1 0 1`. Cập nhật `val` thành 0.

Kết quả **chuỗi nhị phân** là `0b1101`. Tự mình thực hành theo chiều ngược lại và kiểm tra rằng `0b1101` biểu diễn số thập phân $13$.

:::

Quá trình trên dựa vào một vài quan sát thông thường:

* Lũy thừa lớn nhất của hai chúng ta có thể cần nhỏ hơn chính số `val`.
* Lũy thừa nhỏ nhất của hai chúng ta có thể cần luôn là lũy thừa thứ không, tức là, $2^0 = 1$.
* Bắt đầu với các lũy thừa lớn hơn của hai trước. Nếu không bạn có thể gặp tình huống đếm vượt quá số chữ số có sẵn (ở đây, chỉ `0` và `1`).

Đây là một nỗ lực mô tả thuật toán để chuyển đổi một số `val` thành biểu diễn nhị phân:

* Tạo một tập cột, một cho mỗi lũy thừa của hai. Điều này tương ứng với số nhị phân $n$-digit của bạn, ví dụ, `0b _ _ ... _ _`, với $n$ chỗ trống.
* Bắt đầu từ cột ngoài cùng bên trái và đi sang phải (tức là, với $i$ từ $n-1$ đến $0$, bao gồm):
  * Với cột hiện tại $i$, tương ứng với $2^i$:
    * Cột hiện tại có nhỏ hơn hoặc bằng `val` không?
        * Nếu có, đếm xem bao nhiêu $2^i$ vừa vào `val`. Với cơ số 2, số đếm là `1`, nên trừ $1 \times 2^i$ từ `val`. Tiếp tục.
        * Nếu không, đặt `0` và tiếp tục.
* Dừng quá trình này khi `val` về không.

Mục đích của ví dụ này là dạy bạn một số thủ thuật để chuyển đổi giữa các cơ số. Một số sinh viên thấy mô tả thuật toán ở trên dễ hiểu hơn mô tả toán học trong @eq-unsigned-rep. Nếu cái sau có ý nghĩa hơn với bạn, thì hãy dùng nó.

### Thập phân $\rightarrow$ Thập lục phân

Slidedeck bên dưới chuyển đổi $165$ thành biểu diễn thập lục phân, `0xA5`.

:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vTWgqxt2kl5ip7bRfRY7P81WiQGDJSrAuKmkqE1m3mnrerVBeN2s9PGJVVaIQu0aDlsNfuvcRCK09q9/pubembed?start=false&loop=false
:width: 100%
:title: "Animation hướng dẫn từng bước cách chuyển đổi giá trị thập phân 165 thành biểu diễn thập lục phân, như được trình bày trong phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1rHScPLQLom3OzhZyGzhct8vpXtO-AWRiXyr5C-9Qszs/edit?usp=sharing)"
:::

Cho `val` là $165$ trong giải thích bên dưới. Nhấp để hiển thị.

:::{note} Giải thích
:class: dropdown

Tạo các cột 1, 16, 256, 4096 tương ứng với các lũy thừa nguyên tăng dần của mười sáu. Các cột này cũng tương ứng với một số nhị phân 4-digit, ví dụ, `0x _ _ _ _`.

1. Lấy lũy thừa lớn nhất: $16^3 = 4096$. Điều này không vừa vào `val`, nên chúng ta đặt ô thứ 4-từ-phải thành `0`, ví dụ, `0x 0 _ _ _`.
1. Lấy lũy thừa tiếp theo: $16^2 = 256$. Điều này không vừa vào `val`, nên chúng ta đặt ô thứ 3-từ-phải thành `0`, ví dụ, `0x 0 0 _ _`.
1. Lấy lũy thừa tiếp theo: $16^1 = 16$. Đếm xem bạn có thể "dùng" nó bao nhiêu lần; tối đa mười $16$ vừa vào `val`, hiện tại là $165$. Mười là chữ số thập lục phân `A`, nên chúng ta đặt ô thứ 2-từ-phải thành `A`, ví dụ, `0x 0 0 A _`. Trừ $10 \times 16^1$ từ $165$ và cập nhật `val` thành $5$.
1. Lấy lũy thừa tiếp theo, cũng là nhỏ nhất: $16^0 = 1$, hoặc một. Đếm xem bạn có thể "dùng" nó bao nhiêu lần; tối đa năm $1$ vừa vào `val`, hiện tại là $5$. Đặt ô ngoài cùng bên phải thành `5`, ví dụ, `0x 0 0 A 5`. Cập nhật `val` thành $0$.

Kết quả chữ số thập lục phân là `0x00A5`, hoặc tương đương, `0xA5` nếu chúng ta bỏ các số không đứng đầu.

Bạn có thể nhận thấy rằng chúng tôi bao gồm $16^3$ và $16^2$, cả hai đều quá lớn để biểu diễn $165$. Điều này để cho bạn thấy rằng các số không đứng đầu là ổn và có thể bỏ đi sau khi bạn có kết quả.

:::

Chúng tôi để cho bạn tự dịch quá trình chuyển đổi nhị phân mà chúng tôi mô tả thông thường thành quá trình chuyển đổi thập lục phân.

### Nhị phân $\leftrightarrow$ Thập lục phân Rất Đơn Giản

Với những điều trên, xem xét quá trình sau để chuyển đổi nhị phân sang thập lục phân, kết hợp các quá trình chúng ta đã thảo luận ở trên:

1. Chuyển đổi nhị phân sang thập phân.
1. Chuyển đổi thập phân sang thập lục phân.

Quá trình này tẻ nhạt—tính các lũy thừa của hai là khả thi, nhưng liệu mọi kiến trúc sư máy tính có ghi nhớ các lũy thừa của mười sáu không? Thay vào đó, chúng ta có thể chuyển đổi _trực tiếp_ giữa nhị phân và thập lục phân với quan sát:

> Tồn tại một ánh xạ một-một giữa tập các chữ số thập lục phân và tập các chuỗi nhị phân độ dài bốn.

Quan sát trên ngụ ý rằng một chuỗi nhị phân độ dài $4k$ có thể được dịch thành một chuỗi thập lục phân độ dài $k$ bằng cách độc lập chuyển đổi mỗi chuỗi nhị phân độ dài 4 thành một chữ số thập lục phân, sau đó nối kết quả. (Chúng tôi để chứng minh điều này cho những bạn là nhà toán học nhiệt tình.) Điều này làm cho việc chuyển đổi giữa nhị phân và thập lục phân dễ dàng hơn nhiều:

Để chuyển đổi `0x1E` sang nhị phân:

* `1` trong thập lục phân là `0001` trong nhị phân
* `E` trong thập lục phân là `1110` trong nhị phân
* Nối: `0001 1110` (chúng tôi bao gồm khoảng trắng để dễ hình dung hơn)
* Bỏ các số không đứng đầu và nén khoảng trắng. (Tùy chọn) Thêm tiền tố `0b` để biểu thị nhị phân: `0b11110`

Để chuyển đổi `0b11110` sang thập lục phân:

* Đầu tiên nhóm thành các chuỗi 4-bit đầy đủ, đệm không bên trái khi cần thiết: `0001 1110`
* `0001` trong nhị phân là `1` trong thập lục phân
* `1110` trong nhị phân là `E` trong thập lục phân
* Nối: `0x1E` (chúng ta thêm tiền tố `0x` để biểu thị thập lục phân)


:::{tip}
Một lần nữa, ghi nhớ @tbl-dec-hex-bin-16!
:::

## Máy tính cũng biết điều đó

Tại thời điểm này, đáng để nhắc nhở người đọc lần đầu rằng tiền tố hai ký tự `0b` và `0x` biểu thị rằng các chữ số nên được hiểu như biểu diễn nhị phân và thập lục phân, tương ứng. `0` trong `0b` và `0x` không có nghĩa gì đặc biệt.

Các tiền tố này cho phép máy tính phân tích chuỗi chữ số và hiểu chúng theo cơ số dự định.

```c
#include <stdio.h>
int main() {
  const int N = 1234;
  printf("Decimal: %d\n",N);
  printf("Hex: 	  %x\n",N);
  printf("Octal:   %o\n",N);

  printf("Literals (not supported by all compilers):\n");
  printf("0x4d2         = %d (hex)\n", 0x4d2);
  printf("0b10011010010 = %d (binary)\n", 0b10011010010);
  printf("02322         = %d (octal, prefix 0 - zero)\n", 0x4d2);
  return 0;
}
```

Đầu ra:

```
Decimal: 1234
Hex:     4d2
Octal:   2322
Literals (not supported by all compilers):
0x4d2         = 1234 (hex)
0b10011010010 = 1234 (binary)
02322         = 1234 (octal, prefix 0 - zero)
```

Chúng tôi không mong bạn hiểu code này vào lúc này. Chúng ta sẽ thảo luận về cú pháp C, compilers, literals, v.v. rất sớm. Chúng tôi cũng sẽ không đề cập đến octal literals trong khóa học này; hầu hết các compiler C chuẩn sẽ nhận dạng thập lục phân và nhị phân literals.

(which-base)=
## Chúng ta dùng cơ số nào?

Nhớ rằng luôn chỉ có một số; giá trị này có thể được biểu diễn theo nhiều cách. Dưới đây đều là biểu diễn của cùng một số, ba mươi hai:

* $32_{10}$, hoặc đơn giản là $32$. Chúng tôi khuyên bạn viết $32_{ten}$ nếu bạn viết tay.
* `0x20`, hoặc chữ số thập lục phân $20$. Vì cái sau đã là biểu diễn cơ số 10 mặc định của chúng ta, chúng tôi khuyên dùng tiền tố `0x` hoặc (nếu viết tay) dùng chỉ số dưới, ví dụ, $20_{16}$ hoặc $20_{hex}$.
* `0b100000`, hoặc chữ số nhị phân $100000$. Một lần nữa, chúng tôi khuyên dùng tiền tố hoặc (nếu viết tay) dùng chỉ số dưới, ví dụ, $100000_{2}$ hoặc $100000_{two}$.

Các biểu diễn khác nhau phục vụ các mục đích khác nhau:

* **Thập phân**: Tuyệt vời cho con người, đặc biệt khi làm số học. Chúng tôi hy vọng bạn sẽ không bao giờ quên cơ số 10 :-)
* **Nhị phân**: Những gì máy tính sử dụng. Với máy tính, số được lưu trữ dưới dạng dữ liệu nhị phân, bất kể số được chỉ định như thế nào.
* **Hex**: Hy vọng bạn đã nhận ra bây giờ rằng chuỗi dài số nhị phân khó phân tích. Thập lục phân tệ cho số học trên giấy, nhưng nó (1) nhỏ gọn hơn nhiều so với nhị phân, và (2) dễ dàng hơn thập phân nhiều như một định dạng để chuyển đổi sang và từ nhị phân.

Chúng tôi sử dụng hai chiến lược trong khóa học này để dễ hình dung hơn các chuỗi 32 bits, 64 bits, v.v.:

  * Nhóm 4 bits một lần, ví dụ, `0b0011 1010`
  * Chuyển đổi mỗi nhóm 4 bits thành chữ số thập lục phân tương ứng, ví dụ, `0x3A`

Trên hết, nhớ rằng máy tính hoạt động bằng nhị phân, nhưng con người thì không. Vì vậy việc thoải mái hơn với việc chuyển đổi giữa các biểu diễn này trước khi chúng ta tiến xa hơn là điều tốt.
