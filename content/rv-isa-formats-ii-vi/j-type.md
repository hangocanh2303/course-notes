---
title: "J-Type"
subtitle: "TODO. Ghi chú này chưa hoàn chỉnh; xem bài giảng Spring 2026"
---

(sec-j-type-vi)=
## Mục Tiêu Học Tập

* Chuyển đổi qua lại giữa lệnh assembly J-Type và lệnh máy.
* Xác định trường hợp sử dụng cho các lệnh nhảy không điều kiện và giả lệnh khác nhau: `j`, `jr`, `jal`, `jalr`, `ret`.
* Sử dụng lệnh J-Type cho nhánh có điều kiện.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/hkVUmw460Kw
:width: 100%
:title: "[CS61C FA20] Lecture 12.3 - RISC-V Instruction Formats II: J-Format"
:::

::::

## Các Lệnh Nhảy

:::{warning} Phần ghi chú này chưa được viết, nhưng nội dung vẫn nằm trong phạm vi.

Đối với phần này, vui lòng tham khảo [slide bài giảng Spring 2026](https://docs.google.com/presentation/d/18YSyN37XjHEjfkWigPHJWXNJB17AsSiAKW8f9XebE-8/edit?usp=sharing), phần "Jump Instructions."

Ngoài ra, xem `jalr` được đề cập trong [I-Type](#sec-jalr-itype).
:::

Phần còn lại của trang này bao gồm nội dung trong phạm vi không được đề cập trong bài giảng.

## Nhánh Có Điều Kiện Đến Xa

Nhánh có điều kiện thường được sử dụng cho câu lệnh if-else, vòng lặp for/while. Nói chung, vì các cấu trúc điều khiển này thường _khá nhỏ_ (<50 dòng mã C), chúng ta có thể sử dụng **lệnh B-Type**. Tuy nhiên, như chúng ta đã thấy trước đó, lệnh B-Type có phạm vi giới hạn: $\pm 2^{10}$ lệnh từ lệnh hiện tại (PC).

Để nhảy xa hơn nữa, chúng ta có thể sử dụng **nhảy không điều kiện J-Type** kết hợp với lệnh B-Type.

:::{note} `beq x10 x10 far`

Để nhánh đến vị trí xa, ví dụ: `beq x10 x10 far`, chúng ta có thể tương đương chỉ định assembly với thêm một lệnh:

```{code} bash
    bne  x10 x0 next
    j far
next:
    # lệnh tiếp
```
:::

Thừa nhận rằng, lệnh J-Type cũng có phạm vi giới hạn: $\pm 2^{18}$ lệnh từ lệnh hiện tại (PC).

Nếu chúng ta muốn nhảy đến **bất kỳ** địa chỉ nào, RISC-V chọn nhảy với [định địa chỉ tuyệt đối](#sec-absolute-addressing-vi) và `jalr`. Như đã thảo luận trong [phần trước](#sec-jalr-itype), `jalr` là lệnh I-Type đặt PC thành `PC = R[rs1] + imm`, trong đó `imm` chỉ định immediate 12 bit `imm`.

* Để gọi hàm với định địa chỉ tuyệt đối, thay vì `jal ra Label`[^simultaneous]:

    ```{code} bash
    lui  ra <hi20bits*>
    jalr ra ra <lo12bits>
    ```

[^simultaneous]: Lệnh `jalr` sẽ lưu địa chỉ nhảy vào PC và lưu `PC + 4` hiện tại vào `ra` trong cùng chu kỳ (thực tế là "đồng thời")

* Để thoát vòng lặp sử dụng định địa chỉ tuyệt đối, thay vì `j Label` (ví dụ: `jal x0 Label`):

    ```{code} bash
    auipc ra <hi20bits*>
    jalr  x0 ra <lo12bits>
    ```

Xem thảo luận về [lệnh U-Type](#sec-u-type-vi) `lui` và `auipc`. Giống như với việc giải quyết [giả lệnh `li`](#sec-li-lui-vi), `jalr` mở rộng dấu immediate. Vì vậy nếu `lo12bits` có bit dấu được đặt, tăng `hi20bits` thêm 1.

<!--

## Hình Ảnh



:::{figure} images/jal-isa.png
:label: fig-jal-isa-vi
:width: 100%
:alt: "Định dạng lệnh jal J-type: cú pháp jal rd Label với bit 31–12 chứa immediate 20 bit xáo trộn, rd ở bit 11–7 chứa thanh ghi đích, và opcode là 0b1101111 ở bit 6–0 cho lệnh jal; văn bản giải thích offset byte có dấu 21 bit tương đối PC với lsb ngầm định là zero và thanh ghi đích rd được nạp với PC cộng bốn."

Định dạng lệnh jal.
:::

|   |   | opcode |   |
| :-- | -- | -- | --: |
| `imm[20\|10:1\|11\|19:12]` | `rd` | `1101111` | `jal` |

---
title: "Một Chút Về Chương Trình Máy"
subtitle: TODO
---

## Mục Tiêu Học Tập

* TODO
* TODO


TODO: định nghĩa nhánh không điều kiện -- bao gồm sau khi chúng ta thảo luận về định dạng lệnh

Bạn có thể tự hỏi liệu bạn có thể tạo nhánh không điều kiện sử dụng nhánh có điều kiện, như beq x0, x0, label. Mặc dù điều đó sẽ luôn nhảy, có một vấn đề: phạm vi của nhánh ngắn hơn. Vì RISC-V sử dụng lệnh 32 bit, chúng ta phải vừa loại lệnh, các thanh ghi được so sánh, và nhãn (giá trị immediate) vào 32 bit đó. Lệnh nhảy chuyên dụng không cần chỉ định thanh ghi để so sánh, vì vậy giá trị immediate của nó có thể lớn hơn, cho phép nó đến xa hơn trong chương trình.

-->
