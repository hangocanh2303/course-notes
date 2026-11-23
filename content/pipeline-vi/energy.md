---
title: "Hiệu quả Năng lượng"
subtitle: Nội dung này không được kiểm tra
---

(sec-energy-efficiency-vi)=
## Mục tiêu Học tập

* Phát triển khung làm việc cho việc đo hiệu quả năng lượng.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/N9pySCtYFHI
:width: 100%
:title: "[CS61C FA20] Lecture 21.3 - Pipelining I: Energy Efficiency"
:::

::::

Hiệu quả năng lượng rất quan trọng cho cả thiết bị di động và máy tính quy mô trung tâm dữ liệu.
Trong trung tâm dữ liệu, chi phí năng lượng có thể vượt qua chi phí phần cứng chỉ sau một hoặc hai năm.

Cũng quan trọng để lưu ý rằng công suất thường _không_ phải là thước đo hiệu quả tốt. Công suất chỉ là tỷ lệ trao đổi, trong khi năng lượng là những gì thực sự được lưu trữ trong pin và những gì công ty điện tính phí chúng ta.

```{math}
:label: energy-efficiency-vi
\frac{\text{năng lượng}}{\text{chương trình}} = \frac{\text{lệnh}}{\text{chương trình}} * \frac{\text{năng lượng}}{\text{lệnh}}
```

```{math}
:label: power-eq-vi
\text{công suất} = \frac{\text{năng lượng (J)}}{\text{giây}} = \frac{\text{năng lượng (J)}}{\text{lệnh}} * \frac{\text{lệnh}}{\text{chu kỳ}} * \frac{\text{chu kỳ}}{\text{giây}}
```

Xem video để biết thêm thông tin!

<!--

## Visuals

:::{figure} images/processor-perf-trends.png
:label: fig-processor-trends
:width: 70%
:alt: "Historical processor performance trend chart spanning decades with an overall growth curve over time."

48 years of processor performance trends.
:::

:::{figure} images/inv-symbol.png
:label: fig-inv-symbol
:width: 35%
:alt: "Logic symbol for an inverter gate with one input and one inverted output."

Inverter (INV) symbol.
:::

:::{figure} images/inv-schematic.png
:label: fig-inv-schematic
:width: 35%
:alt: "Transistor-level schematic of an inverter showing complementary pull-up and pull-down devices."

Inverter (INV) schematic.
:::

:::{figure} images/clock-rate-power-trends.png
:label: fig-power-trends
:width: 70%
:alt: "Clock-rate and power trend chart over time illustrating frequency growth and power limits."

36 years of clock rate and power trends.
:::
-->
