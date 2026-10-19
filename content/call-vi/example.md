---
title: "Ví Dụ CAL: Hello World"
subtitle: "Quá trình dịch: Compiler, Assembler, Linker"
---

(sec-call-example-vi)=
## Mục Tiêu Học Tập

* Xác định bước nào trong Compile, Assemble, và Link dịch một lệnh assembly cụ thể thành mã máy.
* Xem ví dụ đầu-cuối về việc dịch chương trình `hello.c` thành file thực thi `a.out`.
* Nhận biết các chỉ thị assembly.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/ZG8CSM0CaM4
:width: 100%
:title: "[CS61C FA20] Lecture 13.6 - Compilation, Assembly, Linking, Loading: Example"
:::

::::

Hãy mô tả quá trình dịch chương trình (Compile, Assemble, và Link) với ví dụ `hello_world`.

:::{hint} Kiểm Tra Nhanh

Tại thời điểm nào trong quá trình, tất cả các bit mã máy được xác định cho các lệnh assembly sau?

1. `add  x6 x7 x8`
2. `jal  x1 fprintf`

Sau compilation, sau assembly, sau linking, hay sau loading?

:::

:::{note} Hiển Thị Đáp Án

1. `add  x6 x7 x8` **Sau assembly**. Không cần phân giải địa chỉ nào.
2. `jal  x1 fprintf` **Sau linking**. `fprintf` là hàm external (C `stdio.h`)

:::

## Compiler: `hello.c` $\rightarrow$ `hello.s`

`hello.c`:

```{code} c
:linenos:
#include <stdio.h>
int main() {
    printf("Hello, %s\n", "world");
    return 0;
}
```

`hello.s`:
(code-hello-s-vi)=
```{code} c
:linenos:
.text
    .align 2
    .global main
main:
    addi sp sp -4
    sw   ra 0(sp)
    la   a0 str1
    la   a1 str2
    call printf
    lw   ra 0(sp)
    addi sp sp 4
    li   a0 0
    ret
.section .rodata
    .balign 4
str1:
    .string "Hello, %s!\n"
str2:
    .string "world"
```

Chú ý nhiều **giả lệnh**: `la`, `call`, `li`, `ret`, v.v.

:::{note} Nhấn để xem các chỉ thị
:class: dropdown

* Dòng 1, `.text`: Vào section `text`
  * Dòng 2, `.align`: Căn chỉnh mã theo 2*2 byte
  * Dòng 3, `.global`: Khai báo ký hiệu global main
* Dòng 14, `section`: Vào section `data` chỉ đọc
  * Dòng 15, `balign`: Căn chỉnh data section theo 4 byte
  * Dòng 17, `.string`: chuỗi kết thúc bằng null
  * Dòng 19, `.string`: chuỗi kết thúc bằng null

Thêm [chỉ thị](#sec-directives-vi):

| Chỉ thị | Mô tả |
| :--- | :--- |
| `.text` | Đặt các mục tiếp theo vào Text segment của người dùng (mã máy) |
| `.data` | Đặt các mục tiếp theo vào Data segment của người dùng (dữ liệu file nguồn ở dạng nhị phân) |
| `.globl sym` | Khai báo sym là global và có thể được tham chiếu từ các file khác |
| `.string str` | Lưu chuỗi str trong bộ nhớ và kết thúc bằng null |
| `.word` | Lưu n số lượng 32 bit vào các word bộ nhớ liên tiếp |

:::

:::{note} Nhấn để xem các nhãn
:class: dropdown

* Dòng 4, `main`
* Dòng 16, `str1`
* Dòng 18, `str2`
:::

## Assembler: `hello.s` $\rightarrow$ `hello.o`

Đầu ra của assembler là object file ở dạng nhị phân:

```bash
     ... ff010113 00112623 00000537
00050513 000005b7 00058593 000080e7
00c12083 01010113 00000513 00008067 ...
```

### Text segment

Đây không dễ đọc lắm, nên ta mô tả một số thành phần chi tiết hơn. Dưới đây là text segment dịch phần `main` của `hello.o`. Tham khảo [`hello.s`](#code-hello-s-vi) khi cần.

(code-hello-o-vi)=
```{code} bash
00000000 <main>:
0:  ffc10113 addi sp sp -4
4:  00112023 sw   ra 0(sp)
8:  00000537 lui  a0 0x0
c:  00050513 addi a0 a0 0
10: 000005b7 lui  a1 0x0
14: 00058593 addi a1 a1 0
18: 000080e7 jalr ra 0
1c: 00012083 lw   ra 0(sp)
20: 00410113 addi sp sp 4
24: 00000513 addi a0 zero 0
28: 00008067 jalr ra
```

:::{note} Cách đọc mỗi dòng
:class: dropdown

* Bên trái dấu hai chấm, vd: `10`: địa chỉ tương đối của lệnh trong module
* 8 chữ số thập lục phân, vd: (`000005b7`): mã máy 32 bit, có thể với các placeholder
* lệnh assembly, vd: (`lui a1 0x0`): lệnh assembly, có thể với các placeholder
:::

Giả lệnh được thay thế ở những chỗ có thể.
* `la`[^la] được thay bằng `lui` và `addi`
* `call` được thay bằng `jalr`

[^la]: Refcard CS61C nói rằng giả lệnh `la` phân giải thành `auipc` và `addi`, ngụ ý rằng `la` luôn là địa chỉ tương đối PC. Tuy nhiên, việc phân giải giả->thật này phụ thuộc vào compiler RISC-V. Trong gcc, bạn có thể dùng cờ -fpic hoặc -fno-pic để chỉ định định địa chỉ tương đối vs. tuyệt đối. Tôi nghi rằng cho ví dụ này, chúng ta dùng định địa chỉ tuyệt đối để minh họa đầy đủ các placeholder địa chỉ.

Mã máy bao gồm **các placeholder địa chỉ** bằng zero (vd: `lui a1 0x0` là mã máy `000005b7`), biểu thị các tham chiếu chưa phân giải để linker phân giải.

### Symbol Table

Phần dưới sẽ ở dạng nhị phân, nhưng ta minh họa nó dưới dạng bảng:

:::{table} Symbol Table cho `hello.o`
:label: tab-hello-symbol-table-vi

| Nhãn | Địa chỉ (trong segment của module) | Loại |
| :-- | :-- | :-- |
| `main` | `0x00000000` | global text |
| `str1` | `0x00000000` | local data |
| `str2` | `0x0000000c` | local data |

:::

### Relocation Table

Phần dưới sẽ ở dạng nhị phân, nhưng ta minh họa nó dưới dạng bảng:

:::{table} Relocation Table cho `hello.o`
:label: tab-hello-relocation-table-vi

| Địa chỉ | Loại | Phụ thuộc |
| :-- | :-- | :-- |
| `0x00000008` | `lui `| `%hi(str1)` |
| `0x0000000c` | `addi` | `%low(str1)` |
| `0x00000010` | `lui` | `%hi(str2)` |
| ... | ... | ... |

:::

## Linker: `hello.s` $\rightarrow$ `a.out`

Một phần của file thực thi `a.out` được hiển thị:

```{code} bash
000101b0 <main>:
  101b0: ffc10113 addi sp sp -4
  101b4: 00112023 sw   ra 0(sp)
  101b8: 00021537 lui  a0 0x21
  101bc: a1050513 addi a0 a0 -1520 # 20a10 <str1>
  101c0: 000215b7 lui  a1 0x21
  101c4: a1c58593 addi a1 a1 -1508 # 20a1c <str2>
  101c8: 288000ef jal  ra 10450    # <printf>
  101cc: 00012083 lw   ra 0(sp)
  101d0: 00410113 addi sp sp 4
  101d4: 00000513 addi a0 zero 0
  101d8: 00008067 jalr ra
```

* Gộp tất cả các file .o
* Cập nhật trong symbol table
* Với mỗi mục trong relocation table:
  * Thay các placeholder bằng địa chỉ thực
  * Cập nhật mã máy. Chú ý rằng upper immediate của `lui` được tăng thêm 1; tham khảo [phần trước](#sec-li-lui-vi) về định dạng lệnh U-Type và giả lệnh `li`.
