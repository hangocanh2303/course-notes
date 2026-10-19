---
title: "Transistor và Công Tắc"
subtitle: "Nội dung này không kiểm tra. Tác giả John Wawrzynek"
---

(sec-transistors-vi)=
## Mục Tiêu Học Tập

* Có được trực giác về transistor và công tắc.
* Để tìm hiểu thêm, hãy học EE 105: Microelectronic Devices and Circuits!

::::{note} 🎥 Video Bài Giảng: Công Tắc
:class: dropdown

:::{iframe} https://www.youtube.com/embed/fbtUreUJxR4
:width: 100%
:title: "[CS61C FA20] Lecture 14.1 - Intro to Synchronous Digital Systems: Switches"
:::

từ phút 5:35 trở đi

::::

::::{note} 🎥 Video Bài Giảng: Transistor
:class: dropdown

:::{iframe} https://www.youtube.com/embed/7-zqW5WM4os
:width: 100%
:title: "[CS61C FA20] Lecture 14.2 - Intro to Synchronous Digital Systems: Transistors"
:::

::::

::::{note} 🎥 Trực Giác Bổ Sung: Phép Tương Tự Với Nước
:class: dropdown

:::{iframe} https://www.youtube.com/embed/BoRlclML3tE?
:width: 100%
:title: "Video chi tiết về phép tương tự với nước cho hệ thống điện."
:::

::::


::::{note} 🎥 Trực Giác Bổ Sung: Transistor
:class: dropdown
:::{iframe} https://www.youtube.com/embed/8f3l8KaVcBg?
:width: 100%
:title: "Video chi tiết về mô hình transistor lưỡng cực - hoạt hình Blender."
:::

::::



Slide liên quan:

:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vRnOm001_KVT3IhaHdSJ_6GxXhB5wQAtLrV0c60tdlgD35vtewNsprWXJ9sX-TurxiyCmlE-QHuF7Aj/pubembed?start=false&loop=false
:width: 100%
:title: "Hoạt hình trình bày từng bước về nền tảng và chi tiết của công tắc và transistor, như được trình bày trong các video trong phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1qe4xrtkAFFO6dUE7RPB5main2ESWy51Q-_-6_RjboqQ/edit?usp=sharing)"
:::



<!--
:::{figure} images/n-channel-mos.png
:label: fig-n-mos
:width: 100%
:alt: "Sơ đồ MOSFET kênh n với các cực được đánh nhãn drain, gate, và source và loại kênh được chỉ ra cho NMOS."

Sơ đồ drain, gate, source cho n-channel CMOS.
:::

:::{figure} images/p-channel-mos.png
:label: fig-p-mos
:width: 100%
:alt: "Sơ đồ MOSFET kênh p với các cực được đánh nhãn drain, gate, và source và loại kênh được chỉ ra cho PMOS."

Sơ đồ drain, gate, source cho p-channel CMOS.
:::

:::{figure} images/mos-network.png
:label: fig-mos-network
:width: 100%
:alt: "Sơ đồ dạng lưu đồ liên kết mạng transistor MOS với cổng logic và các khối xây dựng số cấp cao hơn."

Lưu đồ về Mạng MOS và cổng.
:::


:::{figure} images/switch-open.png
:label: fig-switch-open
:width: 100%
:alt: "Mạch đơn giản với nguồn điện áp, tải, và công tắc mở nên đường dẫn bị đứt và dòng điện không chạy."

Sơ đồ mạch với công tắc mở.
:::

:::{figure} images/switch-closed.png
:label: fig-switch-closed
:width: 100%
:alt: "Mạch cùng kiểu với công tắc đóng nên đường dẫn hoàn thành và dòng điện có thể chạy."

Sơ đồ mạch với công tắc đóng.
:::

:::{figure} images/switch-series.png
:label: fig-switch-series
:width: 100%
:alt: "Hai công tắc nối tiếp giữa nguồn và đầu ra nên đầu ra chỉ được cấp điện khi cả hai công tắc đều bật."

Sơ đồ mạch với hai công tắc mắc nối tiếp.
:::

:::{figure} images/switch-parallel.png
:label: fig-switch-parallel
:width: 100%
:alt: "Hai công tắc mắc song song nên đầu ra được cấp điện khi một trong hai công tắc tạo đường dẫn."

Sơ đồ mạch với hai công tắc mắc song song.
:::
-->
