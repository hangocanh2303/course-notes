---
title: "Các Chỉ số Hiệu năng"
---

## Mục tiêu Học tập

* Giải thích sự khác biệt giữa độ trễ (latency) và thông lượng (throughput).
* Đưa ra các ví dụ về các thước đo độ trễ và thông lượng trong kiến trúc máy tính.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/dTmErM78vD4
:width: 100%
:title: "[CS61C FA20] Lecture 21.1 - Pipelining I: Pipelining"
:::

::::

:::{warning} Ôn tập mạch pipeline

Chúng tôi đề cập ngắn gọn đến thông lượng và độ trễ khi thảo luận về thanh ghi trong mạch pipeline, được đề cập trong [phần này](#sec-pipelining-circuits) của đơn vị Hệ thống Số Đồng bộ. Chúng tôi khuyến nghị ôn lại phần đó trước khi tiếp tục.

:::

Trong phần thảo luận về [thời gian thực thi lệnh](#sec-instruction-timing-vi), chúng ta đã tính toán tần số xung clock nhanh nhất có thể cho datapath single-cycle của mình. Với [một số giả định](#tab-timing-steps-vi), chúng ta xác định rằng chu kỳ xung clock ngắn nhất 800 ps được xác định bởi đường tới hạn của các lệnh load. Điều này dẫn đến tần số xung clock tối đa 1.25 GHz, nghĩa là CPU có thể hoàn thành 1.25 tỷ lệnh mỗi giây.

Hãy xem xét một trong những ý tưởng lớn của khóa học này: **Đo lường và cải thiện hiệu năng**. Liệu chúng ta có thể cải thiện hiệu năng của bộ xử lý? Không rõ ràng chúng ta muốn nói gì với câu hỏi này. Có nhiều **thước đo hiệu năng** có thể:

* Thời gian thực thi của một lệnh đơn
* Thời gian thực thi của một công việc đơn
* Tổng số lệnh được thực thi trong một đơn vị thời gian cho trước
* Năng lượng trên mỗi lệnh
* Chi phí trong suốt vòng đời của một chip

## Độ trễ, Thông lượng, và Hiệu quả Năng lượng

Chúng ta xem xét ba thước đo hiệu năng trong khóa học này:

* **Độ trễ (Latency)**: Thời gian từ đầu đến cuối mà máy tính cần để hoàn thành một thứ gì đó: một lệnh, một chương trình/tác vụ, một tập hợp các tác vụ, v.v.
* **Thông lượng (Throughput)**: Máy tính thực thi các tác vụ, lệnh, v.v. hiệu quả như thế nào, một khi nó bắt đầu hoạt động.
* **Hiệu quả Năng lượng**: Bao nhiêu năng lượng được tiêu thụ cho mỗi tác vụ

Tùy thuộc vào đơn vị phân tích (trong ngữ cảnh kiến trúc máy tính, những gì chúng ta đang cố gắng tối ưu hóa), các đơn vị cho các thước đo này có thể khác nhau. Chúng ta thảo luận về sự căng thẳng này với một phép so sánh giao thông vận tải.

## Phép so sánh Giao thông Vận tải

Hãy xem xét hai phương tiện: một chiếc xe thể thao và một chiếc xe buýt, với các đặc điểm trong @tab-transportation-params-vi.

:::{table} Hai phương tiện với các đặc điểm khác nhau.
:label: tab-transportation-params-vi

| Danh mục | Xe Thể thao | Xe Buýt |
| :--- | :--- | :--- |
| Sức chứa Hành khách | 2 người | 50 người |
| Tốc độ Di chuyển | 200 mph | 50 mph |
| Mức tiêu thụ Nhiên liệu | 5 mpg | 2 mpg |

:::

Để so sánh phương tiện nào hoạt động "tốt hơn", chúng ta phải xác định một **tác vụ chuẩn**, chẳng hạn như:

> Vận chuyển 100 hành khách qua quãng đường 50 dặm.[^round-trip-vi]

[^round-trip-vi]: Giả sử 50 dặm khứ hồi, hoặc 50 dặm một chiều trong đó phương tiện ngay lập tức dịch chuyển trở lại khi hoàn thành mỗi chuyến. Cũng giả sử việc lên xuống hành khách, đổ xăng, giao thông, v.v., đều gây ra độ trễ không đáng kể.

Tiếp theo, hãy xem xét hai phương tiện hoạt động như thế nào trong tác vụ này qua nhiều thước đo hiệu năng khác nhau trong @tab-transportation-measures-vi:

:::{table} Hiệu năng phương tiện trên tác vụ chuẩn được chỉ định.
:label: tab-transportation-measures-vi
:align: center

| Thước đo | Xe Thể thao | Xe Buýt |
| :--- | :--- | :--- |
| Thời gian mỗi chuyến | 15 phút | 60 phút |
| Thời gian cho 100 hành khách | 750 phút (50 chuyến 2 người) | 120 phút (2 chuyến 50 người) |
| Hành khách mỗi giờ | 8 hành khách/giờ | 50 hành khách mỗi giờ |
| Gallon mỗi hành khách | 5 gallon | 0.5 gallon |

:::

Xe thể thao phải thực hiện nhiều chuyến, mặc dù thời gian mỗi chuyến riêng lẻ ngắn. Ngược lại, xe buýt hoàn thành tác vụ tổng thể nhanh hơn và với ít năng lượng hơn vì sức chứa cao hơn. Trong khi xe thể thao có **độ trễ** tốt hơn mỗi chuyến, xe buýt có thông lượng và hiệu quả năng lượng mỗi hành khách tốt hơn nhiều.

Hãy dịch phép so sánh này trở lại thế giới máy tính trong @tab-performance-compare-vi.

:::{table} Phép so sánh Giao thông Vận tải vs. Các Thước đo Hiệu năng trong Máy tính.
:label: tab-performance-compare-vi
:align: center

| Phép so sánh Giao thông | Thước đo | Trong Máy tính |
| :--- | :--- | :--- |
| Thời gian mỗi chuyến | **Độ trễ** | Độ trễ lệnh |
| Thời gian cho 100 hành khách | **Độ trễ** | **Thời gian thực thi chương trình** (ví dụ: thời gian để cập nhật màn hình) |
| Hành khách/giờ | **Thông lượng** | **Tổng tác vụ trên đơn vị thời gian** (ví dụ: # yêu cầu máy chủ được xử lý mỗi giờ) hoặc **thông lượng lệnh** (ví dụ: lệnh/giây) |
| Gallon mỗi hành khách | **Hiệu quả Năng lượng** | **Năng lượng mỗi tác vụ** (ví dụ: bạn có thể xem bao nhiêu phim mỗi lần sạc pin) |

:::

## Trong Khóa học Này

Trong đơn vị này, chúng ta sử dụng thảo luận về độ trễ và thông lượng để tạo động lực cho pipeline năm giai đoạn RISC-V, tăng thông lượng lệnh so với trường hợp single-cycle. Sau khi mô tả chi tiết kiến trúc này, chúng ta xem xét cách khắc phục [các hazard pipeline](#sec-pipeline-hazards)—khi các lệnh không thể thực thi đúng cách.

Sau đó, chúng ta thảo luận về độ trễ với "định luật sắt về hiệu năng bộ xử lý", xác định các thành phần cấp cao ảnh hưởng đến thời gian thực thi chương trình trên một bộ xử lý nhất định, bất kể cách triển khai.

Mặc dù không phải trọng tâm của lớp, chúng ta cũng phải đề cập đến các chỉ số ngành thực tế như chi phí trong suốt vòng đời. Xe thể thao có thể có giá \$64,000 với tuổi thọ dự kiến 8 năm, trong khi xe buýt có thể là \$400,000 với tuổi thọ dự kiến 12 năm.
