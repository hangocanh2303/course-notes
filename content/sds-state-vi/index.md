---
title: "Tín Hiệu, Dạng Sóng, và Xung Nhịp"
subtitle: Tác giả John Wawrzynek, chỉnh sửa bởi Lisa Yan
---

(sec-signal-waveform-clock-vi)=
## Mục Tiêu Học Tập

* Diễn giải sơ đồ dạng sóng của tín hiệu xung nhịp.
* Xác định độ trễ lan truyền của mạch logic tổ hợp.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/EQYtPrBWvrU
:width: 100%
:title: "[CS61C FA20] Lecture 14.3 - Intro to Synchronous Digital Systems: Signals and Waveforms"
:::

::::

Trong [chương trước](#sec-intro-sds-vi), chúng ta đã thảo luận về một chip, được tạo thành từ dây dẫn và transistor (cùng với những thứ khác):

:::{figure} #fig-apple-a14
:alt: "Ảnh chụp die có ghi chú của một system-on-chip: các vùng chính được đánh dấu cho GPU, system cache, cụm CPU với bộ nhớ đệm L2, neural engine, và giao diện bộ nhớ DDR dọc theo các cạnh die."
Chip Apple A14 Bionic (nguồn: [Wikipedia](https://en.wikipedia.org/wiki/Apple_A14), [TechInsights](https://www.techinsights.com/blog/two-new-apple-socs-two-market-events-apple-a14-and-m1). @fig-apple-a14 trong [Giới thiệu SDS](#sec-intro-sds-vi).
:::

Bao quanh phần bên trong của chip — lõi — là một tập hợp các kết nối đến thế giới bên ngoài. Thường thì những kết nối này đi qua một số dây trong vỏ nhựa hoặc gốm đến bảng mạch in (PCB). Trong trường hợp hầu hết các máy tính, PCB này sẽ là bo mạch chủ. Một số kết nối này đi đến bộ nhớ chính và bus hệ thống. Khá nhiều chân được sử dụng để kết nối với nguồn điện. Nguồn điện lấy điện AC 110 Volt từ ổ cắm tường (do công ty điện cung cấp) và chuyển đổi thành điện DC điện áp thấp (thường trong khoảng từ 1 đến 5 volt, tùy thuộc vào chip cụ thể được sử dụng). Điện áp DC được đo tương đối so với điện thế đất (GND). Kết nối nguồn đến chip từ nguồn điện có hai loại; GND, và Điện áp DC (được ghi nhãn "Vdd").

Năng lượng được cung cấp bởi nguồn điện điều khiển hoạt động của bộ xử lý. Năng lượng được sử dụng để di chuyển điện tích từ nơi này sang nơi khác trong các mạch và trong quá trình đó bị tiêu tán dưới dạng nhiệt. Ngày nay, một bộ xử lý có thể sử dụng khoảng 10 watt — giống như một bóng đèn không quá sáng.

(sec-clock-vi)=
## Xung Nhịp

Một kết nối đặc biệt khác đến chip là **tín hiệu đầu vào xung nhịp** (clock input signal)[^sds-handout]. Tín hiệu xung nhịp được tạo ra trên bo mạch chủ và gửi đến chip, nơi nó được phân phối khắp bộ xử lý trên các dây nội bộ.

[^sds-handout]: Những ghi chú này được chuyển thể từ ghi chú của Giáo sư John Wawrzynek: [Tài liệu SDS](../resources/sds.pdf).

:::{tip} Tín hiệu xung nhịp là **nhịp tim của hệ thống**.

Nó điều khiển **thời gian** của dòng điện tích (và do đó là **thông tin**) xuyên suốt bộ xử lý và ngoài chip đến bộ nhớ và bus hệ thống.
:::

Nếu chúng ta có một đầu dò rất nhỏ, chúng ta có thể kiểm tra tín hiệu trên các dây xung nhịp, bằng cách nhìn vào mức điện áp trên dây.
Chúng ta sẽ thấy một vệt mờ, vì tín hiệu xung nhịp đang dao động ở tần số rất cao (khoảng **1 tỷ chu kỳ/giây hoặc 1GHz**, ngày nay). Nếu chúng ta có thể ghi lại nó trong một khoảnh khắc ngắn và vẽ ra, chúng ta sẽ thấy một cái gì đó như @fig-clock-waveform-vi.

:::{figure} images/clock-waveform.png
:label: fig-clock-waveform-vi
:width: 100%
:alt: "Dạng sóng xung nhịp vuông lý tưởng: các cao nguyên thấp và cao xen kẽ với chu kỳ T và các cạnh lên xuống sắc nét."

Dạng sóng cho tín hiệu xung nhịp nếu được ghi lại và vẽ theo thời gian.
:::

:::{note} **Tín hiệu xung nhịp** rất đều đặn.

Tín hiệu này là một "sóng vuông" lặp lại. Một chu kỳ xung nhịp (tick) là thời gian từ bất kỳ vị trí nào trên dạng sóng đến cùng vị trí đó trên phiên bản tiếp theo của dạng sóng.
:::

Trong CS 61C, chúng ta hầu như luôn đo chu kỳ xung nhịp từ cạnh lên của tín hiệu đến cạnh lên tiếp theo. Chúng ta sẽ thấy sau rằng tín hiệu xung nhịp là một tín hiệu rất quan trọng và có mặt trong tất cả các hệ thống số đồng bộ.

:::{note} Tại sao chúng được gọi là cạnh *lên* và *xuống* của tín hiệu xung nhịp?
:class: dropdown

Ý tưởng về cạnh *lên* và *xuống* đến từ việc tưởng tượng rằng tín hiệu có thể bắt đầu ở điện áp thấp và tăng lên điện áp cao, giữ ở đó nửa chu kỳ, sau đó giảm xuống điện áp thấp, nơi nó giữ thêm nửa chu kỳ nữa, v.v.
:::

## Tín Hiệu và Dạng Sóng

Tương tự như tín hiệu xung nhịp, chúng ta có thể xem xét các tín hiệu khác trong mạch. Ví dụ, giả sử chúng ta tìm thấy một mạch cộng. Làm sao chúng ta biết đó là mạch cộng? Chúng ta có thể đặt một đầu dò ghi tại đầu vào của nó và quan sát dạng sóng[^sds-handout] như @fig-adder-waveform-vi.

:::{figure} images/adder-waveform.png
:label: fig-adder-waveform-vi
:width: 70%
:alt: "Phía trên mô tả một khối đại diện cho mạch cộng với nhiều đầu vào bit đơn và nhiều đầu ra. Phía dưới mô tả dạng sóng xung nhịp, hiển thị các bit đầu vào của bộ cộng giữ mức ổn định giữa các chuyển đổi và thay đổi theo quan hệ với các cạnh xung nhịp."

Dạng sóng cho các tín hiệu trong mạch cộng nếu được ghi lại và vẽ theo thời gian.
:::

Nếu chúng ta có hai đầu dò, chúng ta có thể xem hai tín hiệu đồng thời. Hãy tưởng tượng rằng chúng ta dò hai bit ít có ý nghĩa nhất của một trong các đầu vào của bộ cộng. Dựa trên các dạng sóng được ghi lại, hiển thị phía trên tín hiệu xung nhịp trong @fig-adder-waveform-vi, chúng ta có thể đưa ra các quan sát sau.

1. Hầu hết thời gian, các tín hiệu ở một trong hai vị trí: **điện áp thấp** hoặc **điện áp cao**.

:::{note} Tại sao?
:class: dropdown

Theo quy ước, điện áp thấp được sử dụng để biểu diễn nhị phân `0` và điện áp cao được sử dụng để biểu diễn nhị phân `1`. Tại các thời điểm khác nhau, vị trí bit cụ thể của đầu vào bộ cộng có giá trị `0`, và vào các thời điểm khác, có giá trị `1`.
:::

2. Khi các tín hiệu ở mức điện áp cao hoặc thấp, đôi khi chúng không hoàn toàn đạt đến các cực điện áp.

:::{note} Tại sao?
:class: dropdown

Các mạch transistor tạo nên bộ cộng và các mạch con của nó có khả năng chống lại các sai số nhỏ về điện áp. Nếu tín hiệu *gần* với điện áp cao, nó được hiểu là logic `1`, và tương tự, nếu nó *gần* với 0 volt, nó được hiểu là logic `0`. Do đó, các sai số (hoặc độ lệch so với điện áp lý tưởng) không tích lũy khi tín hiệu di chuyển từ mạch này sang mạch khác.

Các mạch có thuộc tính nhận tín hiệu đầu vào có thể không lý tưởng, theo nghĩa không chính xác ở các cực điện áp, và tạo ra đầu ra chính xác ở các cực điện áp. **Thuộc tính phục hồi** này là một thuộc tính quan trọng của mạch số!
:::

3. Các thay đổi trong tín hiệu dường như tương ứng với các thay đổi trong tín hiệu xung nhịp, nhưng chúng không phải lúc nào cũng thay đổi mỗi khi chu kỳ xung nhịp thay đổi.

:::{note} Tại sao?
:class: dropdown

Trong các hệ thống đồng bộ, tất cả các thay đổi đều theo các cạnh xung nhịp. Tín hiệu xung nhịp được sử dụng xuyên suốt hệ thống để kiểm soát khi nào tín hiệu có thể nhận giá trị mới.
:::

### Dây Tín Hiệu

Các dây tín hiệu thường được nhóm lại với nhau để truyền các từ dữ liệu, như được hiển thị trong @fig-sig-wires-vi. Hình vẽ này cho thấy một tập hợp 4 dây được sử dụng để truyền một từ 4 bit. Các dạng sóng cho thấy các tín hiệu trên dây có thể thay đổi theo thời gian như thế nào. Các chữ cái viết thường được sử dụng để ghi nhãn các dây và tín hiệu riêng lẻ. Chữ cái viết hoa tương ứng được sử dụng để ghi nhãn tập hợp dây (đôi khi được gọi là *bus*). Vẽ một dạng sóng duy nhất đại diện cho tất cả các tín hiệu trên bus là khó vì, tại bất kỳ thời điểm nào, mỗi dây có thể có giá trị khác nhau. Nếu chúng ta cần hiển thị thời gian của các thay đổi tín hiệu trên bus, chúng ta có thể vẽ dạng sóng như @fig-sig-wires-vi. Ở đây, chúng ta vẽ tín hiệu bus đồng thời ở điện áp cao và thấp, và ghi nhãn mỗi phần đóng với giá trị trên bus tại thời điểm đó. Ngoài các giá trị trên bus, thời gian của các thay đổi là một thông tin quan trọng khác mà chúng ta có thể quan sát từ các dạng sóng như thế này.

:::{figure} images/signal-wires.png
:label: fig-sig-wires-vi
:width: 70%
:alt: "Bốn dạng sóng tín hiệu riêng lẻ cho x3, x2, x1, và x0, cộng với dạng sóng bus được vẽ với các dải chuyển đổi chéo, ghi nhãn giá trị 4 bit được giữ trên bus trong mỗi khoảng ổn định."

Sơ đồ dạng sóng của thay đổi tín hiệu theo thời gian dọc theo 4 dây.
:::

## Độ Trễ Mạch: Độ Trễ Lan Truyền

:::{tip} Độ Trễ Lan Truyền

Một thước đo độ trễ từ đầu vào đến đầu ra, nói chung, được gọi là **độ trễ lan truyền** (propagation delay). Độ trễ lan truyền xảy ra trong tất cả các mạch, bao gồm cả mạch logic tổ hợp.

Tuy nhiên, độ trễ lan truyền của một mạch, như bộ cộng, luôn nhỏ hơn chu kỳ xung nhịp. Chúng ta thảo luận thêm về điều này trong [phần sau](#sec-critical-path-vi).
:::

## Tóm Tắt: Hệ Thống Số Đồng Bộ

@fig-general-model-sds-vi hiển thị mô hình tổng quát cho các hệ thống đồng bộ của chúng ta:

:::{figure} images/general-model-sds.png
:label: fig-general-model-sds-vi
:width: 100%
:alt: "Phác thảo hệ thống đồng bộ: các khối logic tổ hợp xen kẽ với thanh ghi, phản hồi tùy chọn, và phân phối xung nhịp chỉ kết nối với các chân xung nhịp của thanh ghi."

Mô hình tổng quát cho SDS.
:::

* Mạch của chúng ta là một tập hợp các khối logic tổ hợp (khối CL) được phân tách bởi các thanh ghi.
* Các thanh ghi có thể nối tiếp nhau; các khối CL có thể nối tiếp nhau.
* Phản hồi là tùy chọn.
* Tín hiệu xung nhịp chỉ kết nối với đầu vào xung nhịp của thanh ghi. Các khối logic tổ hợp không có trạng thái và không được đồng bộ.
