---
title: "Giới thiệu"
---

## Mục tiêu học tập

* Phân biệt giữa địa chỉ bộ nhớ và giá trị trong bộ nhớ.
* Làm quen với mô hình bộ nhớ địa chỉ theo byte (byte-addressable memory), nghĩa là mỗi byte trong bộ nhớ có một địa chỉ, và mỗi địa chỉ tham chiếu đến một vị trí byte trong bộ nhớ.
* Hiểu rằng con trỏ là các biến lưu trữ địa chỉ.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/tS3MOTQraL4
:width: 100%
:enumerated: false
:title: "Lecture 04.1 - C Intro: Pointers, Arrays, Strings: Pointers and Bugs"
:::
1:39 - 4:22
::::

## Bộ nhớ giống như một mảng địa chỉ theo Byte

Hiện tại, hãy tưởng tượng toàn bộ bộ nhớ của bạn như một mảng thực sự lớn, vô hạn, bắt đầu từ số không, như trong @fig-c-mem-byte-array.

:::{figure} images/c-mem-byte-array.png
:label: fig-c-mem-byte-array
:width: 100%
:alt: "Bộ nhớ được vẽ như một mảng dài địa chỉ theo byte với địa chỉ tăng dần từ trái sang phải. Sơ đồ cho thấy biến x chiếm bốn byte với giá trị 0x12345678 bắt đầu tại địa chỉ 0x100, một mảng ký tự c chứa b, y, e, và ký tự kết thúc null bắt đầu tại địa chỉ 0x104, và con trỏ p lưu địa chỉ 0x00000100 bắt đầu tại địa chỉ bộ nhớ 0x108."

Một góc nhìn về bộ nhớ như một mảng khổng lồ duy nhất, trong đó mỗi byte có một địa chỉ.
:::

Hãy nghĩ về mảng này như một con đường rất, rất, rất dài với nhiều ngôi nhà. Mỗi ô là một ngôi nhà; ô đó có ai đó sống bên trong (ví dụ: một byte dữ liệu), và ô đó cũng có địa chỉ nhà (ví dụ: địa chỉ bộ nhớ). Chúng ta sẽ lo về phạm vi địa chỉ sau; hãy giả sử con đường dài vô hạn, nhưng bắt đầu từ `0x0000000` (địa chỉ toàn số không).

Trong phép so sánh này, mỗi ô của mảng **rộng một byte**. Do đó, mỗi byte có một **địa chỉ** liên kết với nó, và bản thân byte có một **giá trị**. Ví dụ, byte tại địa chỉ `0x00000104` (tức "@" `0x104`) là mẫu 8-bit cho ký tự ASCII `'b'`, hay `0b01100010`.

Tên biến thường có thể tham chiếu đến các vị trí bộ nhớ - cụ thể là các khối bộ nhớ. Trong @fig-c-mem-byte-array, biến `x` là (giả sử) một số nguyên không dấu 32-bit. Biến `x` có giá trị `0x12345678`. **Địa chỉ** của nó, theo quy ước, là địa chỉ của byte _đầu tiên_ của khối bộ nhớ, còn được gọi là địa chỉ **thấp nhất** của khối. Trong sơ đồ, địa chỉ của `x` là `0x00000100`.[^endianness]

[^endianness]: Byte nào của `x` được lưu tại địa chỉ thấp nhất? Phụ thuộc vào kiến trúc của bạn. Đọc về endianness trong [phần khác](#sec-endianness).

Trong chương này, chúng ta sẽ đề cập đến ba khái niệm liên quan chặt chẽ giúp chúng ta hiểu rõ hơn về cách bộ nhớ hoạt động bên dưới: **con trỏ**, **mảng**, và **chuỗi C**. Mỗi khái niệm này đều có những lợi ích và cạm bẫy riêng. Hãy bắt đầu!

## Con trỏ lưu trữ địa chỉ

Bạn có thể đã nghe qua về con trỏ. Bạn có thể sợ con trỏ! Chìa khóa để hiểu con trỏ là thực sự nắm vững định nghĩa của nó:

> Con trỏ: Một biến chứa địa chỉ của một biến khác.

Nói cách khác, nó "trỏ" đến một vị trí bộ nhớ. Phép so sánh "trỏ" này hơi khó hiểu lúc đầu, vì vậy hãy làm rõ.

Trong @fig-c-mem-byte-array-ptr, con trỏ `p` "trỏ đến" `x`. Nhưng mọi thứ trong C đều là các bit bên dưới, điều này _thực sự_ có nghĩa là p là một biến lưu trữ địa chỉ của `x`, hay `0x00000100`. Trong khi `x` là (ví dụ) một số nguyên không dấu chiếm 32 bit, giá trị của `p` là địa chỉ của byte **thấp nhất** của `x`.

:::{figure} images/c-mem-byte-array-ptr.png
:label: fig-c-mem-byte-array-ptr
:width: 100%
:alt: "Cùng một bố cục bộ nhớ địa chỉ theo byte được hiển thị với một mũi tên xanh từ con trỏ p đến biến x. Hình ảnh này cho thấy p lưu địa chỉ bắt đầu của x, 0x00000100."

Con trỏ `p` "trỏ" đến vị trí của `x` trong bộ nhớ. Xem mũi tên xanh.
:::

Vì bản thân con trỏ cũng là biến, chúng cũng có thể có địa chỉ. Trong @fig-c-mem-byte-array-ptr, `p` được lưu tại địa chỉ `0x00000108`.

Một cách diễn đạt khác bạn sẽ nghe là bạn có thể "**đi theo**" con trỏ, nghĩa là, chúng ta truy cập giá trị mà con trỏ trỏ đến (một quá trình chúng ta sẽ gọi chính thức là **giải tham chiếu - dereferencing**). Trong trường hợp này, nếu chúng ta "đi theo" con trỏ `p`, chúng ta sẽ nhận được giá trị của `x`, là `0x12345678`.

Để làm điều đó, C yêu cầu tất cả con trỏ phải có **kiểu**. Nếu chúng ta biết rằng `p` là một con trỏ đến một số nguyên không dấu 32-bit, thì chúng ta biết rằng đi theo con trỏ `p` sẽ lấy 4 byte **bắt đầu từ** `0x00000100`, không chỉ byte tại `0x00000100`. Chúng ta sẽ thảo luận về cú pháp này trong chương tiếp theo.


:::{note} Tóm tắt @fig-c-mem-byte-array-ptr

* `x` là một biến rộng 4-byte. Chúng ta giả sử rằng `x` là một số nguyên không dấu 32-bit. Biến `x` nằm tại địa chỉ `0x100`, và giá trị của nó là `0x12345678`.
* `p` là một biến, cũng rộng 4-byte. Vì sơ đồ cho thấy `p` trỏ đến `x`, chúng ta có thể kết luận rằng `p` là một con trỏ đến một số nguyên không dấu 32-bit. Con trỏ `p` nằm tại địa chỉ `0x00000108`, và giá trị của nó là `0x00000100` vì nó trỏ đến `x`.
* Nếu chúng ta đi theo con trỏ `p`, chúng ta nhận được số nguyên không dấu 32-bit `0x12345678`.
:::
