---
title: "Pipeline 5 Giai đoạn RISC-V"
---

(sec-five-stage-pipeline-vi)=
## Mục tiêu Học tập

* Giải thích cách thanh ghi pipeline truyền dữ liệu giữa các giai đoạn của pipeline 5 giai đoạn.
* So sánh các cách mà năm bước của một lệnh RISC-V được triển khai trong bộ xử lý pipeline 5 giai đoạn và bộ xử lý single-cycle.
* Thảo luận các cách tiếp cận để điều khiển pipelining.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/n5czmvbyX1w
:width: 100%
:title: "[CS61C FA20] Lecture 22.3 - Pipelining II: Pipelining Datapath"
:::

::::

:::{warning} Ôn tập mạch pipeline

Chúng tôi khuyến nghị mạnh mẽ trước tiên đọc về **thanh ghi trong mạch pipeline**, được đề cập trong [phần này](#sec-pipelining-circuits) của đơn vị Hệ thống Số Đồng bộ.

:::

Trong phần này, chúng ta biến đổi bộ xử lý single-cycle thành **bộ xử lý pipeline năm giai đoạn** RISC-V.
Một bộ xử lý pipeline "phân tách" [năm bước thực thi lệnh RISC-V](#sec-five-steps-vi) thành các **stage**, trong đó các lệnh thực thi theo stage, và các stage của các lệnh khác nhau thực thi **song song** trong cùng chu kỳ xung clock.

(sec-pipeline-datapath-vi)=
## Datapath Pipeline

Chuyển đổi giữa các hình ảnh bên dưới để hình dung datapath pipeline năm giai đoạn. Tại mỗi cạnh lên xung clock, **thanh ghi pipeline** mang dữ liệu và tín hiệu điều khiển đến giai đoạn tiếp theo. Chúng ta thảo luận về điều khiển [bên dưới](#sec-pipeline-control-vi).

:::::{tab-set}
::::{tab-item} Datapath Pipeline 5 Giai đoạn
:::{figure} images/five-stage-pipeline.png
:label: fig-five-stage-pipeline
:width: 100%
:alt: "Datapath pipeline năm giai đoạn với IF, ID, EX, MEM, và WB được phân tách bởi thanh ghi pipeline."

Sơ đồ datapath RISC-V năm giai đoạn. Thanh ghi pipeline được chèn giữa các giai đoạn để giữ tín hiệu cho đến chu kỳ xung clock tiếp theo.
:::
::::
::::{tab-item} Datapath Single-Cycle
:::{figure} images/five-stage-single-cycle.png
:label: fig-five-stage-single-cycle
:width: 100%
:alt: "Datapath single-cycle được phân chia thành năm bước thực thi lệnh logic để so sánh."

Datapath RISC-V single-cycle, được phân tách thành [năm bước](#sec-five-steps-vi).
:::
::::
:::::

:::{note} Thanh ghi pipeline

Mỗi giai đoạn cần xử lý dữ liệu từ một lệnh khác nhau. Để làm điều này, pipeline năm giai đoạn chèn **thanh ghi pipeline** giữa mỗi giai đoạn.

**Thanh ghi pipeline** là các thanh ghi được chèn giữa các giai đoạn để giữ lại kết quả của các lệnh riêng lẻ được tính trong một giai đoạn để sử dụng trong giai đoạn tiếp theo. Các thanh ghi này cho phép các phần của một bộ xử lý đơn lẻ được chia sẻ bởi nhiều lệnh.

:::

Giống như [datapath single-cycle](#sec-single-cycle-vi), trong pipeline năm giai đoạn, dữ liệu và tín hiệu điều khiển vẫn thường di chuyển từ trái sang phải. Vẫn có hai vòng lặp. Chúng ta mở rộng trích dẫn ban đầu từ P&H 4.7:

> Lệnh và dữ liệu di chuyển thường từ trái sang phải qua năm giai đoạn khi chúng hoàn thành thực thi. Quay lại phép so sánh giặt đồ, quần áo trở nên sạch hơn, khô hơn, và có tổ chức hơn khi chúng di chuyển qua dây chuyền, và chúng không bao giờ di chuyển ngược lại.
>
> Tuy nhiên, có hai ngoại lệ cho luồng từ trái sang phải của lệnh:
>
> * Giai đoạn write-back, đặt kết quả trở lại tệp thanh ghi ở giữa datapath
> * Việc chọn giá trị tiếp theo của PC, chọn giữa PC tăng dần và địa chỉ branch từ giai đoạn MEM
>
> Dữ liệu chảy từ phải sang trái không ảnh hưởng đến lệnh hiện tại; các chuyển động dữ liệu ngược này chỉ ảnh hưởng đến các lệnh sau trong pipeline. Lưu ý rằng luồng dữ liệu từ phải sang trái đầu tiên có thể dẫn đến data hazard và luồng thứ hai dẫn đến control hazard.[^hazards-vi]

[^hazards-vi]: Chúng ta thảo luận về [hazard](#sec-pipeline-hazards) trong phần sau của chương này.

Chúng ta định nghĩa thanh ghi pipeline bởi hai giai đoạn chúng được chèn giữa, ví dụ: thanh ghi pipeline `IF/ID` chỉ các thanh ghi giữa giai đoạn `IF` và `ID`. Từ P&H 4.7:

> Quay lại phép so sánh giặt đồ, chúng ta có thể có một giỏ giữa mỗi cặp giai đoạn để giữ quần áo cho bước tiếp theo.

## Năm Giai đoạn của Bộ xử lý Pipeline RISC-V

Chúng ta bây giờ xem lại [năm bước thực thi lệnh RISC-V](#sec-five-steps-vi) trong ngữ cảnh của datapath pipeline năm giai đoạn mới trong @fig-five-stage-pipeline. Đây là năm bước.

```{embed} #sec-five-steps-vi
```

Và đây là các giai đoạn của bộ xử lý pipeline RISC-V năm giai đoạn, một bước mỗi giai đoạn.

(sec-five-stages-vi)=
:::{note} Năm _giai đoạn_ của một lệnh RISC-V

1. **Instruction Fetch (`IF`)**: Fetch lệnh hiện tại từ IMEM và tính `PC + 4`.

    Một lệnh thực thi đồng thời ở giai đoạn sau (`MEM`) thiết lập tín hiệu điều khiển `PCSel` để xác định lệnh tiếp theo thực thi.
1. **Instruction Decode (`ID`)**: Giải mã thao tác từ các bit của lệnh, đọc thanh ghi từ RegFile, và tạo immediate 32-bit.

    Các thanh ghi pipeline `IF/ID` cung cấp các bit lệnh.
1. **Execute (`EX`)**: Sử dụng ALU để thực hiện phép toán thanh ghi-thanh ghi (R-Type), phép toán thanh ghi-immediate (I-Type, S-Type), phép toán PC-immediate (load, B-Type, `jal`, `auipc`), hoặc lấy immediate (`lui`).

    Các thanh ghi pipeline `ID/EX` cung cấp các toán hạng cho ALU, cũng như các bit lệnh xác định thao tác ALU.

    Ngoài ra, sử dụng Branch Comparator để so sánh các giá trị thanh ghi nguồn, và truyền kết quả vào đơn vị điều khiển.
1. **Memory Access (`MEM`)**: Load dữ liệu từ (hoặc store dữ liệu vào) DMEM.

    Các thanh ghi pipeline `EX/MEM` cung cấp địa chỉ bộ nhớ và dữ liệu để ghi.

    Ngoài ra, kết quả của Branch Comparator được biết tại thời điểm này. Sử dụng đơn vị điều khiển để xác định tín hiệu điều khiển `PCSel`. Trong [phần sau](#sec-control-hazards), chúng ta thảo luận cách thiết kế này có thể dẫn đến **control hazard**, hoặc thực thi lệnh không theo thứ tự tiềm năng.
1. **Write Back (`WB`)**: Ghi lại vào RegFile.

    Các thanh ghi pipeline `MEM/WB` cung cấp dữ liệu để ghi, cũng như các bit lệnh xác định những gì cần ghi (cho mux `WBSel`).

    Nhớ lại rằng trong thảo luận của chúng ta về [thời gian lệnh](#sec-instruction-timing-vi) giai đoạn `WB` của datapath single-cycle chỉ đơn giản bao gồm MUX và thời gian setup thanh ghi, vì giả định rằng phần tử RegFile thực hiện ghi kích hoạt cạnh lên. Trong [phần sau](#sec-data-hazards), chúng ta thảo luận về một phần tử RegFile được sửa đổi có thể thực hiện ghi vào _cạnh xuống_ của xung clock.
:::

(sec-pipeline-registers-vi)=
## Thanh ghi Pipeline trong Datapath 5 Giai đoạn

Dưới đây, chúng ta giải thích @fig-five-stage-pipeline-registers từ góc nhìn của những gì được đưa _vào_ mỗi tập thanh ghi pipeline. Ví dụ, khi thảo luận về thanh ghi `IF/ID`, chúng ta mô tả lệnh đang thực thi trong giai đoạn `IF`.

::::{figure}
:label: fig-five-stage-pipeline-registers
:alt: "Slide nhúng animation những gì mỗi tập thanh ghi pipeline nắm bắt giữa các giai đoạn IF, ID, EX, MEM, và WB trong datapath RISC-V năm giai đoạn."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vSMgaBei-28IrooFMzb-ygC5ZURZV2WCwkfozWm8e6q74JxnLWYUySPoWurEwVVUgAL7rDPgbnxmULt/pubembed?start=false&loop=false
:width: 100%
:title: "Animation đi qua các thanh ghi pipeline trong datapath năm giai đoạn, bao gồm những gì được đưa vào mỗi thanh ghi, như được thảo luận trong phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1U2q6p5iPYqNPFLD4cDYgE2T4o7FA0dfSVsiAt0qBCsA/edit?usp=sharing)"
:::
Animation đi qua văn bản được đánh số trong phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1U2q6p5iPYqNPFLD4cDYgE2T4o7FA0dfSVsiAt0qBCsA/edit?usp=sharing). Hình ảnh hoàn chỉnh hơn ở @fig-five-stage-summary.
::::

:::{note} `IF/ID`: Hiển thị Giải thích
:class: dropdown

Một lệnh đang ở giai đoạn `IF` truyền những thứ sau vào thanh ghi pipeline `IF/ID`:

* `PC`: PC (tức là, địa chỉ của lệnh này) được lưu trong trường hợp cần trong chu kỳ xung clock sau, ví dụ: trong giai đoạn `EX`, nơi các lệnh branch có điều kiện/jump vô điều kiện sử dụng ALU để tính địa chỉ tương đối PC.
* `inst`: Dữ liệu lệnh, được đọc từ IMEM trong giai đoạn này, được lưu để sử dụng ngay trong giai đoạn tiếp theo của lệnh, `ID`, ví dụ: để xác định thanh ghi nguồn. Nhớ lại rằng trong giai đoạn `IF`, máy tính không thể biết loại lệnh nào đang được fetch tiềm năng, vì vậy nó phải chuẩn bị cho bất kỳ lệnh nào, truyền thông tin có thể cần xuống pipeline (P&H 4.7).
:::

:::{note} `ID/EX`: Hiển thị Giải thích
:class: dropdown

Một lệnh đang ở giai đoạn `ID` truyền những thứ sau vào thanh ghi pipeline `ID/EX`:

* `PC`: Không thay đổi nhưng có thể vẫn cần cho lệnh này ở giai đoạn sau (xem thảo luận `IF/ID`). Chúng ta truyền dữ liệu này trực tiếp giữa các thanh ghi pipeline `IF/ID` và `ID/EX`.
* `RegReadData1`: Tín hiệu đầu ra `rdata1` từ RegFile được đọc trong giai đoạn `ID` của lệnh này.
* `RegReadData2`: Tín hiệu đầu ra `rdata2` từ RegFile, được đọc trong giai đoạn `ID` của lệnh này.
* `imm`: Đầu ra của immediate generator, được tạo trong giai đoạn `ID` của lệnh này.
* `inst`: Một lần nữa, chúng ta truyền lệnh cùng với dữ liệu của nó đến giai đoạn tiếp theo của lệnh, `EX`.

:::

:::{note} `EX/MEM`: Hiển thị Giải thích
:class: dropdown

Một lệnh đang ở giai đoạn `EX` truyền những thứ sau vào thanh ghi pipeline `EX/MEM`:

* `PC`: Không thay đổi như một phần của giai đoạn `EX` của lệnh này nhưng có thể cần trong các chu kỳ sau của lệnh này, ví dụ: với `auipc`.
* `ALUOut`: Kết quả ALU, được tính trong giai đoạn này.
* `RegRead2`: Dữ liệu trong thanh ghi `rs2`; nếu lệnh này là store (S-Type), nó sẽ cần dữ liệu này trong giai đoạn `MEM`.
* `inst`: Một lần nữa, chúng ta truyền lệnh cùng với dữ liệu của nó đến giai đoạn tiếp theo của lệnh, `MEM`.
:::

:::{note} `MEM/WB`: Hiển thị Giải thích
:class: dropdown

Một lệnh đang ở giai đoạn `MEM` truyền những thứ sau vào thanh ghi `MEM/WB`.

* `ALU`: Không thay đổi trong giai đoạn `MEM` của lệnh này, nhưng cần như một trong các tùy chọn để ghi lại vào RegFile trong giai đoạn tiếp theo của lệnh này, `WB`.
* `PC+4`: Để tránh gửi cả `PC` và `PC + 4` xuống pipeline, chúng ta thêm một mạch con bổ sung để tính lại `PC + 4` như một trong các tùy chọn write-back trong giai đoạn tiếp theo của lệnh này, `WB`.
* `Mem`: Kết quả của việc đọc DMEM, được đọc trong giai đoạn này.
* `inst`: Chúng ta đang lặp lại ở đây. :-)
:::

(sec-pipeline-control-vi)=
## Điều khiển Pipeline

Vì pipelining datapath giữ nguyên ý nghĩa của các đường điều khiển, chúng ta có thể sử dụng cùng các giá trị điều khiển nhưng bây giờ nhóm các tín hiệu điều khiển theo giai đoạn pipeline, như trong @tab-controller-signals-pipeline-vi và @fig-five-stage-control.

:::{table} Các tín hiệu cho logic điều khiển, được nhóm theo giai đoạn pipeline.
:label: tab-controller-signals-pipeline-vi

| Tên | Giai đoạn |
| :-- | :-- |
| `ImmSel` | `ID` |
| `BrUn` | `EX` |
| `ASel` | `EX` |
| `BSel` | `EX` |
| `ALUSel` | `EX` |
| `MemRW` | `MEM` |
| `PCSel` | `MEM` |
| `WBSel` | `WB` |
| `RegWEn` | `WB` |

:::

Chúng ta lưu ý không có gì đặc biệt để điều khiển trong giai đoạn `IF`, vì các tín hiệu điều khiển để đọc bộ nhớ lệnh và ghi PC luôn được khẳng định ngầm.

:::{tip} Kiểm tra Nhanh
:label: block-pcsel-mem-vi

Giả sử chúng ta thực thi một lệnh B-Type trên datapath pipeline năm giai đoạn. Tại giai đoạn nào của lệnh này chúng ta sẽ biết liệu branch có điều kiện được thực hiện?

:::

:::{note} Hiển thị Đáp án
:class: dropdown

Câu hỏi được diễn đạt theo cách thông tục có chủ đích: "Biết liệu branch có điều kiện được thực hiện" nghĩa là gì đối với CPU?

Điều này thường thuộc phạm vi của *điều khiển*--không phải dữ liệu. Sử dụng định nghĩa này, hãy giả sử chúng ta định nghĩa "thực hiện/không thực hiện" là khi chúng ta biết giá trị nào để ghi vào PC, tức là giá trị của `PCSel`. Như được hiển thị trong @fig-five-stage-control, tín hiệu `PCSel` được xác định trong **giai đoạn `MEM`.**

Điều khiển tính toán kết quả của `PCSel` trong giai đoạn `MEM`—nhưng đầu ra của Branch Comparator có sẵn trong giai đoạn `EX`.
 Vì vậy *dữ liệu* sẵn sàng sớm hơn một giai đoạn so với tín hiệu *điều khiển* tương ứng! Xem CS 152 cho một thiết kế thay thế tận dụng phát hiện này.

:::

:::{figure} images/five-stage-with-control.png
:label: fig-five-stage-control
:width: 100%
:alt: "Sơ đồ pipeline năm giai đoạn được chú thích với các tín hiệu điều khiển được mang và sử dụng ở các giai đoạn sau."

Sơ đồ bộ xử lý RISC-V năm giai đoạn với điều khiển.
:::

### Triển khai Điều khiển Pipeline

Triển khai điều khiển có nghĩa là thiết lập các đường điều khiển này về giá trị đúng trong mỗi giai đoạn cho mỗi lệnh.

@fig-five-stage-control ở trên hiển thị **một** cách tiếp cận. Mỗi giai đoạn bây giờ có một đơn vị điều khiển riêng biệt xác định các tín hiệu điều khiển dựa trên lệnh đang thực thi trong giai đoạn đó. Điều này được minh họa bởi các đầu vào cho các nhóm tín hiệu điều khiển khác nhau: `inst (ID)`, `inst (EX)`, `inst (M)`, và `inst (WB)`.

::::{note} Click để hiển thị cách tiếp cận thứ hai.
:class: dropdown

Một cách tiếp cận khác tính toán càng nhiều tín hiệu điều khiển càng tốt trong instruction decode (`ID`) vì tất cả các tín hiệu điều khiển ngoại trừ `PCSel` có thể được suy ra từ lệnh. Như được hiển thị trong @fig-pipelined-control, điều này mở rộng thanh ghi pipeline để bao gồm thông tin điều khiển để pipeline các "từ" điều khiển giữa các giai đoạn. Cách tiếp cận này tái sử dụng nhiều mạch điều khiển từ bộ xử lý single-cycle, nhưng nó phá vỡ nhóm trong @tab-controller-signals-pipeline-vi.

:::{figure} images/pipelined-control.png
:label: fig-pipelined-control
:width: 80%
:alt: "Các trường thanh ghi pipeline bổ sung cho các bit điều khiển chảy cùng với các giá trị datapath giữa các giai đoạn."

Sơ đồ thanh ghi pipeline bổ sung cho điều khiển.
:::

::::

## Tổng kết

Bộ xử lý pipeline năm giai đoạn đầy đủ được hiển thị trong @fig-five-stage-summary; điều này cũng có trên thẻ tham khảo khóa học.

:::{figure} images/five-stage-summary.png
:label: fig-five-stage-summary
:width: 100%
:alt: "Sơ đồ tổng kết kết hợp của datapath năm giai đoạn và các đường điều khiển pipeline liên quan."

Sơ đồ bộ xử lý RISC-V năm giai đoạn: datapath và điều khiển.
:::

Pipeline 5 giai đoạn mà chúng ta đã nghiên cứu rất phổ biến trong nhiều thiết bị: ô tô, thiết bị gia dụng, v.v.
