---
title: "Control Hazard"
---

(sec-control-hazards-vi)=
## Mục tiêu Học tập

* Cho các chuỗi lệnh và kiến trúc bộ xử lý, xác định các control hazard tiềm năng.
* Giải thích cách tiếp cận mà pipeline RISC-V năm giai đoạn sử dụng để giải quyết control hazard.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/CnDgqw-bsyQ
:width: 100%
:title: "[CS61C FA20] Lecture 23.2 - Pipelining III: Control Hazards"
:::

::::

Từ [trước](#sec-pipeline-hazards-vi):

:::{embed} #block-def-hazard-control-vi
:::

Control hazard xảy ra khi lệnh được fetch có thể không phải là lệnh cần thiết. Nói cách khác, _liệu_ một lệnh thực thi hay không phụ thuộc vào kết quả của lần thực thi trước. Từ P&H 4.6:

> Giả sử đội giặt của chúng ta được giao nhiệm vụ vui là giặt đồng phục của đội bóng đá. Do quần áo bẩn như thế nào, chúng ta cần xác định liệu bột giặt và cài đặt nhiệt độ nước chúng ta chọn có đủ mạnh để làm sạch đồng phục không...Trong pipeline giặt đồ, chúng ta phải chờ đến giai đoạn thứ hai để kiểm tra đồng phục khô xem có cần thay đổi thiết lập máy giặt hay không.

Control hazard xảy ra với **lệnh jump và branch**. Chúng ta phải bắt đầu fetch lệnh tiếp theo sau jump/branch vào chu kỳ xung clock tiếp theo. Tuy nhiên, pipeline không thể biết lệnh tiếp theo nên là gì—vì nó chỉ _vừa đọc_ lệnh jump/branch từ bộ nhớ.

```{embed} #block-pcsel-mem-vi
```

:::{note} Hiển thị Đáp án
:class: dropdown

Kiểm tra Nhanh này đã được đề cập trong phần trước: [Điều khiển Pipeline](#sec-pipeline-control-vi). Ngoài ra, xem lại `MEM` trong [phần năm giai đoạn](#sec-five-stages-vi).

:::

Chúng ta minh họa control hazard với lệnh B-Type trong @tab-waterfall-branch-vi. Chúng ta thực hiện branch trong giai đoạn `MEM`, _sau khi_ chúng ta biết kết quả của `ALU` và branch comparator từ giai đoạn `EX`. Nếu không, nếu chúng ta không thực hiện branch, chúng ta tiếp tục chọn lệnh tiếp theo là `PC + 4`.


```{list-table} Control hazard có thể xảy ra với lệnh branch, vì các lệnh được thực thi trước khi kết quả branch được biết.
:label: tab-waterfall-branch-vi
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
* - `beq t0 t1 Label`
  - IF
  - ID
  - EX
  - M
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
  - M
  - WB
  -
  -
  -
* - `or  t6 a0 t0`
  -
  -
  - IF
  - ID
  - EX
  - M
  - WB
  -
  -
* - `xor t5 t1 s0`
  -
  -
  -
  - IF
  - ID
  - EX
  - M
  - WB
  -
* - `Label: sw  s0 8(t3)`
  -
  -
  -
  -
  - IF
  - ID
  - EX
  - M
  - WB
```

Đối với lệnh `beq`, chúng ta phải chờ đến chu kỳ thời gian 4 (giai đoạn `MEM` của `beq`) để xác định nếu branch được thực hiện. Lúc đó, ba lệnh tiếp theo đã bắt đầu thực thi, trước khi kết quả branch được biết.

Nếu branch thực sự được thực hiện, để tránh control hazard chúng ta cần đảm bảo rằng ba lệnh `sub`, `or`, và `xor` không thực thi đến hoàn thành. Chúng ta thảo luận cách tiếp cận đơn giản trước, sau đó một cách thông minh hơn.

## Cách tiếp cận 1: Stall khi branch

Một khả năng là bất cứ khi nào chúng ta phát hiện lệnh branch (hoặc jump), chúng ta stall pipeline khi phát hiện branch cho đến khi PC đúng được biết. Triển khai này sẽ liên quan đến nối thêm để ngay khi branch được decode trong giai đoạn `ID`, các lệnh khác bị stall cho đến khi kết quả branch được xác định trong giai đoạn `MEM`.

Trong @tab-waterfall-branch-simple-vi, **mỗi lệnh branch phát sinh stall ba lệnh**, bất kể branch có được thực hiện hay không.

```{list-table} Cách tiếp cận 1. Với lệnh branch, luôn stall cho đến khi lệnh tiếp theo được xác định. Dấu gạch ngang (–) chỉ ra rằng pipeline bị flush và các lệnh bị ảnh hưởng "không làm gì".
:label: tab-waterfall-branch-simple-vi
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
* - `beq t0, t1, Label`
  - IF
  - ID
  - EX
  - M
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
* - `nop`
  - 
  - 
  - 
  - –
  - –
  - –
  - –
  - –
  - 
* - `Label: sw s0, 8(t3)`
  - 
  - 
  - 
  - 
  - IF
  - ID
  - EX
  - M
  - WB
```

Cách tiếp cận này đơn giản nhưng chậm. Nó khá tốn kém nếu chương trình của chúng ta có nhiều branch. Sau cùng, nếu chúng ta _không_ thực hiện branch, chúng ta nên tiến hành thực thi lệnh `sub` sau cùng.

## Cách tiếp cận 2: Giả định branch không được thực hiện

Một cách tiếp cận khác có thể là _chỉ stall khi cần_. Nói cách khác, tiến hành thực thi các lệnh theo tuần tự, và ngay lập tức **flush pipeline** một khi được xác định rằng branch nên được thực hiện. So sánh hai trường hợp dưới đây.

Nếu branch được thực hiện, chúng ta stall **ba chu kỳ**. Trong @tab-waterfall-branch-taken-vi, một khi được xác định rằng branch được thực hiện trong chu kỳ 4, flush pipeline. Điều này liên quan đến việc chuyển các lệnh ở giai đoạn `IF`, `ID`, `EX` thành no-op. Sau đó, trong chu kỳ 5, lệnh đúng (lệnh `sw` được branch đến) được thực thi.

```{list-table} Cách tiếp cận 2. Trong chu kỳ 4, chúng ta xác định rằng branch được thực hiện. Dấu gạch ngang (–) chỉ ra rằng pipeline bị flush và các lệnh bị ảnh hưởng "không làm gì".
:label: tab-waterfall-branch-taken-vi
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
* - `beq t0 t1 Label`
  - IF
  - ID
  - EX
  - M
  - WB
  - 
  - 
  - 
  - 
* - `sub → nop`
  - 
  - IF
  - ID
  - EX
  - –
  - –
  - 
  - 
  - 
* - `or  → nop`
  - 
  - 
  - IF
  - ID
  - –
  - –
  - –
  - 
  - 
* - `xor  → nop`
  - 
  - 
  - 
  - IF
  - –
  - –
  - –
  - –
  - 
* - `Label: sw  s0 8(t3)`
  - 
  - 
  - 
  - 
  - IF
  - ID
  - EX
  - M
  - WB
```

Nếu branch không được thực hiện, chúng ta stall **không chu kỳ nào**. Trong @tab-waterfall-branch-not-taken-vi, nếu được xác định rằng branch _không_ được thực hiện trong chu kỳ 4, không làm gì bất thường. Vì các lệnh `sub`, `or`, và `xor` đã ở trong pipeline, chúng ta không lãng phí bất kỳ chu kỳ nào vào stalling.

```{list-table} Cách tiếp cận 2. Nếu branch không được thực hiện, tiến hành bình thường. Bảng này do đó trông giống hệt @tab-waterfall-branch-vi!
:label: tab-waterfall-branch-not-taken-vi
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
* - `beq t0 t1 Label`
  - IF
  - ID
  - EX
  - M
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
  - M
  - WB
  -
  -
  -
* - `or  t6 a0 t0`
  -
  -
  - IF
  - ID
  - EX
  - M
  - WB
  -
  -
* - `xor t5 t1 s0`
  -
  -
  -
  - IF
  - ID
  - EX
  - M
  - WB
  -
* - `Label: sw  s0 8(t3)`
  -
  -
  -
  -
  - IF
  - ID
  - EX
  - M
  - WB
```

:::{hint} Pipeline 5 Giai đoạn giải quyết control hazard bằng cách giả định rằng branch không được thực hiện.

Từ P&H 4.6:

> Một cách tiếp cận đơn giản là luôn dự đoán rằng branch có điều kiện sẽ không được thực hiện. Khi bạn đúng, pipeline tiến hành ở tốc độ đầy đủ. Chỉ khi branch có điều kiện được thực hiện thì pipeline mới stall.

:::

Như hầu hết tất cả các trường hợp chúng ta đã thấy cho đến nay, chúng ta có thể đánh giá hiệu năng của thiết kế này—tức là giả định rằng branch không được thực hiện, sau đó stall nếu chúng được thực hiện—thiết kế trên chương trình chuẩn. Nếu chúng ta có chương trình thực hiện nhiều branch, thiết kế này sẽ hoạt động kém. Nếu chúng ta có chương trình không thực hiện nhiều branch (mặc dù có nhiều lệnh branch tồn tại), thiết kế này sẽ hoạt động tuyệt vời!

## Cách tiếp cận 3: Dự đoán Branch

Cách tiếp cận đơn giản được sử dụng bởi pipeline năm giai đoạn của chúng ta, được mô tả trong tiểu mục trước, là ví dụ về **dự đoán branch**.

Máy tính ngày nay sử dụng phiên bản phức tạp hơn của dự đoán branch. Một phiên bản cứng nhắc có thể dự đoán trước khi chương trình thực thi rằng một số branch có điều kiện được thực hiện và một số không được thực hiện. Cách tiếp cận này có thể dựa vào hành vi khuôn mẫu hoặc trường hợp trung bình, hoặc thậm chí "tung đồng xu" mỗi chu kỳ.

Các phiên bản phức tạp hơn của dự đoán branch là động và xảy ra trong phần cứng, thực hiện các dự đoán tùy thuộc vào hành vi của từng branch có điều kiện. Các bộ dự đoán phần cứng động này có thể thay đổi dự đoán cho branch có điều kiện trong suốt vòng đời của chương trình.

Để biết thêm các cách tiếp cận dự đoán branch và control hazard, xem P&H 4.9.
