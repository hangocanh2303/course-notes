---
title: "Superscalar"
subtitle: Nội dung này nằm ngoài phạm vi
---

(sec-superscalar-vi)=
## Mục tiêu Học tập

* Hiểu rằng bộ xử lý superscalar có CPI < 1.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/M95RT-gqah8
:width: 100%
:title: "[CS61C FA20] Lecture 23.3 - Pipelining III: Superscalar"
:::

::::

Chúng ta có thể thử nhiều chiến lược để tăng hiệu năng hơn nữa:

1. **Tăng tốc độ xung clock.** Bị giới hạn bởi công nghệ và tản nhiệt công suất.
2. **Tăng độ sâu pipeline.** "Chồng chéo" thực thi lệnh thông qua pipeline sâu hơn, ví dụ: 10 hoặc 15 giai đoạn. Ít công việc hơn mỗi giai đoạn nghĩa là chu kỳ xung clock ngắn hơn/công suất thấp hơn. Nhưng có **nhiều tiềm năng hơn** cho cả ba loại hazard. Và nhiều stall hơn nghĩa là CPI trung bình sẽ lớn hơn 1.
3. Thiết kế **bộ xử lý superscalar**. Desktop, laptop, điện thoại di động, v.v. thường có một vài trong số này, kết hợp với các bộ xử lý pipeline 5 giai đoạn đơn giản hơn.

Ở mức cao, bộ xử lý superscalar là **multiple-issue**, nghĩa là chúng bắt đầu nhiều lệnh mỗi chu kỳ xung clock. Trong bộ xử lý superscalar, nhiều đơn vị thực thi thực thi lệnh song song, trong đó mỗi đơn vị thực thi có pipeline riêng.

:::{figure} images/superscalar-processors.png
:label: fig-superscalar
:width: 100%
:alt: "Timeline superscalar hiển thị nhiều lệnh bắt đầu thực thi trong cùng chu kỳ xung clock sử dụng các slot issue song song."

Bộ xử lý superscalar bắt đầu nhiều lệnh mỗi chu kỳ xung clock.
:::

Phần cứng bộ xử lý cũng phải có một số loại thực thi động "out-of-order", trong đó nó sắp xếp lại lệnh động để giảm tác động của hazard.

Chúng ta lưu ý rằng bộ xử lý superscalar, trong đó nhiều lệnh được thực thi trên nhiều pipeline trong một bộ xử lý đơn, nên được đối lập với bộ xử lý đa lõi, có các thread riêng biệt mỗi lõi.

Bộ xử lý superscalar bắt đầu nhiều lệnh mỗi chu kỳ xung clock. Trong [Định luật Sắt về Hiệu năng Bộ xử lý](#sec-iron-law-vi), bộ xử lý superscalar có CPI trung bình < 1 vì IPC > 1 (nhiều lệnh hoàn thành mỗi chu kỳ xung clock).

<!-- reprint of iron-law -->
<!-- ```{math} :label: superscalar-law
\frac{\text{time}}{\text{program}} = \frac{\text{instructions}}{\text{program}} * \frac{\text{cycles}}{\text{instructions}} * \frac{\text{time}}{\text{cycles}}
``` -->

:::{figure} images/arm-a53-benchmark.png
:label: fig-arm-a53-benchmark
:width: 70%
:alt: "Biểu đồ benchmark ARM Cortex-A53 với CPI trên trục y và đường tham chiếu ngang được tô sáng tại CPI bằng một."

Benchmark ARM A53 (đường vàng ngang tại $\text{CPI}=1$).
:::
