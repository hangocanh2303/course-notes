---
title: "Pipelining qua Ví dụ Giặt đồ"
---

(sec-laundry-analogy-vi)=
## Mục tiêu Học tập

* Sử dụng giặt đồ làm phép so sánh để hiểu kiến trúc pipeline.
* So sánh độ trễ và thông lượng của kiến trúc tuần tự và pipeline.
* Hiểu rằng pipelining cải thiện hiệu năng bằng cách tăng thông lượng lệnh, trái ngược với việc giảm thời gian thực thi của một lệnh riêng lẻ. Thông lượng lệnh là chỉ số quan trọng vì các chương trình thực thi hàng tỷ lệnh.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/y52uRSQGYms
:width: 100%
:title: "[CS61C FA20] Lecture 21.4 - Pipelining I: Introduction to Pipelining"
:::

::::

Khái niệm **pipelining** sẽ tăng thông lượng tính toán của chúng ta. Trước khi nhảy vào pipeline RISC-V, hãy bắt đầu với một ví dụ thực tế về giặt đồ. Bất kỳ ai đã giặt nhiều đồ đều đã sử dụng pipelining một cách trực giác. Bốn bước giặt đồ được hiển thị trong @fig-laundry-setup:
:::{figure} images/laundry-setup.png
:label: fig-laundry-setup
:width: 100%
:alt: "Sơ đồ thiết lập phòng giặt ánh xạ máy giặt, máy sấy, và các tác vụ gấp đồ thành các giai đoạn kiểu pipeline trong đó mỗi giai đoạn mất 30 phút tuần tự."

Phép so sánh giặt đồ.
:::

1. **Giai đoạn Máy giặt** (30 phút). Đặt một mẻ quần áo bẩn vào máy giặt.
1. **Giai đoạn Máy sấy** (30 phút). Khi máy giặt xong, đặt mẻ ướt vào máy sấy.
1. **Giai đoạn "Gấp đồ"** (30 phút). Khi máy sấy xong, đặt mẻ khô lên bàn và gấp.
1. **Giai đoạn "Cất đồ"** (30 phút). Khi gấp xong, cất quần áo vào ngăn kéo.

Giả sử **tác vụ chuẩn giặt đồ** của chúng ta liên quan đến bốn người trong tiệm giặt, hoặc phòng giặt chung[^laundry-assumptions-vi], tất cả đều cần giặt đồ vào tối Chủ nhật. Trong @fig-laundry-setup, mỗi người được biểu thị bằng túi giặt với số của họ (1 đến 4).

[^laundry-assumptions-vi]: Phòng giặt chung có một máy giặt duy nhất, một máy sấy duy nhất, một quầy gấp đồ duy nhất, và một bộ ngăn kéo vô hạn nhưng đơn lẻ nằm bí ẩn trong chính căn phòng.

## Giặt đồ Tuần tự

Hãy xem xét cách tiếp cận **tuần tự** để sử dụng giặt đồ, trong đó chỉ một người có thể sử dụng phòng tại một thời điểm,[^sequential-assumptions-vi] như được hiển thị trong @fig-laundry-sequential.
Người 1 bắt đầu lúc 6pm và hoàn thành cả bốn bước lúc 8pm; sau đó, Người 2 bắt đầu và hoàn thành 8-10pm, tiếp theo là Người 3, và cuối cùng Người 4, không hoàn thành cho đến 2am.

[^sequential-assumptions-vi]: Hãy tưởng tượng có một chìa khóa duy nhất cho phòng giặt chung. Khi bắt đầu giai đoạn giặt, bạn ngay lập tức lấy chìa khóa, mở khóa phòng và vào, và khóa lại phòng để không ai khác có thể vào. Sau đó ngay cuối giai đoạn gấp, bạn ngay lập tức mở khóa phòng, ra ngoài và khóa lại, và trả lại chìa khóa.

:::{figure} images/sequential-laundry.png
:label: fig-laundry-sequential
:width: 100%
:alt: "Lịch trình giặt đồ tuần tự trong đó mẻ của một người hoàn thành hoàn toàn tất cả các giai đoạn (máy giặt, máy sấy, gấp, và cất) trước khi người tiếp theo bắt đầu."

Timeline cho giặt đồ tuần tự.
:::

Cách triển khai này mất **8 giờ** chỉ cho **4 mẻ** giặt. Nếu chúng ta nhìn kỹ bốn tài nguyên chính, chúng hầu hết bị lãng phí; sau cùng, mỗi người độc quyền sử dụng một trong máy giặt, máy sấy, quầy và ngăn kéo khi họ tiến qua bốn bước. Máy giặt, ví dụ, chỉ được sử dụng 30 phút trong toàn bộ hai giờ.

Chúng ta chắc chắn có thể làm tốt hơn bằng cách cố gắng thực hiện các bước này đồng thời, trong đó mỗi người thực hiện một bước khác nhau. Đây là trực giác đằng sau **pipelining**.

## Giặt đồ Pipeline

Bây giờ hãy xem xét cách tiếp cận **pipeline** để sử dụng giặt đồ,[^pipelined-assumptions-vi] trong đó ngay khi Người 1 xong máy giặt và lấy quần áo ướt ra để chuyển sang máy sấy, Người 2 có thể đặt quần áo bẩn của riêng họ vào máy giặt, và cứ thế. Trong @fig-laundry-parallel, Người 1 vẫn bắt đầu lúc 6pm và hoàn thành lúc 8pm, nhưng bây giờ Người 4 có thể bắt đầu sớm hơn nhiều lúc 7:30pm và hoàn thành lúc 9:30pm. Phù!

[^pipelined-assumptions-vi]: Giả sử cửa không khóa bây giờ. Chủ tiệm giặt đã hiểu ra.

:::{figure} images/parallel-laundry.png
:label: fig-laundry-parallel
:width: 50%
:alt: "Lịch trình giặt đồ pipeline với công việc máy giặt, máy sấy, và gấp đồ chồng chéo giữa các khách hàng khác nhau để tối ưu thời gian. Người tiếp theo có thể bắt đầu giai đoạn máy giặt (giai đoạn 1) ngay khi người trước họ chuyển sang giai đoạn máy sấy (giai đoạn 2)."

Timeline cho giặt đồ pipeline.
:::

Cách triển khai này chỉ mất **3.5 giờ** cho cùng **4 mẻ** giặt. Tại các khoảng 30 phút khác nhau, khách hàng chia sẻ bốn tài nguyên vì họ đang hoàn thành các bước khác nhau của quy trình giặt (@fig-different-phases). Từ 7:30-8pm, cả bốn tài nguyên đều được khách hàng sử dụng.

:::{figure} images/customers-different-phases.png
:label: fig-different-phases
:width: 100%
:alt: "Hai góc nhìn đồng bộ của giặt đồ pipeline: bên trái, một timeline hiển thị bốn khách hàng mỗi người bắt đầu giai đoạn máy giặt ngay khi người trước họ bắt đầu giai đoạn máy sấy, và bên phải, một tiến trình tập trung của các giai đoạn giặt theo thứ tự cho một khoảng 30 phút cho trước, hiển thị cách mỗi giai đoạn trong 30 phút đó được sử dụng bởi một khách hàng khác."

Hai góc nhìn của giặt đồ pipeline: Theo thời gian (trái) so với ảnh chụp phòng giặt trong một khoảng thời gian cho trước (ví dụ: 7-7:30pm).
:::

## Giặt đồ: Độ trễ vs. Thông lượng

Từ P&H 4.6:

> Nghịch lý pipelining là thời gian từ khi đặt một chiếc tất bẩn vào máy giặt cho đến khi nó được sấy khô, gấp, và cất đi không ngắn hơn với pipelining; lý do pipelining nhanh hơn cho nhiều mẻ là vì mọi thứ hoạt động song song, nên nhiều mẻ hơn được hoàn thành mỗi giờ. Pipelining cải thiện \*thông lượng\* của hệ thống giặt đồ. Do đó, pipelining sẽ không giảm thời gian để hoàn thành một mẻ giặt, nhưng khi chúng ta có nhiều mẻ giặt phải làm, sự cải thiện về thông lượng giảm tổng thời gian để hoàn thành công việc.

@tab-laundry-compare-vi hiển thị các thước đo hiệu năng trên tác vụ chuẩn giặt đồ:

:::{table} Độ trễ vs. Thông lượng: Giặt đồ Tuần tự vs. Song song.
:label: tab-laundry-compare-vi
:align: center

| Thước đo | Phép so sánh Giặt đồ | Tuần tự | Pipeline |
| :--- | :--- | :--- | :--- |
| Thời gian thực thi chương trình | Thời gian hoàn thành cả 4 mẻ | 8 giờ | 3.5 giờ |
| Độ trễ lệnh | Thời gian hoàn thành một mẻ | 2 giờ | 2 giờ |
| Thông lượng lệnh | Số mẻ trung bình\* mỗi 30 phút | 0.25 | 1\*|
:::

Lưu ý (\*): Thông lượng là xấp xỉ cho tác vụ 4 khách hàng nhỏ bé. Trong trường hợp này, 4 mẻ hoàn thành trong bảy khoảng 30 phút, hoặc xấp xỉ 0.57 mẻ mỗi khoảng 30 phút. Tuy nhiên, nếu chúng ta có nhiều khách hàng, pipeline được "lấp đầy" hoàn toàn, trong đó chính xác 1 khách hàng sẽ hoàn thành mỗi 30 phút. Chúng tôi báo cáo trạng thái ổn định này trong @tab-laundry-compare-vi.

<!--
:::{figure} images/ipc-timeline.png
:label: fig-ipc-timeline
:width: 100%
:alt: "IPC-style timeline showing one completed customer per cycle after initial pipeline fill."

IPC Timeline diagram showing one customer per "cycle" in laundry analogy.
:::
-->

(sec-processor-hl-vi)=
## Bộ xử lý RISC-V: Tuần tự và Pipeline

Hãy dịch phép so sánh giặt đồ này trở lại bộ xử lý RISC-V.

Hãy xem xét cách một lệnh như `add t0 t1 t2` truy cập phần cứng trong **bộ xử lý single-cycle**. Giống như khi chúng ta tính [thời gian lệnh](#sec-instruction-timing-vi), @fig-single-cycle-processor-hl biểu diễn năm giai đoạn[^pedantic-vi] với [đơn vị phần cứng chính](#tab-timing-steps-vi).

[^pedantic-vi]: Bước? Giai đoạn? Stage? Xem [ghi chú này](#sec-phases-stages) từ khi chúng ta thảo luận về năm bước thực thi lệnh RISC-V.

:::{figure} images/single-cycle-processor-hl.png
:label: fig-single-cycle-processor-hl
:width: 100%
:alt: "Biểu tượng giai đoạn bộ xử lý single-cycle cho một lệnh add với tô màu đọc và ghi theo tài nguyên phần cứng. Các giai đoạn bao gồm IF với khối IMEM, ID với khối Reg, EX với ALU, MEM với khối DMEM, và WB với khối Reg."

Sơ đồ cấp cao của bộ xử lý single-cycle thực thi lệnh `add t0 t1 t2`. Chúng ta áp dụng biểu diễn đồ họa của mỗi [năm giai đoạn](#sec-five-steps-vi), trong đó mỗi biểu tượng đại diện cho tài nguyên phần cứng chính được truy cập trong giai đoạn đó. Tô màu minh họa nếu đơn vị được đọc (tức là, nửa bên phải được tô cho IMEM trong `IF` và RegFile trong `ID`) hoặc đơn vị được ghi (tức là, nửa bên trái được tô cho RegFile trong `WB`). Đối với lệnh R-Type, MEM trong suốt vì `add` không truy cập bộ nhớ dữ liệu.
:::

Giả sử chúng ta sử dụng bộ xử lý single-cycle để thực thi ba lệnh theo tuần tự.

```bash
add t0 t1 t2
or  t3 t4 t5
lw  t6 8(t7)
```

Trong **CPU single-cycle**, _chỉ một lệnh có thể truy cập bất kỳ tài nguyên nào trong một chu kỳ xung clock, theo tuần tự_. Như trong @fig-sequential-processor, thực thi ba lệnh này sẽ mất thời gian $3 \cdot t_{cycle}$, trong đó $t_{instruction} = t_{cycle}$.

:::{figure} images/sequential-processor.png
:label: fig-sequential-processor
:width: 100%
:alt: "Timeline bộ xử lý single-cycle cho ba lệnh, mỗi lệnh chiếm toàn bộ chu kỳ và chờ bắt đầu cho đến khi lệnh trước đã hoàn thành tất cả các giai đoạn."

Sử dụng bộ xử lý single-cycle cho ba lệnh. Thời gian lệnh $t_{instruction}$ bằng độ dài của một chu kỳ xung clock, $t_{cycle}$.
:::

Trong bộ xử lý **pipeline**, mặt khác, nhiều lệnh truy cập các tài nguyên khác nhau trong một chu kỳ duy nhất. Theo phép so sánh giặt đồ, chúng ta muốn thiết kế một bộ xử lý pipeline có thể tạo ra timeline sau trong @fig-pipelined-processor.

:::{figure} images/pipelined-processor.png
:label: fig-pipelined-processor
:width: 100%
:alt: "Timeline bộ xử lý pipeline năm giai đoạn cho ba lệnh chồng chéo qua các giai đoạn IF đến WB."

Sử dụng bộ xử lý RISC-V pipeline cho ba lệnh. Trong chu kỳ đầu tiên, `add` ở giai đoạn `IF`. Trong chu kỳ thứ hai, `or` ở giai đoạn `IF`, trong khi `add` đã chuyển sang giai đoạn `ID`. Trong chu kỳ thứ ba, `lw` ở giai đoạn `IF`, trong khi `or` và `add` đã chuyển sang các giai đoạn `ID` và `EX` tương ứng. Mỗi lệnh bây giờ mất **năm chu kỳ** để thực thi, nhưng chu kỳ xung clock có thể được định thời theo thời lượng của giai đoạn dài nhất.
:::

Nếu chúng ta xem xét lệnh `add t0 t1 t2` và cách nó truy cập bộ xử lý _pipeline_, chúng ta quan sát thấy điều gì đó như @fig-pipelined-processor-hl.

:::{figure} images/pipelined-processor-hl.png
:label: fig-pipelined-processor-hl
:width: 100%
:alt: "Sơ đồ sử dụng tài nguyên bộ xử lý pipeline cấp cao cho add hiển thị hoạt động phần cứng theo từng giai đoạn."

Sơ đồ cấp cao của bộ xử lý pipeline thực thi lệnh `add t0 t1 t2`. Tô màu được mô tả trong chú thích của @fig-single-cycle-processor-hl.
:::

Ghi chú:

* Chúng ta có [thanh ghi pipeline](#sec-pipelining-circuits) phân tách mỗi giai đoạn, nghĩa là **mỗi giai đoạn pipeline mất một chu kỳ xung clock**. Chúng ta thảo luận thiết kế này chi tiết hơn trong [phần tiếp theo](#sec-five-stage-pipeline-vi).
* Lệnh `add` bây giờ mất **năm chu kỳ** để thực thi, nên $t_{instruction} = 5 \cdot t_{cycle}$.
* Ba lệnh cùng nhau bây giờ mất thời gian $7 \cdot t_{cycle}$ để thực thi.
* Mỗi chu kỳ _ngắn hơn nhiều_, nên tổng thời gian thực thi chương trình ngắn hơn bộ xử lý single-cycle.

:::{hint} Xung clock của bộ xử lý pipeline

Trong bộ xử lý pipeline, **vẫn có một xung clock**, được sử dụng cho tất cả các giai đoạn. Chu kỳ xung clock bị giới hạn bởi thời lượng của giai đoạn _chậm nhất_.
:::

Tất cả các giai đoạn trong bộ xử lý pipeline mất _cùng_ độ dài. Hãy giả sử cùng thời lượng giai đoạn từ bộ xử lý single-cycle như @tab-timing-steps-vi từ [thời gian lệnh](#sec-instruction-timing-vi). Trong pipeline, mặc dù phần cứng trong các giai đoạn `ID` và `WB` có thể tính toán giá trị nhanh hơn, một xung clock đơn nghĩa là chúng sẽ mất cùng thời gian như các giai đoạn khác.

:::{table} Thời gian bước: một giai đoạn trong bộ xử lý single-cycle vs. một stage trong bộ xử lý pipeline năm giai đoạn. Trong bộ xử lý single-cycle, cả năm bước phải xảy ra trong một chu kỳ, nên độ dài giai đoạn là thời gian hoạt động. Trong bộ xử lý pipeline, mỗi bước được định thời theo độ dài của một stage, phải là độ dài của một chu kỳ xung clock (vì [thanh ghi pipeline](#sec-five-stage-pipeline-vi)).
:label: tab-timing-steps-compare-vi

| Bước | Thời gian hoạt động | Thời lượng giai đoạn Single-cycle | Thời lượng stage Pipeline |
| :-- | :-- | :-- | :-- |
| Instruction Fetch (`IF`) | 200 ps | 200 ps | 200 ps |
| Instruction Decode (`ID`) | 100 ps | 100 ps | 200 ps |
| Execute (`EX`) | 200ps | 200 ps | 200 ps |
| Memory Access (`MEM`) | 200ps | 200 ps | 200 ps |
| Write Back (`WB`) | 100ps | 100 ps | 200 ps |
:::

Hãy so sánh hiệu năng này trong @tab-processor-compare-vi.

:::{table} So sánh hiệu năng: Bộ xử lý single-cycle vs. bộ xử lý pipeline 5 giai đoạn.
:label: tab-processor-compare-vi
:align: center

| So sánh | Tuần tự | Pipeline |
| :--- | :--- | :--- |
| Sơ đồ cấp cao | INSERT | INSERT |
| Thời gian bước | (xem @tab-timing-steps-compare-vi) <br/> 200 ps (`IF`, `EX`, `MEM`); 100 ps (`ID`, `WB`) | 200 ps (tất cả stage cùng độ dài) |
| Chu kỳ xung clock, $t_{cycle}$ | 200 + 100 + 200 + 200 + 100 = 800 ps | 200 ps |
| Độ trễ lệnh<br/>$t_{instruction}$ | $t_{cycle}$ = 800 ps | $5 \cdot t_{cycle}$ = 1000 ps |
| Tần số xung clock | 1/800 ps = 1.25 GHz | 1/200 ps = 5 GHz |

:::

### Tăng tốc Pipeline

Dựa trên @tab-processor-compare-vi, **tăng thông lượng**, hay tăng tốc pipelining, gần với tỷ lệ thời gian giữa các lệnh. Một cách tiếp cận là so sánh thời gian bộ xử lý single-cycle và pipeline trên một chương trình chuẩn như 1 triệu lệnh `lw`[^no-hazards-vi]:

[^no-hazards-vi]: Chúng ta giả sử không có [load data hazard](#sec-data-hazards-load) trong chương trình này.

* Bộ xử lý single-cycle mất $10^{6} \cdot t_{cycle}$ = 800,000,000 ps.
* Bộ xử lý pipeline mất $10^{6} \cdot t_{cycle} + 4 \cdot t_{cycle}$ = 200,000,800 ps, trong đó có bốn chu kỳ để chứa thời gian "khởi động" của pipeline (nơi không phải tất cả các giai đoạn của pipeline được sử dụng).

Nếu chúng ta lấy tỷ lệ của tổng thời gian thực thi:

$$ \dfrac{800,000,000 \text{ps}}{200,000,800 \text{ps}} \approx \dfrac{800 \text{ps}}{200 \text{ps}} = 4$$

:::{warning} Tại sao không tăng tốc 5x?

Các giai đoạn được cân bằng không hoàn hảo, và có một số chi phí bổ sung cho pipelining, mà chúng ta sẽ thấy trong [chương tiếp theo](#sec-pipeline-hazards)).

:::

(sec-ilp-vi)=
:::{hint} Song song mức lệnh

Pipelining là một phương tiện **thực thi các lệnh song song** sử dụng cùng phần cứng bộ xử lý.

Từ P&H 4.6:
> Pipelining cải thiện hiệu năng bằng cách \*tăng thông lượng lệnh, trái ngược với việc giảm thời gian thực thi của một lệnh riêng lẻ\*. nhưng thông lượng lệnh là chỉ số quan trọng vì các chương trình thực thi hàng tỷ lệnh.

:::

(sec-waterfall-vi)=
### Biểu đồ thác nước

Thay vì vẽ lại cùng sơ đồ nhỏ như trong @fig-pipelined-processor, chúng ta sẽ sử dụng @tab-pipelined-waterfall-vi để minh họa việc thực thi lệnh trong bộ xử lý pipeline. **Biểu đồ thác nước** này nên được đọc từ trái sang phải: cột ngoài cùng bên trái là lệnh RISC-V; tất cả các cột khác được đánh chỉ mục theo bước thời gian (chu kỳ xung clock, bắt đầu từ 1). Đối với mỗi hàng, một mục trong cột bước thời gian chỉ ra giai đoạn mà lệnh đang thực thi, hoặc trống nếu không. Để đơn giản, chúng ta viết `MEM` là M.

```{list-table} Biểu đồ thác nước của @fig-pipelined-processor.
:header-rows: 1
:label: tab-pipelined-waterfall-vi
* - Lệnh
  - 1
  - 2
  - 3
  - 4
  - 5
  - 6
  - 7
* - `add s0 t0 t1`
  - IF
  - ID
  - EX
  - M
  - WB
  -  
  -  
* - `or  t3 t4 t5`
  -  
  - IF
  - ID
  - EX
  - M
  - WB
  -  
* - `lw  t6 8(t7)`
  - 
  - 
  - IF
  - ID
  - EX
  - M
  - WB
```
