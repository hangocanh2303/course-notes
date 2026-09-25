---
title: "Các thành phần kiến trúc: Bộ xử lý, Thanh ghi và Bộ nhớ"
short_title: "Bộ xử lý, Thanh ghi và Bộ nhớ"
---

(sec-architecture-elements)=
## Mục tiêu học tập

* Hiểu rằng thanh ghi là bộ nhớ cực nhỏ, cực nhanh nằm bên trong bộ xử lý. Trong bố cục khái niệm của máy tính, bộ xử lý và bộ nhớ nằm ở vị trí riêng biệt.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/mIDxHr5_sxo
:width: 100%
:title: "[CS61C FA20] Lecture 07.2 - RISC-V Intro: Elements of Architecture: Registers"
:::

::::


::::{note} 🎥 Video bài giảng - Phân cấp bộ nhớ
:class: dropdown

:::{iframe} https://www.youtube.com/embed/Vo2WL9acC5M
:width: 100%
:title: "[CS61C FA20] Lecture 08.2 - RISC-V lw, sw, Decisions I: Data Transfer Instructions"
:::

Đến 5:00

::::

## Bố cục khái niệm của máy tính

Để học một ISA, trước tiên chúng ta phải hiểu @fig-von-neumann, cho thấy bố cục khái niệm của máy tính:

* Một **bộ xử lý** (ví dụ: **Đơn vị xử lý trung tâm**, hay CPU), chịu trách nhiệm tính toán. Bên trong bộ xử lý, có một **đơn vị điều khiển** và một **đường dữ liệu**. Các thành phần chính của đường dữ liệu là **thanh ghi** và đơn vị thực thi, thường được gọi là **Đơn vị Logic Số học** (ALU). Chúng ta sẽ thảo luận tất cả các chi tiết này [sớm](#sec-single-cycle).
* **Bộ nhớ chính**, chịu trách nhiệm lưu trữ dữ liệu dài hạn.
* **Thiết bị I/O**, tức là Thiết bị Input/Output như bàn phím, màn hình, v.v.

:::{figure} ../great-ideas/images/von-neumann.png
:label: fig-von-neumann
:width: 100%
:alt: "Sơ đồ khối của máy kiểu von Neumann: hộp bộ xử lý chứa điều khiển và đường dữ liệu với PC, thanh ghi, và ALU; bộ nhớ chính lưu trữ các byte; các mũi tên được gắn nhãn cho địa chỉ, dữ liệu đọc, dữ liệu ghi, và điều khiển đọc-ghi kết nối bộ xử lý và bộ nhớ, với các đường input và output riêng biệt đến bộ nhớ."

Bố cục máy tính cơ bản (Xem: [kiến trúc von Neumann](https://en.wikipedia.org/wiki/Von_Neumann_architecture)).
:::

## Thanh ghi

Quan trọng là, bộ xử lý được thiết kế để _nhanh_. Ví dụ, nếu bộ xử lý chạy ở 4 GHz, thì nó có thể thực thi các lệnh trên một số dữ liệu mỗi chu kỳ, hay cứ 0.25 ns (nano giây). Dữ liệu này cũng phải được đặt ở vị trí vật lý gần với bộ xử lý!

Xét rằng tốc độ ánh sáng (xấp xỉ $3.0 \times 10^8$ m/s), xác định về mặt vật lý tốc độ nhanh nhất để truy cập dữ liệu từ một vị trí vật lý nhất định. Nói cách khác, truy cập thứ gì đó cách khoảng 10 cm sẽ mất 0.3 ns (may mắn là hầu hết các chip tích hợp của chúng ta nhỏ hơn nhiều so với khoảng cách này). Tuy nhiên, trong tất cả các kiến trúc hiện đại, chúng ta có ít nhất hai phần cứng cho dữ liệu:

* **Thanh ghi**, nằm bên trong bộ xử lý. Các đối tượng phần cứng này có không gian hạn chế[^register-rv32] nhưng nhanh như chớp; bộ xử lý thực hiện các phép toán trên các dữ liệu này bằng đơn vị logic số học.
* **Bộ nhớ**, lớn hơn nhiều[^memory-laptop] và nằm bên ngoài bộ xử lý. Truy cập bộ nhớ thường được giả định mất khoảng 100 ns (@fig-3-locality). Bộ xử lý giao tiếp với bộ nhớ bằng cách phát hành địa chỉ để đọc hoặc ghi dữ liệu. Tín hiệu "enable" bổ sung đảm bảo chúng ta không vô tình thay đổi giá trị bộ nhớ khi chỉ muốn đọc; chúng ta thảo luận điều này sau.

[^register-rv32]: 32 x 4B = 128 B dữ liệu thanh ghi trên kiến trúc RV32.

[^memory-laptop]: 2-64 GB bộ nhớ trên laptop hiện đại.

:::{figure} ../great-ideas/images/3-locality.png
:label: fig-3-locality
:width: 100%
:alt: "Biểu đồ tương tự độ trễ ánh xạ các mức bộ nhớ từ thanh ghi qua cache, RAM, ổ đĩa, và băng từ đến độ trễ nano giây tăng dần, kết hợp với các ẩn dụ thời gian và khoảng cách theo quy mô con người như đầu so với khuôn viên, Sacramento, Pluto, và Andromeda."

Ý tưởng lớn 3: Nguyên lý cục bộ / Phân cấp bộ nhớ
:::

(sec-memory-hierarchy-early)=
## Phân cấp bộ nhớ

Mỗi ISA chỉ định một số thanh ghi phần cứng được xác định trước, định nghĩa cách mỗi thanh ghi nên được sử dụng cho việc thực thi lệnh. RISC-V định nghĩa 32 thanh ghi; đọc thêm trong [phần tiếp theo](#sec-rv32i-registers).

Nhớ lại hình ảnh về phân cấp bộ nhớ chính trong (@fig-3-memory-hierarchy):

:::{figure} ../great-ideas/images/3-memory-hierarchy.png
:label: fig-3-memory-hierarchy
:width: 100%
:alt: "Kim tự tháp phân cấp bộ nhớ từ lõi CPU, thanh ghi, và L1 qua L3 cache ở đỉnh hẹp, bộ nhớ chính DRAM ở giữa, và SSD, flash, ổ đĩa từ, và bộ nhớ ảo về phía đáy rộng, với các ghi chú về tốc độ, chi phí và dung lượng ở mỗi tầng."

Ý tưởng lớn 3: Nguyên lý cục bộ / Phân cấp bộ nhớ
:::

Ở trên cùng, chúng ta có lõi bộ xử lý với các thanh ghi của nó. Trên một chip riêng, chúng ta thường có bộ nhớ chính, được triển khai bằng DRAM (Dynamic Random Access Memory). Bạn có thể đã nghe về các loại như DDR3, 4, hoặc 5, hoặc High Bandwidth Memory (HBM). Mặc dù DRAM nhanh, nó không nhanh bằng thanh ghi. Ở mức giá hợp lý, bạn có thể có nhiều gigabyte với vài chục đô la, cung cấp dung lượng trung bình.

Vật lý quy định rằng **nhỏ hơn là nhanh hơn**. Khoảng cách giữa thanh ghi và bộ nhớ _lớn_ như thế nào? Trong khi bộ xử lý chỉ có khoảng 128 byte tổng lưu trữ thanh ghi, một laptop có thể có 2 đến 64 gigabyte DRAM, và một server có thể có một terabyte. Mặt khác, nếu chúng ta nghĩ về độ trễ, thanh ghi nhanh hơn DRAM khoảng 50 đến 500 lần.

Hãy quay lại phép tương tự độ trễ lưu trữ của Jim Gray[^great-ideas] trong @fig-3-locality. Nói cách khác–nếu việc lấy dữ liệu từ thanh ghi trong đầu bạn mất một phút, việc lấy dữ liệu từ bộ nhớ (chậm hơn 100 lần) sẽ giống như **lái xe đến Sacramento** để lấy một mảnh giấy bạn quên. Nếu khoảng cách là 500 lần, đó giống như lái xe đến Los Angeles và quay lại. Đó là một hình phạt khổng lồ chỉ để lấy một mục dữ liệu riêng lẻ!

[^great-ideas]: Tại một thời điểm nào đó, chúng ta sẽ quay lại và viết về Great Ideas (bài giảng giới thiệu của chúng ta). <!-- TODO -->

Chúng ta chỉ có một số lượng nhỏ thanh ghi–chúng cực kỳ nhanh và chia sẻ không gian quý giá với lõi bộ xử lý, làm cho chúng cực kỳ đắt tiền. Thiết kế một ISA (và một kiến trúc liên quan) do đó liên quan đến một điệu tango cẩn thận (?) để thực hiện các phép toán trên dữ liệu trong thanh ghi khi có thể, và dãn cách các chuyến đi hạn chế nhưng nặng nề đến bộ nhớ và ổ đĩa. Chúng ta sẽ xem lại điệu tango cẩn thận này trong một [phần sau](#sec-memory-hierarchy-revisited).
