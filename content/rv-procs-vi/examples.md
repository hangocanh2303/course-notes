---
title: "Gọi thủ tục đệ quy"
---


## Mục tiêu học tập

* Xác định prologue và epilogue của một thủ tục.
* Thực hành gọi thủ tục, sử dụng thủ tục đệ quy làm ví dụ.
* Xác định các trường hợp sử dụng cho lệnh và pseudoinstruction jump vô điều kiện–đặc biệt, biết cách nhảy đến thủ tục và trả về từ thủ tục.

Chúng tôi khuyến nghị mở [phần quy ước thanh ghi](#tab-calling-convention) của [Green Card RISC-V](#sec-green-card). Chúng tôi cũng khuyến nghị tham chiếu [Các bước cơ bản](#sec-rv-procedure-call-steps) ([phiên bản xem lại](#sec-rv-procedure-call-steps-revisited)) của gọi thủ tục.


```{embed} #sec-rv-procedure-call-steps-revisited
```

## Ví dụ đệ quy: `factorial`

Xét triển khai hợp ngữ bên dưới của `factorial`. Chúng ta đã bỏ qua một số dòng để đơn giản; xem mã đầy đủ ở cuối phần này.

(code-factorial-recursive)=
```{code} bash
:linenos:
main:
  li a0 3
  jal ra factorial
  …
factorial:
  addi sp sp -8
  sw ra 0(sp)
  sw s0 4(sp)
  mv s0 a0
  li t0 1
  bne s0 t0 recurse
  li a0 1
  j epilogue
recurse:
  addi a0 s0 -1
  jal ra factorial
  mul a0 s0 a0
epilogue:
  lw ra 0(sp)
  lw s0 4(sp)
  addi sp sp 8
  jr ra
```

:::{hint} Kiểm tra nhanh
Các dòng nào tương ứng với prologue và epilogue của thủ tục `factorial`?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Bắt đầu với **epilogue**, được gắn nhãn (nhãn `epilogue` trên dòng 18 gắn với lệnh `lw` trên dòng 19). Các lệnh trên dòng 19, 20, và 21 load giá trị trở lại thanh ghi saved `s0` và địa chỉ trả về `ra`, sau đó tăng con trỏ stack `sp`. Cuối cùng, dòng 22 `jr ra` trả về hàm caller.

Nếu epilogue là "dọn dẹp," **prologue** là "thiết lập." Chúng ta có thể thấy rằng các lệnh trên dòng 6, 7, và 8 giảm con trỏ stack `sp` và store các thanh ghi **callee-saved** `s0` và `ra` lên stack, được sử dụng trong chính thủ tục.
:::

### Câu hỏi thảo luận

[Di chuột ở đây](#code-factorial-recursive) để xem hợp ngữ RISC-V.

:::{note} Mở rộng để xem mã C tương đương
```{code} c
:linenos:
// giả sử số dương
int factorial(int n) {
  if (n == 1) return 1;
  int a = n * factorial(n-1);
  return a;
}
```
:::

:::{hint} Prologue

1. Stack frame lớn bao nhiêu?
1. Thanh ghi nào được lưu lên stack?
:::

:::{note} Hiển thị đáp án
:class: dropdown

1. 8B
1. `ra` của chính chúng ta, vì chúng ta (có thể) thực hiện cuộc gọi hàm đệ quy. `s0` của caller của chúng ta
:::

:::{hint} Cuộc gọi đệ quy
[Dòng 16](#code-factorial-recursive): Như caller, tại sao chúng ta lưu `s0` trước khi thực hiện cuộc gọi đệ quy?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Factorial đệ quy (cho số dương) tính $\text{factorial}(n) = n! = n \cdot (n - 1)!$, cho $n > 1$. $\text{factorial}(1) = 1! = 1.$

[Calling convention](#sec-rv-calling-convention) có nghĩa là các thanh ghi saved như `s0` được bảo toàn qua cuộc gọi đệ quy, nên sao chép đối số `a0` ($n$) vào thanh ghi saved `s0` để chúng ta vẫn có thể sử dụng giá trị này sau khi cuộc gọi đệ quy $(n-1)!$ hoàn thành.

:::

:::{hint} Thanh ghi tạm thời
[Dòng 10-11](#code-factorial-recursive): Tại sao chúng ta sử dụng thanh ghi tạm thời `t0`?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Lệnh branch `bne` so sánh hai thanh ghi. Lệnh này kiểm tra trường hợp cơ sở của chúng ta, (theo Dòng 10) là `1`. Do đó chúng ta load immediate `1` vào thanh ghi `t0`.[^a0-s0] [^base-case-zero]

[^a0-s0]: Với lệnh `mv s0 a0`, chúng ta có thể thay thế lệnh branch `bne s0 t0 recurse` bằng `bne a0 t0 recurse`. Mã có lẽ sử dụng cái trước để phân biệt quy ước đặt tên thanh ghi cho thanh ghi `a0`, vừa là đối số hàm đầu tiên vừa là giá trị trả về. Thanh ghi `s0` được chỉ định là giá trị của $n$ trong $\text{factorial}(n)$, và thanh ghi `a0` sẽ sớm trở thành giá trị trả về (theo dòng 12).
[^base-case-zero]: Factorial được [định nghĩa toán học](https://en.wikipedia.org/wiki/Factorial) trên tất cả số không âm, bao gồm $0!= 1$. Mã bài giảng chọn một cách sư phạm để bỏ qua trường hợp zero để cho bạn thấy `bne` với hai giá trị thanh ghi khác không.
:::


:::{hint} Epilogue
[Dòng 18-22](#code-factorial-recursive) Chúng ta khôi phục thanh ghi trước hay pop stack frame trước?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Trước tiên khôi phục thanh ghi, mà giá trị của chúng được lưu trên stack. Sau đó, pop stack frame, ngay trước khi trả về caller.
:::

:::{hint} [Dòng 13](#code-factorial-recursive)
Tại sao `j epilogue`? Tại sao không phải `jr ra`?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Khi trả về, **callee** luôn chịu trách nhiệm bảo toàn calling convention: khôi phục thanh ghi saved và giải phóng stack frame (ví dụ: khôi phục `sp`). Mặc dù Dòng 12 và 13 ngụ ý callee đã đạt đến trường hợp cơ sở, trường hợp cơ sở cũng phải hoàn thành epilogue trước khi trả về caller.

:::

### Chạy Demo

Mã bên dưới để tham khảo.

:::{note} Mở rộng để xem `recursive.s`
:class: dropdown

```bash
#### Factorial đệ quy
.globl factorial

.text
main:
    li a0 4
    jal ra, factorial

    addi a1, a0, 0
    addi a0, x0, 1
    ecall # In kết quả

    addi a1, x0, '\n'
    addi a0, x0, 11
    ecall # In dòng mới

    addi a0, x0, 10
    ecall # Thoát

# factorial nhận một đối số:
# a0 chứa số mà chúng ta muốn tính factorial
# Giá trị trả về nên được lưu trong a0

factorial:
    # Prologue
    addi sp sp -8
    sw ra 0(sp)
    sw s0 4(sp)
    
    # Body
    mv s0 a0
    li t0 1
    
    bne s0 t0 recurse
    
    # Trường hợp cơ sở
    li a0 1
    j epilogue
    
recurse:
    addi a0 s0 -1
    jal ra factorial
    mul a0 s0 a0

epilogue:
    lw ra 0(sp)
    lw s0 4(sp)
    addi sp sp 8
    jr ra
```

:::

:::{note} Mở rộng để xem gợi ý cài đặt trình mô phỏng Venus
:class: dropdown

* Trong `main`:
  * `jal ra, factorial` (ví dụ: cuộc gọi hàm `factorial(4)`)
  * cập nhật `li a0 3` để mô phỏng `factorial(3)`
* Đặt breakpoint trong `factorial`:
  * đầu thủ tục, tức là nhãn `factorial`
  * Trường hợp cơ sở
  * cuộc gọi đệ quy

Kiểm tra giá trị thanh ghi `a0` và `s0`; gợi ý chuyển sang chế độ xem `Decimal` của giá trị thanh ghi
:::

## Một ví dụ đệ quy khác

Hãy xem!

```{code} c
:linenos:
int foo(int i) {
  if (i == 0) return 0;
  int a = i + foo(i-1);
  return a;
}
int j = foo(3);
int k = foo(100);
int m = j+k;
```

```{code} bash
:linenos:
  j main
foo:          # int foo(int i)
  addi sp sp -8 # Prologue
  sw ra 0(sp)   # Prologue
  sw s0 4(sp)   # Prologue
  mv s0 a0      # Di chuyển i
  bne s0 x0 Next # nếu i != 0, bỏ qua này
  li a0 0       # int a = 0;
  j Epilogue    # Đi đến Epilogue
              # (để khôi phục stack)
Next: 
  addi a0 s0 -1 # int j = i - 1;
  jal ra foo    # j = foo(j);
  add a0 s0 a0  # int a = i + j;
Epilogue: 
  lw ra 0(sp)   # Epilogue
  lw s0 4(sp)   # Epilogue
  addi sp sp 8  # Epilogue
  jr ra         # return a;
main:
  li a0 3       # int j = foo(3);
  jal ra foo    # gọi foo
  mv s0 a0      # mv rd rs1 đặt rd = rs1
  li a0 100     # int k = foo(100);
  jal ra foo    # gọi foo
  mv s1 a0      # Lưu giá trị trả về trong s1
  add a0 s0 s1  # int m = j+k;
```
