---
title: "Định luật Sắt về Hiệu năng Bộ xử lý"
---

(sec-iron-law-vi)=
## Mục tiêu Học tập

* Giải thích các thành phần phương trình của "định luật sắt" về hiệu năng bộ xử lý.
* Áp dụng "định luật sắt" về hiệu năng bộ xử lý để so sánh các kiến trúc bộ xử lý trên các chương trình chuẩn.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/4vLGgSuP27E
:width: 100%
:title: "[CS61C FA20] Lecture 21.2 - Pipelining I: Processor Performance Iron Law"
:::

::::

Một trong những chỉ số chính để đo hiệu năng bộ xử lý là thời gian thực thi một chương trình, còn được gọi là **thời gian thực thi chương trình**. Tuy nhiên, có nhiều tham số ảnh hưởng đến chỉ số hiệu năng này.

Để tách rời các tham số này, chúng ta chia chúng thành những gì chúng ta sẽ gọi là **"định luật sắt" về hiệu năng bộ xử lý**. Phương trình @eq-iron-law-vi hiển thị phương trình hiệu năng CPU cổ điển này:

```{math}
:enumerated: true
:label: eq-iron-law-vi
\frac{\text{thời gian}}{\text{chương trình}} = \frac{\text{lệnh}}{\text{chương trình}} \cdot \frac{\text{chu kỳ}}{\text{lệnh}} \cdot \frac{\text{thời gian}}{\text{chu kỳ}}
```

Phương trình này sử dụng phân số để mở rộng **thời gian thực thi chương trình** ($\text{thời gian}/\text{chương trình}$ đo bằng s, ms, ns, v.v.) thành tích của ba thành phần liên quan đến số lệnh, chu kỳ trên mỗi lệnh, và chu kỳ xung clock. Từ P&H 1.6:

> Công thức này đặc biệt hữu ích vì chúng tách riêng [ba] yếu tố chính ảnh hưởng đến hiệu năng. Chúng ta có thể sử dụng các công thức này để so sánh hai triển khai khác nhau hoặc đánh giá một phương án thiết kế thay thế nếu chúng ta biết tác động của nó lên ba tham số này.

### Lệnh trên mỗi chương trình

$$\frac{\text{lệnh}}{\text{chương trình}}$$

Thành phần đầu tiên của Phương trình @eq-iron-law-vi là số lệnh trong **chương trình chuẩn**. Chương trình chuẩn được xác định bởi:

* Đặc tả tác vụ. Ví dụ, thực hiện nén ảnh rất khác so với cố gắng chơi một ván cờ vây.
* Thuật toán và thời gian chạy, ví dụ: O(N{sup}`2`) hoặc O(N)[^big-o-vi].
* Ngôn ngữ lập trình. Các ngôn ngữ cấp cao cung cấp mô tả nhỏ gọn của tác vụ nhưng có thể "nổ ra" thành rất nhiều lệnh assembly; có thể hiệu quả hơn khi code bằng assembly, nhưng chúng ta đã thấy rằng code bằng assembly rất tẻ nhạt.
* Trình biên dịch, gắn liền chặt chẽ với ngôn ngữ lập trình. Một số trình biên dịch (hoặc tùy chọn trình biên dịch) tạo mã assembly với tập lệnh tối thiểu, hoặc mã assembly với tập tiềm năng tối thiểu về control hoặc data hazard.
* Kiến trúc tập lệnh (ISA). Hầu hết các kiến trúc RISC yêu cầu nhiều lệnh cấp assembly hơn các bộ xử lý loại CISC, mặc dù đây không thể là kỳ vọng "cố định" một cách riêng lẻ.

[^big-o-vi]: Về ký hiệu Big-O, xem CS 61B hoặc khóa học Cấu trúc Dữ liệu và Thuật toán tương đương.

### Chu kỳ trên Lệnh (CPI)

$$\frac{\text{chu kỳ}}{\text{lệnh}}$$

Thành phần thứ hai trong Phương trình @eq-iron-law-vi là **chu kỳ trên mỗi lệnh**, hay CPI. Từ P&H 1.6:

> Vì các lệnh khác nhau có thể mất lượng thời gian khác nhau tùy thuộc vào những gì chúng làm, CPI là trung bình của tất cả các lệnh được thực thi trong chương trình.

Ví dụ:
* Bộ xử lý RISC-V [single-cycle](#sec-single-cycle-vi): CPI = 1.
* [Bộ xử lý pipeline năm giai đoạn](#sec-five-stage-pipeline-vi) RISC-V được thảo luận trong chương này: CPI $\approx$ 1. CPI thường lớn hơn 1 một chút vì stall do [pipeline hazard](#sec-pipeline-hazards).
* **Lệnh phức tạp**: CPI >> 1, nếu được sử dụng trong chương trình chuẩn. Ví dụ, lấy một ISA chỉ định hàm C `strcpy` như `strcpy` cấp assembly. `strcpy` cấp assembly sẽ nhất thiết phát sinh nhiều chu kỳ hơn, giả sử, `add` cấp assembly.
* [Bộ xử lý superscalar](#sec-superscalar-vi): CPI < 1.

Để đo CPI, chạy bộ xử lý trên chương trình chuẩn. Trên cùng chương trình chuẩn, các bộ xử lý sẽ có CPI khác nhau vì sự khác biệt trong ISA và triển khai bộ xử lý. Tuy nhiên, CPI cung cấp một cách so sánh **hai triển khai khác nhau của cùng ISA**, vì chương trình chuẩn (và số lệnh trong chương trình) sẽ giống nhau.

### Chu kỳ xung clock

$$ \frac{\text{thời gian}}{\text{chu kỳ}}$$

Chu kỳ xung clock là thời gian cho một chu kỳ xung clock đơn và là nghịch đảo của tần số xung clock và được đo bằng giây (hoặc ms, ns, v.v.). Chu kỳ xung clock được xác định bởi:[^ee-105-vi]

* Triển khai bộ xử lý, ví dụ: [đường tới hạn](#sec-critical-path) qua logic tổ hợp.
* Độ trễ qua các cổng logic, được xác định bởi công nghệ (ví dụ: 5nm vs. 14nm vs. 28nm)
* Ngân sách công suất. Ví dụ, bộ xử lý desktop chạy ở tần số xung clock như 4GHz nhưng đốt nhiều năng lượng hơn nhiều so với điện thoại, chạy ở tần số gần 2-2.5 GHz. Công suất cũng được xác định bởi điện áp cung cấp. Điện áp thấp hơn giảm tốc độ transistor nhưng cải thiện hiệu quả năng lượng. Để biết thêm thông tin, đọc [phần tùy chọn](#sec-energy-efficiency-vi) về hiệu quả năng lượng.

[^ee-105-vi]: Để biết thêm thông tin, học các khóa học cao hơn như EE 105.

## Sử dụng Định luật Sắt

Từ P&H 1.6:

> Luôn ghi nhớ rằng thước đo hiệu năng máy tính hoàn chỉnh và đáng tin cậy duy nhất là thời gian. Ví dụ, thay đổi tập lệnh để giảm số lệnh có thể dẫn đến một tổ chức với thời gian chu kỳ xung clock chậm hơn hoặc CPI cao hơn bù đắp cho sự cải thiện về số lệnh. Tương tự, vì CPI phụ thuộc vào loại lệnh được thực thi, mã thực thi số lệnh ít nhất có thể không phải là nhanh nhất.

Ví dụ, hãy tưởng tượng cố gắng giảm phương sai trong thành phần lệnh trên mỗi chương trình. Điều này khó hơn vẻ ngoài và sẽ phụ thuộc vào những gì bạn muốn so sánh:

* So sánh hiệu năng bộ xử lý trên **hai tác vụ khác nhau**, như nén ảnh vs. cờ vây. Giữ ngôn ngữ lập trình và trình biên dịch giống nhau. ISA sẽ nhất thiết giống nhau nếu bạn sử dụng một bộ xử lý.
* So sánh **hai triển khai** của một ISA, giữ chương trình lệnh assembly giống nhau (điều này cố định tác vụ, thuật toán, ngôn ngữ lập trình, và trình biên dịch).
* Để so sánh hai ISA, giữ triển khai ngôn ngữ cấp cao của tác vụ giống nhau. Cố gắng tìm trình biên dịch với các tối ưu hóa tương tự cho cả hai ISA.

### Thông lượng Lệnh

**Thông lượng lệnh** có thể được đo như số lệnh hoàn thành trên đơn vị thời gian và là tích của nghịch đảo của hai thành phần cuối. Tương đương, nó là nghịch đảo của CPI nhân với tần số xung clock $f$.

```{math}
:enumerated: true
:label: eq-throughput-vi
\begin{aligned}
\text{thông lượng} &= \frac{\text{lệnh}}{\text{thời gian}} \\
&= \frac{\text{lệnh}}{\text{chu kỳ}} \cdot \frac{\text{chu kỳ}}{\text{thời gian}} \\
&= \frac{f}{\text{CPI}}
\end{aligned}
```

Chúng ta lưu ý rằng thông lượng lệnh được nhúng vào Định luật Sắt như tích của nghịch đảo của hai thành phần cuối của Phương trình @eq-iron-law-vi:

```{math}
\begin{aligned}
\frac{\text{thời gian}}{\text{chương trình}} &= \frac{\text{lệnh}}{\text{chương trình}} \cdot \frac{\text{chu kỳ}}{\text{lệnh}} \cdot \frac{\text{thời gian}}{\text{chu kỳ}} \\
&= \frac{\text{lệnh}}{\text{chương trình}} \cdot \frac{1}{\text{thông lượng}} \\
\end{aligned}
```

```{math}
:enumerated: true
:label: eq-throughput-2-vi
\begin{aligned}
\frac{1}{\text{thông lượng}} &= \frac{\text{chu kỳ}}{\text{lệnh}} \cdot \frac{\text{thời gian}}{\text{chu kỳ}}
\end{aligned}
```

::::{exercise} Kiểm tra Nhanh
:label: ex-processor-speed-vi
Giả sử chúng ta có hai bộ xử lý, A và B, và đánh giá hiệu năng trên tác vụ như nén ảnh. Bộ xử lý nào sẽ thực thi chương trình này nhanh hơn?

:::{table} Ví dụ So sánh Hiệu năng Bộ xử lý.
:label: tab-processor-speed-vi
:align: center

|  | Bộ xử lý A | Bộ xử lý B |
| :--- | :--- | :--- |
| # Lệnh | 1 triệu | 1.5 triệu |
| CPI Trung bình | 2.5 | 1 |
| Tốc độ xung clock | 2.5 GHz | 2 GHz |
:::

::::

::::{solution} ex-processor-speed-vi
:label: ex-processor-speed-sol-vi
:class: dropdown

Mặc dù thực thi nhiều lệnh hơn và có tốc độ xung clock chậm hơn, **Bộ xử lý B nhanh hơn cho tác vụ này**! CPI tốt hơn đáng kể của Bộ xử lý B giúp nó vượt qua các bất lợi khác.

:::{table} Ví dụ So sánh Hiệu năng Bộ xử lý.
:label: tab-processor-example-vi
:align: center

| Thước đo | Bộ xử lý A | Bộ xử lý B |
| :--- | :--- | :--- |
| # Lệnh | 1 triệu | 1.5 triệu |
| CPI Trung bình | 2.5 | 1 |
| Tốc độ xung clock, `f` | 2.5 GHz | 2 GHz |
| **Thời gian thực thi chương trình** | 1 ms | 0.75 ms |
| **Thông lượng lệnh (mỗi ns)** | 1 lệnh/ns | 2 lệnh/ns |
:::

Thời gian thực thi chương trình cho Bộ xử lý A:

$$10^6 \times 2.5 \times 1/(2.5 \times 10^9) = 1 \times 10^{-3} = 1 \text{ms}$$

Thời gian thực thi chương trình cho Bộ xử lý B:
$$(1.5 \times 10^6 )\times 1 \times 1/(2 \times 10^9) = 1.5/2 \times 10^{-3} = 0.75 \text{ms}$$

::::

::::{exercise} Kiểm tra Nhanh
:label: ex-cpi-tf-vi

Một chỉ số liên quan đến CPI là IPC, hay lệnh trên mỗi chu kỳ. Các phát biểu sau đây đúng hay sai? Chọn tất cả các phát biểu đúng.

1. IPC là số lệnh trung bình sẽ hoàn thành với một chu kỳ xung clock bổ sung.
1. CPI là số chu kỳ trung bình cần để hoàn thành một lệnh bổ sung.
1. CPI là số chu kỳ xung clock trung bình cần để thực thi mỗi lệnh.

::::

::::{solution} ex-cpi-tf-vi
:label: ex-cpi-tf-sol-vi
:class: dropdown

1. Đúng
1. Đúng
1. **Sai**. Nếu nhiều lệnh đang thực thi, định nghĩa CPI này sẽ "tính đôi" thời gian. Định nghĩa này mô tả **độ trễ lệnh**.
::::
