---
title: "Giới thiệu về Hazard"
---

(sec-pipeline-hazards-vi)=
## Mục tiêu Học tập

* Xác định ba loại hazard gặp phải trong pipeline RISC-V.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/71Mb1OjKwbs
:width: 100%
:title: "[CS61C FA20] Lecture 22.2 - Pipelining II: Pipeline Hazards"
:::

::::

Một trong những chi phí của pipelining là nó đưa vào **pipeline hazard**. Một pipeline hazard, hay đơn giản là **hazard**, là tình huống trong đó một lệnh đã lên kế hoạch không thể thực thi trong chu kỳ xung clock "thích hợp". Nói cách khác, hazard là khi việc thực thi một tổ hợp lệnh sẽ không thể hoặc sẽ dẫn đến thực thi chương trình không chính xác.

Có ba loại hazard:

+++ {"label": "block-def-hazard-structural-vi"}
[**Structural hazard**](#sec-structural-hazards-vi): Phần cứng trong bộ xử lý không thể hỗ trợ tổ hợp các lệnh mà chúng ta muốn thực thi trong cùng chu kỳ xung clock.
+++ {"label": "block-def-hazard-data-vi"}
[**Data hazard**](#sec-data-hazards-vi): Các lệnh có phụ thuộc dữ liệu, và một số lệnh phải chờ các lệnh trước hoàn thành—nếu không các giá trị lỗi thời sẽ được sử dụng trong tính toán.
+++ {"label": "block-def-hazard-control-vi"}
[**Control hazard**](#sec-control-hazards-vi): Luồng thực thi phụ thuộc vào các lệnh trước. Các lệnh sai được thực thi.
+++

Chúng ta thảo luận cách giải quyết hazard thông qua nhiều giải pháp khác nhau trong phần cứng, trong thời gian thực thi, hoặc trong mã chương trình:

* **Stalling** là một giải pháp không hiệu quả để giải quyết bất kỳ loại hazard nào, trong đó chúng ta trì hoãn các lệnh cho đến khi chúng ta có thể thực thi chúng mà không phát sinh hazard. Vì hiệu năng bị ảnh hưởng với stalling, chúng ta sẽ thảo luận các cách tránh stalling khi có thể (mặc dù nó luôn là phương án cuối cùng tốt). Xem [phần này](#sec-data-hazards-stall-vi).
* Chỉ định **yêu cầu phần cứng**, tức là trên các đơn vị phần cứng cụ thể trong pipeline.
* **Forwarding**, còn được gọi là bypassing, là khi chúng ta nối thêm các kết nối trong datapath và thay vào đó sử dụng kết quả khi được tính toán. Xem [phần này](#sec-data-hazards-forward-vi).
* **Code scheduling**, trong đó chúng ta sắp xếp lại các lệnh tại thời điểm biên dịch để tránh hazard.

Trong thực tế, máy tính sử dụng kết hợp các kỹ thuật trên để tối đa hóa thông lượng và duy trì các lợi ích của **song song mức lệnh** mà pipelining cung cấp.
