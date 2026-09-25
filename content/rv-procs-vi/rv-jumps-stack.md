---
title: "Jump và Stack Frame"
---

(sec-jump-rv-stack)=
## Mục tiêu học tập

* Xác định các trường hợp sử dụng cho lệnh và pseudoinstruction jump vô điều kiện–đặc biệt, biết cách nhảy đến thủ tục và trả về từ thủ tục.
* So sánh stack frame RISC-V với stack frame C.
* Viết các lệnh RISC-V để cấp phát và giải phóng stack frame.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/XPlSOQKV8mM
:width: 100%
:title: "[CS61C FA20] Lecture 10.3 - RISC-V Procedures: Memory Allocation"
:::

::::
::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/XZAHwb7Smj0
:width: 100%
:title: "[CS61C FA20] Lecture 09.3 - RISC-V Decisions II: RISC-V Function Calls"
:::

Từ 9:10 trở đi

::::

(sec-jumps)=
## Các lệnh Jump

Chuyển điều khiển giữa các thủ tục đơn giản có nghĩa là jump vô điều kiện đến các lệnh chương trình khác nhau. Đáng chú ý, nếu một thủ tục (**caller**) gọi thủ tục khác (**callee**), callee phải biết cách **trả về** cho caller.

Nhớ lại rằng jump vô điều kiện là các lệnh khi được thực thi, đặt PC thành một lệnh khác. Do đó chúng ta cần các lệnh jump theo dõi **địa chỉ trả về** của lệnh.

:::{note} Một cặp lệnh quan trọng

Sử dụng thanh ghi có tên `ra` (thực sự là thanh ghi số `x1`) với các lệnh sau:

* **Chuyển điều khiển đến thủ tục.** `jal ra fnLabel` để đồng thời lưu địa chỉ trả về trong thanh ghi `ra` và nhảy đến địa chỉ của `fnLabel`, chuyển điều khiển cho callee.
* **Trả điều khiển về điểm gốc.** `jr ra` để nhảy đến địa chỉ trả về, đã lưu trong thanh ghi `ra`.

:::

Ở trên, "lưu địa chỉ trả về" có nghĩa là lưu địa chỉ của **lệnh tiếp theo**, `PC + 4`, vào thanh ghi có tên `ra`. "Nhảy đến một địa chỉ" có nghĩa là cập nhật `PC` để trong chu kỳ tiếp theo, máy tính thực thi một lệnh khác, không theo thứ tự.

Hãy thảo luận chi tiết hơn bên dưới.

### Pseudoinstruction Jump vs. lệnh thực

Jump vô điều kiện không đặc biệt khó hiểu (chúng tôi hy vọng). Tuy nhiên, điều quan trọng cần lưu ý là trong nhiều trường hợp sử dụng của chúng, chỉ có hai lệnh jump vô điều kiện **thực** được hiển thị trong @tab-rv-jumps: `jal` và `jalr`. Phần còn lại (`jr`, `ret`, `j,` một `jal` khác) là **pseudoinstruction**.

:::{table} Jump vô điều kiện; xem green card RISC-V cho [Control](#tab-rv32i-control) và [Pseudoinstructions](#tab-rv32i-pseudoinstructions).
:label: tab-rv-jumps
:align: center

| Lệnh hoặc Pseudoinstruction | Tên | Mô tả | Nếu là pseudo, bản dịch |
| --- | --- | --- | --- |
| `jal rd label` | Jump And Link | `R[rd] = PC + 4;`<br/>`PC = PC + offset` |  - |
| `jalr rd rs1 imm` | Jump And Link Register | `R[rd] = PC + 4;`<br/>`PC = R[rs1] + imm` | - |
| `j label` | Jump | `PC = PC + offset` | `jal x0 label` |
| `jal label` | Jump And Link (Pseudo) | `R[ra] = PC + 4`<br/>`PC = PC + offset` | `jal ra label` |
| `jr rs1` | Jump Register | `PC = R[rs1]` | `jalr x0 rs1 0` |
| `ret` | RETurn (`jr ra`) | `PC = R[ra]` | `jalr x0 ra 0` |

:::

Có hai lệnh thực ở trên.

**J**ump **a**nd **L**ink (`jal rd label`). Ghi địa chỉ của **lệnh tiếp theo**, `PC + 4`, vào thanh ghi `rd`. Sau đó thực hiện jump vô điều kiện đến `label` bằng cách đặt `PC` thành địa chỉ của lệnh có nhãn `label`. **Linking** có nghĩa là chúng ta tạo một liên kết có thể được sử dụng để trả về caller. (Về mặt này, `jal` thực sự nên được gọi là "Link and Jump").

* Pseudoinstruction `j label` được sử dụng để triển khai các câu lệnh điều kiện và vòng lặp, như đã thảo luận trong [phần trước](#sec-branches). `jal x0 label` hiệu quả loại bỏ liên kết/địa chỉ trả về, vì thanh ghi `x0` được cố định bằng zero.
* Pseudoinstruction `jal label` được mở rộng thành `jal ra label`, trong đó tên thanh ghi `ra` là **địa chỉ trả về** hoặc thanh ghi số `x1`. Chúng ta thảo luận lý do này bên dưới.

**J**ump **a**nd **L**ink **R**egister (`jalr rd rs1 imm`). Liên kết "địa chỉ trả về" (`PC + 4`) với thanh ghi `rd`. Sau đó thực hiện jump vô điều kiện bằng cách đặt `PC` thành `R[rs1] + imm`.

* Pseudoinstruction `jr rs1` là lệnh `jalr x0 rs1 0`, có nghĩa là chúng ta loại bỏ liên kết và nhảy trực tiếp đến địa chỉ trong thanh ghi `rs1`.
* Pseudoinstruction `ret` là lệnh `jalr x0 ra 0` và tương đương với `jr ra`. Loại bỏ liên kết và nhảy trực tiếp đến địa chỉ trả về trong thanh ghi có tên `ra`.

:::{tip} Kiểm tra nhanh

Tại sao có cả `jal` và `jalr`?

:::

:::{note} Hiển thị giải thích
:class: dropdown

`jal` chỉ định mục tiêu nhảy bằng **nhãn**. `jal` hỗ trợ gọi thủ tục vì có lẽ, bạn nên biết tên của thủ tục bạn đang gọi.

`jalr` chỉ định mục tiêu nhảy bằng **thanh ghi**. `jalr` hỗ trợ trả về thủ tục vì bạn không nhất thiết biết tên của thủ tục đã gọi bạn (cũng không nên mong đợi rằng bạn được gọi ở đầu thủ tục đó); thay vào đó bạn chỉ nên biết địa chỉ để trả về. Chúng ta sẽ thấy trong chương sau cách `jalr` tạo điều kiện cho các jump với *địa chỉ tuyệt đối*.
:::

## Ví dụ hàm `leaf`

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/THhKfRlQTyU
:width: 100%
:title: "[CS61C FA20] Lecture 10.1 - RISC-V Procedures: Function Call Example"
:::
::::

(sec-rv-stack)=
## Stack Frame RISC-V

Trong [phần trước](#sec-example-sp) chúng ta đã thấy cách chúng ta có thể store và load mảng đến và từ stack. **Con trỏ stack** giữ địa chỉ của đỉnh stack. Trong phần này chúng ta thảo luận cách stack frame được cấp phát và giải phóng giữa **gọi hàm**.

Khi chúng ta thảo luận [stack C](#sec-stack), chúng ta đã thấy [một animation](#fig-c-stack-anim) push và pop stack frame giữa các cuộc gọi hàm. Quan trọng:

> Stack phát triển xuống dưới. Con trỏ stack (`sp`) trỏ đến đỉnh stack, tức là địa chỉ của stack frame hiện tại.

Stack frame RISC-V (hầu hết) hoạt động như stack frame C. Theo [quy ước thanh ghi RV32I](#sec-register-conventions), con trỏ stack được lưu trong **thanh ghi `sp`**, là thanh ghi số `x2`.

Một thủ tục RISC-V có thể chọn sử dụng stack frame bằng cách thao tác sp:

* Khi callee giành được điều khiển, trong **prologue**, cấp phát/push stack frame bằng cách **giảm** `sp` (một lần nữa, stack phát triển xuống dưới).

* Khi callee kết thúc trong **epilogue**, giải phóng/pop stack frame bằng cách **tăng** `sp`.

Slidedeck trong @fig-rv-stack-anim animate cấp phát và giải phóng trên stack qua con trỏ stack.

::::{figure}
:label: fig-rv-stack-anim
:alt: "Slide nhúng animate giảm và tăng con trỏ stack RISC-V qua các prologue và epilogue thủ tục lồng nhau trên stack phát triển xuống dưới."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vRmZPYooswNdpDJwrvmnf4LB5h0emERgb162lLWy88ytNPuWI-qcS0X_HiNt5XQgIPvtQ4Ed-6nW2I2/pubembed?start=false&loop=false
:width: 100%
:enumerated: false
:title: "Animation theo dõi qua cấp phát và giải phóng trên stack bộ nhớ sử dụng con trỏ stack, như được chi tiết trong phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1Ns11j8poIPDE7Bwg-qg5Dk6pokKuU-LdP_c8SwWpFTU/edit?usp=sharing)"
:::
Animation mở rộng về quản lý bộ nhớ stack trong RISC-V.
::::

:::{note} Giải thích @fig-rv-stack-anim
:class: dropdown

1. `main` cấp phát 12B không gian cho stack frame của nó bằng cách giảm `sp` đi `12`. Điều này xảy ra trong dòng đầu tiên của `main`, ngụ ý rằng cấp phát stack frame là một phần của prologue của `main`. Từ góc nhìn của `main`, con trỏ stack `sp` giữ `0xFFFFFFD4`, là đáy stack frame của `main`. Tại một thời điểm nào đó, `main` gọi `fooA`.
1. Trong prologue của `fooA`, `fooA` cấp phát 8B không gian stack bằng cách giảm `sp` đi `8`. Từ góc nhìn của `fooA`, con trỏ stack `sp` giữ `0xFFFFFFCC`, là đáy stack frame của `foo`.
1. Trong epilogue của `fooA`, nó giải phóng cùng 8B không gian bằng cách tăng `sp` đi `8`. Trước khi trả về với `jr ra`, `fooA` (như callee) đã khôi phục con trỏ stack của `main` thành `0xFFFFFFD4`.
1. Khi `main` giành lại "điều khiển," nó thấy con trỏ stack đúng, `0xFFFFFFD4`.

Qua các cuộc gọi hàm, con trỏ stack của caller `main` được bảo toàn. Chúng ta thảo luận quy ước gọi thanh ghi này trong [phần khác](#sec-rv-calling-convention).
:::

```{warning} Không phải tất cả thủ tục RISC-V đều cần không gian stack!

Một số thủ tục có footprint lưu trữ cục bộ nhỏ; có lẽ mã C tương đương của chúng chỉ sử dụng vài biến 32-bit. Khi dịch các thủ tục này, chúng ta có thể tránh các phép toán load/store bộ nhớ tốn kém và sử dụng [quy ước thanh ghi](#tab-calling-convention) để giới hạn logic trong các thanh ghi volatile/tạm thời.
```

Giống như trong C, push và pop stack frame đơn giản tương ứng với giảm và tăng con trỏ stack. Dữ liệu của callee trước đó do đó có thể ở lại trong bộ nhớ được đánh dấu là "free" cho callee tiếp theo ghi đè. Tham khảo [thảo luận stack C](#sec-stack) cho các vấn đề bảo mật tiềm ẩn.
