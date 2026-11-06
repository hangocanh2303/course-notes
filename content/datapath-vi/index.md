---
title: "Giới Thiệu"
subtitle: "Datapath vs. Control; 5 Bước Thực Thi Lệnh RISC-V"
---

(sec-single-cycle-vi)=
## Mục Tiêu Học Tập

* Xác định năm bước để thực thi một lệnh RISC-V.
* Giải thích rằng datapath của bộ xử lý mô tả luồng dữ liệu giữa các phần tử trạng thái của bộ xử lý.
* Giải thích rằng control của bộ xử lý mô tả các tín hiệu cần thiết để đảm bảo các hoạt động datapath thực thi đúng các lệnh.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/YAMqWaTZy8k
:width: 100%
:title: "[CS61C FA20] Lecture 18.1 - Single-Cycle CPU Datapath I: RISC-V Processor Design"
:::

::::

Trong chương này, chúng ta sẽ thiết kế một bộ xử lý RISC-V và kết nối các phần phần mềm và phần cứng trong câu chuyện CS 61C của chúng ta. Trong [hình bên dưới](#fig-great-idea-1), chương này thiết kế phần cứng có thể thực sự thực thi mã RISC-V.

:::{figure} #fig-great-idea-1
:width: 100%
:enumerated: false
:alt: "Sơ đồ trừu tượng phân lớp: compiler, assembler, sau đó mã máy phía trên đường ISA, với kiến trúc phần cứng và mạch logic bên dưới. Bên phải hiển thị các ví dụ tương ứng từ C và hợp ngữ RISC-V qua nhị phân, sơ đồ khối bộ xử lý, và logic cổng NAND."

Ý Tưởng Lớn #1: Trừu tượng hóa (Abstraction).
:::

Có **nhiều** biến thể phần cứng bộ xử lý RISC-V như vậy; chúng ta sẽ thảo luận một bộ xử lý như vậy trong chương này, và một cái thứ hai sẽ sớm thôi. Có nhiều thiết kế chip RISC-V khác trên thị trường[^sifive]. Cuối cùng, _tất cả bộ xử lý RISC-V phải thực thi đúng các lệnh RISC-V_. Hãy xem cách thực hiện!

[^sifive]: Xem [SiFive](https://www.sifive.com/boards/hifive-premier-p550) để có thêm ví dụ.

:::{hint} Mô tả phần cứng vốn dĩ là song song

Trước khi tiếp tục, hãy nhớ rằng chúng ta nhấn mạnh tính song song trong khóa học này. Ngay cả ở cấp độ phần cứng, tất cả các cổng hoạt động song song, và các phần tử trạng thái đều được đồng bộ với cùng một clock. Điều này có nghĩa là các đơn vị chức năng này đều hoạt động song song!

:::

:::{figure} ../great-ideas/images/new-school-machine-structures.png
:width: 100%
:enumerated: false
:alt: "Stack cấu trúc máy hiện đại nhấn mạnh tính song song: các lớp phần mềm phía trên ISA, tổ chức phần cứng bên dưới, với các thành phần song song multicore, vector, và accelerator rõ ràng."
:label: fig-great-idea-new-school-vi

Cấu trúc Máy "Trường Phái Mới" tận dụng tính song song trong cả phần mềm và phần cứng.
:::

Nếu chúng ta thực thi một lệnh đơn trong một chu kỳ clock đơn, các khối logic tổ hợp có thể đang hoạt động trên dữ liệu rác cho đến khi chúng nhận được các giá trị đúng từ các phần tử trạng thái, và chúng ta thường cần đợi đến cuối chu kỳ clock trước khi có thể đảm bảo các giá trị ổn định cho lệnh tiếp theo. Trong chương tiếp theo, chúng ta sẽ thảo luận về pipelining, cho phép chúng ta thực thi nhiều lệnh song song.

## Datapath vs. Control

Để xây dựng một bộ xử lý, hãy nhớ lại bố cục máy tính cơ bản bên dưới (từ [chương trước](#sec-architecture-elements)). Bộ xử lý ở bên trái hình là phần hoạt động của máy tính thực hiện tất cả công việc. Nó truy cập bộ nhớ để đọc/ghi dữ liệu, nó thực thi các lệnh, và nó đưa ra quyết định về lệnh nào sẽ thực thi tiếp theo.

:::{embed} #fig-von-neumann
:::

Bên trong bộ xử lý, có hai thành phần chính:

* **Datapath** ("cơ bắp"): Phần của bộ xử lý chứa phần cứng cần thiết để thực hiện các hoạt động mà bộ xử lý yêu cầu. Datapath RISC-V mô tả luồng dữ liệu giữa các phần tử cốt lõi của bộ xử lý RISC-V.
* **Control** ("bộ não"): Phần của bộ xử lý (cũng trong phần cứng) cho datapath biết cần làm gì. Control mô tả các tín hiệu cần thiết để đảm bảo các phần tử của datapath thực thi đúng một lệnh RISC-V.

:::{warning} Datapath vs. Control

Control và Datapath là hai _sự trừu tượng_ cùng nhau tạo thành CPU. Bên trong, cả hai đều được thiết kế với logic tổ hợp và các phần tử trạng thái.
:::

Một phép so sánh đến từ hoạt động của [Trạm Vũ trụ Quốc tế](https://en.wikipedia.org/wiki/International_Space_Station) (ISS). Trạm vũ trụ giống như datapath có tất cả phần cứng cần thiết để thực hiện các hoạt động trong không gian. Trung tâm điều khiển nhiệm vụ ở trên Trái Đất và, với một số dữ liệu từ trạm vũ trụ, gửi chỉ dẫn trở lại ISS. Cả trạm vũ trụ và trung tâm điều khiển nhiệm vụ đều có máy tính (ví dụ: logic tổ hợp) và trạng thái, và cả hai đều cần thiết để đảm bảo hoạt động đúng của Trạm Vũ trụ Quốc tế như một hệ thống.

:::::{grid} 2
::::{grid-item}
:::{figure} images/space-iss.jpg
:enumerated: false
:width: 50%
:alt: "Ảnh Trạm Vũ trụ Quốc tế được sử dụng như phép so sánh cho datapath bộ xử lý như phần cứng thực hiện các hoạt động."
Datapath RISC-V giống như một trạm vũ trụ.
:::
::::

::::{grid-item}
:::{figure} images/space-mission-control.jpg
:width: 50%
:alt: "Ảnh các nhà điều hành trung tâm điều khiển nhiệm vụ được sử dụng như phép so sánh cho logic control điều khiển các hành động datapath."
:enumerated: false
Logic control RISC-V giống như trung tâm điều khiển nhiệm vụ.
:::
::::
:::::

## Bộ Xử Lý Chu Kỳ Đơn (Single-Cycle Processor)

CPU là một hệ thống logic số phức tạp cập nhật trạng thái sử dụng logic tổ hợp. Nó có hai loại phần tử (được thảo luận trong [phần tiếp theo](#sec-state-elements-vi)):

* [Phần tử trạng thái](#sec-state-elements-vi) chứa trạng thái, ví dụ: thanh ghi, bộ nhớ, v.v.
* Các Khối Logic Tổ hợp hoạt động trên các giá trị dữ liệu, ví dụ: [ALU](#sec-alu), [mux](#sec-mux), logic tổ hợp, v.v.

Mục tiêu của chúng ta cho chương này là thiết kế một **bộ xử lý chu kỳ đơn** **thực thi một lệnh mỗi chu kỳ**, bắt đầu từ việc fetch lệnh cho đến cập nhật PC cho lệnh tiếp theo.

* Trong mỗi chu kỳ clock, các đầu ra hiện tại của các phần tử trạng thái _điều khiển các đầu vào_ đến logic tổ hợp, có đầu ra ổn định tại các đầu vào đến các phần tử trạng thái trước cạnh lên clock tiếp theo.
* Sau đó, tại **cạnh lên clock**, tất cả các phần tử trạng thái được _cập nhật_ với các đầu ra logic tổ hợp, và việc thực thi chuyển sang chu kỳ clock tiếp theo.

### Bộ Xử Lý như một Máy Trạng Thái?

Về mặt lý thuyết, để thiết kế CPU của chúng ta, chúng ta có thể triển khai thứ gì đó như @fig-monolithic-datapath-vi. Chúng ta có thể tạo một máy trạng thái "hữu hạn" xem xét mỗi lệnh như một trạng thái riêng biệt, sau đó chỉ định logic tổ hợp để chuyển đổi sang mọi lệnh khả thi khác.

:::{figure} images/monolithic-datapath.png
:label: fig-monolithic-datapath-vi
:width: 50%
:alt: "Datapath CPU nguyên khối strawman được vẽ như một khối logic cồng kềnh kết nối với các phần tử trạng thái, minh họa tại sao sao chép phần cứng cho mỗi lệnh là không thực tế."

Cách tiếp cận strawman, cồng kềnh để triển khai datapath của chúng ta. Chúng ta thảo luận về các phần tử trạng thái được liệt kê trong hình ở [phần tiếp theo](#sec-state-elements-vi).
:::

:::{warning} Vấn đề: Một khối "nguyên khối" đơn lẻ sẽ cồng kềnh và không hiệu quả

Mặc dù về mặt lý thuyết chúng ta có thể xây dựng một "bong bóng" logic tổ hợp riêng biệt cho mỗi lệnh đơn và sử dụng các bộ ghép kênh để chọn giữa chúng, điều đó không thực tế vì nhiều lệnh chia sẻ cùng một đường dữ liệu.
:::

## 5 Bước Thực Thi Lệnh RISC-V

Thay vì cách tiếp cận FSM phức tạp ở trên, chúng ta sẽ chia quy trình thành **năm bước**, sau đó kết nối các bước để tạo ra toàn bộ mạch bộ xử lý. Với mỗi lệnh, chúng ta sẽ xác định liệu logic bổ sung có cần được kết hợp vào mỗi giai đoạn hay không.

Lợi ích của cách tiếp cận này là hai mặt:

1. Các bước nhỏ hơn dễ thiết kế hơn
1. Tính module có nghĩa là chúng ta có thể tối ưu một bước mà không cần chạm vào các bước khác.

(sec-five-steps-vi)=
:::{note} Năm bước thực thi lệnh RISC-V

1. **Fetch Lệnh (`IF`)**: Fetch lệnh từ [bộ nhớ](#sec-element-imem-vi) và tăng [bộ đếm chương trình](#sec-element-pc-vi).
1. **Decode Lệnh (`ID`)**: Xác định phép toán từ các bit của lệnh và đọc thanh ghi từ [register file](#sec-element-regfile-vi).
1. **Thực Thi (`EX`)**: Sử dụng [Đơn vị Số học Logic (ALU)](#sec-alu) để thực hiện phép toán.
1. **Truy Cập Bộ Nhớ (`MEM`)**: Load dữ liệu từ [bộ nhớ](#sec-element-dmem-vi), hoặc store dữ liệu[^mem-phase-vi] vào bộ nhớ.
1. **Ghi Lại (`WB`)**: Ghi lại[^wb-phase-vi] vào [register file](#sec-element-regfile-vi).
:::

Trong datapath chu kỳ đơn, tất cả các giai đoạn của một lệnh RV32I sẽ thực thi trong cùng một chu kỳ[^mem-phase-vi][^wb-phase-vi]: Fetch Lệnh (`IF`) bắt đầu ở cạnh lên đầu tiên của clock, và Ghi Lại (`WB`) hoàn thành[^wb-phase-vi] kết quả cuối cùng ở cạnh lên tiếp theo.

:::{warning} Không phải tất cả các bước đều cần thiết cho mọi lệnh!

Ví dụ, [các lệnh R-Type](#sec-datapath-r-type-vi) của chúng ta, không cần truy cập bộ nhớ, và [store](#sec-datapath-load-store) không cần ghi lại vào register file. Tuy nhiên, qua các định dạng lệnh RISC-V khác nhau, các hành động phần lớn giống nhau, bất kể lệnh cụ thể nào. Từ P&H 4.1:

> Sự đơn giản và tính quy luật của tập lệnh RISC-V đơn giản hóa việc triển khai bằng cách làm cho việc thực thi nhiều lớp lệnh tương tự nhau.

:::

[^mem-phase-vi]: Chính xác hơn là trong `MEM`, chúng ta thiết lập dữ liệu để store lại vào `DMEM` bằng cách đảm bảo rằng đầu vào đến `DMEM` ổn định trước cạnh lên tiếp theo của clock. Sau đó, ở cạnh lên, `MEM` store giá trị đúng vào bộ nhớ. Xem thêm: [DMEM](#sec-element-dmem-vi)

[^wb-phase-vi]: Đối với datapath chu kỳ đơn của chúng ta, chính xác hơn là trong `WB`, chúng ta thiết lập giá trị để ghi lại vào Register File bằng cách đảm bảo rằng đầu vào D của thanh ghi `rd` ổn định tại thời điểm setup, trước cạnh lên tiếp theo của clock. Sau đó, ở cạnh lên, thanh ghi `rd` nhận giá trị này, sau đó sau một độ trễ clk-to-q, có giá trị đúng ở đầu ra Q của nó. Xem thêm: [RegFile](#sec-element-regfile-vi)

Trong phần tiếp theo, chúng ta giới thiệu các phần tử chính của **datapath** RISC-V. Hiện tại, chúng ta chia sẻ @fig-five-step-single-cycle-datapath-vi và để bạn đoán ý nghĩa của mỗi khối phần cứng.

:::{figure} images/five-step-single-cycle-datapath.png
:label: fig-five-step-single-cycle-datapath-vi
:alt: "Tổng quan datapath chu kỳ đơn được gắn nhãn theo IF, ID, EX, MEM, và WB hiển thị các phần tử trạng thái, ALU và các đường mux qua một chu kỳ lệnh."

Năm bước của datapath chu kỳ đơn. Xem [phần này](#sec-state-elements-vi) để biết mô tả của mỗi khối phần cứng.
:::

Chúng ta nhấn mạnh lớp trừu tượng giữa datapath và control với @fig-five-step-single-cycle-control-vi bên dưới. Logic control chọn các đường datapath "cần thiết" dựa trên lệnh. Logic control chỉ định liệu dữ liệu có cần được ghi vào bộ nhớ hoặc thanh ghi, phép toán số học hoặc logic nào cần thực thi, các bộ chọn cho MUX, v.v.

:::{figure} images/five-step-single-cycle-control.png
:label: fig-five-step-single-cycle-control-vi
:alt: "Cùng datapath năm bước được chú thích với ảnh hưởng của control, làm nổi bật cách các tín hiệu control chọn các đường dữ liệu hoạt động cho mỗi lệnh."

Khi datapath tính toán các giá trị, logic control chọn các giá trị cần thiết để thực thi lệnh.
:::


Từ P&H Section 4.7:

:::{blockquote}
Các lệnh và dữ liệu thường di chuyển từ trái sang phải qua năm [bước] khi chúng hoàn thành việc thực thi.

Tuy nhiên, có hai ngoại lệ cho luồng từ trái sang phải này của các lệnh:

* Bước ghi lại, đặt kết quả trở lại vào register file ở giữa datapath
* Việc chọn giá trị tiếp theo của PC, chọn giữa PC tăng dần và địa chỉ branch...

Dữ liệu chảy từ phải sang trái không ảnh hưởng đến lệnh hiện tại; những chuyển động dữ liệu ngược này chỉ ảnh hưởng đến các lệnh sau trong pipeline.
:::

(sec-phases-stages-vi)=
:::{warning} 5 Giai đoạn (Phases)? 5 Tầng (Stages)?

Hiện tại với cách tiếp cận datapath chu kỳ đơn hiện tại, chúng ta sẽ sử dụng thuật ngữ **phase** (giai đoạn) để chỉ mỗi bước trong năm bước để thực thi một lệnh trong một chu kỳ clock đơn.

Bạn sẽ thấy tài liệu đề cập đến "pipeline RISC-V 5-stage." Trong lớp của chúng ta, chúng ta dành thuật ngữ **stage** (tầng) để chỉ một bước trong **kiến trúc pipeline**, nơi một lệnh mất đến năm chu kỳ clock để thực thi, một chu kỳ cho mỗi bước. Chúng ta sẽ thảo luận điều này sau.

:::

Cuối cùng, chúng ta lưu ý rằng cách tiếp cận chu kỳ đơn này không thực tế, vì chu kỳ clock phải kéo dài để phù hợp với lệnh chậm nhất mất tất cả năm bước. Sau khi thiết kế datapath chu kỳ đơn, chúng ta sẽ xem xét các triển khai nhanh hơn tận dụng **pipelining**.
