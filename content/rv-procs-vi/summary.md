---
title: "Tóm tắt"
---

## Và để kết luận$\dots$

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/E2QbJ3pOnts
:width: 100%
:title: "[CS61C FA20] Lecture 10.4 - RISC-V Procedures: Summary"
:::

::::

<table border="1" style="border-collapse: collapse; width: 100%;">
  <thead>
    <tr>
      <th style="padding: 10px; text-align: left;">Danh mục</th>
      <th style="padding: 10px; text-align: left;">Các lệnh</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding: 10px; vertical-align: middle;">Số học</td>
      <td style="padding: 10px;">
        <ul style="margin: 0;">
          <li><code>add</code></li>
          <li><code>sub</code></li>
          <li><code>and</code></li>
          <li><code>or</code></li>
          <li><code>xor</code></li>
          <li><code>sll</code></li>
          <li><code>srl</code></li>
          <li><code>sra</code></li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="padding: 10px; vertical-align: middle;">Immediate</td>
      <td style="padding: 10px;">
        <ul style="margin: 0;">
          <li><code>addi</code></li>
          <li><code>andi</code></li>
          <li><code>ori</code></li>
          <li><code>xori</code></li>
          <li><code>slli</code></li>
          <li><code>srli</code></li>
          <li><code>srai</code></li>
          <li><code>li</code> (pseudo)</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="padding: 10px; vertical-align: middle;">Load/Store</td>
      <td style="padding: 10px;">
        <ul style="margin: 0;">
          <li><code>lw</code></li>
          <li><code>lb</code></li>
          <li><code>lbu</code></li>
          <li><code>sw</code></li>
          <li><code>sb</code></li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="padding: 10px; vertical-align: middle;">Branch/Jump</td>
      <td style="padding: 10px;">
        <ul style="margin: 0;">
          <li><code>beq</code></li>
          <li><code>bne</code></li>
          <li><code>bge</code></li>
          <li><code>blt</code></li>
          <li><code>bgeu</code></li>
          <li><code>bltu</code></li>
          <li><code>j</code> (pseudo)</li>
          <li><code>jalr</code></li>
          <li><code>jal</code></li>
          <li><code>jr</code> (pseudo)</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

### Calling Convention

Hãy ôn lại ý nghĩa đặc biệt chúng ta gán cho mỗi loại thanh ghi trong RISC-V.

| **Thanh ghi** | **Quy ước** | **Người lưu** |
|---|----|-----|
| `x0` | Lưu **zero** | N/A |
| `sp` | Lưu **con trỏ stack** | Callee |
| `ra` | Lưu **địa chỉ trả về** | Caller |
| `a0` - `a7` | Lưu **đối số** và **giá trị trả về** | Caller |
| `t0` - `t6` | Lưu giá trị **tạm thời** *không tồn tại* sau gọi hàm | Caller |
| `s0` - `s11` | Lưu giá trị **saved** *tồn tại* sau gọi hàm | Callee |

Để lưu và gọi lại giá trị trong thanh ghi, chúng ta sử dụng lệnh `sw` và `lw` để lưu và load từ
đến và từ bộ nhớ, và chúng ta thường tổ chức hàm như sau:

```
# Prologue
addi sp, sp, -8 # Chỗ cho hai thanh ghi. (Tại sao?)
sw s0, 0(sp) # Lưu s0 (hoặc bất kỳ thanh ghi saved nào)
sw s1, 4(sp) # Lưu s1 (hoặc bất kỳ thanh ghi saved nào)

# Mã được bỏ qua

# Epilogue
lw s0, 0(sp) # Load s0 (hoặc bất kỳ thanh ghi saved nào)
lw s1, 4(sp) # Load s1 (hoặc bất kỳ thanh ghi saved nào)
addi sp, sp, 8 # Khôi phục con trỏ stack
```
### Ví dụ Calling Convention trong mã
Dưới đây là ví dụ về calling convention trong hàm RISC-V.

Các thanh ghi callee-saved (như `s0`) được lưu ở đầu hàm và khôi phục trước khi trả về, vì các thanh ghi này phải được bảo toàn bởi hàm.

Các thanh ghi caller-saved (như `t1` và `ra`) được caller lưu trước khi gọi hàm khác, vì callee có thể sửa đổi các thanh ghi này. **Lưu ý**: Mặc dù `ra` là thanh ghi caller-saved, nó thường được lưu ở đầu và cuối hàm theo quy ước, như được hiển thị bên dưới.
```
func_a:

 # Prologue: Lưu thanh ghi callee-saved & địa chỉ trả về
 addi sp, sp, -8 # Cấp phát không gian stack
 sw ra, 0(sp) # Lưu địa chỉ trả về
 sw s0, 4(sp) # Lưu s0

 addi t1, x0, 10 # Sửa đổi t1
 addi s0, x0, 20 # Sửa đổi s0

 # Lưu thanh ghi caller-saved trước gọi hàm
 addi sp, sp, -4 # Cấp phát thêm không gian stack
 sw t1, 0(sp) # Lưu t1 (thanh ghi caller-saved)

 jal func_b # Gọi hàm khác

 # Khôi phục thanh ghi caller-saved sau gọi hàm
 lw t1, 0(sp) # Khôi phục t1 (thanh ghi caller-saved)
 addi sp, sp, 4 # Giải phóng không gian cho thanh ghi caller-saved
 addi t1, t1, 5 # Sửa đổi t1
 addi s0, s0, 5 # Sửa đổi s0

 # Epilogue: Khôi phục thanh ghi callee-saved
 lw ra, 0(sp) # Khôi phục địa chỉ trả về
 lw s0, 4(sp) # Khôi phục s0
 addi sp, sp, 8 # Giải phóng không gian stack

 ret # Trả về từ func_a
 ```

## Bài đọc giáo trình

P&H 2.8

<!-- ## Tài liệu tham khảo bổ sung -->

## Bài tập
Kiểm tra kiến thức của bạn!

### Ôn tập khái niệm

:::{exercise}
:label: procs-01
1. Sau khi gọi hàm và hàm trả về, các thanh ghi `t` có thể đã bị thay đổi trong quá trình thực thi hàm, trong khi các thanh ghi a không thể.
:::

:::{solution} procs-01
:label: procs-01-sol
:class: dropdown

**Sai.** Thanh ghi `a0` và `a1` thường được sử dụng để lưu giá trị trả về từ hàm, nên hàm có thể đặt giá trị của chúng thành giá trị trả về trước khi trả về.

:::

:::{exercise}
:label: procs-02
2. Để sử dụng thanh ghi saved (`s0`-`s11`) trong hàm, chúng ta phải lưu giá trị của chúng trước khi sử dụng và khôi phục giá trị trước khi trả về.
:::

:::{solution} procs-02
:label: procs-02-sol
:class: dropdown

**Đúng.** Thanh ghi saved là callee-saved, nên chúng ta phải lưu và khôi phục chúng ở đầu và cuối hàm. Điều này thường được thực hiện trong các khối mã có tổ chức gọi là "function prologue" và "function epilogue."

:::

:::{exercise}
:label: procs-03
3. Stack chỉ nên được thao tác ở đầu và cuối hàm, nơi thanh ghi callee-saved được lưu tạm thời.
:::

:::{solution} procs-03
:label: procs-03-sol
:class: dropdown

**Sai.** Mặc dù việc tạo 'prologue' và 'epilogue' riêng biệt để lưu thanh ghi callee lên stack là ý tưởng tốt, stack có thể thay đổi ở bất kỳ đâu trong hàm. Một ví dụ tốt là nếu bạn muốn bảo toàn giá trị hiện tại của thanh ghi tạm thời, bạn có thể giảm sp để lưu thanh ghi lên stack ngay trước cuộc gọi hàm.

:::
