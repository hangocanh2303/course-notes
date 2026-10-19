---
title: "Thanh Ghi"
subtitle: "Chuyển thể từ John Wawrzynek"
---

(sec-registers-vi)=
## Mục Tiêu Học Tập

* Nhận biết các thuộc tính của mạch nhớ.
* Mô tả hành vi của flip-flop loại d kích hoạt cạnh dương.
* Xác định thời gian thiết lập, thời gian giữ, và độ trễ clk-to-q trong sơ đồ thời gian FF.
* Giải thích tầm quan trọng của thời gian thiết lập và thời gian giữ để lấy mẫu đầu vào.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/OPj9eUw5O_M
:width: 100%
:title: "[CS61C FA20] Lecture 15.2 - State, State Machines: Register Details Flip-flops"
:::

::::

Như chúng ta đã thấy trong [phần trước](#sec-cl-practice-vi), mạch logic tổ hợp cung cấp tất cả các chức năng cần thiết để thực thi lệnh, bao gồm tất cả các phép toán số học và logic, tăng giá trị bộ đếm chương trình, xác định khi nào cần rẽ nhánh, tính toán các giá trị PC mục tiêu mới, v.v., nhưng chúng không có bộ nhớ về các đầu vào hoặc đầu ra trước đó. **Mạch nhớ** (memory circuit)[^sds-handout] là một ví dụ về loại mạch khác được sử dụng để tạo bộ xử lý có khả năng ghi nhớ đầu vào của nó. Mạch nhớ này được sử dụng để hiện thực thanh ghi mục đích chung, như được hiển thị trong @fig-reg-circuit-vi.

[^sds-handout]: Những ghi chú này được chuyển thể từ ghi chú của Giáo sư John Wawrzynek: [Tài liệu SDS](../resources/sds.pdf).

:::{figure} images/register-circuit.png
:label: fig-reg-circuit-vi
:width: 70%
:alt: "Ký hiệu thanh ghi với đầu vào dữ liệu n-bit D, đầu ra n-bit được lưu trữ Q, và điều khiển LOAD xác định khi nào giá trị mới được bắt."

Sơ đồ của thanh ghi được hiện thực thông qua mạch nhớ.
:::

## Thanh Ghi Tổng Quát: Một Mạch Nhớ

Dưới sự điều khiển của một đầu vào đặc biệt, gọi là **LOAD**, thanh ghi bắt giá trị của tín hiệu đầu vào và giữ nó vô thời hạn, hoặc cho đến khi giá trị khác được nạp vào. Giá trị được lưu trữ bởi thanh ghi xuất hiện tại đầu ra của thanh ghi (một thời gian rất ngắn sau khi nó được nạp). Nếu bộ xử lý nạp một giá trị vào thanh ghi tại một thời điểm, nó có thể quay lại sau và đọc chính xác giá trị đó. Ngay cả khi các giá trị khác nhau xuất hiện tại đầu vào của thanh ghi trong khoảng thời gian trung gian, giá trị đầu ra của thanh ghi sẽ không thay đổi trừ khi được hướng dẫn bằng cách phát tín hiệu với đầu vào **LOAD**. Vì vậy, miễn là bộ xử lý có quyền kiểm soát việc phát tín hiệu của đầu vào **LOAD**, nó có toàn quyền kiểm soát những gì được lưu và khi nào.

Thường thì tín hiệu xung nhịp hệ thống **CLK** (xem [Xung Nhịp](#sec-clock-vi)), được sử dụng làm tín hiệu LOAD trên thanh ghi, như được hiển thị trong @fig-clk-reg-vi. Cấu hình này có nghĩa là, với thanh ghi này, một giá trị mới được nạp vào mỗi chu kỳ xung nhịp. Đây là một cấu hình có giá trị mà chúng ta sẽ khám phá trong [phần sau](#sec-use-register-vi).

:::{figure} images/clocked-register.png
:label: fig-clk-reg-vi
:width: 70%
:alt: "Thanh ghi được đồng bộ, việc bắt giá trị được xác định bởi CLK thay vì đường LOAD rõ ràng, cập nhật đầu vào n-bit được lưu trữ thành đầu ra n-bit trên mỗi cạnh xung nhịp hoạt động."

Sơ đồ của thanh ghi được đồng bộ được hiện thực thông qua mạch nhớ.
:::


(sec-flip-flops-vi)=
## Bên Trong Thanh Ghi Là Gì? Flip-Flop

Như được hiển thị trong @fig-reg-comps-vi, một thanh ghi rộng `n` bit không gì khác ngoài `n` phiên bản của một mạch đơn giản hơn[^state-handout].

[^state-handout]: Những ghi chú này được chuyển thể từ ghi chú của Giáo sư John Wawrzynek: [Tài liệu State](../resources/state.pdf).

:::{figure} images/register-components.png
:label: fig-reg-comps-vi
:width: 100%
:alt: "Thanh ghi n-bit được vẽ như n flip-flop D song song chia sẻ xung nhịp chung và tạo ra đầu ra Q n-bit được gộp."

Sơ đồ của thanh ghi được xây dựng từ `n` phiên bản của các mạch đơn giản hơn gọi là **flip-flop**.
:::

Chúng ta gọi những mạch đơn giản này là **flip-flop** (FF), là các thanh ghi rộng 1 bit được đặt tên từ thực tế là khi hoạt động, nó lật (flip) và bật (flop) giữa việc giữ `0` hoặc `1`. Tín hiệu `CLK` (hoặc `LOAD`) được gửi đến đầu vào `CLK` (hoặc `LOAD`) của tất cả `n` FF. Mỗi FF chịu trách nhiệm lưu trữ một bit của dữ liệu `n` bit được lưu trữ bởi thanh ghi. Theo quy ước thanh ghi, đầu vào được ghi nhãn `D` và đầu ra được ghi nhãn `Q`. Trong trường hợp bit đơn, đầu vào được ghi nhãn `d` và đầu ra được ghi nhãn `q`.

### Flip-Flop Loại D Kích Hoạt Cạnh

Loại FF phổ biến nhất được gọi là **flip-flop loại d kích hoạt cạnh** (edge-triggered d-type flip flop). Bên trong, mỗi FF bao gồm khoảng 10 transistor. Hoạt động của FF được minh họa như một sơ đồ dạng sóng trong @fig-ff-waveform-vi. Hình cho thấy hoạt động của flip-flop loại d kích hoạt cạnh **dương**, một trong hai loại flip-flop loại d kích hoạt cạnh. Loại còn lại là flip-flop loại d kích hoạt cạnh _âm_ — nhưng trong CS 61C, chúng ta sẽ tập trung vào flip-flop cạnh dương.

:::{figure} images/ff-waveform.png
:label: fig-ff-waveform-vi
:width: 100%
:alt: "Dạng sóng xung nhịp, dữ liệu d, và đầu ra q cho flip-flop D, hiển thị đầu ra q được cập nhật để khớp với đầu vào d chỉ sau cạnh lên của xung nhịp."

Sơ đồ dạng sóng của hoạt động của FF loại d kích hoạt cạnh.
:::

### Flip-Flop Loại D Kích Hoạt Cạnh Dương

Với FF loại d kích hoạt cạnh dương, trên mỗi cạnh dương (lên) của xung nhịp, đầu vào `d` được lấy mẫu và chuyển đến đầu ra `q`. Vào tất cả các thời điểm khác, đầu vào `d` bị bỏ qua.

:::{tip} Làm thế nào để xác minh hành vi của FF loại d kích hoạt cạnh dương?

Chúng tôi để việc xác minh quá trình này như một bài tập cho bạn! Bắt đầu bằng cách nhìn vào các dạng sóng của FF loại d kích hoạt cạnh dương. Bạn sẽ thấy rằng thời điểm duy nhất đầu ra `q` thay đổi là ngay sau cạnh lên của xung nhịp. Đầu vào `d` có thể lên xuống nhiều lần trong một chu kỳ xung nhịp duy nhất, nhưng chỉ giá trị của nó ngay tại cạnh lên của xung nhịp là quan trọng. Đây là khi đầu vào được lấy mẫu.

Bạn cũng có thể nhận thấy những thời điểm khi đầu ra FF không thay đổi để phản hồi cạnh lên của xung nhịp. Điều này chỉ xảy ra khi đầu vào `d` và đầu ra `q` đã có cùng giá trị!
:::

::::{note} Nhấp để xem một sơ đồ dạng sóng mô phỏng ví dụ!
:class: dropdown

:::{figure} images/waveform-diagram.png
:label: fig-waveform-ff-vi
:width: 100%
:alt: "Sơ đồ dạng sóng ví dụ được tạo thông qua công cụ số mô tả các đường ray thời gian cho clk, d, và q hiển thị lấy mẫu kích hoạt cạnh nhất quán với dấu vết Verilog hoặc mô phỏng."

Sơ đồ dạng sóng mô phỏng cho FF loại d dương.
:::
::::

### Sơ Đồ Thời Gian FF

Một sơ đồ thời gian FF chi tiết cho FF loại d kích hoạt cạnh dương được hiển thị trong @fig-ff-timing-vi. Giống như mạch logic tổ hợp, FF không thể thay đổi đầu ra của chúng ngay lập tức. Ngoài ra, cần thời gian để chuyển đầu vào bên trong. Do đó, đầu vào `d` phải ổn định trong một khoảng thời gian ngắn trước cạnh lên của xung nhịp (gọi là **thời gian thiết lập** - setup time) và giữ ổn định trong một khoảng thời gian ngắn sau cạnh (gọi là **thời gian giữ** - hold time). Cùng nhau, thời gian thiết lập và thời gian giữ tạo ra một cửa sổ thời gian khi đầu vào `d` không thể thay đổi. Nếu nó thay đổi trong cửa sổ này, FF sẽ không bắt đầu vào mới một cách đáng tin cậy. Lưu ý rằng, một khi FF bắt đầu vào mới để phản hồi cạnh lên của xung nhịp, nó cũng mất một khoảng thời gian nhỏ để chuyển giá trị mới đến đầu ra (gọi là **độ trễ clk-to-q**)[^hold-time-clk-to-q].

:::{figure} images/timing-diagram.png
:label: fig-ff-timing-vi
:width: 55%
:alt: "Sơ đồ thời gian có chú thích tập trung vào cạnh lên của xung nhịp CLK. Sơ đồ ghi nhãn các khoảng thời gian xung quanh cạnh lên của xung nhịp cho thời gian thiết lập trước cạnh (A), thời gian giữ sau cạnh (B), và độ trễ clk-to-q cho đến khi đầu ra q ổn định sau cạnh (C); d phải giữ ổn định qua cửa sổ thiết lập-giữ."

Sơ đồ thời gian của FF loại d kích hoạt cạnh dương.
:::

Nhìn vào @fig-ff-timing-vi ở trên:

* `A` minh họa **thời gian thiết lập** (setup time).
* `B` minh họa **thời gian giữ** (hold time).
* `C` minh họa **độ trễ clk-to-q**.
* Các đường kẻ dọc vàng đứt nét minh họa cửa sổ thời gian trong đó đầu vào `d` phải ổn định (từ đầu thời gian thiết lập đến cuối thời gian giữ).

[^hold-time-clk-to-q]: Không có mối quan hệ cụ thể giữa thời gian giữ và độ trễ clk-to-q, vì cái trước mô tả đầu vào và cái sau mô tả đầu ra. Tuy nhiên, trong thực tế, thời gian giữ nhỏ hơn độ trễ clk-to-q.
