---
title: "Thời gian Thực thi Lệnh"
---

(sec-instruction-timing-vi)=
## Mục tiêu Học tập

* Thực hành diễn giải dạng sóng trong biểu đồ thời gian.
* Cho một lệnh, xác định đường tới hạn qua datapath single-cycle.
* Ước tính thời gian lệnh dựa trên năm giai đoạn thực thi lệnh.

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/eLT0FOLdtn8
:width: 100%
:title: "[CS61C FA20] Lecture 20.3 - Single-Cycle CPU Control: Instruction Timing"
:::

::::

Chúng ta nên định thời datapath single-cycle như thế nào? Chúng ta nên thiết lập tần số xung clock như thế nào? Trong phần này, chúng ta phát triển một xấp xỉ về thời gian lệnh sử dụng [năm bước thực thi lệnh RISC-V](#sec-five-steps-vi).

```{embed} #sec-five-steps-vi
```

## Biểu đồ Thời gian cho `add`

Đầu tiên, hãy xem xét các độ trễ trong lệnh `add` thân thuộc của chúng ta. Xem lại datapath `add` trong @anim-datapath-add-full.

<!-- ::::{figure}
:label: anim-datapath-add-full
:::{iframe} https://view.officeapps.live.com/op/embed.aspx?src=https://github.com/61c-teach/course-notes/raw/refs/heads/main/content/datapath/pptx/datapath-add-full.pptx
:width: 100%
:title: "Slides tracing through the `add` Datapath (Full)."
:::
The `add` datapath, updated from an [earlier section](#sec-datapath-r-type)'s simple `add`-only datapath. Use the menu bar to trace through the animation or download a copy of the PDF/PPTX file.
:::: -->

::::{figure}
:label: anim-datapath-add-full
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vTNTZZVVOA6hREQLorL6y7AEwzfidYgLpsBBc9YEAda0Dxih_yFBL3ykPOPSysa7w/pubembed?start=false&loop=false
:width: 100%
:alt: "Truy vết datapath add đầy đủ hiển thị tất cả các bus chính và đường điều khiển hoạt động cho lệnh add qua một chu kỳ."
:title: "Slides theo dõi Datapath `add` (Đầy đủ)."
:::
Datapath `add`, được cập nhật từ datapath `add` đơn giản của [phần trước](#sec-datapath-r-type-vi). Sử dụng thanh menu để theo dõi animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/10ORU8oWu4rSY7J6uSWl_6OxrJLVLt9EQ/edit?usp=sharing).
::::

@fig-timing-add hiển thị các dạng sóng cho việc thực thi lệnh `add x1 x2 x3` tại địa chỉ `0x100`, tiếp theo là `add x6 x7 x9` tại địa chỉ `0x104`.

:::{figure} images/timing-add.png
:label: fig-timing-add
:alt: "Dạng sóng thời gian cho hai lệnh add liên tiếp, hiển thị PC, lệnh, đọc thanh ghi, ALU, và writeback ổn định trong một chu kỳ xung clock."

Biểu đồ thời gian cho `add`. Chỉ hiển thị các dạng sóng tín hiệu liên quan.

:::

:::{note} Giải thích @fig-timing-add

1. **Instruction Fetch** (`IF`).

    * Tại cạnh lên của xung clock, cập nhật thanh ghi program counter với tín hiệu đầu vào của nó. Sau một độ trễ $t_{clk-to-q}$, giá trị của program counter `0x100` xuất hiện tại tín hiệu đầu ra `pc`.

    * Đồng thời thực hiện các công việc sau:[^concurrent-vi]
    
        * Tăng PC đến lệnh tiếp theo với bộ cộng đơn giản. Sau một độ trễ, tín hiệu `pc + 4` sẵn sàng với `0x104` tại đầu vào mux PCSel.[^bundle-delay-vi]

        * Fetch một lệnh từ `IMEM`. Sau một độ trễ, `inst[31:0]` được cập nhật với mã máy cho `add x1 x2 x3`.

1. **Instruction Decode** (`ID`). Đồng thời thực hiện các công việc sau:[^concurrent-2-vi]
    * Lấy giá trị của các thanh ghi nguồn `rs1` và `rs2` từ RegFile. Sau một độ trễ, các tín hiệu đầu ra `R[rs1]` và `R[rs2]` sẵn sàng với giá trị của các thanh ghi `x2` và `x3`.
    * Giải mã lệnh để xác định các [tín hiệu logic điều khiển](#tab-control-truth-table-vi).
1. **Execute** (`EX`).
    * Sử dụng hai mux để chọn các tín hiệu đầu vào thích hợp cho ALU. Sau một độ trễ, các tín hiệu đầu vào này mang hai giá trị thanh ghi nguồn.[^mux-delay-vi]
    * Sau thêm một độ trễ, tín hiệu đầu ra `alu` của ALU được thiết lập thành tổng của hai thanh ghi nguồn `x2` và `x3`.
1. **Memory** (`MEM`). (Chúng ta không truy cập bộ nhớ, vì vậy không tính độ trễ bộ nhớ trong phân tích.)
1. **Write Back** (`WB`). Sử dụng mux `WBSel` để chọn tín hiệu đầu ra `alu` làm tín hiệu `wb` đến đầu vào `wdata` của RegFile. Sau một độ trễ, tín hiệu được thiết lập thành tổng của hai thanh ghi nguồn.

    Ngoài ra, tính thời gian setup cần thiết để giữ tín hiệu `wb` ổn định trước cạnh lên xung clock.

[^concurrent-vi]: Các quá trình này mất thời gian tương đương, mặc dù cái nào dài hơn phụ thuộc vào công nghệ cụ thể. Trong @fig-timing-add, bộ cộng tình cờ hoàn thành nhanh hơn thao tác fetch bộ nhớ `IMEM`.

[^concurrent-2-vi]: Trong @fig-timing-add, việc giải mã logic điều khiển của lệnh tình cờ hoàn thành nhanh hơn thao tác đọc thanh ghi `RegFile`. Chúng ta sẽ giả định thứ tự ưu tiên này trong các phân tích sau.

[^bundle-delay-vi]: Lưu ý rằng dạng sóng đại diện cho các bó dây với giá trị thập lục phân (đối lập với tín hiệu cao-thấp nhị phân của xung clock). Bó dây đầu ra PC `pc` cập nhật cùng lúc, vì các [flip-flop](#sec-flip-flops) được nối song song. Ngược lại, đầu ra `pc+4` không ổn định đồng thời. Vì [bộ cộng](#sec-adder-subtractor) xếp tầng các bộ cộng một-bit theo chuỗi, các bit ít quan trọng nhất ổn định sớm hơn các bit quan trọng hơn. Trong biểu đồ thời gian, chúng ta luôn hiển thị sự chuyển đổi đến giá trị đúng. Đối với `pc+4`, điều này xảy ra sau độ trễ lan truyền của bit quan trọng nhất.

[^mux-delay-vi]: Có hai bộ ghép kênh được điều khiển bởi ASel và BSel tương ứng. Cả hai độ trễ lan truyền xảy ra đồng thời, vì vậy chúng ta chỉ tính độ trễ lan truyền của một mux.

:::

## Độ trễ đường tới hạn theo lệnh

Các lệnh khác nhau sử dụng các thành phần khác nhau của datapath. Chúng ta bây giờ cập nhật định nghĩa của [đường tới hạn](#sec-critical-path) để xem xét đường giữa các đầu vào và đầu ra của phần tử xung clock _quan trọng_ cho lệnh đã cho. Ví dụ, truy cập DMEM không quan trọng cho `add`, trong khi thiết lập dữ liệu RegFile để ghi lại không quan trọng cho `sw`.

:::{table} Mô tả thời gian của các thành phần.
:label: tab-timing-vi

| Độ trễ | Mô tả |
| :-- | :-- |
| $t_{\texttt{clk-to-q}}$ | Độ trễ clk-to-q để chuyển giá trị đầu vào thanh ghi sang đầu ra. |
| $t_{\texttt{setup}}$ | Thời gian setup để giữ đầu vào thanh ghi ổn định trước cạnh lên xung clock. |
| $t_{\texttt{mux}}$ | Độ trễ lan truyền qua một mux; giả sử cùng độ trễ cho tất cả các mux. |
| $t_{\texttt{add}}$ | Độ trễ lan truyền qua bộ cộng đơn giản tăng PC đến lệnh tiếp theo. |
| $t_{\texttt{RegFile}}$ | Độ trễ để đọc giá trị thanh ghi từ RegFile. |
| $t_{\texttt{IMEM}}$ | Độ trễ để đọc lệnh từ IMEM. |
| $t_{\texttt{DMEM}}$ | Độ trễ để đọc một từ từ DMEM. |
| $t_{\texttt{ALU}}$ | Độ trễ lan truyền qua ALU. |
| $t_{\texttt{Imm}}$ | Độ trễ lan truyền qua immediate generator. |
| $t_{\texttt{BrComp}}$ | Độ trễ lan truyền qua branch comparator. |
:::

::::{tip} Tính độ trễ dọc theo đường tới hạn cho mỗi lệnh.

1. `add`
1. `lw`
1. `beq`

Các lựa chọn:
* **A.** $t_{\texttt{clk-to-q}} + t_{\texttt{Add}} + t_{\texttt{IMEM}} + t_{\texttt{RegFile}} + t_{\texttt{BrComp}} + t_{\texttt{ALU}} + t_{\texttt{DMEM}} + t_{\texttt{mux}} + t_{\texttt{Setup}}$
* **B.** $t_{\texttt{clk-to-q}} + t_{\texttt{IMEM}} + t_{\texttt{RegFile}} + 2 \cdot t_{\texttt{mux}} + t_{\texttt{ALU}} + t_{\texttt{Setup}}$
* **C.** $t_{\texttt{clk-to-q}} + t_{\texttt{IMEM}} + \max\{t_{\texttt{RegFile}}, t_{\texttt{Imm}}\} + t_{\texttt{ALU}} + 2 \cdot t_{\texttt{mux}} + t_{\texttt{DMEM}} + t_{\texttt{Setup}}$
* **D.** $t_{\texttt{clk-to-q}} + t_{\texttt{IMEM}} + \max\{t_{\texttt{RegFile}}, t_{\texttt{Imm}}\} + t_{\texttt{ALU}} + 3 \cdot t_{\texttt{mux}} + t_{\texttt{Setup}}$
* **E.** Cái khác

::::

::::{note} Hiển thị Đáp án cho `add`
:class: dropdown

**B**. Có hai "vòng" mà chúng ta xem xét:[^concurrent-2-vi]

* Vòng cập nhật PC, đo từ đầu ra PC đến đầu vào PC: $t_{\texttt{clk-to-q}} + t_{\texttt{Add}} + t_{\texttt{mux}} + t_{\texttt{setup}}$
* Vòng qua ALU, đo từ đầu ra PC đến đầu vào RegFile: $t_{\texttt{clk-to-q}} + t_{\texttt{IMEM}} + t_{\texttt{Reg}} + t_{\texttt{mux}} + t_{\texttt{ALU}} + t_{\texttt{mux}} + t_{\texttt{setup}}$

```{math}
\begin{aligned}
\text{Độ trễ đường tới hạn}
  =& t_{\texttt{clk-to-q}} \\
    & + \max \{ t_{\texttt{Add}} + t_{\texttt{mux}}, t_{\texttt{IMEM}} + t_{\texttt{Reg}} + t_{\texttt{mux}} + t_{\texttt{ALU}} + t_{\texttt{mux}} \} \\
    & + t_{\texttt{setup}} \\
  =& t_{\texttt{clk-to-q}} + t_{\texttt{IMEM}} + t_{\texttt{Reg}} + t_{\texttt{mux}} + t_{\texttt{ALU}} + t_{\texttt{mux}} + t_{\texttt{setup}} \\
  =&t_{\texttt{clk-to-q}} + t_{\texttt{IMEM}} + t_{\texttt{RegFile}} + 2 \cdot t_{\texttt{mux}} + t_{\texttt{ALU}} + t_{\texttt{Setup}}
\end{aligned}
```

Đường tới hạn sử dụng vòng dài hơn qua ALU.
::::

<!-- ::::{figure}
:label: anim-datapath-beq-full
:::{iframe} https://view.officeapps.live.com/op/embed.aspx?src=https://github.com/61c-teach/course-notes/raw/refs/heads/main/content/datapath/pptx/datapath-beq-full.pptx
:width: 100%
:title: "Slides tracing through the `beq` Datapath (Full)."
:::
The `beq` datapath, updated from an [earlier section](#sec-datapath-b-type)'s simpler datapath. Use the menu bar to trace through the animation or download a copy of the PDF/PPTX file.
:::: -->

::::{figure}
:label: anim-datapath-beq-full
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vRFDNojlD-1rvzMR9dQTJ8JZ2CllIZwaS6yYCdE1CumfoKjRXqeOekErcrTywda-w/pubembed?start=false&loop=false
:width: 100%
:alt: "Truy vết datapath beq đầy đủ với branch comparator, tính địa chỉ đích PC, và các đường chọn PC phụ thuộc điều khiển."
:title: "Slides theo dõi Datapath `beq` (Đầy đủ)."
:::
Datapath `beq`, được cập nhật từ datapath đơn giản hơn của [phần trước](#sec-datapath-b-type-vi). Sử dụng thanh menu để theo dõi animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/1iI-seGm2A7lSpHoGKM81b67utfyINT1H/edit?usp=sharing).
::::

:::{note} Hiển thị Đáp án cho `beq`
:class: dropdown

**E**. Cái khác.

Chúng tôi để việc suy diễn này cho bạn. Lưu ý bạn có thể cần tạo các độ trễ giữ chỗ mới cho logic điều khiển...!
:::

<!-- ::::{figure}
:label: anim-datapath-lw-full
:::{iframe} https://view.officeapps.live.com/op/embed.aspx?src=https://github.com/61c-teach/course-notes/raw/refs/heads/main/content/datapath/pptx/datapath-lw-full.pptx
:width: 100%
:alt: "Full lw datapath trace slide including IMEM fetch, ALU address generation, DMEM read, and writeback mux path to RegFile."
:title: "Slides tracing through the `lw` Datapath (Full)."
:::
The `lw` datapath, updated from an [earlier section](#sec-datapath-load-store)'s simpler datapath. Use the menu bar to trace through the animation or download a copy of the PDF/PPTX file.
:::: -->

::::{figure}
:label: anim-datapath-lw-full
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vQqpuC67HJ6YsCXhnQ5YA1zqtUPBy-KRSnwjJzHgYdry8wwPkA6fm5pvToseVpxhQ/pubembed?start=false&loop=false
:width: 100%
:alt: "Phác thảo thời gian dựa trên giai đoạn gắn nhãn các khoảng IF, ID, EX, MEM, và WB được sử dụng để ước tính độ trễ lệnh single-cycle."
:title: "Slides theo dõi Datapath `lw` (Đầy đủ)."
:::
Datapath `lw`, được cập nhật từ datapath đơn giản hơn của [phần trước](#sec-datapath-load-store-vi). Sử dụng thanh menu để theo dõi animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/1QV6naVOhLr3PVoKszzY-ipUCigknXs3D/edit?usp=sharing).
::::

::::{note} Hiển thị Đáp án cho `lw`
:class: dropdown

**C**. Load sử dụng phần cứng trong cả năm giai đoạn của datapath. Chúng ta vẫn xem xét hai "vòng" qua datapath[^concurrent-2-vi]:

* Vòng cập nhật PC, vẫn đo từ đầu ra PC đến đầu vào PC.
* Vòng dài hơn nhiều, đo từ đầu ra PC qua ALU _và_ DMEM, đến đầu vào RegFile. Chúng ta bây giờ xem xét thêm phần cứng cho load:
  * **Instruction Decode**: Khối immediate generation thiết lập `imm` đồng thời với RegFile lấy giá trị thanh ghi nguồn `R[rs1]`. Chúng ta ký hiệu độ trễ này là lớn hơn trong hai, $\max\{t_{\texttt{RegFile}}, t_{\texttt{Imm}}\}$.
  * **Execute**: Đầu ra ALU tính địa chỉ bộ nhớ, vì vậy chúng ta phát sinh $t_{\texttt{ALU}}$.
  * **Memory**: Thao tác đọc DMEM bây giờ quan trọng, vì vậy chúng ta phát sinh thời gian đọc DMEM, $t_{\texttt{DMEM}}$.

```{math}
\begin{aligned}
\text{Độ trễ đường tới hạn}
  =& t_{\texttt{clk-to-q}} \\
   & + \max \{ t_{\texttt{add}} + t_{\texttt{mux}}, \\
   & t_{\texttt{IMEM}} + t_{\texttt{Imm}} + t_{\texttt{mux}} + t_{\texttt{ALU}} + t_{\texttt{DMEM}} + t_{\texttt{mux}}, \\
   & t_{\texttt{IMEM}} + t_{\texttt{RegFile}} + t_{\texttt{mux}} + t_{\texttt{ALU}} + t_{\texttt{DMEM}} + t_{\texttt{mux}} \} \\
   & + t_{\texttt{setup}} \\
  =& t_{\texttt{clk-to-q}} + t_{\texttt{IMEM}} + \max\{t_{\texttt{RegFile}}, t_{\texttt{Imm}}\} + t_{\texttt{ALU}} + 2 \cdot t_{\texttt{mux}} + t_{\texttt{DMEM}} + t_{\texttt{setup}}
\end{aligned}
```
::::



:::{warning} Load sử dụng cả [năm bước](#sec-five-steps-vi)!

Trong khi hầu hết các lệnh sử dụng ba hoặc bốn trong năm bước, load là lệnh duy nhất phải vừa đọc từ DMEM (Memory) _và_ Write Back đến RegFile.
:::

## Xung clock datapath single-cycle chậm

Để xác định **tần số xung clock** cho datapath single-cycle, chúng ta tính độ trễ đường tới hạn của mỗi lệnh, sau đó đặt chu kỳ xung clock là độ trễ **trường hợp xấu nhất** phát sinh qua tất cả các lệnh.

Để đưa một số con số vào phân tích trước đó, chúng ta sẽ _đơn giản hóa_ các ước tính thời gian với @tab-timing-steps-vi, giả định rằng thời gian của mỗi [năm bước thực thi lệnh RISC-V](#sec-five-steps-vi) được chi phối bởi các đơn vị phần cứng chức năng chính.

:::{table} Giả sử mỗi trong [năm bước](#sec-five-steps-vi) được chi phối bởi một đơn vị phần cứng chính. Các bộ ghép kênh, đơn vị điều khiển, truy cập PC, immediate generation, và dự đoán branch phát sinh độ trễ tối thiểu.
:label: tab-timing-steps-vi

| Bước | Thời gian hoạt động | Đơn vị phần cứng chính |
| :--- | :--- | :--- |
| Instruction Fetch (`IF`) | 200 ps | Đọc một từ lệnh từ IMEM. |
| Instruction Decode (`ID`) | 100 ps | Đọc giá trị thanh ghi từ RegFile. |
| Execute (`EX`) | 200 ps | Thực hiện các phép toán số học/logic trong ALU. |
| Memory Access (`MEM`) | 200 ps | Đọc hoặc ghi dữ liệu từ DMEM. |
| Write Back (`WB`) | 100 ps | Ghi lại vào RegFile. Đối với single-cycle, chúng ta giả định đây là độ trễ của mux WBSel và thời gian setup. |

:::

Chúng ta sau đó có thể tạo ra biểu đồ thời gian đơn giản hóa trong @fig-timing-phases cho một lệnh sử dụng tất cả các giai đoạn—như lệnh `lw` trước đó.
Chúng ta cũng có thể xây dựng @tab-timing-instructions-vi, hiển thị thời gian cần thiết cho các định dạng lệnh khác nhau.

:::{figure} images/timing-phases.png
:label: fig-timing-phases
:alt: "Biểu đồ thời gian dựa trên giai đoạn gắn nhãn các khoảng IF, ID, EX, MEM, và WB được sử dụng để ước tính độ trễ lệnh single-cycle."

Biểu đồ thời gian gần đúng cho [năm bước thực thi lệnh RISC-V](#sec-five-steps-vi) trong datapath single-cycle.

:::

:::{table} (P&H Hình 4.28). Tổng thời gian cho mỗi lệnh được tính từ thời gian đơn giản hóa cho mỗi giai đoạn.
:label: tab-timing-instructions-vi

| Lệnh | IF (200ps) | ID (100ps) | EX (200ps) | MEM (200ps) | WB (100ps) | Tổng |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| `add` | X | X | X | | X | 600ps |
| `beq` | X | X | X | | | 500ps |
| `jal` | X | X | X | | X | 600ps |
| `lw` | X | X | X | X | X | 800ps |
| `sw` | X | X | X | X | | 700ps |
:::

Trong khi @tab-timing-instructions-vi ở trên hiển thị thời gian ngắn nhất để hoàn thành mỗi lệnh, chúng ta lưu ý rằng datapath single-cycle, như tất cả các hệ thống số đồng bộ, chia sẻ một xung clock duy nhất.

:::{warning} Datapath single-cycle được định thời theo lệnh chậm nhất

Mặc dù một số lệnh có thể nhanh đến 500 ps, thiết kế datapath single-cycle của chúng ta yêu cầu thời gian cho mỗi lệnh là **800 ps**, vì chúng ta phải đặt chu kỳ xung clock để cho phép lệnh chậm nhất: **load**.

Trong datapath single-cycle của chúng ta, điều này có nghĩa là **tần số xung clock tối đa 1.25 GHz**—mặc dù mỗi giai đoạn riêng lẻ mất nhiều nhất 200 ps.
:::

Chúng ta cũng lưu ý rằng đường tới hạn của mỗi lệnh thường liên quan đến việc truy cập các đơn vị phần cứng chính theo _chuỗi_. Nói cách khác, trong hầu hết mỗi chu kỳ xung clock, phần lớn phần cứng của chúng ta nhàn rỗi và không tính toán dữ liệu bổ sung!

Chúng ta giải quyết các vấn đề hiệu suất này và nhiều hơn nữa trong thiết kế datapath _pipeline_ tiếp theo. Đón chờ nhé!
