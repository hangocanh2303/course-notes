---
title: "Các Chế Độ Định Địa Chỉ RISC-V"
subtitle: "Định Địa Chỉ Tương Đối PC và Định Địa Chỉ Tuyệt Đối"
---

## Mục Tiêu Học Tập

* So sánh định địa chỉ tương đối PC với định địa chỉ tuyệt đối.
* Cho mã assembly với nhãn, tính toán offset tương đối PC cho các nhánh có điều kiện và nhảy không điều kiện.

Chúng tôi khuyên bạn nên xem lại luồng điều khiển RISC-V trước khi tiếp tục.

* [Một phần](#sec-rv-pc) mô tả cách mặc định, **Bộ đếm chương trình (PC)** được tăng thêm 4 byte, tương ứng với lệnh tuần tự tiếp theo.
* [Phần khác](#sec-branches-jumps) mô tả nhảy không điều kiện (ví dụ: `j Label`) và nhánh có điều kiện (ví dụ: `beq rs1 rs2 Label`).


## Các Chế Độ Định Địa Chỉ

Có các **chế độ định địa chỉ** khác nhau–tức là, các cách sử dụng toán hạng và/hoặc địa chỉ được mã hóa trong lệnh.

Chúng ta đã thấy một chế độ định địa chỉ với [loads và stores](#sec-data-transfer). Các lệnh này sử dụng định địa chỉ cơ sở hoặc dịch chuyển, tính toán địa chỉ như tổng của một thanh ghi trong tệp thanh ghi (`x0-31`) và một hằng số immediate được mã hóa trong lệnh.

RISC-V sử dụng hai chế độ định địa chỉ để tính toán địa chỉ _lệnh_ để cập nhật PC:

* **Định địa chỉ tương đối PC**, tính toán địa chỉ bằng cách cộng PC với offset hằng số có dấu, ví dụ: `PC = PC + offset`
* **Định địa chỉ tuyệt đối**, tính toán địa chỉ từ thanh ghi, ví dụ: `PC = R[rs1] + imm`.

### Định Địa Chỉ Tương Đối PC

Trong hầu hết các trường hợp, các lệnh cập nhật PC sử dụng **định địa chỉ tương đối PC**.

* Lệnh số học, loads, và stores: `offset` là $+4$
* Nhánh
* Lệnh J-Type (xem [phần sau](#sec-j-type-vi))
* (Thực tế, tất cả các lệnh trừ `jalr`)

Tại sao? **Mã Độc Lập Vị Trí**. Nếu toàn bộ khối mã được di chuyển, các offset tương đối trong các lệnh sẽ không thay đổi!

:::{hint} Từ Nhãn đến Offset Tương Đối PC

Nhớ rằng trong assembly, nhảy không điều kiện và nhánh có điều kiện sử dụng nhãn, ví dụ: `j Label` và `beq rs1 rs2 Label`. [Nhãn](#sec-labels) không phải là lệnh và **không thực sự "tồn tại"** trong mã máy.

Xây dựng lệnh máy do đó yêu cầu chuyển đổi nhãn thành các hằng số số có thể được sử dụng cho định địa chỉ. Tất cả các lệnh nhánh và nhảy sử dụng nhãn đều dùng **định địa chỉ tương đối PC**.

Để chuyển đổi các lệnh nhánh và nhảy sang mã máy, chúng ta phải tính toán **offset tương đối PC**, là các hằng số số.

:::

:::{note} Ví Dụ

```{code} bash
:linenos:
Loop:
    beq  x19 x10 End
    add  x18 x18 x10
    addi x19 x19  -1
    j    Loop
End:
    # lệnh đích
```

Xem xét mã assembly ở trên. Trong mỗi trường hợp sau, offset tương đối PC là gì?

1. `beq x19 x10 End`, nếu nhánh không được thực hiện
1. `beq x19 x10 End`, nếu nhánh được thực hiện
1. `j Loop`
:::

**1.** `beq x19 x10 End`, **nếu nhánh không được thực hiện.** Trong trường hợp này, PC cập nhật đến lệnh tuần tự tiếp theo. Offset tương đối PC là **$+4$**.

**2.** `beq x19 x10 End`, **nếu nhánh được thực hiện**. Trong trường hợp này, PC cập nhật đến lệnh được gắn nhãn `End`. Xem xét @fig-offsets, gán địa chỉ mẫu cho mỗi lệnh trong assembly ở trên. Ở đây, `pc` cập nhật từ `beq` (tại địa chỉ `0x0C`) đến lệnh của `End` (tại địa chỉ `0x1C`). Hiệu số này là `0x10`, hoặc **+16**. Điều này tương ứng với lệnh thứ tư _sau_ `beq`.

:::{figure} images/offsets.png
:label: fig-offsets-vi
:width: 80%
:align: center
:alt: "Assembly tại địa chỉ bộ nhớ 0x0c–0x1c: Loop beq x19 x10 End, sau đó các lệnh add và addi, lệnh j Loop không điều kiện, và nhãn End cho lệnh đích; các mũi tên cong cho thấy nhánh tiến từ beq đến End và nhảy lùi từ j Loop đến Loop."

Ví dụ mã minh họa với thao tác nhảy.
:::

**3.** `j Loop`. Trong trường hợp này, PC cập nhật đến lệnh được gắn nhãn `Loop` (ở đây là `beq`). Vẫn xem xét @fig-offsets, `pc` cập nhật từ `j` (tại địa chỉ `0x18`) đến lệnh của `Loop` (tại địa chỉ `0x0C`). Hiệu số này là **$-12$**. Điều này tương ứng với ba lệnh _trước_ `j`.

(sec-absolute-addressing-vi)=
### Định Địa Chỉ Tuyệt Đối

Ngược lại, **định địa chỉ tuyệt đối** cung cấp địa chỉ mới để ghi đè PC. Chế độ định địa chỉ này **phụ thuộc vị trí** và dễ bị ảnh hưởng khi di chuyển mã (như chúng ta sẽ thấy sau).

Chỉ `jalr` (một [lệnh I-Type](#sec-jalr-itype)) sử dụng định địa chỉ tuyệt đối bằng cách đặt `PC = R[rs1] + imm`. Làm như vậy thường liên quan đến việc xây dựng immediate 32 bit, điều này có thể thực hiện được bằng định dạng lệnh [U-Type](#sec-u-type-vi) mà chúng ta thảo luận trong chương này.
