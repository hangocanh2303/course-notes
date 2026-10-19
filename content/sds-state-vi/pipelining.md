---
title: "Đường Ống Hóa Để Tăng Hiệu Năng"
subtitle: "Tác giả John Wawrzynek, chỉnh sửa bởi Lisa Yan"
---

:::{warning} Nội dung này nằm ngoài phạm vi của bài thi giữa kỳ.

Nội dung này được đề cập khi chúng ta thảo luận về đường ống trong [đường dữ liệu RISC-V năm giai đoạn](#sec-five-stage-pipeline).
:::

(sec-pipelining-circuits-vi)=
## Mục Tiêu Học Tập

* Giải thích cách thanh ghi có thể được sử dụng để cải thiện hiệu năng **thông lượng** của mạch.
* So sánh cách dữ liệu đầu vào được nạp qua mạch *có đường ống* và mạch *không có đường ống*.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/fxjbBygDzBo
:width: 100%
:title: "[CS61C FA20] Lecture 15.4 - State, State Machines: Pipelining for Performance"
:::

::::

Trong [phần trước](#sec-use-register-vi), chúng ta đã thấy một ví dụ về nơi thanh ghi cần thiết để đảm bảo hành vi mạch đúng đắn. Bây giờ, chúng ta sẽ thấy cách thanh ghi cũng có thể được sử dụng để tăng tần số xung nhịp có thể đạt được và, do đó, cải thiện hiệu năng của mạch thông qua đường ống hóa [^state-handout].

[^state-handout]: Những ghi chú này được chuyển thể từ ghi chú của Giáo sư John Wawrzynek: [Tài liệu State](../resources/state.pdf).

## Mạch Không Có Đường Ống

Giả sử chúng ta có nhu cầu nối tiếp hai mạch logic tổ hợp, một bộ cộng và một bộ dịch. Ý tưởng của mạch này là, khi các giá trị đầu vào đến, chúng được cộng với nhau và sau đó dịch đi một lượng nào đó. Chúng ta có thể tưởng tượng rằng kết hợp mạch này là một phần của bộ xử lý. @fig-non-pipelined-adder-vi minh họa mạch ví dụ này bên trái và sơ đồ thời gian bên phải.

:::{figure} images/non-pipelined-adder.png
:label: fig-non-pipelined-adder-vi
:width: 100%
:alt: "Bên trái, một bộ cộng không có đường ống, gồm một thanh ghi, logic cộng, bộ dịch, và một thanh ghi thứ hai. Bên phải, sơ đồ thời gian hiển thị xung nhịp chung và cách các giá trị đầu vào và đầu ra bộ dịch thay đổi để dẫn đến một lần cộng-rồi-dịch hoàn chỉnh mỗi chu kỳ xung nhịp."

Sơ đồ của mạch cộng/dịch *không có đường ống* và sơ đồ thời gian liên kết.
:::

Chúng ta giả sử rằng các giá trị đầu vào đến từ một thanh ghi (được hiển thị trong @fig-non-pipelined-adder-vi như một khối thanh ghi kết hợp) và đầu ra đi vào một thanh ghi. Trên mỗi chu kỳ xung nhịp, chúng ta đồng thời bắt một cặp giá trị đầu vào mới trong thanh ghi đầu vào và kết quả trước đó trong thanh ghi đầu ra. Các dạng sóng cho hoạt động của mạch này được hiển thị bên phải trong @fig-non-pipelined-adder-vi. Có một độ trễ một chu kỳ xung nhịp từ đầu vào (đầu ra của thanh ghi đầu vào) đến đầu ra (đầu ra của thanh ghi đầu ra $R_{i-1}$).

## Mạch Có Đường Ống

Tần số xung nhịp tối đa (chu kỳ xung nhịp tối thiểu) bị giới hạn bởi độ trễ lan truyền của phép cộng/dịch. Nếu chúng ta cố gắng làm cho chu kỳ xung nhịp quá ngắn, thì logic cộng/dịch sẽ không có đủ thời gian để tạo ra đầu ra của nó. Do đó, thanh ghi đầu ra sẽ bắt giá trị không chính xác.

Nếu chúng ta cảm thấy rằng chu kỳ xung nhịp cho mạch hoạt động đúng là quá dài, chúng ta có thể chọn chia phép cộng/dịch thành hai chu kỳ, thực hiện phép *cộng* trên chu kỳ đầu tiên và phép *dịch* trên chu kỳ thứ hai. Chúng ta có thể hiện thực ý tưởng này bằng cách đưa vào một thanh ghi mới giữa hai khối, như được hiển thị bên trái trong @fig-pipelined-adder-vi.

:::{figure} images/pipelined-adder.png
:label: fig-pipelined-adder-vi
:width: 100%
:alt: "Bên trái, một bộ cộng có đường ống, gồm một thanh ghi, logic cộng, thanh ghi đường ống bổ sung, bộ dịch, và thanh ghi thứ ba. Bên phải, sơ đồ thời gian hiển thị xung nhịp chung và cách các đầu vào và tất cả các giá trị đầu ra trung gian thay đổi để dẫn đến sự chồng chéo hai giai đoạn nên mỗi giai đoạn hoạt động mỗi chu kỳ với chu kỳ ngắn hơn."

Sơ đồ của mạch cộng/dịch *có đường ống* và sơ đồ thời gian liên kết.
:::

Trên mỗi chu kỳ xung nhịp, dữ liệu di chuyển từ đầu ra của `reg1`, qua bộ cộng, đến đầu vào của `reg2`, hoặc di chuyển từ đầu ra của `reg2`, qua bộ dịch, đến đầu vào của `reg3`. Vì dữ liệu không còn cần di chuyển qua cả hai khối trong một chu kỳ xung nhịp duy nhất, tần số xung nhịp có thể được tăng lên. Các dạng sóng cho phép cộng/dịch có đường ống này được hiển thị bên phải trong @fig-pipelined-adder-vi.

:::{note} Mạch này có thể nhận giá trị đầu vào mới trên mỗi chu kỳ xung nhịp không?

**Có!** Một quan sát thú vị về mạch có đường ống mới này là, sau khi dữ liệu di chuyển qua bộ cộng và được bắt trong `reg2`, trên chu kỳ xung nhịp tiếp theo, khi dữ liệu di chuyển vào bộ dịch, dữ liệu mới có thể đồng thời di chuyển vào bộ cộng vì bộ cộng bây giờ rảnh. Do đó, các giá trị dữ liệu đầu vào mới có thể được nạp vào mạch trên mỗi chu kỳ xung nhịp.
:::

## Thêm Thanh Ghi Để Cải Thiện Hiệu Năng

Như được minh họa bởi các phương trình bên dưới, bây giờ sẽ có độ trễ **hai** chu kỳ xung nhịp từ việc chèn một tập dữ liệu vào mạch cho đến khi nó xuất hiện tại đầu ra. Tuy nhiên, chu kỳ xung nhịp mới ngắn hơn, vì vậy về thời gian tuyệt đối, độ trễ từ chèn dữ liệu cho đến đầu ra (tức là **độ trễ** - latency) không tệ hơn đáng kể. Quan trọng hơn, vì sự biến đổi và tốc độ xung nhịp mới cao hơn, kết quả sẽ được tạo ra với tốc độ cao hơn (nhiều đầu ra hơn mỗi giây, tức là **thông lượng** - throughput).

::::{card}
:header: Mạch không có đường ống (@fig-non-pipelined-adder-vi)

```{math}
\begin{aligned}
\text{Độ trễ đường găng} &= t_{\texttt{clk-to-q}} + t_{\texttt{cộng}} + t_{\texttt{dịch}} + t_{\texttt{thiết lập}} \\
\text{độ trễ} &= t_{\texttt{clk-to-q}} + t_{\texttt{cộng}} + t_{\texttt{dịch}} + t_{\texttt{thiết lập}} \\
\end{aligned}
```
::::


::::{card}
:header: Mạch có đường ống (@fig-pipelined-adder-vi)

```{math}
\begin{aligned}
\text{Độ trễ đường găng} &= \max \{ t_{\texttt{clk-to-q}} + t_{\texttt{cộng}} + t_{\texttt{thiết lập}}, t_{\texttt{clk-to-q}} + t_{\texttt{dịch}} + t_{\texttt{thiết lập}} \} \\
\\
\text{Độ trễ} &= t_{\texttt{clk-to-q}} + t_{\texttt{cộng}} + t_{\texttt{thiết lập}} + t_{\texttt{clk-to-q}} + t_{\texttt{dịch}} + t_{\texttt{thiết lập}}
\end{aligned}
```
::::

Đường ống hóa là một biến đổi tốt nếu bạn đánh giá dựa trên kết quả/thời gian, hay **thông lượng**. Nếu bạn quan tâm nhiều hơn đến **độ trễ** cho bất kỳ tập giá trị đầu vào nào, thì biến đổi này ít lý tưởng hơn.
