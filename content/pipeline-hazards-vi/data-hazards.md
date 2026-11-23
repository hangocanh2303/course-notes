---
title: "Data Hazard"
short_title: "Data Hazard, Forwarding"
---

(sec-data-hazards-vi)=
## Mục tiêu Học tập

* Cho các chuỗi lệnh và kiến trúc bộ xử lý, xác định các data hazard tiềm năng.
* Giải thích tại sao tệp thanh ghi ghi-rồi-đọc giảm stall pipeline do data hazard tiềm năng.
* Giải thích cách forwarding (tức là bypassing) giảm stall pipeline do data hazard tiềm năng.
* Triển khai hai đường forwarding, một từ `MEM` đến `EX` và một từ `WB` đến `EX`, trong bộ xử lý pipeline năm giai đoạn.
* Giải thích tại sao data hazard liên quan đến lệnh load trong chuỗi lệnh cho trước có thể phát sinh stall pipeline không tránh được.

::::{note} 🎥 Video Bài giảng: Data Hazard
:class: dropdown

:::{iframe} https://www.youtube.com/embed/iOKJ-oW1urY
:width: 100%
:title: "[CS61C FA20] Lecture 22.4 - Pipelining II: Data Hazards"
:::

::::

::::{note} 🎥 Video Bài giảng: Load Data Hazard
:class: dropdown

:::{iframe} https://www.youtube.com/embed/VWCAqieFkHI
:width: 100%
:title: "[CS61C FA20] Lecture 23.1 - Pipelining III: Load Data Hazard"
:::

::::

Từ [trước](#sec-pipeline-hazards-vi):

:::{embed} #block-def-hazard-data-vi
:::

Data hazard xảy ra vì các lệnh đọc từ và ghi vào cùng các thanh ghi và bộ nhớ. Từ P&H 4.6:

> Giả sử bạn tìm thấy một chiếc tất ở trạm gấp mà không có chiếc nào khớp. Một chiến lược có thể là chạy xuống phòng và tìm kiếm qua tủ quần áo để xem liệu bạn có thể tìm thấy chiếc khớp không. Rõ ràng, trong khi bạn đang tìm kiếm, các mẻ đã sấy xong sẵn sàng để gấp và những mẻ đã xong sẵn sàng để sấy.

Trong phần này, chúng ta thảo luận cách bộ xử lý pipeline năm giai đoạn có thể được sửa đổi để giảm thiểu ảnh hưởng hiệu năng do data hazard.

(data-hazard-1-vi)=
### Ví dụ 1

Xem xét [biểu đồ thác nước](#sec-waterfall-vi) dưới đây. Các lệnh `add` và `sub` có data hazard vì lệnh trước ghi vào _và_ lệnh sau đọc từ thanh ghi `s0`.

```{list-table} Ví dụ 1. Data hazard.
:header-rows: 1
:label: tab-data-hazard-1-vi

* - Lệnh
  - 1
  - 2
  - 3
  - 4
  - 5
  - 6
  - 7
  - 8
  - 9
* - `add s0 t0 t1`
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  - 
  - 
* - `sub t2 s0 t0`
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  -
* - `or  t3 t4 t5`
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
```

Lệnh `sub` phải đọc giá trị cập nhật của `s0` sau khi lệnh `add` hoàn thành. Trong chu kỳ 5, lệnh `add` ghi vào thanh ghi `s0`. Tuy nhiên, trong chu kỳ 3, `sub` đọc từ thanh ghi `s0`, nhận được giá trị cũ của `s0`, trước khi `add` cập nhật nó. Sau đó `sub` thực hiện phép trừ sai của giá trị cũ này trước khi ghi kết quả sai.

(sec-data-hazards-stall-vi)=
## Stalling

Để giải quyết data hazard trong [Ví dụ 1](#data-hazard-1-vi), chúng ta có thể **stall** pipeline cho đến khi tài nguyên "sẵn sàng", tức là `add` đã ghi giá trị đúng vào thanh ghi `s0`. Các stall pipeline, hay **bubble**, thực chất là "no-op" (nop) trong đó các pipeline bị ảnh hưởng không làm gì.

Sơ đồ dưới đây minh họa giải pháp ba-stall. Trong @tab-data-hazard-1-stall-vi, `sub` chắc chắn sẽ đọc giá trị cập nhật đúng của thanh ghi `s0` vào cuối chu kỳ 6.

```{list-table} [Ví dụ 1](#data-hazard-1-vi): Giải quyết data hazard với stall. Dấu gạch ngang (–) chỉ ra rằng pipeline bị flush và các lệnh bị ảnh hưởng "không làm gì".
:label: tab-data-hazard-1-stall-vi
:header-rows: 1

* - Lệnh
  - 1
  - 2
  - 3
  - 4
  - 5
  - 6
  - 7
  - 8
  - 9
  - 10
* - `add s0 t0 t1`
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  - 
  - 
  - 
* - `sub → nop`
  - 
  - IF
  - ID
  - –
  - –
  - –
  - 
  - 
  - 
  - 
* - `nop`
  - 
  - 
  - –
  - –
  - –
  - –
  - –
  - 
  - 
* - ` nop`
  - 
  - 
  - 
  - –
  - –
  - –
  - –
  - –
  - 
  - 
* - `sub t2 s0 t0`
  - 
  - 
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 

* - `or t3 t4 t5`
  - 
  - 
  - 
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
```

Vì hiệu năng bị ảnh hưởng với stalling, chúng ta sẽ thảo luận các cách tránh stalling khi có thể (mặc dù nó luôn là phương án cuối cùng tốt).

### Triển khai Stall

Các chi tiết trong tiểu mục này nằm ngoài phạm vi.
Để biết thêm thông tin, đọc P&H 4.8.

Triển khai stall trong phần cứng yêu cầu điều khiển và trạng thái pipeline bổ sung để ngăn các thay đổi trạng thái không mong muốn trong các giai đoạn bị stall, ví dụ: ghi vào program counter, thanh ghi, hoặc bộ nhớ.

Một cách tiếp cận được mô tả trong P&H 4.8 là đơn vị phát hiện hazard. Đối với data hazard, đơn vị phát hiện này có thể được triển khai trong giai đoạn `ID` để xác định nếu các thanh ghi nguồn của lệnh này phụ thuộc vào thanh ghi đích của (các) thanh ghi vẫn còn trong pipeline.[^stall-pipeline-regs-vi] Để stall một lệnh, chúng ta có thể hủy kích hoạt tất cả các tín hiệu điều khiển (bằng cách đặt chúng về 0[^control-note-vi]) để khi lệnh đi qua các giai đoạn sau, các giai đoạn thực chất không làm gì.[^stall-instruction-vi]

Chúng ta minh họa điều này trong @tab-data-hazard-1-stall-vi, trong đó trong chu kỳ 2, đơn vị phát hiện hazard phát hiện rằng lệnh ở giai đoạn `ID`, `sub`, có thanh ghi nguồn phụ thuộc vào lệnh `add`. Đơn vị phát hiện hazard sau đó đẩy nop qua pipeline và bảo tồn lệnh `sub` cho đến khi nó có thể được hoàn thành an toàn.[^stall-stages-vi]

[^stall-pipeline-regs-vi]: Làm thế nào chúng ta kiểm tra thanh ghi đích? Đơn vị phát hiện hazard kiểm tra các thanh ghi pipeline. Ví dụ, nếu thanh ghi `rd` được chỉ định trong thanh ghi pipeline `ID/EX` là một trong các thanh ghi nguồn cho lệnh ở giai đoạn `ID`, thì stall lệnh ở giai đoạn `ID`.

[^control-note-vi]: Đây có phần là nói quá; đọc P&H 4.9 để biết thêm chi tiết.

[^stall-instruction-vi]: Nếu lệnh ở giai đoạn `ID` bị stall, thì lệnh ở giai đoạn `IF` cũng phải bị stall, v.v. Chúng ta có thể hoàn thành điều này bằng cách (1) ngăn thanh ghi PC tăng, và (2) ngăn thanh ghi pipeline `IF/ID` thay đổi. Từ P&H 4.8: "Giống như bạn khởi động lại máy giặt với cùng quần áo, và để máy sấy tiếp tục quay không. Tất nhiên, giống máy sấy, nửa sau của pipeline bắt đầu với giai đoạn EX phải đang làm gì đó; những gì nó đang làm là thực thi các lệnh không có hiệu ứng: nop."

[^stall-stages-vi]: Chúng ta lưu ý rằng trong @tab-data-hazard-1-stall-vi, lệnh `sub` thực sự được fetch và decode trong chu kỳ 2 và 3, nhưng giai đoạn `EX` của nó bị trì hoãn đến chu kỳ 7. Tương tự, lệnh `or` được fetch trong chu kỳ xung clock 3 nhưng giai đoạn `ID` của nó bị trì hoãn đến chu kỳ 7.

## RegFile: Ghi-Rồi-Đọc

(data-hazard-2-vi)=
### Ví dụ 2
Xem xét một ví dụ khác, được minh họa bởi biểu đồ thác nước trong @tab-data-hazard-2-vi. Liệu sự phụ thuộc giữa `add` và `sw` có gây ra data hazard không?

```{list-table} Ví dụ 2. Data hazard...?
:header-rows: 1
:label: tab-data-hazard-2-vi

* - Lệnh
  - 1
  - 2
  - 3
  - 4
  - 5
  - 6
  - 7
  - 8
  - 9
* - `add t0 t1 t2`
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  - 
  - 
* - `lw  t0 8(t3)`
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  - 
* - `or  t3 t4 t5`
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
* - `sw  t0 4(t6)`
  - 
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
* - `sll t6 t0 t3`
  - 
  - 
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
```

Điều gì đang xảy ra trong chu kỳ 5? Nếu chúng ta giả sử [thiết kế RegFile ban đầu](#sec-element-regfile), thì lệnh `add` ở giai đoạn `WB` chỉ thiết lập MUX, vì vậy việc ghi vào `t0` xảy ra tại cạnh xung clock lên _tiếp theo_, hoặc chu kỳ 6. Điều này có nghĩa là trong cùng chu kỳ 5, lệnh `sw` ở giai đoạn `ID` thực sự sẽ đọc giá trị cũ, gây ra data hazard.[^not-structural-vi]

[^not-structural-vi]: Chúng ta lưu ý hazard này **không phải là structural hazard**. Sau cùng, [thiết kế RegFile](#sec-element-regfile) _không ngăn_ `add` và `sw` đọc/ghi vào cùng thanh ghi trong cùng chu kỳ, vì có đủ cổng đầu vào. Tuy nhiên, điều đáng lo ngại là _giá trị_ `sw` đọc phải là giá trị đúng mà `add` ghi.

### Yêu cầu phần cứng

Do đó, pipeline RISC-V năm giai đoạn "nâng cao" **yêu cầu phần cứng** trên tệp thanh ghi. Chúng ta tận dụng tốc độ cao của tệp thanh ghi (100 ps cho mỗi lần đọc/ghi) để giả định rằng đơn vị phần cứng hỗ trợ **ghi-rồi-đọc**:

* Lệnh giai đoạn `WB` cập nhật giá trị trong nửa đầu chu kỳ, ví dụ: vào cạnh _xuống_.
* Giai đoạn `ID` đọc giá trị mới.

::::{hint} Giả định phần cứng mới: RegFile là ghi-rồi-đọc

Giả định này được minh họa thêm bởi tô màu của các giai đoạn `ID` và `WB` trong sơ đồ bộ xử lý pipeline cấp cao được thảo luận trong [phần trước](#sec-processor-hl-vi):

:::{figure} #fig-pipelined-processor-hl
:alt: "In lại sơ đồ bộ xử lý pipeline cấp cao: nửa trái giai đoạn WB được tô màu cho ghi tệp thanh ghi trong nửa đầu chu kỳ và nửa phải giai đoạn ID được tô màu cho đọc thanh ghi trong nửa sau, minh họa thời gian ghi-rồi-đọc."

Nửa trái của giai đoạn `WB` được tô màu, chỉ ra rằng RegFile được ghi trong nửa đầu của chu kỳ xung clock. Tương tự, nửa phải của giai đoạn `ID` được tô màu, chỉ ra rằng RegFile được đọc trong nửa sau của chu kỳ xung clock. In lại từ [phần trước](#sec-processor-hl-vi).
:::

Lưu ý rằng **có thể không phải lúc nào cũng có thể** hỗ trợ tệp thanh ghi với khả năng ghi-rồi-đọc, đặc biệt trong các thiết kế tần số cao. Luôn kiểm tra các giả định thiết kế bộ xử lý trước, đặc biệt khi trả lời câu hỏi bài tập.
::::

Nếu chúng ta giả sử RegFile hỗ trợ ghi-rồi-đọc, thì trong chu kỳ 5, việc đọc của lệnh `sw` ở giai đoạn `ID` trả về những gì được ghi bởi lệnh `add` ở giai đoạn `WB`, vì vậy **không có data hazard**.

::::{tip} Kiểm tra Nhanh
Hãy xem lại [Ví dụ 1](#data-hazard-1-vi).

:::{figure} #tab-data-hazard-1-vi
:alt: "In lại bảng sơ đồ pipeline cho ví dụ add sau đó các lệnh phụ thuộc đơn giản, hiển thị chiếm dụng giai đoạn IF đến WB mỗi chu kỳ trước khi áp dụng stall."
:::

Nếu chúng ta giả sử RegFile hỗ trợ ghi-rồi-đọc, cần stall bao nhiêu chu kỳ để tránh data hazard?

::::

:::{note} Hiển thị Đáp án
:class: dropdown
Chúng ta có thể chỉ stall _hai_ chu kỳ, như được hiển thị trong @tab-data-hazard-1-stall-fast-vi. Trong nửa đầu chu kỳ 5, lệnh `add` ghi vào thanh ghi `s0`; trong nửa sau, lệnh `sub` đọc `s0`.

```{list-table}  [data-hazard-1-vi](@data-hazard-1-vi): Giải quyết data hazard với stall **và** giả định rằng tệp thanh ghi hỗ trợ ghi-rồi-đọc trong cùng chu kỳ. Dấu gạch ngang (–) chỉ ra rằng pipeline bị flush và các lệnh bị ảnh hưởng "không làm gì".
:header-rows: 1
:label: tab-data-hazard-1-stall-fast-vi

* - Lệnh
  - 1
  - 2
  - 3
  - 4
  - 5
  - 6
  - 7
  - 8
  - 9
* - `add s0, t0, t1`
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  - 
  - 
* - `sub → nop`
  - 
  - IF
  - –
  - –
  - –
  - –
  - 
  - 
  - 
* - `nop`
  - 
  - 
  - –
  - –
  - –
  - –
  - –
  - 
  - 
* - `sub t2, s0, t0`
  - 
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
```
:::

(sec-data-hazards-forward-vi)=
## Forwarding

Cho đến nay, chúng ta đã thảo luận một số giải pháp cho một số hazard bằng cách (1) chỉ định các yêu cầu phần cứng thích hợp, và nếu tất cả đều thất bại, (2) stall pipeline cho đến khi không có hazard.

Tuy nhiên, chúng ta quan sát rằng với data hazard, chúng ta không cần chờ lệnh hoàn thành trước khi cố gắng giải quyết data hazard. Nói cách khác, dữ liệu được đề cập sẵn sàng _sớm hơn nhiều_ so với giai đoạn `WB` của lệnh trước.

(data-hazard-3-vi)=
### Ví dụ 3

Xem xét ví dụ trong @tab-data-hazard-3-vi, có hai data hazard vì các lệnh `sub` và `or` phụ thuộc vào kết quả của lệnh `add` ghi vào thanh ghi `s0`.

:::{list-table} Ví dụ 3.
:label: tab-data-hazard-3-vi
:header-rows: 1

* - Lệnh
  - 1
  - 2
  - 3
  - 4
  - 5
  - 6
  - 7
  - 8
  - 9
* - `add s0 t0 t1`
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  - 
  - 
* - `sub t2 s0 t0`
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  - 
* - `or  t6 s0 t3`
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
:::

Kết quả cộng `t0` và `t1` sẵn sàng vào đầu chu kỳ 4, một khi lệnh `add` hoàn thành giai đoạn `EX` trong chu kỳ 3. Vì vậy chúng ta có thể thêm phần cứng bổ sung để cung cấp tổng này làm đầu vào cho lệnh `sub` _và_ lệnh `or`.

### Bypassing với nhiều kết nối hơn

Nối thêm các kết nối trong datapath để sử dụng kết quả khi được tính là một quá trình được gọi là **forwarding** hay **bypassing**.[^forwarding-bypassing-vi] Thay vì chờ giá trị được ghi vào RegFile, chúng ta có thể lấy toán hạng trực tiếp từ giai đoạn pipeline _tiếp theo_.

[^forwarding-bypassing-vi]: Từ P&H 4.6: "Tên _forwarding_ đến từ ý tưởng rằng kết quả được truyền tiếp từ lệnh trước đến lệnh sau. _Bypassing_ đến từ việc truyền kết quả vòng qua tệp thanh ghi đến đơn vị mong muốn."

Chúng ta sử dụng @fig-forwarding-hl để mô tả ở mức cao dữ liệu nào được forward.

:::{figure} images/forwarding-hl.png
:label: fig-forwarding-hl
:width: 80%
:alt: "Hình ảnh pipeline năm giai đoạn cấp cao với ba lệnh đang thực thi: ADD ghi s0, SUB tiêu thụ s0, và OR cũng tiêu thụ s0. Các mũi tên bypass màu hiển thị kết quả ALU của ADD được forward từ ranh giới EX/MEM trực tiếp vào mux toán hạng giai đoạn EX của SUB, và bypass thứ hai từ MEM/WB vào mux giai đoạn EX của OR để không lệnh phụ thuộc nào chờ write-back tệp thanh ghi. Nhãn giai đoạn IF đến WB đóng khung mỗi hàng lệnh để người xem có thể thấy căn chỉnh chu kỳ."

Forwarding thêm các kết nối bổ sung giữa [thanh ghi pipeline](#sec-pipeline-registers-vi) và các thành phần khác trong datapath.
:::

Ghi chú:

* Vào đầu chu kỳ 4, kết quả ALU từ lệnh `add` được forward từ thanh ghi pipeline `EX/MEM` trực tiếp đến ALU (cho giai đoạn `EX` của lệnh `sub`).
* Vào đầu chu kỳ 5, kết quả ALU từ lệnh `add` được forward từ thanh ghi pipeline `MEM/WB` trực tiếp đến ALU (cho giai đoạn `EX` của lệnh `or`).
* Giá trị thanh ghi `s0` vẫn được cập nhật trong chu kỳ 5, từ giá trị cũ 5 đến giá trị mới 9. Các giai đoạn `ID` của lệnh `sub` và `or` vẫn đọc giá trị cũ của thanh ghi `s0` trong chu kỳ 2 và 3 tương ứng. Điều quan trọng là các toán hạng đúng được đưa vào ALU trong giai đoạn `EX` cho cả hai lệnh này.
* Lưu ý rằng với hardware forwarding, chúng ta **không cần** cập nhật biểu đồ thác nước trong [Ví dụ 3](#data-hazard-3-vi) vì không có stall xảy ra.

### Các Đường Forwarding

:::{hint} Forward dữ liệu từ thanh ghi pipeline

Trong khóa học này, chúng ta thảo luận **hai** loại đường forwarding (tức là bypass) từ thanh ghi pipeline đến **mỗi** trong hai đầu vào ALU, như được mô tả trong @fig-forwarding-all-hl.

1. **`MEM` đến `EX` forwarding**. Forward dữ liệu từ thanh ghi pipeline `EX/MEM` đến đầu vào ALU (ví dụ: trong [Ví dụ 3](#data-hazard-3-vi), để giải quyết data hazard `add`/`sub`).
1. **`WB` đến `EX` forwarding**. Forward dữ liệu từ thanh ghi pipeline `MEM/WB` đến đầu vào ALU (ví dụ: trong [Ví dụ 3](#data-hazard-3-vi), để giải quyết data hazard `add`/`or`).

:::

:::{figure} images/forwarding-all-hl.png
:label: fig-forwarding-all-hl
:width: 100%
:alt: "Sơ đồ forwarding mở rộng với mọi giai đoạn pipeline được gắn nhãn và hai mạng bypass được tô sáng. Đường tím: từ thanh ghi sau MEM/WB quay lại mux ALU đưa toán hạng B, chú thích là WB-to-EX forwarding. Đường hồng: từ thanh ghi sau EX/MEM đến cùng mux, chú thích là MEM-to-EX forwarding. Thanh ghi pipeline giữa các giai đoạn được vẽ rõ ràng để sinh viên có thể theo dõi các giá trị được forward được lấy từ đâu so với cổng đọc tệp thanh ghi."

Các bypass forwarding cho tín hiệu đầu vào B của ALU. Để đơn giản, chúng ta không vẽ các bypass cho tín hiệu đầu vào A, mặc dù chúng chắc chắn cần thiết. Ngoại trừ PC, các thanh ghi giữa các giai đoạn là thanh ghi pipeline. Chuyển thể từ Spring 2026 CS 152.[^cs152-diagram-vi]
:::

[^cs152-diagram-vi]: [Sơ đồ gốc](https://inst.eecs.berkeley.edu/~cs152/exams/pdfs/mt1/sp26-mt1-sols.pdf) bởi Wen Cao, TA CS 152/252 Spring 2026. Sơ đồ CS 152/252 dành cho datapath pipeline khác với các đường forwarding khác (ví dụ: xem MUX vào thanh ghi pipeline ID/EX). Vui lòng sử dụng @fig-forwarding-all-hl cho khóa học này.

::::{tip} Kiểm tra Nhanh
Hãy xem [Ví dụ 1](#data-hazard-1-vi) trước đó.

:::{figure} #tab-data-hazard-1-vi
:alt: "In lại bảng sơ đồ pipeline cho ví dụ add sau đó các lệnh phụ thuộc đơn giản, hiển thị chiếm dụng giai đoạn IF đến WB mỗi chu kỳ trước khi áp dụng stall."
:::

Giả sử RegFile hỗ trợ ghi-rồi-đọc, _và_ chúng ta triển khai các đường forwarding được mô tả từ `MEM` đến `EX` và từ `WB` đến `EX`. Cần stall bao nhiêu chu kỳ để tránh data hazard?

::::

:::{note} Hiển thị Đáp án
:class: dropdown

Chúng ta **không cần stall pipeline**. Kết quả ALU từ lệnh `add` có sẵn vào đầu chu kỳ 4. Chúng ta có thể tận dụng đường forwarding `MEM` đến `EX` để forward kết quả ALU của lệnh `add` trực tiếp từ thanh ghi pipeline `EX/MEM` đến ALU cho giai đoạn `EX` của lệnh `sub`, cũng trong chu kỳ 4.
:::

### Forwarding: Triển khai

Forwarding được triển khai bằng cách thêm các dây bypass giữa thanh ghi pipeline và các thành phần khác, chèn mux, và bao gồm logic điều khiển bổ sung.

@fig-forwarding-ex-mem hiển thị triển khai của đường forwarding `MEM` đến `EX`. Đường forwarding (ví dụ: **bypass**) kết nối đầu ra của ALU từ thanh ghi pipeline `EX/MEM` đến các mux đầu vào ALU. Hai mux này bây giờ rộng hơn để tính đến tùy chọn bypass bổ sung. Các tín hiệu điều khiển `ASel` và `BSel` bây giờ cũng phải sử dụng các bit lệnh để xác định nếu bypass nên được sử dụng cho đầu vào nào đến ALU.

:::{figure} images/forwarding-ex-mem.png
:label: fig-forwarding-ex-mem
:width: 100%
:alt: "Sơ đồ pipeline năm giai đoạn đầy đủ với các giai đoạn được gắn nhãn hiển thị trường hợp forwarding EX-MEM được tô sáng. Đường vàng chỉ ra đường forwarding được thực hiện bởi các giá trị đầu ra từ thanh ghi pipeline giai đoạn EX, quay lại vào các mux trong giai đoạn Execute cho lệnh sau sử dụng trong giai đoạn EX của nó để giải quyết phụ thuộc đọc-sau-ghi."

Triển khai: đường forwarding `MEM` đến `EX`.

:::

@fig-forwarding-ex-mem triển khai một trong hai đường forwarding được mô tả trong @fig-forwarding-all-hl. Chúng ta bỏ qua mạch forwarding đầy đủ cho đường `WB` đến `EX`, để bạn tự làm.

(data-hazard-4-vi)=
:::{hint} Kiểm tra Nhanh: Ví dụ 4

Xem xét chuỗi lệnh dưới đây. Các data hazard tiềm năng nào được giải quyết bằng cách chèn các đường forwarding được mô tả từ `MEM` đến `EX` và từ `WB` đến `EX`?

```{list-table} Ví dụ 4.
:header-rows: 1
:label: tab-data-hazard-4-vi

* - Lệnh
  - 1
  - 2
  - 3
  - 4
  - 5
  - 6
  - 7
  - 8
  - 9
* - `add s0 t1 t2`
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  - 
  - 
* - `lw  s1 8(s0)`
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  - 
* - `or t3 s1 t5`
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
* - `and t4 s1 t2`
  - 
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
* - `sll t0 t1 t2`
  - 
  - 
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
```

* **A.** hazard `add`-`lw` với thanh ghi `s0`
* **B.** hazard `lw`-`or` với thanh ghi `s1`
* **C.** hazard `lw`-`and` với thanh ghi `s1`
* **D.** Không có cái nào ở trên

:::

:::{note} Hiển thị Đáp án
:class: dropdown

* **A.** Data hazard `add`-`lw` được **giải quyết** bởi forwarding `MEM` đến `EX`. Kết quả lệnh `add` (cộng `t1` và `t2`) có sẵn trong thanh ghi pipeline `EX/MEM` vào đầu chu kỳ 4. Chu kỳ 4 cũng là giai đoạn `EX` của lệnh `lw`. Trong chu kỳ này, giá trị đúng được forward từ thanh ghi pipeline `EX/MEM` đến đầu vào A của ALU, ghi đè giá trị cũ của thanh ghi `s0` được fetch trước đó (trong giai đoạn `ID` của lệnh `lw` trong chu kỳ 3).
* **C.** Data hazard `lw`-`and` được **giải quyết** bởi forwarding `WB` đến `EX`. Kết quả đọc bộ nhớ từ lệnh `lw` có sẵn từ thanh ghi pipeline `MEM/WB` vào đầu chu kỳ 6. Chu kỳ 6 cũng là giai đoạn `EX` của lệnh `and`. Trong chu kỳ này, giá trị đúng được forward từ thanh ghi pipeline `MEM/WB` đến đầu vào A của ALU, ghi đè giá trị cũ của thanh ghi `s1` được fetch trước đó (trong giai đoạn `ID` của lệnh `and` trong chu kỳ 5).
:::

:::{warning} Forwarding không giải quyết tất cả data hazard!

Data hazard `lw`-`or` trong tùy chọn B **không được giải quyết** bởi logic forwarding được đề xuất. Chu kỳ 5 là giai đoạn `EX` của lệnh `or`. Tuy nhiên, lệnh `lw` không hoàn thành đọc giá trị từ DMEM (để load vào thanh ghi `s1`) cho đến cuối chu kỳ 5. Kết quả của lần đọc bộ nhớ này không có sẵn trong thanh ghi pipeline `MEM/WB` cho đến _chu kỳ 6_.
:::

(sec-data-hazards-load-vi)=
## Load Data Hazard

Data hazard `lw`-`or` được mô tả [ở trên](#data-hazard-4-vi) là ví dụ về **load-use data hazard**. Hazard bắt nguồn từ giai đoạn `EX` của lệnh phụ thuộc vào đọc bộ nhớ từ giai đoạn `MEM` của lệnh load ngay trước đó **trong cùng chu kỳ xung clock**.

### Cách tiếp cận 1: Stall

:::{hint} Load có thể dẫn đến stall pipeline không tránh được.

Nếu một lệnh ngay sau lệnh load (tức là trong **load delay slot**) sử dụng kết quả của load, phần cứng phải **stall một chu kỳ** _và_ tận dụng forwarding `WB` đến `EX`.
:::

Xem xét chuỗi lệnh trong [Ví dụ 4](#data-hazard-4-vi) (Kiểm tra Nhanh trước). Pipeline phải stall một chu kỳ để tránh data hazard `lw`-`or`:


:::{list-table} [Ví dụ 4](#data-hazard-4-vi): Với đơn vị phát hiện hazard trong giai đoạn ID, một bubble được chèn bắt đầu từ chu kỳ 5, thay đổi lệnh `or` thành nop. Lệnh `or` được fetch và decode trong chu kỳ 3 và 4, nhưng giai đoạn `EX` của nó bị trì hoãn đến chu kỳ xung clock 6.
:label: data-hazard-4-load-vi
:header-rows: 1

* - Lệnh
  - 1
  - 2
  - 3
  - 4
  - 5
  - 6
  - 7
  - 8
  - 9
* - `add s0 t1 t2`
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  - 
  - 
* - `lw  s1 8(s0)`
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
  - 
  - 
* - `or → nop`
  - 
  - 
  - IF
  - ID
  - —
  - —
  - —
  - 
  - 
* - `or  t3 s1 t5`
  - 
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
  - 
* - `and t4 s1  t2`
  - 
  - 
  - 
  - 
  - IF
  - ID
  - EX
  - MEM
  - WB
:::

### Cách tiếp cận 2: Code scheduling

Xem xét [chuỗi lệnh](#tab-data-hazard-4-vi) trong Kiểm tra Nhanh trước. Chúng ta quan sát rằng nếu các lệnh `or` và `sll` được đổi chỗ, chúng ta có thể tránh stall không tránh được do data hazard `lw`-`or` tiềm năng.

Từ P&H 4.8:

> Mặc dù trình biên dịch thường dựa vào phần cứng để giải quyết hazard và do đó đảm bảo thực thi đúng, trình biên dịch phải hiểu pipeline để đạt hiệu năng tốt nhất. Nếu không, các stall không mong đợi sẽ giảm hiệu năng của mã được biên dịch.

Nói cách khác, nếu trình biên dịch biết bộ xử lý giải quyết data hazard như thế nào, nó có thể thiết kế các chuỗi lệnh để tránh stall không tránh được, ví dụ: do `load`. Cách tiếp cận này được gọi là **code scheduling**. Với kiến thức về kiến trúc bộ xử lý nền tảng, trình biên dịch sắp xếp lại mã để cải thiện hiệu năng.

Xem xét mã C dưới đây.

```c
A[3] = A[0] + A[1];
A[4] = A[0] + A[2];
```

Giả sử địa chỉ của mảng `int A[]` ở trong thanh ghi `a0` và các phần tử từ 0 đến 4 của `A` ở trong `t0` đến `t4` tương ứng.

Một biên dịch đơn giản sẽ dẫn đến stall không tránh được do các lệnh trong load delay slot cần kết quả load. Nếu pipeline triển khai forwarding `WB` đến `EX`, stalling phát sinh _hai chu kỳ bổ sung_, như dưới đây.

```bash
lw  t0 0(a0)
lw  t1 4(a0)
add t2 t0 t1    # stall một chu kỳ
sw  t2 12(a0)
lw  t3 8(a0)    # stall một chu kỳ
add t4 t0 t3
sw  t4 16(a0)
```

Trình biên dịch có thể sử dụng code scheduling bằng cách chèn các lệnh vào load delay slot không liên quan đến kết quả load. Với forwarding, chuỗi bảy lệnh mới dưới đây _không phát sinh bất kỳ mất hiệu năng nào do stalling_.

```bash
lw  t0 0(a0)
lw  t1 4(a0)
lw  t3 8(a0)
add t2 t0 t1
sw  t2 12(a0)
add t4 t0 t3
sw  t4 16(a0)
```

## Tổng kết: Phát hiện Data Hazard và Triển khai Forwarding

Một lần nữa, data hazard xảy ra giữa các giai đoạn khác nhau, khi một lệnh đọc thanh ghi trước khi lệnh trước hoàn thành ghi vào cùng thanh ghi.

Giả sử chúng ta có các tín hiệu `rs1`, `rs2`, `RegWEn`, và `rd` cho hai lệnh (lệnh *n* và lệnh *n + 1*) và chúng ta muốn xác định nếu data hazard tồn tại giữa các lệnh. Chúng ta có thể kiểm tra để xem nếu thanh ghi `rd` cho lệnh *n* khớp với thanh ghi `rs1` hoặc `rs2` của lệnh *n + 1*, chỉ ra data hazard.

Sau đó chúng ta có thể sử dụng phát hiện hazard để xác định đường forwarding/số stall (nếu có) cần thiết để đảm bảo thực thi lệnh đúng. Bằng mã giả, một phần có thể trông giống như sau:

```pseudo-code
if (rs1(n + 1) == rd(n) && RegWen(n) == 1) {
    set ASel for (n + 1) to forward ALU output from n
}
if (rs2(n + 1) == rd(n) && RegWen(n) == 1) {
    set BSel for (n + 1) to forward ALU output from n
}
```

Đọc P&H 4.8 để biết thêm thông tin.
