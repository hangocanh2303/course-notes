---
title: "Gọi thủ tục"
---

(sec-rv-procedure-calls)=
## Mục tiêu học tập

* Nhớ sáu bước cơ bản để gọi thủ tục.
* Phân biệt giữa caller và callee.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/XZAHwb7Smj0
:width: 100%
:title: "[CS61C FA20] Lecture 09.3 - RISC-V Decisions II: RISC-V Function Calls"
:::

Đến 9:10

::::

Để hoàn tất thảo luận về RISC-V, hãy thảo luận cách RISC-V triển khai **gọi thủ tục**[^procedures].

[^procedures]: Thuật ngữ: Hướng dẫn RISC-V đề cập đến gọi và trả về **thủ tục**, trong khi C đề cập đến gọi và trả về **hàm**. Để biết thêm thông tin, xem [trang Wikipedia này](https://en.wikipedia.org/wiki/Function_(computer_programming)) và [trang wikibooks C này](https://en.wikibooks.org/wiki/C_Programming/Procedures_and_functions).


Đầu tiên chúng ta phác thảo các bước cơ bản của gọi thủ tục trong phần này. Sau đó chúng ta xem lại một vài chủ đề chi tiết:

| Chủ đề | Các chương trước | Chương này |
| :-- | :--: | :--: |
| Jump vô điều kiện | [Sử dụng trong branch](#sec-branches) | [Các lệnh jump chung](#sec-jumps) |
| Stack Frame | [Mảng trên stack](#sec-example-sp) | [Push/pop stack frame](#sec-rv-stack) |
| Quy ước thanh ghi | [Tên thanh ghi](#sec-register-names), <br/>[Quy ước thanh ghi](#sec-register-conventions) | [Calling convention](#sec-rv-calling-convention-top) |

Cuối cùng, chúng ta xem một số ví dụ về thủ tục đệ quy trong RISC-V.

## Gọi hàm trong C

Chúng ta bắt đầu bằng việc phát triển trực giác về cách **trạng thái** thay đổi trong một cuộc gọi hàm trong C.
Xét mã C hơi ngớ ngẩn này. Compiler hoặc lập trình viên cần theo dõi thông tin gì để thực hiện hai cuộc gọi `mult` trong `main`?

```{code} c
:linenos:
int main() {
  int j = ...;
  int k = ...;

  int i = mult(j, k); // cuộc gọi đầu tiên đến mult
  int m = mult(i, i); // cuộc gọi thứ hai đến mult
}

/* hàm mult rất ngớ ngẩn với tham số,
   biến cục bộ, và giá trị trả về */
int mult(int mcand, int mlier) {
  int product = 0;
  while (mlier > 0) {
    product = product + mcand;
    mlier = mlier - 1;
  }
  return product;
}
```

Một số ghi chú:

* Tham số: `j` và `k` cần được sao chép như đối số cho `mult`
* `mult` có thể truy cập các biến cục bộ của chính nó (ví dụ: `product`)
* `mult` nên có khả năng thực thi các lệnh
* Các lệnh `mult` nằm ở đâu đó, và các lệnh `main` ở nơi khác
* Sau khi `mult` trả về `main`, các biến cục bộ của `main` (ví dụ: `j` và `k`) vẫn nên ở đó, không bị động

### Phép tương tự trông nhà

Nhớ lại rằng trong bố cục máy tính của chúng ta, chỉ có **một máy tính** với một tập hợp cố định các thanh ghi. Do đó, gọi thủ tục liên quan đến "chia sẻ" không gian tính toán và lưu trữ. Khi chúng ta thực hiện một cuộc gọi thủ tục, chúng ta cần theo dõi **dữ liệu** chương trình (ví dụ: biến cục bộ nào thuộc về ai), **điều khiển** thủ tục (ví dụ: thủ tục nào đang thực thi và thủ tục nào đang chờ), và **truyền tham số**.

Gọi thủ tục giống như trông nhà. Hãy tưởng tượng bạn có cha mẹ[^out-of-state] nhờ ("gọi") bạn trông nhà một đêm.

[^out-of-state]: Hoặc bất kỳ ai, thực sự. Hãy tưởng tượng phép tương tự này áp dụng cho bất kỳ nhóm người nào gọi bạn làm điều gì đó cho họ, và bạn phải sử dụng tài sản của họ để hoàn thành nhiệm vụ. Giả sử có bồi thường hợp lý. :-)

1. **Thiết lập.** Trước khi bạn đến, cha mẹ bạn chuẩn bị một số vật dụng cần thiết mà họ biết bạn sẽ cần; họ đặt những thứ này trên bàn.
1. **Chuyển điều khiển.** Cha mẹ bạn giấu chìa khóa. Họ gọi bạn đến và rời đi; họ cũng cho bạn biết họ giấu chìa khóa ở đâu (dưới chậu cây.)
1. **Prologue.** Điều đầu tiên bạn làm (sau khi mở khóa cửa) là tạo không gian cho đồ của bạn. Cha mẹ bạn để lại một số thứ trên bàn không phải cho bạn; bạn đặt những thứ này trong tủ. Bạn biết bạn sẽ sử dụng thêm một số thứ không vừa trên bàn, nên bạn tạo thêm không gian trong tủ hành lang.
1. **Thực hiện nhiệm vụ mong muốn.** Bạn trông con chó (hoặc mèo). Ngủ qua đêm. Ăn tối. Làm bài tập. Xem TV. Lướt internet.
1. **Epilogue.** Để dọn dẹp, bạn đặt lại đồ của cha mẹ từ tủ lên bàn. Bạn khôi phục nhà về trạng thái ban đầu (bao gồm cả tủ). Bạn để lại cho họ một món quà đẹp, cũng trên bàn.
1. **Epilogue: Trả điều khiển về điểm gốc.** Bạn đặt chìa khóa trở lại vị trí đã thỏa thuận (dưới chậu cây). Sau khi bạn rời đi, bạn gọi cha mẹ bảo họ quay lại.

Trong phép tương tự này, nhà của cha mẹ bạn là máy tính. Thanh ghi là bàn. Tủ/rác/kho nhà là bộ nhớ. Các vật dụng cần thiết cha mẹ bạn chuẩn bị cho bạn là các đối số. Món quà đẹp bạn để lại là giá trị trả về. Cha mẹ bạn là thủ tục **caller**, và bạn là thủ tục **callee**.

### Các bước cơ bản

:::{hint} Gọi thủ tục thường liên quan đến hai bên:

* **CalleR**: thủ tục gọi
* **CalleE**: thủ tục được gọi

:::

(sec-rv-procedure-call-steps)=
:::{note} Sáu bước cơ bản để gọi thủ tục

1. [Caller] **Thiết lập đối số**. Đặt đối số vào thanh ghi.
1. [Caller] **Chuyển điều khiển đến thủ tục**. Sử dụng lệnh `jal` (jump-and-link): `jal ra fnLabel`
1. [Callee] **Prologue**. Thu nhận tài nguyên lưu trữ (cục bộ): không gian stack (ví dụ: push một frame lên stack), lưu giá trị thanh ghi, v.v.
1. [Callee] **Thực hiện nhiệm vụ mong muốn**.
1. [Callee] **Epilogue**. Đặt giá trị trả về ở nơi caller có thể truy cập, khôi phục giá trị thanh ghi, và giải phóng lưu trữ cục bộ trên stack (ví dụ: pop frame khỏi stack).
1. [Callee] **Epilogue: Trả điều khiển về điểm gốc**. Sử dụng lệnh `jr`: `jr ra`

:::

Chúng ta xem lại sáu bước cơ bản này trong [phần sau](#sec-rv-procedure-call-steps-revisited).
