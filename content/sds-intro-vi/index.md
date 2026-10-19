---
title: "Hệ Thống Số Đồng Bộ"
subtitle: Trích từ tài liệu của John Wawrzynek
---

(sec-intro-sds-vi)=
## Mục Tiêu Học Tập

* Định nghĩa hệ thống số đồng bộ (Synchronous Digital System).
* Nhận biết transistor và dây dẫn là nền tảng của mạch số.
* Phân biệt hai loại mạch số: mạch logic tổ hợp và mạch nhớ.

::::{note} 🎥 Video Bài Giảng: Giới Thiệu
:class: dropdown

:::{iframe} https://www.youtube.com/embed/fbtUreUJxR4
:width: 100%
:title: "[CS61C FA20] Lecture 14.1 - Intro to Synchronous Digital Systems: Switches"
:::

đến phút 5:35

::::

Bây giờ, chúng ta sẽ đi **xuống dưới** đường cam trong Ý Tưởng Lớn #1. **Làm thế nào để thiết kế phần cứng thực thi mã máy?** Trong phần tiếp theo của khóa học, chúng ta sẽ hiện thực một bộ xử lý RISC-V dưới dạng **hệ thống số đồng bộ** (Synchronous Digital System), có khả năng thực thi các lệnh RISC-V.

:::{figure} #fig-great-idea-1
:width: 100%
:alt: "Sơ đồ trừu tượng hóa theo lớp: compiler, assembler, rồi mã máy nằm trên đường ISA, với kiến trúc phần cứng và mạch logic bên dưới. Phía bên phải hiển thị các ví dụ tương ứng từ C và assembly RISC-V qua mã nhị phân, sơ đồ khối bộ xử lý, và logic cổng NAND."

Ý Tưởng Lớn #1: Trừu Tượng Hóa.
:::

:::{hint} Tại sao cần học thiết kế phần cứng?

Để thực sự hiểu cách máy tính hoạt động, chúng ta cần hiểu toàn bộ ngăn xếp (stack), bao gồm cả tầng vật lý. Phần này sẽ giúp bạn có trực giác để trả lời những câu hỏi thường gặp:

* Tại sao máy tính của tôi chạy chậm?
* Tại sao pin nhanh hết?

Có giới hạn trong những gì bạn có thể làm với bộ xử lý tiêu chuẩn! Chúng tôi hy vọng phần này giúp bạn hiểu khả năng và giới hạn của phần cứng nói chung — và bộ xử lý nói riêng — để bạn biết khi nào cần chuyển sang phần cứng tùy chỉnh.

Các nguyên lý chúng tôi dạy trong CS 61C có thể vẫn áp dụng được trong 30 năm tới, ngay cả khi công nghệ nền tảng thay đổi. Để tìm hiểu về phần cứng hiện đại và nâng cao, hãy học các môn chuyên sâu như EECS 151: Digital Logic Design và CS 152: Computer Architecture.
:::

Cốt lõi của máy tính hiện đại là nhiều loại **mạch tích hợp** (integrated circuits), bản thân chúng được tạo thành từ nhiều hệ thống số đồng bộ kết nối với nhau (cùng với những thứ khác). Ví dụ, [chip Apple A14 Bionic](#fig-apple-a14) được sử dụng trong nhiều thiết bị iPad và iPhone của Apple vào đầu những năm 2020.

:::{figure} images/apple-a14.jpg
:label: fig-apple-a14
:width: 70%
:alt: "Ảnh chụp die có ghi chú của một system-on-chip: các vùng chính được đánh dấu cho GPU, system cache, cụm CPU với bộ nhớ đệm L2, neural engine, và giao diện bộ nhớ DDR dọc theo các cạnh die."

Chip Apple A14 Bionic (nguồn: [Wikipedia](https://en.wikipedia.org/wiki/Apple_A14), [TechInsights](https://www.techinsights.com/blog/two-new-apple-socs-two-market-events-apple-a14-and-m1))
:::

Hãy chú ý các khối được ghi chú trên bức ảnh. Chúng ta sẽ thảo luận về chúng trong phần sau của học kỳ. Dưới đây là thông số kỹ thuật của bộ xử lý này:

* 11,8 tỷ transistor
* Công suất tiêu thụ 5W
* Sáu lõi (ARM v8.5a) tốc độ lên đến 3.0 GHz:
  * Hai lõi hiệu năng cao, Bốn lõi tiết kiệm năng lượng
* CPU hỗ trợ dữ liệu 64-bit
* GPU để xử lý dữ liệu đồ họa
* NPU "Neural Engine" - phần cứng chuyên dụng cho mạng nơ-ron
* Bộ tăng tốc nhân ma trận AMX
* Bộ nhớ đệm L2: 4-8MB

Trong lớp này, chúng ta tập trung vào **đơn vị xử lý trung tâm** (CPU), hay bộ xử lý (phía bên trái của [hình bố cục máy tính](#fig-von-neumann)). Để hiểu cách một bộ xử lý hiện đại được xây dựng, chúng ta bắt đầu với định nghĩa các khối cơ bản.

## Dây Dẫn và Transistor

Tất cả mạch điện được tạo từ **dây dẫn** và **transistor**.[^others]

[^others]: Mạch điện còn có thể bao gồm các điện trở, tụ điện, cuộn cảm "ký sinh" **tương tự** (analog). Tuy nhiên, dây dẫn và transistor là cốt lõi trong thiết kế mạch **số**.

* **Dây dẫn** (tức là các nút điện) cung cấp tín hiệu điện và được dùng để biểu diễn các biến.
* **Transistor** là thiết bị bán dẫn dùng để khuếch đại hoặc **chuyển mạch** tín hiệu.

### Dây Dẫn

Trong mạch số, mỗi dây dẫn có thể mang một trong hai giá trị thông qua biểu diễn nhị phân của mức điện áp để tín hiệu `0` hoặc `1`. Ở mức cao, dây dẫn có dòng điện chạy qua đủ mạnh sẽ được kéo lên điện áp "cao" và biểu diễn `1`; dây dẫn được kéo xuống điện áp "thấp" biểu diễn `0`.

:::{figure} images/bin-wire.jpg
:label: fig-bin-wire
:width: 60%
:alt: "Sơ đồ trục điện áp: các dải riêng biệt cho vùng 0 và vùng 1 với các đường mức lý tưởng, và khoảng trống ở giữa được ghi là vùng trung gian không xác định giữa các mức nhị phân hợp lệ."

Điện áp thấp là `0`; điện áp cao là `1`.
:::

Đối với mạch số, chúng ta giữ tín hiệu **đơn giản** (tức là chỉ có giá trị nhị phân) và đẩy sự phức tạp sang cách chúng ta kết hợp các tín hiệu. **Bó dây** (bundles) biểu diễn các biến nhiều bit (@fig-bin-rep-signals).

:::{figure} images/binary-rep-signals.png
:label: fig-bin-rep-signals
:width: 60%
:alt: "Tám dây song song được đánh nhãn từ x7 đến x0 tương đương với một dây dày đánh nhãn X với dấu gạch chéo, biểu thị một bus 8-bit như cách viết tắt cho tám dây đơn bit."

(trái) tám dây, mỗi dây biểu diễn biến nhị phân $x_0$ đến $x_7$; (phải) một bó dây biểu diễn biến 8-bit $X$.
:::

:::{note} Tại sao không dùng biểu diễn tín hiệu vượt quá nhị phân?
:class: dropdown

Máy tính thời kỳ đầu biểu diễn tín hiệu dưới dạng thập phân — không phải nhị phân. Tuy nhiên, tất cả dây dẫn đều chịu nhiễu/không lý tưởng, và điều này trở nên tệ hơn ở kích thước nhỏ hơn.

Khi chip trở nên phức tạp hơn, dây dẫn trở nên nhỏ hơn. Mạch phân biệt giữa _hai_ đầu vào có thể được thiết kế đơn giản và đã mở rộng tốt theo Định luật Moore. Do đó, **biểu diễn nhị phân** cho tín hiệu tạo ra độ tin cậy thông qua khả năng chống nhiễu tốt.

:::

Một dây dẫn có thể mang các giá trị (nhị phân) khác nhau tại các thời điểm khác nhau. Dây dẫn được kéo lên điện áp thấp và cao bởi **transistor**.

### Transistor, Tóm Tắt

Transistor có trong tất cả thiết bị điện tử hiện đại: **mạch tích hợp** và **vi xử lý**.
Sự phát triển và thiết kế của transistor được thảo luận trong [Ý Tưởng Lớn #2: Định luật Moore](#fig-great-idea-2), cho thấy sự tăng trưởng theo thời gian của mật độ transistor trong (vi) xử lý.

:::{figure} ../great-ideas/images/2-moores-law.png
:label: fig-great-idea-2
:width: 100%
:alt: "Đồ thị thang log của số transistor trên mỗi die theo năm từ 1960 đến 2010, so sánh dữ liệu năm 1965 của Moore, chip nhớ, và vi xử lý Intel, với ảnh die nhỏ cho các bộ xử lý đầu và giữa những năm 1990."

Minh họa Định luật Moore theo thời gian.
:::

Đồng sáng lập Intel Gordon Moore[^go-bears]:

> "Giảm chi phí là một trong những điểm hấp dẫn lớn của điện tử tích hợp, và lợi thế chi phí tiếp tục tăng khi công nghệ phát triển hướng tới việc sản xuất các chức năng mạch lớn hơn trên một đế bán dẫn duy nhất."
> 
> -- _Electronics_, Tập 38, Số 8, ngày 19 tháng 4 năm 1965

[^go-bears]: Cử nhân Cal 1950!

Hiểu sâu về transistor nằm ngoài phạm vi khóa học. Với những ai tò mò, chúng tôi đã đưa vào một [phần bổ sung](#sec-transistors-vi) về transistor và công tắc, mặc dù chúng tôi khuyên bạn nên học các môn nâng cao như EE 105: Microelectronic Devices and Circuits.

Chúng ta sẽ tập trung vào việc nhớ rằng transistor rất quan trọng trong việc tạo thành các _khối xây dựng_ để thiết kế logic bộ xử lý. Nhờ hành vi chuyển mạch và khuếch đại, chúng có thể được kết hợp với nhau để thiết kế các mạch thực hiện **logic** trên các biến nhị phân (tức là dây dẫn) và lưu trữ **trạng thái** của các giá trị nhị phân cụ thể.

### Ví Dụ Mạch Số: NAND

Xét cổng NAND, nhận hai đầu vào nhị phân và tạo ra đầu ra nhị phân là `0` (thấp) chỉ khi cả hai đầu vào đều cao (`1`).

:::{figure} images/equivalence-nand.png
:label: fig-equivalence-nand
:width: 100%
:alt: "Năm cách biểu diễn NAND tương đương trên một hàng: sơ đồ transistor CMOS, khối NAND, định nghĩa logic y bằng NAND của a và b, ký hiệu cổng NAND tiêu chuẩn, và bảng chân trị hai đầu vào."

Cổng NAND. Chúng ta sẽ chủ yếu sử dụng các biểu diễn ở phía bên phải.
:::

Có năm cách biểu diễn NAND được hiển thị trong @fig-equivalence-nand. Trong khóa học này, chúng ta tập trung vào bốn cách biểu diễn bên phải nhất.

* (ngoài cùng bên trái): Sơ đồ mạch transistor của cổng NAND với đầu vào `a` và `b` và đầu ra `c`
* (ngay bên trái mũi tên): Sơ đồ khối của NAND, cũng với đầu vào `a` và `b` và đầu ra `c`
* (ngay bên phải mũi tên) Mô tả chức năng của NAND, trong đó `y = NAND(a, b)`
* (thứ hai từ phải) Cổng logic NAND, với đầu vào `a` và `b` và đầu ra `y`
* (ngoài cùng bên phải): Bảng chân trị cho NAND, với đầu vào `a` và `b` và đầu ra `y`

:::{hint} Độ Trễ Mạch

Một thuộc tính quan trọng của mạch số là chúng không thể tạo ra kết quả ngay lập tức.
Nếu chúng ta xem xét dạng sóng đầu vào và đầu ra của bất kỳ mạch số nào, chúng ta sẽ nhận thấy một độ trễ nhỏ từ khi đầu vào thay đổi đến khi đầu ra có kết quả mới. Độ trễ xuất hiện vì cần thời gian để điện tích di chuyển qua dây dẫn và transistor.

Một thước đo độ trễ từ đầu vào đến đầu ra, nói chung, được gọi là **độ trễ lan truyền** (propagation delay). Đọc [tài liệu SDS](https://inst.eecs.berkeley.edu/~cs61c/sp21/resources-pdfs/sds.pdf) để biết thêm thông tin.
:::

## Hệ Thống Số Đồng Bộ

Phần cứng bên dưới hầu hết mọi bộ xử lý là một **Hệ Thống Số Đồng Bộ** (Synchronous Digital System).

:::{note} Hệ Thống Số Đồng Bộ

* **Đồng bộ** (Synchronous): Tất cả các hoạt động được phối hợp bởi một xung nhịp (clock) trung tâm. Xung nhịp bộ xử lý là "nhịp tim" của hệ thống. Một **chu kỳ** (cycle) là khoảng thời gian giữa các nhịp đập.
* **Số** (Digital): Biểu diễn tất cả các giá trị bằng các giá trị rời rạc, tức là các chữ số nhị phân `1` và `0`.

Hệ thống số đồng bộ kết hợp các **mạch** với nhau để tạo ra các hệ thống logic số phức tạp. Về cốt lõi, các mạch này bản thân được tạo thành từ **dây dẫn** và **transistor**, mà chúng ta thảo luận ngắn gọn bên dưới.

:::

Chúng ta sẽ thảo luận về xung nhịp sau. Hiện tại, hãy biết rằng xung nhịp cũng có thể được biểu diễn qua tín hiệu "sóng vuông" số, dao động giữa `1` và `0` theo chu kỳ. Ngày nay, xung nhịp hoạt động trong phạm vi 3-4GHz, nghĩa là có bốn tỷ chu kỳ của sóng vuông này mỗi giây. Chúng ta sẽ thấy sau cách xung nhịp hỗ trợ **trạng thái**.

:::{note} Nhấp để xem các lựa chọn thay thế cho SDS
:class: dropdown

* **Hệ thống bất đồng bộ** (Asynchronous) phải phối hợp cục bộ các hành động và giao tiếp giữa các thành phần; khó thiết kế/gỡ lỗi hơn nhiều.
* **Mạch tương tự** (Analog) sử dụng điện áp/dòng điện để biểu diễn phạm vi giá trị liên tục. Ngày nay, ngay cả nhiều mạch tương tự cũng sử dụng thiết kế số đồng bộ bằng cách dùng bộ chuyển đổi tương tự sang số (ADC), và ngược lại. Học EE 105 để biết thêm!

:::

## Các Loại Mạch Số

Hệ Thống Số Đồng Bộ bao gồm hai loại mạch cơ bản: mạch logic tổ hợp và mạch có trạng thái.

**Mạch logic tổ hợp** (Combinational logic circuits) nhận đầu vào và kết hợp chúng để tạo ra đầu ra mới sau một độ trễ nhỏ. Các thành phần cốt lõi của mạch logic tổ hợp là **cổng logic** (logic gates).

Mạch logic tổ hợp **không có bộ nhớ** về các đầu vào hoặc đầu ra trước đó. Đầu ra là hàm của không gì khác ngoài đầu vào — giống như hàm toán học, ví dụ: $y = f(x)$. Mạch logic tổ hợp không có cách lưu trữ thông tin từ lần gọi này sang lần tiếp theo và không tạo ra bất kỳ hiệu ứng phụ nào.

Mạch logic tổ hợp được sử dụng xuyên suốt trong các hiện thực bộ xử lý. Chúng cung cấp tất cả các chức năng cần thiết để thực thi lệnh, bao gồm tất cả các phép toán số học và logic, tăng giá trị bộ đếm chương trình, xác định khi nào cần rẽ nhánh, tính toán giá trị PC mục tiêu mới, v.v. Không tính diện tích tiêu thụ bởi cache, _hầu hết diện tích trên chip vi xử lý_ được dùng cho các khối logic tổ hợp.

**Phần tử trạng thái** (State elements), hay mạch nhớ, là các mạch lưu trữ thông tin. Không giống như mạch logic tổ hợp, các mạch này ghi nhớ đầu vào của chúng (trong CL, đầu vào chỉ được dùng để tạo đầu ra, nhưng không bao giờ được ghi nhớ). Thành phần cốt lõi của mạch có trạng thái là **phần tử trạng thái**. Chúng ta sẽ thảo luận chi tiết về một phần tử trạng thái như vậy, **thanh ghi** (register), trong phần sau.
