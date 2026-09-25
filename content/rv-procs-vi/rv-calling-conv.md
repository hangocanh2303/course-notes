---
title: "Quy ước gọi hàm"
---

<!-- không tham chiếu cái này một cách tổng quát; tham chiếu sec-rv-calling-convention-->
(sec-rv-calling-convention-top)=
## Mục tiêu học tập

* Giải thích tại sao quy ước gọi thanh ghi giúp triển khai gọi thủ tục trong RISC-V.
* Xác định caller hay callee chịu trách nhiệm cho mỗi trong sáu bước cơ bản để gọi thủ tục.
* Sử dụng green card RISC-V để xác định thanh ghi là caller-saved, callee-saved, hay không phải cả hai.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/TLskZ9Ic-T8
:width: 100%
:title: "[CS61C FA20] Lecture 10.2 - RISC-V Procedures: Register Conventions"
:::

::::

Chúng ta đã thảo luận trước đó về [tên thanh ghi](#sec-register-names) và [quy ước thanh ghi](#sec-register-conventions):

> Tên thanh ghi định nghĩa **quy ước**—tức là, chỉ định cách các lệnh hợp ngữ nên sử dụng các thanh ghi cụ thể cho các chức năng phổ biến cụ thể. Các hạn chế này giúp xây dựng "thỏa thuận" về cách dịch các thành phần riêng biệt của chương trình để các lệnh hợp ngữ khớp với nhau.

Xét [bảng quy ước thanh ghi](#tab-calling-convention) trên green card RISC-V. Cho đến nay chúng ta chỉ thảo luận vài quy ước thanh ghi–cụ thể là, con trỏ stack `sp` và địa chỉ trả về `ra`.

Phần còn lại của các quy ước thanh ghi chúng ta sẽ thảo luận trong khóa học này[^gp-tp] liên quan đến cách sử dụng thanh ghi giữa **gọi thủ tục**.

[^gp-tp]: Ngoài phạm vi: `gp` (global pointer, được sử dụng để lưu tham chiếu đến heap) và `tp` (thread pointer, được sử dụng để lưu các stack riêng biệt cho thread). Hãy coi các thanh ghi này là "cấm sử dụng"–sử dụng chúng vi phạm quy ước thanh ghi!


## Động lực

:::{warning} Tránh clobbering

Vì tất cả các lệnh sử dụng cùng quy ước thanh ghi RISC-V, các giá trị thanh ghi có thể bị **clobber** với các cuộc gọi thủ tục lồng nhau–có nghĩa là các giá trị có thể bị ghi đè.
:::

Trong mã C bên dưới, `main` gọi `sum_square`, gọi hai lần `mult`.

```{code} c
:linenos:

int main() {
  int z = sum_square(3, 4);
  ...
}

int mult(int x, int y) {
  return x * y;
}

int sum_square(int x, int y) {
  return mult(x, x) + mult(y, y);
}
```

Như chúng ta đã thảo luận trong [phần trước](#sec-rv-procedure-calls), `jal ra Label` và `jr ra` là một cặp lệnh phổ biến lưu địa chỉ trả về vào thanh ghi `ra`. Quy ước đặt tên thanh ghi có nghĩa là trong cả hai lệnh, `ra` đề cập đến cùng thanh ghi số `x1`. Tuy nhiên, nhất thiết `sum_square` sẽ muốn nhảy lại về một số `ra`, nhưng điều này sẽ bị ghi đè bởi cả hai cuộc gọi đến `mult`. Do đó chúng ta cần lưu địa chỉ trả về của `sum_square` ở đâu đó trước cuộc gọi đến `mult`—hãy sử dụng stack!


:::{tip} Kiểm tra nhanh

Giả sử rằng chúng ta không sử dụng stack để lưu giá trị thanh ghi. Từ [quy ước thanh ghi](#tab-calling-convention) green card RISC-V, chúng ta biết:

* Thanh ghi `ra` là thanh ghi `x1` và có địa chỉ trả về mà callee nên sử dụng để trả về caller.
* Thanh ghi `a0` là thanh ghi `x10` và là đối số thứ zero cho callee.
* Thanh ghi `a0` **cũng** là thanh ghi giữ giá trị trả về khi callee trả về cho caller.

Nếu chúng ta _không_ lưu gì vào stack, điều gì có thể sai khi chúng ta cố gọi `factorial(2)` sử dụng khai báo factorial bên dưới?

```c
// giả sử số không âm
int factorial(int n) {
  if (n == 1) return 1;
  int a = n*factorial(n-1);
  return a;
}
```

:::

:::{note} Hiển thị giải thích
:class: dropdown

Sử dụng [các bước cơ bản của gọi thủ tục](#sec-rv-procedure-call-steps) được thảo luận trong phần trước, chúng ta chia sẻ những gì xảy ra từ góc nhìn của **caller** `factorial(2)`.

1. `factorial(2)` Chuẩn bị cho **callee** `factorial(1)` bằng cách đặt đối số `1` vào thanh ghi `a0`.
1. Chuyển điều khiển cho callee `factorial(1)` bằng cách thực thi lệnh `jal ra factorial`.
1. (bỏ qua; nhiệm vụ của `factorial(1)`)
1. (bỏ qua; nhiệm vụ của `factorial(1)`)
1. (bỏ qua; nhiệm vụ của `factorial(1)`)
1. (bỏ qua; nhiệm vụ của `factorial(1)`)

Tại thời điểm này, `factorial(2)` đã giành lại điều khiển và mong đợi `a0` có giá trị trả về từ `factorial(1)`.

Tiếp theo, `factorial(2)` muốn nhân giá trị trả về này với đối số của chính nó, `2`. Thật không may, tại thời điểm này, thanh ghi `a0` đã bị ghi đè!
:::

(sec-rv-calling-convention)=
## Quy ước gọi thanh ghi

Xét [các bước cơ bản của gọi hàm](#sec-rv-procedure-call-steps). Như một phần của Bước 2 (nơi caller chuyển điều khiển và thực thi cho callee), caller có thể "lưu" các thanh ghi hiện tại của họ như thế nào?

:::{warning} Giải pháp strawman

Chúng ta _có thể_ push và pop 31 thanh ghi `x1` đến `x31` của caller lên stack giữa các cuộc gọi thủ tục. Mặc dù đơn giản, cách tiếp cận này tốn kém: chúng ta hiếm khi sử dụng tất cả 31 thanh ghi (theo quy ước thanh ghi) nên chúng ta có thể sao chép dữ liệu thừa với các phép toán bộ nhớ đắt tiền.
:::

Thay vào đó, RISC-V định nghĩa một **calling convention**:

> Một tập hợp các quy tắc được chấp nhận chung về việc thanh ghi nào sẽ không thay đổi sau một cuộc gọi thủ tục, và thanh ghi nào có thể thay đổi.

Nói cách khác, để giảm thiểu các load và store đắt tiền giữa các cuộc gọi thủ tục, calling convention RISC-V chia thanh ghi thành hai loại:

* Thanh ghi volatile, tạm thời **không** được bảo toàn qua cuộc gọi thủ tục.
* Thanh ghi saved **được** bảo toàn qua cuộc gọi thủ tục.

:::{hint} Calling convention không ngụ ý rằng bất kỳ thanh ghi nào bị cấm sử dụng!

Một thủ tục có thể vừa là caller _và_ callee! Khi một thủ tục đang điều khiển, nó có quyền truy cập vào tất cả thanh ghi. Điều calling convention *ngụ ý* là một hợp đồng hai chiều:

1. **Mong đợi của Caller**: Khi callee trả về từ thực thi, caller nên mong đợi rằng các thanh ghi volatile có thể đã thay đổi, nhưng các thanh ghi saved không thay đổi.
1. **Mong đợi của Callee**: Callee có thể thay đổi giá trị trong các thanh ghi saved, nhưng nó phải khôi phục giá trị gốc trước khi trả về. Callee có thể thay đổi giá trị trong các thanh ghi tạm thời mà không cần khôi phục giá trị.
:::

Có nhiều cách để chỉ định quy ước này. [ASM Manual](https://github.com/riscv-non-isa/riscv-asm-manual/blob/main/src/asm-manual.adoc#general-registers) chỉ định quy ước như liệu thanh ghi có được bảo toàn qua cuộc gọi thủ tục không ("yes" hoặc "no"). Quy ước trong @tab-calling-convention-copy chỉ định ai phải **lưu** giá trị thanh ghi ("caller" hoặc "callee").

* **Thanh ghi volatile caller-saved**. Caller chịu trách nhiệm lưu các giá trị thanh ghi này cho chính nó trước khi gọi callee. Sau khi callee trả về, caller sau đó có thể khôi phục các giá trị này nếu cần.
* **Thanh ghi saved callee-saved**. Callee chịu trách nhiệm lưu và khôi phục các giá trị thanh ghi này. Hai bước này thường được thực hiện trong [prologue và epilogue](#sec-rv-procedure-call-steps), tương ứng.

:::{table} Quy ước gọi thanh ghi RV32I. Bảng này cũng có sẵn trên [green card](#sec-green-card) khóa học của chúng ta.
:label: tab-calling-convention-copy
:align: center

| Thanh ghi | Tên | Mô tả | Người lưu |
| :--- | :--- | :--- | :---: |
| `x0` | `zero` | Hằng số 0 | - |
| `x1` | `ra` | Địa chỉ trả về | Caller |
| `x2` | `sp` | Con trỏ stack | Callee |
| `x3` | `gp` | Global Pointer[^gp-tp] | - |
| `x4` | `tp` | Thread Pointer[^gp-tp] | - |
| `x5-7` | `t0-2` | Thanh ghi tạm thời | Caller |
| `x8` | `s0` / `fp` | Thanh ghi saved 0 / Frame Pointer | Callee |
| `x9` | `s1` | Thanh ghi saved | Callee |
| `x10-11` | `a0-1` | Đối số thủ tục / Giá trị trả về | Caller |
| `x12-17` | `a2-7` | Đối số thủ tục | Caller |
| `x18-x27` | `s2-11` | Thanh ghi saved | Callee |
| `x28-31` | `t3-6` | Tạm thời | Caller |
:::

:::{note} Lưu và khôi phục thanh ghi

Caller có thể sử dụng thanh ghi saved để lưu/khôi phục giá trị thanh ghi; trong khi đó, callee có thể sử dụng thanh ghi tạm thời để lưu/khôi phục thanh ghi caller.

Cả caller và callee cũng có thể sử dụng stack để lưu/khôi phục giá trị thanh ghi:

* **Lưu thanh ghi**. Cấp phát không gian trên stack frame trước bằng cách giảm `sp`, sau đó store giá trị thanh ghi vào bộ nhớ stack.
* **Khôi phục thanh ghi**. Load giá trị từ bộ nhớ stack vào thanh ghi, sau đó giải phóng không gian trên stack frame (bằng cách tăng `sp`).

Nhìn chung, lưu và khôi phục thanh ghi được coi là một phần của stack frame. Nhiều nhà thiết kế thấy hữu ích khi giảm và tăng con trỏ stack chính xác một lần trong prologue và epilogue, tương ứng—điều này tương ứng với push và pop stack frame.

:::

## Các bước cơ bản, xem lại

Theo calling convention, chúng ta xem lại [Sáu bước cơ bản để gọi thủ tục](#sec-rv-procedure-call-steps) từ [phần trước](#sec-rv-procedure-calls) chi tiết hơn:

(sec-rv-procedure-call-steps-revisited)=
:::{note} Sáu bước cơ bản để gọi thủ tục, xem lại

| Bước | Ai | Mô tả gốc | Mô tả xem lại
| :---: | :--- | :--- | :--- |
| 1 | Caller | **Thiết lập đối số**. Đặt đối số vào thanh ghi. | **Thiết lập cho callee**. Lưu thanh ghi caller-saved nếu cần, bằng cách sao chép vào `s0-s11` hoặc lên stack. Đặt đối số vào `a0`, …, `a7`, stack, v.v. |
| 2 | Caller | **Chuyển điều khiển đến thủ tục**. Sử dụng lệnh `jal` (jump-and-link): `jal ra fnLabel` | - |
| 3 | Callee | **Prologue**. Thu nhận tài nguyên lưu trữ (cục bộ): không gian stack (ví dụ: push một frame lên stack), lưu giá trị thanh ghi, v.v. | **Prologue.** Push một stack frame mới bằng cách giảm `sp`. Lưu thanh ghi như `s0-s11` nếu callee cần sử dụng chúng. Lưu `ra` nếu callee sẽ gọi subroutine. Cấp phát đủ không gian cho biến cục bộ không phải thanh ghi như mảng stack. |
| 4 | Callee | **Thực hiện nhiệm vụ mong muốn.** | - |
| 5 | Callee | **Epilogue**. Đặt giá trị trả về ở nơi caller có thể truy cập, khôi phục giá trị thanh ghi, và giải phóng lưu trữ cục bộ trên stack (ví dụ: pop frame khỏi stack). | Đặt giá trị trả về vào `a0` (hoặc `a1` nếu cần). Khôi phục `ra` nếu callee đã gọi subroutine. Pop stack frame hiện tại bằng cách tăng `sp`. |
| 6 | Callee | **Epilogue: Trả điều khiển về điểm gốc**. Sử dụng lệnh `jr`: `jr ra` | **Epilogue: Trả điều khiển về caller.** Sử dụng lệnh `jr`: `jr ra` |

:::
