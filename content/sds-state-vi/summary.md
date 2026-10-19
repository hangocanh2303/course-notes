---
title: "Tổng Kết"
---

## Kết Luận$\dots$

Một **máy trạng thái hữu hạn** là một loại automaton đơn giản trong đó trạng thái tiếp theo và đầu ra chỉ phụ thuộc vào trạng thái hiện tại và đầu vào. Mỗi trạng thái được biểu diễn bằng một vòng tròn, và mọi máy trạng thái hữu hạn đúng đắn đều có một trạng thái bắt đầu, được đánh dấu bằng nhãn "Start" hoặc một mũi tên đơn dẫn vào nó. Mỗi chuyển đổi giữa các trạng thái được ghi nhãn [đầu vào]/[đầu ra].

Ví dụ, @fig-disc8-fsm-vi bên dưới là một máy trạng thái hữu hạn với hai trạng thái (0 và 1). Nó xuất ra 1 khi trạng thái thay đổi, và 0 khi trạng thái giữ nguyên.

Máy bắt đầu ở trạng thái 0. Khi đầu vào là 0, nó ở lại trạng thái hiện tại và xuất ra 0. Khi đầu vào là 1, nó chuyển sang trạng thái khác và xuất ra 1. Khi ở trạng thái 1, máy hoạt động theo cách tương tự: nó ở lại trạng thái 1 và xuất ra 0 khi đầu vào là 0, và chuyển về trạng thái 0 với đầu ra 1 khi đầu vào là 1.

:::{figure} images/FSM_Precheck_Disc8.png
:label: fig-disc8-fsm-vi
:width: 60%
:alt: "Máy hai trạng thái với bong bóng trạng thái 0 và 1: đầu vào 0 ở bất kỳ trạng thái nào tạo vòng lặp tự thân với đầu ra 0, trong khi đầu vào 1 ở bất kỳ trạng thái nào thay đổi trạng thái và dẫn đến đầu ra 1. Trạng thái bắt đầu được ghi nhãn bên trái và chỉ ngay đến trạng thái 0."

Sơ đồ FSM cho mô tả ở trên.
:::

Với logic tổ hợp và thanh ghi, bất kỳ FSM nào cũng có thể được hiện thực trong phần cứng!

Đầu vào cho thanh ghi phải ổn định trong một khoảng thời gian nhất định xung quanh cạnh lên của xung nhịp để đầu vào được lấy mẫu chính xác. Khoảng thời gian trước cạnh lên mà đầu vào phải ổn định được gọi là **thời gian thiết lập** (setup time), và thời gian sau cạnh lên mà đầu vào phải ổn định được gọi là **thời gian giữ** (hold time). Thời gian giữ thường được bao gồm trong độ trễ clk-to-q, vì vậy thời gian clk-to-q thường sẽ lớn hơn hoặc bằng thời gian giữ.

Về mặt logic, thực tế rằng clk-to-q ≥ thời gian giữ có ý nghĩa vì chỉ mất clk-to-q giây để sao chép giá trị, vì vậy không cần thiết phải giữ giá trị được nạp vào thanh ghi lâu hơn. Xem xét mạch thanh ghi trong @fig-reg-circuit-ex-disc8-vi và giả sử thời gian **thiết lập** 2.5ps, thời gian **giữ** 1.5ps, và thời gian **clk-to-q** 1.5ps. Tín hiệu xung nhịp có chu kỳ 13ps.

:::{figure} images/register_circuit_example.png
:label: fig-reg-circuit-ex-disc8-vi
:width: 100%
:alt: "Sơ đồ khối thanh ghi bên trái cạnh sơ đồ dạng sóng thời gian với các sóng xung nhịp, dữ liệu, và đầu ra hiển thị thời gian thiết lập, giữ, và clk-to-q. Sơ đồ thời gian cũng mô tả khoảng không ổn định được tô bóng cho đến sau thời gian clk-to-q của cạnh lên xung nhịp đầu tiên (khi đầu ra đầu tiên ổn định)."

Mạch thanh ghi ví dụ với sơ đồ thời gian.
:::

Lưu ý rằng giá trị của đầu ra trong sơ đồ không thay đổi ngay lập tức sau cạnh lên của xung nhịp. Cho đến khi đủ thời gian trôi qua để đầu ra phản ánh đầu vào, giá trị được giữ bởi đầu ra là rác; điều này được biểu diễn bằng phần tô xám của đồ thị đầu ra. Thời gian chu kỳ xung nhịp phải đủ nhỏ để các đầu vào cho thanh ghi không thay đổi trong thời gian giữ và đủ lớn để tính đến thời gian clk-to-q, thời gian thiết lập, và độ trễ logic tổ hợp.

Một số mối quan hệ SDS quan trọng được liệt kê bên dưới:

$$\tau_{\text{độ trễ đường găng}} = \tau_{\text{clk-to-q}} + \tau_{\text{độ trễ logic tổ hợp}} + \tau_{\text{thời gian thiết lập}}$$

trong đó $\tau_{\text{độ trễ logic tổ hợp}}$ là độ trễ logic tổ hợp tối đa cho bất kỳ đường thanh ghi $\rightarrow$ thanh ghi nào trong mạch. Đường có độ trễ tối đa được gọi là **đường găng** (critical path).

Ngoài ra, các mạch phải thỏa mãn các ràng buộc thời gian giữ vì thời gian giữ có thể bị vi phạm nếu dữ liệu lan truyền quá nhanh:

$$\tau_{\text{clk-to-q}} + \tau_{\text{độ trễ tổ hợp nhỏ nhất}} \geq \tau_{\text{thời gian giữ}}$$

## Đọc Thêm Từ Sách Giáo Khoa

P&H A.3-A.6

## Tài Liệu Tham Khảo Bổ Sung

* [Tài liệu Giới thiệu SDS](../resources/sds.pdf) - Xung nhịp, Độ trễ
* [Tài liệu State](../resources/state.pdf) - Thanh ghi, FSM

## Bài Tập
Kiểm tra kiến thức của bạn!

### Ôn Tập Khái Niệm

:::{exercise}
:label: state-01-vi
1. **Đúng/Sai**: Độ trễ "clk-to-q" của thanh ghi là thời gian giữa cạnh lên của tín hiệu xung nhịp và thời gian giữ của thanh ghi.
:::

:::{solution} state-01-vi
:label: state-01-sol-vi
:class: dropdown
**Sai.** Độ trễ "clk-to-q" là thời gian giữa cạnh lên của tín hiệu xung nhịp và thời gian cần thiết để đầu ra của thanh ghi phản ánh đầu vào mới.
::: 

:::{exercise}
:label: state-02-vi
2. **Đúng/Sai**: Các phần tử trạng thái chỉ cập nhật đầu ra của chúng trên cạnh lên của xung nhịp, ngay cả khi các đầu vào thay đổi giữa các cạnh lên của xung nhịp.
:::

:::{solution} state-02-vi
:label: state-02-sol-vi
:class: dropdown
**Đúng.** Các phần tử trạng thái sẽ cập nhật đầu ra của chúng trên cạnh lên của tín hiệu xung nhịp. Giữa các cạnh lên, đầu ra của phần tử trạng thái sẽ giữ nguyên bất kể các đầu vào.
:::
