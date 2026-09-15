---
title: "Giới thiệu"
---

## Mục tiêu học tập

* Làm quen với lịch sử ngắn gọn về máy tính
* Hiểu tại sao viết chương trình bằng C cho phép chúng ta khai thác các tính năng cơ bản của kiến trúc (có chủ đích hoặc không có chủ đích)

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/p5QYzRGWGKo?si=CUN0igmuK0dZVVR1
:width: 100%
:title: "[CS61C FA20] Lecture 03.1 - C Intro: Basics: Intro and Background"
:::

::::

(sec-stored-program)=
## Máy tính Chương trình Lưu trữ

Hãy hiểu một chút về những điều cơ bản của tổ chức máy tính. Một trong những máy tính đầu tiên là ENIAC tại UPenn vào năm 1945-46.

:::{figure} images/eniac.jpg
:label: fig-eniac
:width: 60%
:align: center
:alt: "Một bức ảnh đen trắng chụp một căn phòng lớn chứa đầy các bảng điện tử cao và dây cáp chằng chịt của hệ thống máy tính ENIAC. Hai người được định vị giữa các thiết bị, bao gồm một phụ nữ ở phía trước đứng cạnh một tủ điều khiển lớn trong khi cầm một tài liệu."

ENIAC (Electronic Numerical Integrator and Computer) là máy tính số điện tử có thể lập trình, đa năng đầu tiên, hoàn thành vào năm 1945. ([Wikipedia](https://en.wikipedia.org/wiki/ENIAC))
:::

ENIAC là máy tính điện tử đa năng đầu tiên. Sau Thế chiến II, nó thường được sử dụng để tính toán quỹ đạo đạn đạo. Nó nhân trong 2.8 mili giây! Vấn đề là, phải mất hai hoặc ba ngày để lập trình.

Chú ý các dây nối trong @fig-eniac; sau khi đọc sơ đồ, một người sẽ viết các dây nối. Chưa kể—các đèn chân không cứ hỏng liên tục.

Hãy dành một chút thời gian để ghi nhận những người bạn thấy trong hình. Nhiều lập trình viên đầu tiên là **máy tính** (computers):

:::{epigraph}
Những phụ nữ làm việc cho NASA ... đang tính toán các phương trình để giúp cải thiện đặc tính bay của máy bay. Các kỹ sư sẽ làm việc trên thiết kế và vạch ra các phương trình. Các phương trình sau đó sẽ được giao cho một máy tính, một phụ nữ, để tính toán.

-- [_Computer History_](https://computerhistory.org/blog/hidden-figures-no-longer/)
:::

Các lập trình viên đầu tiên thường là phụ nữ thuộc nhóm UPenn, và họ không được đưa tin đủ vì là những lập trình viên đầu tiên của thời đại. Hãy xem phim [_Hidden Figures_](https://www.imdb.com/title/tt4846340/) (2016) để tìm hiểu về những nữ lập trình viên và nhà toán học đầu tiên này.

Tiếp theo, **EDSAC** tại Cambridge (1949) là máy tính **chương trình lưu trữ** đa năng đầu tiên.

:::{figure} images/edsac.jpg
:label: fig-edsac
:width: 60%
:align: center
:alt: "Một bức ảnh đen trắng cho thấy nhiều giá dọc cao của hệ thống máy tính EDSAC chứa đầy đèn chân không và linh kiện điện tử. Hình ảnh chụp kiến trúc vật lý quy mô lớn của các máy tính chương trình lưu trữ đầu tiên trong một môi trường phòng thí nghiệm."

EDSAC (Electronic Delay Storage Automatic Calculator) là một trong những máy tính chương trình lưu trữ đa năng đầu tiên, hoàn thành vào năm 1949.
[Wikipedia](https://en.wikipedia.org/wiki/EDSAC)
:::

Chú ý rằng @fig-edsac không còn có dây nối. EDSAC được thiết kế xung quanh một khái niệm đáng kinh ngạc: **chương trình lưu trữ**, nghĩa là dữ liệu không chỉ phải đại diện cho số; nó có thể đại diện cho chính chương trình. Nó có nghĩa là bits có thể là bits, và bits cũng có thể là một chương trình. Ngày nay, chúng ta coi khái niệm này là đương nhiên; chúng ta tải một ứng dụng trên điện thoại và thậm chí không nghĩ về nó.

Lưu ý phụ: Vào thời điểm đó, khái niệm byte 8-bit ít chuẩn hơn. EDSAC sử dụng **words** two's complement nhị phân 35-bit để biểu diễn một phạm vi thông tin, từ số nguyên đến lệnh chương trình.

## Ý tưởng Lớn #1

> Trừu tượng hóa. Bất cứ thứ gì cũng có thể là số: dữ liệu, số, lệnh, v.v.

Trong suốt khóa học này, chúng ta sẽ thấy cách tất cả các lớp trong @fig-great-idea-abstraction được liên kết với nhau bởi ý tưởng này.

:::{figure} images/great-idea-abstraction.png
:label: fig-great-idea-abstraction
:width: 100%
:align: center
:alt: "Sơ đồ phân cấp của trừu tượng hóa máy tính với năm mức xếp chồng từ trên xuống dưới: chương trình ngôn ngữ cấp cao (đoạn code C), assembly RISC-V, ngôn ngữ máy nhị phân, sơ đồ khối kiến trúc phần cứng của datapath bộ xử lý và giao diện bộ nhớ, và mức mạch logic với sơ đồ cấp cổng. Mũi tên hoặc nhóm cho thấy mỗi lớp trên biên dịch hoặc tinh chỉnh thành lớp tiếp theo, kết thúc ở các cổng vật lý triển khai ISA."

Ý tưởng Lớn #1: Trừu tượng hóa. Bất cứ thứ gì cũng có thể là số (tức là, chuỗi bits): dữ liệu, lệnh, v.v.
:::

## Ngôn ngữ lập trình cấp cao

Trong @fig-great-idea-abstraction, chúng ta gọi C là ngôn ngữ cấp cao. Điều này sẽ có vẻ lố bịch với bạn: Rốt cuộc, Python là cấp cao, và C là cấp thấp. Một **ngôn ngữ lập trình cấp cao** là ngôn ngữ thường xử lý biến, vòng lặp, v.v., thay vì các chi tiết cụ thể của kiến trúc bộ xử lý.

Code ngôn ngữ cấp cao biên dịch xuống **assembly code**, sau đó được assemble thành **machine code** (tức là, bits). Chúng ta sẽ xem lại hai biểu diễn chương trình cấp thấp này rất sớm.

Ngày xưa, C là cách mạng. Nó tạo điều kiện cho hệ điều hành đầu tiên (Unix) không được viết bằng assembly. [Unix](https://en.wikipedia.org/wiki/Unix) là một OS di động, nghĩa là bạn có thể viết code cho OS và sau đó di chuyển nó sang một kiến trúc khác. Đó là một điều lớn! Nếu bạn viết bằng assembly, bạn phải tùy chỉnh nó cho máy cụ thể đó. Nhưng nếu bạn có thể biên dịch xuống từ một mức trên—đó là trừu tượng hóa, ý tưởng quan trọng nhất trong lớp này.

## Tại sao C?

:::{epigraph}
C không phải là ngôn ngữ "cấp rất cao", cũng không phải là ngôn ngữ "lớn", và không chuyên biệt cho bất kỳ lĩnh vực ứng dụng cụ thể nào. Nhưng sự thiếu hạn chế và tính tổng quát của nó làm cho nó thuận tiện và hiệu quả hơn cho nhiều nhiệm vụ so với các ngôn ngữ được cho là mạnh mẽ hơn.

-- [Kernighan and Ritchie (K&R), ấn bản thứ 2, 1988](http://9p.io/cm/cs/cbook/)
:::

**1. Học khoa học máy tính, không phải ngôn ngữ lập trình**. Tại Berkeley, chúng tôi tự hào có ba ngôn ngữ khác nhau trong các khóa học EECS cấp thấp: Python, Java, và C. Mục tiêu không bao giờ chỉ là dạy bạn một ngôn ngữ; mà là dạy bạn khoa học máy tính. Chúng tôi muốn bạn thành thạo để có thể quyết định ngôn ngữ nào để chọn cho một vấn đề cụ thể.

**2. Khai thác các tính năng cơ bản của kiến trúc.** Chúng ta học C trong lớp này vì sau đó chúng ta có thể viết các chương trình liên quan đến *quản lý bộ nhớ, song song, và nhiều hơn nữa*. Tuy nhiên, lưu ý rằng C đủ thấp đến silicon để bạn có thể chọc vào bits và có nhiều quyền kiểm soát hơn. Nó cũng làm cho việc lập trình khó khăn!

Chúng tôi sẽ cho bạn thấy bạn có thể gặp bao nhiêu rắc rối với con trỏ và rò rỉ bộ nhớ. C không được định kiểu mạnh, nên compiler không thể kiểm tra mọi thứ. Nó gần như giống một chiếc xe mà bạn đã tháo bỏ tất cả nhựa và tất cả những gì bạn có là dây... và bạn đang khởi động lại nó bằng cách nối lại những dây đó. Thú vị. Thêm sau.

**3. Sau hơn 40 năm, một trong những ngôn ngữ lập trình phổ biến nhất là C** (và các dẫn xuất của nó, C++, Objective C, C#). Chúng ta sẽ sớm thấy rằng Java đã thích nghi rất nhiều từ C—vì hầu hết các lập trình viên đã quen thuộc với cú pháp C.

## C: Tuyên bố từ chối trách nhiệm

C "cấp thấp" hơn nhiều so với các ngôn ngữ khác bạn đã thấy. Nó vốn không an toàn (thêm sau), [có các quy ước từ khóa tệ hại](https://inst.eecs.berkeley.edu/~cs61c/resources/HarveyNotesC1-3.pdf), phạm vi biến kỳ lạ, ..., danh sách còn dài.

Xem xét tất cả, C là một lựa chọn _hợp lý_ để dạy kiến trúc máy tính nhập môn. Tuy nhiên, khi bạn viết chương trình trong thế giới thực, bạn có các lựa chọn tốt hơn, tùy thuộc vào loại chương trình bạn muốn.

Nếu hiệu năng quan trọng:

* **Rust** giống như ngôn ngữ "C nhưng an toàn"—nếu bạn muốn sức mạnh của C nhưng với độ an toàn cao hơn. Khi chương trình C của bạn (về lý thuyết) đúng với tất cả các kiểm tra cần thiết, nó không nên nhanh hơn Rust.
* **Go**: "Đồng thời": Lập trình đồng thời thực tế tận dụng các bộ vi xử lý đa lõi hiện đại.

Nếu tính toán khoa học quan trọng:

* **Python** có các thư viện tốt để truy cập tài nguyên GPU cụ thể. Trình thông dịch Python được viết bằng C. Python có thể sử dụng Cython để gọi code C cấp thấp để làm việc. PyTorch, một thư viện Python phổ biến cho machine learning, sử dụng C++.
* **Spark** có thể quản lý nhiều máy khác song song.

### Ngôn ngữ C Liên tục Phát triển

Chuẩn ngôn ngữ lập trình C đã có nhiều bản sửa đổi quan trọng kể từ khi ra đời vào năm 1972.
Giống như Python 2 vs Python 3 – cùng ngôn ngữ, nhưng cú pháp/tính năng hơi khác.

Lịch sử, từ [StackOverflow](https://stackoverflow.com/questions/17206568/what-is-the-difference-between-c-c99-ansi-c-and-gnu-c):

* Trước 1989: K&R C (lưu ý K&R ấn bản 1 năm 1978, [ấn bản 2 năm 1988](http://9p.io/cm/cs/cbook/))
* 1989/1990: ANSI C
* 1999: C99
* 2011: C11
* 2017: C17
* 2024: C23

Chúng tôi sẽ dạy chuẩn C17 trong khóa học này, là phiên bản C được giả định bởi `gcc` trên các server khóa học của chúng tôi. `gcc` là gì? Bạn sắp tìm ra!

### Tự mình thử!

Tuyên bố từ chối trách nhiệm cuối cùng: bạn sẽ không học code C chỉ bằng cách xem video hoặc đọc sách. Bạn phải tự mình thử. Hãy mở một editor và bắt đầu!

Chúng tôi sẽ cố gắng hết sức để bao quát các khía cạnh chính của ngôn ngữ C trong các ghi chú khóa học này. Nhưng bạn vẫn nên có một tài liệu tham khảo C trong tay. Cuốn sách K&R ([Kernighan & Ritchie, ấn bản thứ 2, 1988](http://9p.io/cm/cs/cbook/) phiên bản có logo đỏ ANSI) là sách cần có cho mọi lập trình viên C.
