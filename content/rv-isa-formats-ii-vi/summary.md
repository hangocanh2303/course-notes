---
title: "Tổng Kết"
---

## Và Kết Luận$\dots$

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/Ntp8UOhJleU
:width: 100%
:title: "[CS61C FA20] Lecture 12.4 - RISC-V Instruction Formats II: Summary"
:::

::::

### Chuyển Đổi Lệnh

Nhớ rằng mọi lệnh trong RISC-V có thể được biểu diễn như một giá trị nhị phân 32 bit, mã hóa loại lệnh, cũng như bất kỳ thanh ghi/immediate nào có trong lệnh. Để chuyển đổi lệnh RISC-V sang nhị phân, và ngược lại, bạn có thể sử dụng các bước dưới đây. Bảng tham chiếu 61C sẽ rất hữu ích cho việc chuyển đổi!

**RISC-V ⇒ Nhị Phân**
1. Xác định loại lệnh (R, I, I*, S, B, U, hoặc J)
2. Tìm định dạng lệnh tương ứng
3. Chuyển đổi các thanh ghi và giá trị immediate, nếu có, sang nhị phân
4. Sắp xếp các bit nhị phân theo định dạng lệnh, bao gồm các bit opcode (và có thể bit funct3/funct7)

**Nhị Phân ⇒ RISC-V**
1. Xác định lệnh sử dụng các bit opcode (và có thể funct3/funct7)
2. Chia biểu diễn nhị phân thành các phần dựa trên định dạng lệnh
3. Chuyển đổi thanh ghi + giá trị immediate
4. Ghép lệnh cuối cùng dựa trên loại/định dạng lệnh

Dưới đây là ví dụ về một chuỗi lệnh RISC-V với các bản dịch nhị phân tương ứng.

| `example.S` | `example.bin` |
| :--- | :--- |
| `main:`| (N/A) |
| `addi sp,sp,-4` | `11111111110000010000000100010011` |
| `sw   ra 0(sp)` | `00000000000100010010000000100011` |
| `addi s0 sp 4`  | `00000000010000010000010000010011` |
| `mv   a0 a5`    | `00000000000001111000010100010011` |
| `call printf`   | `00000000010001000000000011101111` |
| `...` | (bỏ qua)

## Đọc Thêm Từ Sách Giáo Khoa

P&H 2.5, 2.10

<!-- ## Tài Liệu Tham Khảo Bổ Sung -->

## Bài Tập
Kiểm tra kiến thức của bạn!

### Ôn Tập Khái Niệm

:::{exercise}
:label: isa-01-vi
1. **Đúng hay Sai**: Trong RISC-V, trường opcode của lệnh xác định loại của nó (R-Type, S-Type, v.v.).
:::

:::{solution} isa-01-vi
:label: isa-01-sol-vi
:class: dropdown

**Đúng.** Trường opcode của lệnh xác định duy nhất loại lệnh và cho phép chúng ta xác định định dạng lệnh mà chúng ta đang làm việc. Opcode nằm ở 7 bit thấp nhất của lệnh máy (bit 0-6).
:::

:::{exercise}
:label: isa-02-vi
2. **Đúng hay Sai**: Trong RISC-V, lệnh li x5 0x44331416 sẽ luôn được mã hóa trong 32 bit khi được chuyển đổi sang nhị phân.
:::

:::{solution} isa-02-vi
:label: isa-02-sol-vi
:class: dropdown

**Sai.** Đây là câu hỏi mẹo. Đúng là mọi lệnh thông thường trong RISC-V sẽ luôn được mã hóa trong 32 bit. Tuy nhiên, `li` thực sự là giả lệnh! Nhớ rằng giả lệnh có thể chuyển đổi thành một hoặc nhiều lệnh RISC-V. Trong trường hợp này, li sẽ được chuyển đổi thành lệnh `addi` và `lui`. Do đó, `li x5 0x44331416` thực sự sẽ được mã hóa trong 64 bit, vì nó đại diện cho hai lệnh RISC-V.
:::

:::{exercise}
:label: isa-03-vi
3. **Đúng hay Sai**: Chúng ta có thể sử dụng lệnh nhánh để di chuyển PC một byte.
:::

:::{solution} isa-03-vi
:label: isa-03-sol-vi
:class: dropdown

**Sai.** Offset lệnh nhánh có số không ngầm định là bit ít quan trọng nhất, vì vậy chúng ta chỉ có thể di chuyển PC theo offset chia hết cho 2 (tham khảo lại [phần này](#sec-j-type-vi) để giải thích tại sao!). Offset đầy đủ cho lệnh nhánh sẽ là offset 13 bit `{imm[12:1], 0}`, trong đó chúng ta lấy các bit immediate từ mã hóa nhị phân của lệnh và thêm số không ngầm định.
:::

### Bài Tập Ngắn

:::{exercise}
:label: isa-04-vi
1. Chuyển đổi các thanh ghi RISC-V sau sang biểu diễn nhị phân:
* `s0`
* `sp`
* `x9`
* `t4`
:::

:::{solution} isa-04-vi
:label: isa-04-sol-vi
:class: dropdown
Chú ý rằng vì chúng ta có 32 thanh ghi khác nhau trong RISC-V, chúng ta cần 5 bit để mã hóa chúng.
Nhìn vào bảng tham chiếu 61C, chúng ta có thể thấy `s0` tham chiếu đến thanh ghi `x8`. Để có đáp án cuối cùng, chúng ta chuyển 8 sang nhị phân: `0b01000`. Làm theo quy trình tương tự như trên, chúng ta có phần còn lại của đáp án...
* `s0`: `x8` = `0b01000`
* `sp`: `x2` = `0b00010`
* `x9`: `x9` = `0b01001`
* `t4`: `x29` = `0b11101`
:::
