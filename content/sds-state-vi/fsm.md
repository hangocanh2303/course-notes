---
title: "Máy Trạng Thái Hữu Hạn"
subtitle: "Chuyển thể từ John Wawrzynek"
---

## Mục Tiêu Học Tập

* Định nghĩa các thành phần của Máy Trạng Thái Hữu Hạn (FSM).
* Mô tả cách các mạch có thể được biểu diễn dưới dạng FSM.
* Mô tả cách FSM chuyển đổi giữa các trạng thái.
* Biểu diễn một mạch phát hiện chuỗi ví dụ dưới dạng sơ đồ dạng sóng, FSM, bảng chân trị, và mạch.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/zVilpcMgdf0
:width: 100%
:title: "[CS61C FA20] Lecture 15.5 - State, State Machines: Finite State Machines"
:::

::::

Máy Trạng Thái Hữu Hạn (FSM)[^state-handout] là một trừu tượng hữu ích cho nhiều loại tác vụ tính toán. Chúng xuất hiện trong nhiều lĩnh vực của khoa học máy tính và thường được sử dụng trong thiết kế phần cứng. Ví dụ, với bộ nhớ đệm được điều khiển bằng phần cứng trong thiết kế bộ xử lý, một FSM sẽ được sử dụng để điều khiển tập hợp các hành động liên quan đến việc phục vụ các lệnh load và store từ bộ xử lý. Nó sẽ đi qua một loạt các "trạng thái" khi phản hồi các yêu cầu từ bộ xử lý, đôi khi cần truy cập bộ nhớ chính và đưa dữ liệu mới vào bộ nhớ đệm. Chúng ta có thể tưởng tượng việc mô tả hành vi bộ nhớ đệm như một thuật toán hoặc chương trình, và thuật toán này sẽ được hiện thực trong phần cứng với một FSM!

[^state-handout]: Những ghi chú này được chuyển thể từ ghi chú của Giáo sư John Wawrzynek: [Tài liệu State](../resources/state.pdf).

## Máy Trạng Thái Hữu Hạn
Một FSM có một tập hợp **đầu vào**, một tập hợp **đầu ra**, và một tập hợp hữu hạn các **trạng thái**. Tại bất kỳ thời điểm nào, máy đang ở trong một trong các trạng thái. Mỗi trạng thái có thể có một hoặc nhiều cung đưa máy từ trạng thái hiện tại sang trạng thái mới khi các giá trị đầu vào nhất định xuất hiện. Các giá trị đầu ra được liên kết với các cung, có nghĩa là khi máy rời khỏi một trạng thái, nó có thể thay đổi giá trị đầu ra, tùy thuộc vào trạng thái mới nào nó đang chuyển đến và giá trị đầu vào nào được nhìn thấy. Các trạng thái FSM được đặt tên duy nhất và, theo quy ước, các cung được ghi nhãn với các giá trị đầu vào và đầu ra liên kết như được thấy trong @fig-fsm-vi.

:::{figure} images/fsm.png
:label: fig-fsm-vi
:width: 100%
:alt: "Sơ đồ bong bóng và cung ba trạng thái với các di chuyển hai chiều giữa các trạng thái và một vòng lặp tự thân trên trạng thái giữa."

Sơ đồ FSM ba trạng thái với các cung chuyển đổi và một vòng lặp tự thân.
:::

:::{note} "Vòng lặp tự thân" là gì?

Vòng lặp tự thân (self-loop) là các cung FSM đại diện cho không có thay đổi trạng thái (xem Trạng thái 2 trong @fig-fsm-vi). Những cung này chỉ ra rằng một đầu vào nhất định, liên kết với cung đó, không thay đổi trạng thái hiện tại của máy. Tuy nhiên, những cung vòng lặp tự thân này vẫn có thể được liên kết với thay đổi giá trị đầu ra.
:::

## Ví Dụ: Ba Số Một
Với mạch logic tổ hợp và thanh ghi, FSM có thể được hiện thực trong phần cứng. Ví dụ, hãy xem xét một mạch nhỏ được sử dụng để phát hiện sự xuất hiện của ba số 1 liên tiếp trong một chuỗi bit đầu vào.

### Sơ Đồ Dạng Sóng

Trong mạch này, chúng ta giả sử các giá trị bit đơn được áp dụng một lần mỗi chu kỳ cho đầu vào của FSM, như được hiển thị trong sơ đồ thời gian trong @fig-three1s-timing-vi. Vào chu kỳ khi số 1 thứ ba của một chuỗi ba số 1 xuất hiện tại đầu vào của FSM, đầu ra của máy thay đổi thành 1 trong một chu kỳ. Sau đó, đầu ra trở về 0 cho đến khi FSM phát hiện một chuỗi ba số 1 khác.

:::{figure} images/three1s-timing.png
:label: fig-three1s-timing-vi
:width: 100%
:alt: "Sơ đồ thời gian cho đầu vào và đầu ra của FSM phát hiện chuỗi trong đó đầu ra xác nhận trong một chu kỳ khi số một liên tiếp thứ ba đến trong đầu vào."

Sơ đồ thời gian đầu vào và đầu ra cho mạch phát hiện ba bit 1 liên tiếp.
:::

### Máy Trạng Thái Hữu Hạn
Để thiết kế FSM cho mạch ví dụ này, chúng ta bắt đầu bằng cách vẽ sơ đồ trạng thái.

* Trạng thái `S0` là trạng thái máy ở trong khi nó bắt đầu đếm số 1. Điều này xảy ra cả ở đầu chuỗi, và mỗi lần máy reset và bắt đầu tìm kiếm chuỗi ba số 1 lại.
* Trạng thái `S1` là trạng thái máy ở trong khi nó đã thấy một `1`.
* Trạng thái `S2` là trạng thái máy ở trong khi nó đã thấy hai bit `1` liên tiếp.

Trong trạng thái `S0`, nếu máy thấy `1`, nó chuyển sang trạng thái `S1` với đầu ra `0`. Nếu máy thấy `0`, có vòng lặp tự thân và nó ở lại `S0` với đầu ra `0`.

Trong trạng thái `S1`, nếu máy thấy `1`, đầu ra vẫn là `0` và máy chuyển sang trạng thái `S2`. Nếu máy thấy `0`, nó reset, quay lại trạng thái `S0` với đầu ra `0`.

Trong trạng thái `S2`, nếu máy thấy thêm `1`, thì đầu ra được thay đổi thành `1` và máy reset, quay lại trạng thái `S0`. Nếu máy ở trong trạng thái `S2` và thấy `0`, thì đầu ra vẫn là `0` và máy reset, quay lại trạng thái `S0`.

Sử dụng các trạng thái và chuyển đổi trạng thái được xác định ở trên, chúng ta có thể vẽ sơ đồ trạng thái FSM (@fig-three1s-fsm-vi) với các trạng thái được ghi nhãn và các cung chuyển đổi được ghi nhãn.

:::{figure} images/three1s-fsm.png
:label: fig-three1s-fsm-vi
:width: 100%
:alt: "Sơ đồ máy trạng thái hữu hạn với các trạng thái S0, S1, và S2 đếm độ dài chuỗi của các số một đầu vào, reset khi có số không đầu vào, và đặt đầu ra cao trong một chu kỳ khi rời trạng thái S2."

Sơ đồ trạng thái FSM hoàn chỉnh cho mạch ba trạng thái phát hiện ba bit `1` liên tiếp.
:::

### Bảng Chân Trị
Sử dụng sơ đồ trạng thái FSM chúng ta vẽ ở trên (@fig-three1s-fsm-vi), bây giờ chúng ta có thể xây dựng bảng chân trị cho mạch ví dụ của chúng ta. Để ánh xạ sơ đồ trạng thái của chúng ta sang bảng chân trị, chúng ta sử dụng các mẫu bit `00`, `01`, và `10`, cho các trạng thái `S0`, `S1`, và `S2`, tương ứng.

Nửa trái của bảng hiển thị trạng thái hiện tại (PS), còn gọi là trạng thái hiện hành, và giá trị đầu vào. Nửa phải của bảng hiển thị trạng thái tiếp theo kết quả (NS) và giá trị đầu ra. @tab-three1s-tt-vi hiển thị bảng chân trị cho mạch phát hiện chuỗi ví dụ của chúng ta.

:::{table} Bảng Chân Trị Mạch Ba Số Một.
:label: tab-three1s-tt-vi
:align: center

| PS | ĐẦU VÀO | NS | ĐẦU RA |
| :--- | :--- | :--- | :--- |
| `00` | `0` | `00` | `0` |
| `00` | `1` | `01` | `0` |
| `01` | `0` | `00` | `0` |
| `01` | `1` | `10` | `0` |
| `10` | `0` | `00` | `0` |
| `10` | `1` | `00` | `1` |
:::

### Mạch
Trong hiện thực phần cứng của FSM, chúng ta giả sử rằng các chuyển đổi trạng thái được điều khiển bởi xung nhịp.

Trên mỗi chu kỳ xung nhịp, máy:
1. Kiểm tra các đầu vào
2. Chuyển sang trạng thái mới
3. Tạo ra đầu ra mới

Để thực hiện các bước này, chúng ta cần một thanh ghi (@fig-three1s-reg-vi) để giữ trạng thái hiện tại của máy. Chúng ta gán một mẫu bit duy nhất, như trong bảng chân trị, cho mỗi trạng thái trong sơ đồ trạng thái. Thanh ghi sau đó có thể theo dõi máy hiện đang ở trạng thái nào bằng cách giữ mẫu bit tương ứng với trạng thái cụ thể.

Tiếp theo, chúng ta cần thiết kế một mạch hiện thực hàm ánh xạ đầu vào và trạng thái hiện tại sang trạng thái tiếp theo và đầu ra, như được phác thảo trong bảng chân trị trong @tab-three1s-tt-vi. Chúng ta có thể sử dụng mạch logic tổ hợp (@fig-three1s-cl-vi) vì mạch này không cần tự ghi nhớ bất cứ điều gì.

:::::{grid} 2

::::{grid-item}
:::{figure} images/three1s-register.png
:label: fig-three1s-reg-vi
:width: 70%
:alt: "Khối thanh ghi trạng thái hiện tại giữ các bit trạng thái được mã hóa với cập nhật được đồng bộ cho bộ phát hiện chuỗi."

1. Thiết kế thanh ghi cho trạng thái hiện tại (PS).
:::
::::

::::{grid-item}
:::{figure} images/three1s-cl.png
:label: fig-three1s-cl-vi
:width: 70%
:alt: "Khối logic tổ hợp ánh xạ trạng thái hiện tại và đầu vào bên ngoài thành các bit trạng thái tiếp theo và đầu ra."

2. Thiết kế CL cho PS/ĐẦU VÀO $\rightarrow$ NS/ĐẦU RA.
:::
::::
:::::

Nhớ lại rằng các bit đầu vào được áp dụng cho FSM một lần mỗi chu kỳ xung nhịp. Do đó, chúng ta muốn buộc một chuyển đổi trạng thái trên mỗi chu kỳ xung nhịp và có thể đặt khối logic tổ hợp trong một vòng lặp với thanh ghi trạng thái như được hiển thị trong @fig-three1s-circuit-vi.

:::{figure} images/three1s-circuit.png
:label: fig-three1s-circuit-vi
:width: 70%
:alt: "Đường dữ liệu vòng kín nơi thanh ghi trạng thái nạp logic trạng thái tiếp theo, và đầu ra của nó lặp lại các đầu vào thanh ghi với xung nhịp chung."

Hiện thực thanh ghi trạng thái và logic tổ hợp của mạch phát hiện chuỗi ví dụ.
:::

Trên mỗi chu kỳ xung nhịp, khối logic tổ hợp xem xét trạng thái hiện tại và giá trị đầu vào để tạo ra mẫu bit cho trạng thái tiếp theo và giá trị của đầu ra (`0` hoặc `1`). Trên cạnh lên của xung nhịp, giá trị trạng thái mới được bắt bởi thanh ghi và quá trình bắt đầu lại.

::::{note} Bên trong khối CL là gì?

Các khối CL chứa logic tổ hợp cần thiết để hiện thực bảng chân trị của mạch. Trong hầu hết các trường hợp, chúng ta sử dụng một tập hợp các cổng logic để hiện thực bất kỳ hàm CL nào. Trong CS 61C, chúng ta sử dụng những cổng logic này như các khối xây dựng cơ bản cho mạch của chúng ta!

CL cho mạch phát hiện chuỗi ví dụ này được hiển thị trong @fig-three1s-cl-gates-vi.

Xem [Cổng Logic](#sec-logic-gates-vi) và [Đại Số Boolean](#sec-boolean-algebra-vi) để biết thêm chi tiết về các cổng phổ biến và cách dịch bảng chân trị thành logic tổ hợp!

:::{figure} images/three1s-cl-gates.png
:label: fig-three1s-cl-gates-vi
:width: 70%
:alt: "Sơ đồ mức cổng cho logic trạng thái tiếp theo và đầu ra được suy ra từ bảng chân trị bộ phát hiện chuỗi ba số một."

Logic tổ hợp cho mạch phát hiện chuỗi ví dụ.
:::
::::
