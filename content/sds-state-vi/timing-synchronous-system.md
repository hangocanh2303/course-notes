---
title: "Định Thời Hệ Thống Đồng Bộ"
subtitle: Tác giả John Wawrzynek, chỉnh sửa bởi Lisa Yan
---

## Mục Tiêu Học Tập

* Giải thích tại sao thanh ghi giúp ổn định hệ thống đồng bộ.
* Tính toán độ trễ của đường găng (critical path) của mạch.
* Xác định tần số xung nhịp tối đa cho một mạch cho trước.
* Giải thích vi phạm thời gian giữ và cách tránh nó.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/NJp8PSGsV9U
:width: 100%
:title: "[CS61C FA20] Lecture 15.1 - State, State Machines: Accumulator"
:::

::::

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/Q8jlosmglo0
:width: 100%
:title: "[CS61C FA20] Lecture 15.3 - State, State Machines: Accumulator revisited"
:::
::::

Mạch logic tổ hợp tạo ra đầu ra hoàn toàn dựa trên tín hiệu đầu vào của chúng. Chúng được sử dụng cho nhiều chức năng khác nhau. Mặt khác, các phần tử trạng thái có một số ít công dụng rất cụ thể. Đây là các mạch ghi nhớ giá trị tín hiệu đầu vào của chúng.

:::{tip} Thanh ghi có thể được sử dụng để kiểm soát dòng tín hiệu giữa các mạch logic tổ hợp.

Ví dụ về bộ tích lũy trong phần này sẽ thuyết phục bạn rằng có những nơi mà phần tử trạng thái là cần thiết cho chức năng mạch đúng đắn.

Tuy nhiên, việc đưa vào phần tử trạng thái sẽ ảnh hưởng đến bao nhiêu có thể xảy ra trong một chu kỳ xung nhịp cho trước. Chúng ta thảo luận điều này bên dưới.

:::

:::{tip} Thanh ghi có thể được sử dụng để tăng tần số xung nhịp có thể đạt được.

Với khái niệm thanh ghi đường ống, chúng ta có thể thêm thanh ghi vào mạch để thực sự cải thiện hiệu năng. Chúng ta thảo luận ý tưởng này trong [phần tiếp theo](#sec-pipelining-circuits-vi).
:::

## Ví Dụ Bộ Tích Lũy

Xét thiết kế của một mạch có nhiệm vụ tính tổng của một danh sách các số nguyên, $X_0, X_1, X_2, \dots, X_{n-1}$. Giả sử rằng một mạch khác áp dụng các số từ danh sách một lần một — một số mỗi chu kỳ xung nhịp. Dưới đây là một mô tả đồ họa của một trừu tượng của mạch chúng ta sử dụng để tính tổng — chúng ta sẽ gọi khối là "sum". Các giá trị $X$ được áp dụng, một giá trị mỗi chu kỳ, và sau $n$ chu kỳ, tổng có mặt tại đầu ra, S.

:::{figure} images/accumulator-block.png
:label: fig-acc-block-vi
:width: 45%
:alt: "Ký hiệu hộp bộ tích lũy trừu tượng với các đầu vào x_i được áp dụng và tích lũy trong tổng S, có sẵn sau n chu kỳ xung nhịp."

Sơ đồ khối cho bộ tích lũy.
:::

Chúng ta nên đặt gì bên trong khối sum để đạt được chức năng mong muốn? Rõ ràng mạch liên quan đến một bộ cộng. Ngoài ra, ở mỗi bước, chúng ta cần lấy tổng hiện tại và chuyển nó trở lại bộ cộng để nó có thể cộng thêm một giá trị X vào nó.

### Giải Pháp Thử Nghiệm: Mạch Không Ổn Định

@fig-accumulator-circuit-unstable-vi là lần thử đầu tiên của chúng ta:

:::{figure} images/accumulator-circuit-unstable.png
:label: fig-accumulator-circuit-unstable-vi
:width: 55%
:alt: "Bộ cộng thử nghiệm với đầu ra tổng được nạp thẳng trở lại đầu vào bộ cộng, thiếu thanh ghi nên tổng chạy đua và đếm quá trong một chu kỳ."

Sơ đồ mạch cho bộ tích lũy.
:::

Hãy kiểm tra hoạt động của mạch này chi tiết để xem nó có làm đúng công việc không. Giả sử $S$ bắt đầu ở $0$. Sau đó chúng ta áp dụng $X_0$. Sau một độ trễ ngắn (độ trễ lan truyền của bộ cộng, $\tau_{add}$) $S$ sẽ thay đổi thành $X_0$. Sau đó sau một $\tau_{add}$ độ trễ nữa, S sẽ thay đổi thành $X_0 + X_0$, rồi sau một $\tau_{add}$ độ trễ nữa, S sẽ thay đổi thành $X_0 + X_0 + X_0$, v.v. Vì $\tau_{add}$ thường nhỏ hơn chu kỳ xung nhịp, **tất cả các phép cộng $X_0$ này sẽ xảy ra trước khi $X_1$ được áp dụng.**

Đây rõ ràng không phải là hoạt động đúng của "sum". Mạch mất kiểm soát! Chúng ta cần một cách để kiểm soát việc tính toán, **từng bước một**.

(sec-use-register-vi)=
### Sử Dụng Thanh Ghi

Cách để kiểm soát việc tính toán là đặt một thanh ghi trong **đường phản hồi** (feedback path) (kết nối từ đầu ra của bộ cộng đến đầu vào của nó), như được hiển thị trong @fig-accumulator-circuit-register-vi.

:::{figure} images/accumulator-circuit-register.png
:label: fig-accumulator-circuit-register-vi
:width: 55%
:alt: "Mạch bộ cộng với đường phản hồi qua thanh ghi được đồng bộ có reset, giữ tổng một phần ổn định giữa các cập nhật X."

Sơ đồ mạch cho bộ tích lũy có reset.
:::

Thanh ghi giữ giá trị hiện tại của $S$ trong khi giá trị tiếp theo đang được hình thành. Sau khi chúng ta hài lòng với $S$ tiếp theo, chúng ta nạp nó vào thanh ghi và áp dụng giá trị $X$ mới, sau đó chờ $S$ mới xuất hiện tại đầu ra của bộ cộng. Quá trình được lặp lại cho tất cả $n$ giá trị $X$.

**Tín hiệu reset** là một tính năng phổ biến của mạch thanh ghi. Trong [sơ đồ mạch](#fig-accumulator-circuit-register-vi), đây là tín hiệu đầu vào "reset" cho thanh ghi. Đây là một tín hiệu có thể được sử dụng để xóa giá trị thanh ghi, và do đó cho chúng ta cách để khởi tạo mạch.

:::{note} Thêm về Tín Hiệu Reset
:class: dropdown

Đầu vào "reset" cho thanh ghi là một đầu vào đặc biệt xóa thanh ghi để nó giữ tất cả 0.

Loại đầu vào reset phổ biến nhất được gọi là **reset đồng bộ**: Nếu tín hiệu đầu vào reset có giá trị 1 trên _cạnh lên_ của tín hiệu xung nhịp, thì thanh ghi được xóa, bất kể giá trị của đầu vào D. Nói cách khác, tín hiệu reset có ưu tiên hơn đầu vào dữ liệu và buộc thanh ghi về tất cả 0.
:::

### Sơ Đồ Dạng Sóng

@fig-accumulator-timing-vi hiển thị các dạng sóng minh họa hoạt động **thô** của [mạch bộ tích lũy](#fig-accumulator-circuit-register-vi), cho một vài lần lặp. Kết quả được tạo ra một cách đẹp đẽ từng cái một. Trong ví dụ này, thanh ghi được sử dụng để giữ lại việc chuyển dữ liệu từ một nơi này đến nơi khác trong mạch.

Đầu ra của mạch được ghi nhãn $S_i$, và đầu ra của thanh ghi được ghi nhãn $S_{i-1}$ để nhắc chúng ta rằng thanh ghi trì hoãn tín hiệu 1 chu kỳ. Vì vậy nếu đầu ra của mạch đang giữ kết quả của lần lặp thứ i, thì thanh ghi giữ kết quả của lần lặp thứ (i − 1).

:::{figure} images/accumulator-timing.png
:label: fig-accumulator-timing-vi
:width: 100%
:alt: "Sơ đồ thời gian với các dạng sóng căn chỉnh cho xung nhịp, đầu vào X, đầu ra bộ cộng S_i, và đầu ra thanh ghi bị trễ một chu kỳ S_i-1, hiển thị một lần tích lũy mỗi chu kỳ trong mạch này."

Sơ đồ thời gian thô cho bộ tích lũy, bây giờ với dạng sóng cho đầu ra thanh ghi $S_{i-1}$.
:::

:::{note} Hiển thị giải thích của @fig-accumulator-timing-vi
:class: dropdown

Bắt đầu bằng cách nhìn vào thời gian của thay đổi trên đầu ra của thanh ghi $S_{i−1}$. Điều này theo sau cạnh dương của xung nhịp sau một độ trễ nhỏ (thời gian clk-to-q của các flip-flop được sử dụng để hiện thực thanh ghi). Chúng ta giả sử rằng đầu vào $X$ được áp dụng chính xác cùng thời điểm. Hai giá trị di chuyển qua bộ cộng cùng nhau và sau một độ trễ nhỏ (độ trễ lan truyền của bộ cộng $\tau_{add}$) một kết quả mới xuất hiện tại đầu ra của bộ cộng, $S_i$. Sau đó tất cả yên tĩnh cho đến cạnh lên của xung nhịp. Tại thời điểm đó, giá trị đầu ra được chuyển đến thanh ghi và toàn bộ quá trình lặp lại.
:::

### Thời Gian Đến Của Tín Hiệu Khác Nhau

Trong thực tế $X_i$ có thể không nhất thiết đến cùng thời điểm với giá trị phản hồi, $S_{i−1}$. Các dạng sóng bên dưới trong @fig-accumulator-timing-realistic-vi hiển thị $X$ đến **muộn hơn một chút** so với $S_{i−1}$.

:::{figure} images/accumulator-timing-realistic.png
:label: fig-accumulator-timing-realistic-vi
:width: 100%
:alt: "Sơ đồ thời gian bộ tích lũy tương tự như trường hợp lý tưởng, nhưng X chuyển đổi hơi sau S_i-1 nên đầu ra bộ cộng tạm thời trở nên không xác định trong khi đầu vào không ổn định, và sau đó ổn định trước cạnh lấy mẫu tiếp theo."

Mặc dù $X_i$ và $S_{i-1}$ đến bộ tích lũy vào các thời điểm khác nhau, chu kỳ xung nhịp đủ dài để $S_i$ ổn định trước cạnh lên tiếp theo của xung nhịp.
:::

:::{note} Hiển thị giải thích của @fig-accumulator-timing-realistic-vi
:class: dropdown
Khi thanh ghi lần đầu tiên bắt $X_0$, trong một khoảng thời gian nhỏ đầu vào $X$ vẫn có $X_0$, do đó bộ cộng bắt đầu tính $X_0 + X_0$! Tuy nhiên, phép tính sai lầm này nhanh chóng bị hủy bỏ khi đầu vào $X$ thay đổi thành $X_1$. Thật không may, phép tính bị hủy bỏ có thể sẽ đi qua bộ cộng, tạo ra một loại không ổn định tại đầu ra. Tuy nhiên, sự không ổn định trong $S_i$ không có ảnh hưởng đến $S_{i−1}$, vì nó bắt giá trị của nó từ $S_i$ trước khi nó trở nên xấu.
:::

Như đã thấy ở trên, trên mỗi chu kỳ có một khoảng thời gian nhỏ nơi bộ cộng có các đầu vào không nhất quán. Loại sai lệch thời gian đến và sự không ổn định đầu ra tiếp theo là phổ biến trong nhiều mạch. Trong các mạch được thiết kế đúng cách, sự không ổn định này không bao giờ xảy ra xung quanh cạnh lên của xung nhịp và do đó bị bỏ qua bởi các thanh ghi và mạch hạ lưu.

## Tính Toán Tần Số Xung Nhịp Tối Đa

Lưu ý rằng tần số xung nhịp tối đa (chu kỳ xung nhịp tối thiểu) bị giới hạn bởi độ trễ lan truyền của phép cộng/dịch. Nếu chúng ta cố gắng làm cho chu kỳ xung nhịp quá ngắn, thì logic cộng/dịch sẽ không có đủ thời gian để tạo ra đầu ra của nó và thanh ghi đầu ra sẽ bắt giá trị không chính xác.

Đồng thời, chúng ta tìm cách tạo ra các mạch băng thông cao, tạo ra nhiều đầu ra mỗi giây. Chúng ta muốn tìm một sự cân bằng lý tưởng giữa xung nhịp tần số cao và đầu ra ổn định.

(sec-critical-path-vi)=
:::{hint} Chu kỳ xung nhịp tối thiểu là độ trễ trên đường găng.

**Đường găng của mạch** (Critical path): Đường đi giữa (các) đầu vào và (các) đầu ra gây ra độ trễ trường hợp xấu nhất (tối đa). Thường điều này được xác định bằng cách xác định độ trễ dài nhất từ bất kỳ đầu ra thanh ghi (flip-flop) nào đến bất kỳ đầu vào thanh ghi (flip-flop) nào.
:::

(sec-max-clock-frequency-vi)=
:::{hint} Tần số xung nhịp tối đa là nghịch đảo của chu kỳ xung nhịp tối thiểu.

$T$ là chu kỳ xung nhịp (tính bằng giây, hoặc nano giây), và $f$ là tần số xung nhịp (tính bằng chu kỳ/giây, hoặc s{sup}`-1`, hoặc Hz, hoặc GHz).

Mối quan hệ: $f = 1/T$.
:::

@fig-critical-path-vi hiển thị một mạch điển hình.

:::{figure} images/critical-path.png
:label: fig-critical-path-vi
:width: 55%
:alt: "Đường thanh ghi-đến-thanh ghi mẫu: clk-to-q (mũi tên xanh), độ trễ logic tổ hợp (mũi tên hồng), và ngân sách thời gian thiết lập (mũi tên xanh lá) hiển thị những yếu tố nào trong đường thiết lập chu kỳ xung nhịp tối thiểu."

Để xác định chu kỳ xung nhịp tối thiểu, tính độ trễ trên đường găng.
:::

Trong mạch này, đường găng là:

$$\text{đường găng} = \text{độ trễ clk-to-q} + \text{độ trễ CL} + \text{thời gian thiết lập}$$

trong đó độ trễ clk-to-q và thời gian thiết lập là các thuộc tính của thanh ghi (ví dụ: flip-flop nội bộ), và độ trễ CL là độ trễ qua khối logic tổ hợp.

::::{tip} Kiểm Tra Nhanh

Tần số xung nhịp tối đa cho mạch được hiển thị trong @fig-critical-path-practice-vi là bao nhiêu?

* **A.** 5 GHz
* **B.** 200 MHz
* **C.** 500 MHz
* **D.** 1/7 GHz
* **E.** 1/6 GHz
* **F.** Cái gì khác

Giả sử tất cả thanh ghi có độ trễ clk-to-q 1 ns, thời gian thiết lập, và thời gian giữ; cổng AND có độ trễ lan truyền 1 ns; và tất cả đầu vào không kết nối đến từ thanh ghi.

:::{figure} images/critical-path-practice.png
:label: fig-critical-path-practice-vi
:width: 55%
:alt: "Bản vẽ logic tổ hợp thực hành với vài cổng AND 2 đầu vào và một flip-flop. Độ trễ đường găng hiện chưa được ghi nhãn và để lại cho bài tập."

Mạch ví dụ.
:::
::::

::::{note} Hiển thị Đáp Án
:class: dropdown

* **B.** 200 MHz

Đường găng được tô sáng trong @fig-critical-path-answer-vi.

:::{figure} images/critical-path-answer.png
:label: fig-critical-path-answer-vi
:width: 55%
:alt: "Cùng mạch chuỗi AND với đường găng thanh ghi-đến-thanh ghi dài nhất được tô sáng màu vàng. Đường găng này đi qua ba trong số bốn cổng AND tổng cộng."

Mạch ví dụ, với đường găng được tô sáng.
:::

$$
\begin{align}
\text{đường găng}
  &= \text{độ trễ clk-to-q} + \text{độ trễ của 3 cổng AND} + \text{thời gian thiết lập} \\
  &= 1 \text{ns} + 3 \text{ns} + 1 \text{ns} \\
  &= 5 \text{ns}
\end{align}
$$

Chu kỳ xung nhịp tối thiểu là 5 ns = $10^{-9}$ giây mỗi chu kỳ. Tần số tối đa do đó là $1/(5 \times 10^{-9} \text{s}) = (0.2 \times 10^9) \text{s}^{-1} = (200 \times 10^6) \text{Hz}$, hoặc 200 MHz.

::::

## Vi Phạm Thời Gian Giữ

Ở trên, đường găng xác định tần số xung nhịp tối đa mà chúng ta có thể sử dụng để đảm bảo đầu vào ổn định cho các phần tử thanh ghi của chúng ta. Tiếp theo, chúng ta trình bày một vấn đề khác: **vi phạm thời gian giữ** (hold time violations).

Nhớ lại từ [trước đó](#sec-registers-vi) rằng **thời gian giữ** là khoảng thời gian trong đó đầu vào d của thanh ghi phải ổn định _sau_ cạnh lên của xung nhịp. Có một số trường hợp dữ liệu lan truyền qua mạch quá nhanh mà đầu vào cho thanh ghi trở nên không ổn định trong thời gian giữ.

Mặc dù hiếm[^hold-time-violation], điều này có thể xảy ra nếu độ trễ _trường hợp tốt nhất_ giữa các phần tử được đồng bộ _ngắn hơn_ thời gian giữ. Để giảm thiểu điều này, chúng ta có thể thêm độ trễ tùy ý vào logic tổ hợp của mạch để tăng độ trễ trường hợp tốt nhất.

[^hold-time-violation]: Xem chú thích liên quan trong [phần trước](#sec-registers-vi).
