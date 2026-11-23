---
title: "Structural Hazard"
---

(sec-structural-hazards-vi)=
## Mục tiêu Học tập

* Giải thích tại sao các yêu cầu phần cứng của pipeline RISC-V không gây ra structural hazard.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/71Mb1OjKwbs
:width: 100%
:title: "[CS61C FA20] Lecture 22.2 - Pipelining II: Pipeline Hazards"
:::

::::

Từ trước:

:::{embed} #block-def-hazard-structural-vi
:::

Từ P&H 4.6:

> Một structural hazard trong phòng giặt sẽ xảy ra nếu chúng ta sử dụng máy giặt-sấy kết hợp thay vì máy giặt và máy sấy riêng biệt. Các kế hoạch pipeline được lên lịch cẩn thận của chúng ta sẽ bị phá hỏng.
> 
> Tập lệnh RISC-V được thiết kế để pipeline, giúp các nhà thiết kế khá dễ dàng tránh structural hazard khi thiết kế pipeline.

Nói cách khác, trong bộ xử lý năm giai đoạn hiện tại của chúng ta, **structural hazard không phải là vấn đề** trừ khi có thay đổi được thực hiện cho pipeline. Các structural hazard _có thể_ tồn tại được ngăn chặn bởi các yêu cầu phần cứng của RV32I.

Giả sử chúng ta có năm lệnh sau thực thi đồng thời trong pipeline năm giai đoạn như trong @fig-structural-pipeline.

:::{figure} images/structural-pipeline.png
:label: fig-structural-pipeline
:width: 70%
:alt: "Sơ đồ chiếm dụng pipeline với các biểu tượng cho mỗi trong năm giai đoạn pipeline. Nhãn dưới các giai đoạn hiển thị năm lệnh riêng biệt đồng thời trong mỗi giai đoạn IF, ID, EX, MEM, và WB. Lệnh 1 ở giai đoạn write-back, cùng lúc lệnh 2 ở giai đoạn MEM, lệnh 3 ở giai đoạn EX, lệnh 4 ở giai đoạn ID, và lệnh 5 ở giai đoạn IF."

Năm lệnh `inst1`, `inst2`, `inst3`, `inst4`, `inst5` đang thực thi theo thứ tự và chiếm tất cả năm giai đoạn của pipeline trong cùng chu kỳ xung clock.
:::

Nếu chúng ta thay đổi các thành phần phần cứng chính của pipeline, những thay đổi này có thể dẫn đến structural hazard như thế nào?

### RegFile

[Tệp thanh ghi](#sec-element-regfile) (RegFile) được sử dụng trong bộ xử lý của chúng ta hỗ trợ đọc và ghi đồng thời bởi hai lệnh khác nhau.

::::::{grid} 2

:::::{grid-item}

:::{figure} images/regfile-no-structural-hazard.png
:label: fig-regfile-no-hazard
:width: 70%
:alt: "Biểu tượng hình chữ nhật tệp thanh ghi hiển thị các đầu vào wdata, rd, rs1, và rs2, tín hiệu điều khiển RegWEn, và các đầu ra rdata1 và rdata2. Điều này mô tả mô hình truy cập đọc và ghi riêng biệt tránh xung đột cấu trúc."

Thiết kế RegFile RISC-V
:::
:::::

:::::{grid-item}

:::{figure} images/regfile-structural-hazard.png
:label: fig-regfile-hazard
:width: 70%
:alt: "Biểu tượng khối tệp thanh ghi thay thế chỉ với các đầu vào wdata và rs, cùng tín hiệu điều khiển RegWEn, và một đầu ra rdata. Tổ chức này gây ra xung đột tài nguyên đọc-ghi nếu cố gắng đọc và ghi từ RegFile trong cùng chu kỳ."

Thiết kế RegFile thay thế
:::
:::::
::::::
<!-- grid -->

Trong @fig-structural-pipeline, tệp thanh ghi được truy cập đồng thời bởi `inst4` ở giai đoạn `ID` (để đọc hai thanh ghi), và `inst1` ở giai đoạn `WB` (để ghi vào thanh ghi). Thiết kế RegFile RV32I trong @fig-regfile-no-hazard có nghĩa là `inst1` và `inst4` có thể đồng thời thực hiện ghi và đọc tương ứng vào RegFile mà không gây ra structural hazard.

Ngược lại, thiết kế RegFile thay thế trong @fig-regfile-hazard có thể gây ra structural hazard. `RegWEn` chỉ định nếu RegFile sẽ được sử dụng để đọc hoặc ghi trong chu kỳ này, và cổng `rs` chỉ định giá trị thanh ghi 5-bit để sử dụng cho đọc hoặc ghi. Tuy nhiên, thiết kế RegFile này không hỗ trợ đọc/ghi đồng thời, và do đó việc cố gắng thực thi `inst1` và `inst4` trong cùng chu kỳ sẽ gây ra structural hazard.

### Bộ nhớ

[Bộ nhớ lệnh](#sec-element-imem) (IMEM) và [bộ nhớ dữ liệu](#sec-element-dmem) (DMEM) được sử dụng trong bộ xử lý của chúng ta là các phần tử phần cứng riêng biệt.

:::{figure} images/separate-mem.png
:label: fig-separate-mem
:width: 60%
:alt: "Bên trái, một biểu tượng hình chữ nhật cho IMEM với đầu vào địa chỉ và đầu ra lệnh. Bên phải, một biểu tượng hình chữ nhật cho DMEM riêng biệt với đầu vào địa chỉ và wdata, tín hiệu điều khiển MemRW, và đầu ra rdata. Thiết kế bộ nhớ riêng biệt này tránh hazard bộ nhớ lệnh-dữ liệu."

IMEM và DMEM RV32I là hai đơn vị phần cứng riêng biệt.
:::

Trong @fig-structural-pipeline, bộ nhớ được truy cập bởi `inst5` ở giai đoạn `IF` (để đọc lệnh từ IMEM) và `inst2` ở giai đoạn `MEM` (để đọc dữ liệu từ DMEM). IMEM và DMEM riêng biệt trong @fig-separate-mem không gây ra structural hazard, vì `inst5` và `inst2` có thể đồng thời truy cập bộ nhớ.

Ngược lại, nếu chúng ta sử dụng một khối bộ nhớ duy nhất (ví dụ: đơn giản gọi là "MEM") chỉ có thể hỗ trợ một lần đọc/ghi tại một thời điểm (đến bất kỳ phần nào của bộ nhớ—lệnh hoặc dữ liệu), thì `inst5` và `inst2` cố gắng truy cập bộ nhớ trong cùng chu kỳ sẽ gây ra structural hazard.

Chúng ta sẽ thảo luận sau, nhưng bên dưới, IMEM và DMEM thực sự là cache của bộ nhớ chính, như được hiển thị trong @fig-separate-onchip-mem. Thiết kế này cho phép chúng gần bộ xử lý hơn nhiều để giữ cho truy cập bộ nhớ nhanh.

:::{figure} images/separate-onchip-mem.png
:label: fig-separate-onchip-mem
:width: 80%
:alt: "Sơ đồ Bộ xử lý-Bộ nhớ hiển thị bộ nhớ lệnh và dữ liệu on-chip riêng biệt được kết nối riêng với datapath bộ xử lý."

Sơ đồ Bộ xử lý và Bộ nhớ của IMEM/DMEM riêng biệt trong bộ nhớ.
:::
