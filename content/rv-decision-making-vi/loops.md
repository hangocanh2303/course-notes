---
title: "Vòng lặp"
---

## Mục tiêu học tập

* Dịch vòng lặp C thành lệnh hợp ngữ RISC-V.
* Xem vòng lặp `for` trong hợp ngữ.


::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/OWxcAqFNkpo
:width: 100%
:title: "[CS61C FA20] Lecture 08.3 - RISC-V lw, sw, Decisions I: Decision Making"

Từ 8:50 trở đi
::::


:::{hint} Học một vòng lặp, học tất cả


Có nhiều loại vòng lặp trong C: `while`, `for`, `do-while`.

Như bạn có thể nhớ từ lớp trước như CS 61A, mỗi cấu trúc vòng lặp có thể được viết thành một trong các vòng lặp khác (hãy thử với vòng lặp for và while). Một khi chúng ta học cách viết branch có điều kiện với một vòng lặp, chúng ta có thể áp dụng cùng phương pháp branch cho tất cả các vòng lặp.

:::

Chúng ta sẽ đề cập chi tiết việc triển khai vòng lặp `for` bên dưới, sau đó để các vòng lặp khác cho bạn tham khảo sau.

## Mã C

Vòng lặp C bên dưới cộng 20 số nguyên trong mảng `arr` và lưu kết quả trong `sum`.

(code-c-for-loop)=
:::{code} c
:linenos:

int arr[20];
... // điền dữ liệu vào arr
int sum = 0;
for (int i=0; i<20; i++) {
    sum += arr[i];
}
// ...
:::

Việc triển khai vòng lặp trong hợp ngữ tương tự như viết vòng lặp với cú pháp `goto` [](#sec-goto-warning). Nhấp để hiển thị cách chúng ta có thể viết lại vòng lặp ở trên với `goto` và [nhãn](#sec-labels).

::::{note} Nhấp để hiển thị vòng lặp `goto`
:class: dropdown

(code-goto-for-loop)=
:::{code} c
:linenos:
      int arr[20];
      …
      int sum = 0;
      int i = 0;
Loop: if(i >= 20) goto End
      sum += arr[i];
      i++;
      goto Loop
End:  // …
:::

(sec-goto-warning)=
:::{warning} `goto` không phải là thực hành C tốt
Đừng viết `goto` trong mã C thực! Vì lý do chúng ta thảo luận trong [Phụ lục](#sec-appendix-goto) của trang này.
:::

::::


## Bản dịch RISC-V

Đây là bản dịch hợp ngữ đầy đủ của [mã C gốc](#code-c-for-loop).

(code-for-loop-rv)=
```{code} bash
    add   x9  x8  x0
    add  x10  x0  x0
    add  x11  x0  x0
    addi x13  x0  20
Loop:
    bge  x11 x13 End
    lw   x12  0(x9)
    add  x10 x10 x12
    addi  x9  x9   4
    addi x11 x11   1
    j Loop
End:
    ...
```

### Gán thanh ghi

:::{tip} Kiểm tra nhanh

Khi bạn theo dõi mã, cố gắng hiểu cách các số thanh ghi được sử dụng ánh xạ đến các biến cục bộ trong [mã C gốc](#code-c-for-loop).

Đọc [hợp ngữ ở trên](#code-for-loop-rv), khớp các thanh ghi bên trái với một trong các biểu thức C bên phải.

| Thanh ghi | Biểu thức C |
| :--: | :-- |
| **1.**   `x8` | **A.** `sum` |
| **2.**  `x11` | **B.** `i` |
| **3.**   `x9` | **C.** `20` |
| **4.**  `x12` | **D.** `arr[0]` |
| **5.**  `x10` | **E.** `&arr[0]` |
| **6.**  `x13` | **F.** `arr[i]` |
| | **G.** `&arr[i]` |

**Gợi ý**: Thanh ghi `x8` giữ địa chỉ của `arr`.

:::

:::{note} Hiển thị đáp án
:class: dropdown

Tham chiếu [hợp ngữ ở trên](#code-for-loop-rv) khi bạn đi qua các đáp án.

1. Thanh ghi `x8` giữ `&arr[0]`, địa chỉ của phần tử đầu tiên của `arr` (tương đương, địa chỉ của `arr`).
1. Thanh ghi `x11` giữ `i`, biến vòng lặp. Gợi ý là từ lệnh `add x11 x0 x0` ngay trước `Loop`, và lệnh `addi x11 x11 1` trước khi chúng ta nhảy lại về đầu `Loop`.
1. Thanh ghi `x9` giữ `&arr[i]`, địa chỉ của phần tử hiện tại của `arr`. Gợi ý là từ lệnh đầu tiên `add x9 x8 x0` và lệnh **số học con trỏ số nguyên** `add x9 x9 4` trước khi chúng ta nhảy lại về đầu `Loop`.
1. Thanh ghi `x12` giữ `arr[i]`, phần tử hiện tại của `arr` (giá trị, không phải địa chỉ). Gợi ý là từ "khởi tạo"[^wording] của nó với `lw x12 0(x9)` và việc sử dụng nó trong vòng lặp, `add x10 x10 x12`.
1. Thanh ghi `x10` giữ `sum`, giá trị hiện tại của `sum`. Gợi ý là lệnh `add x10 x10 x12` duy nhất trong `Loop`.
1. Thanh ghi `x13` có giá trị immediate `20` và nó được đặt trong `addi x13 x0 20`. Giá trị này được sử dụng trong lệnh branch `bge`, phải so sánh **thanh ghi**.

[^wording]: Lưu ý rằng thanh ghi vật lý luôn ở đó, nên thanh ghi không thể được khởi tạo. Trước lệnh `lw`, chỉ có một số dữ liệu khác trong `x12`; sau lệnh `lw`, `x12` giữ các bit cần cho chuỗi lệnh này.
:::


### Từng dòng

Hợp ngữ một lần nữa, bây giờ có comment. Di chuột để tham chiếu [mã C gốc](#code-c-for-loop).

```{code} bash
:linenos:
    add   x9  x8  x0 # &arr[0]
    add  x10  x0  x0 # sum = 0
    add  x11  x0  x0 # i=0
    addi x13  x0  20
Loop:
    bge  x11 x13 End # nếu i >= 20, thì branch
    lw   x12  0(x9)
    add  x10 x10 x12 # sum += arr[i]
    addi  x9  x9   4 # &arr[i+1]
    addi x11 x11   1 # i++
    j Loop
End:
    ...
```

**Dòng 1**. `add x9 x8 x0`. Đặt phần tử đầu tiên (thứ zero) vào thanh ghi `x9`.

**Dòng 2**. `add x10 x0 x0`. Khởi tạo[^wording-2] `sum` bằng zero trong thanh ghi `x10`.

[^wording-2]: Thanh ghi `x10` không được khởi tạo; nó là vị trí lưu trữ vật lý cho bit. Chúng ta sử dụng khởi tạo để ám chỉ biến C. Ở đây, thanh ghi `x10` đang được sử dụng để giữ các giá trị của biến `sum`; biến `sum` được khởi tạo bằng giá trị zero.

**Dòng 3**: `add x11 x0 x0`. Khởi tạo biến vòng lặp `i` bằng zero trong thanh ghi `x11`.

**Dòng 4**: `addi x13 x0 20` (`li x13 20`). Đặt giá trị `20` vào thanh ghi `x13` để sử dụng cho lệnh branch.

**Dòng 5-6**: `bge x11 x13 End`. Nếu biến vòng lặp hiện tại lớn hơn hoặc bằng 20, branch đến lệnh End trong Dòng 13.

* `bge` thoát vòng lặp nếu `R[x11] >= R[x13]` là true.
* Nếu không, nếu false, thì `i < 20`, nên tiếp tục trong vòng lặp.
* Lưu ý rằng các nhãn `Loop` và `End` không phải là lệnh; thay vào đó, chúng được gắn với địa chỉ lệnh (tức là, `Loop` là địa chỉ của lệnh `bge`).

**Dòng 7**: `lw   x12  0(x9)`. Lấy giá trị của `arr[i]` và đặt nó vào thanh ghi `x12`.

**Dòng 8**: `add  x10 x10 x12`. Cộng `arr[i]` vào tổng hiện tại trong thanh ghi `x10`; ghi kết quả trở lại thanh ghi `x10`.

**Dòng 9**: `addi  x9  x9   4`. Lấy địa chỉ của phần tử tiếp theo, `&arr[i+1]`. Cập nhật địa chỉ byte trong thanh ghi `x9` bằng cách cộng 4 (`sizeof(int)` là 4).

**Dòng 10**: `addi x11 x11 1`. Tăng `i` thêm một. Điều này cần cho so sánh lệnh branch.

**Dòng 11**: `j Loop` Nhảy đến lệnh branch.

**Dòng 12**: Nếu chúng ta đến lệnh này, chúng ta đã thoát khỏi vòng lặp.

### Chạy Demo

Bạn có thể chạy demo bên dưới trong trình mô phỏng RISC-V như Venus.

:::{note} `arr20.s`
:class: dropdown

```bash
.data
output: .word   1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20

.text
    # `output` là con trỏ đến mảng int
    # đặt x8 bằng `output`
    la    x8 output

    add   x9  x8  x0
    add  x10  x0  x0
    add  x11  x0  x0
    addi x13  x0  20
Loop:
    bge  x11 x13 End
    lw   x12  0(x9)
    add  x10 x10 x12
    addi  x9  x9   4
    addi x11 x11   1
    j Loop
End:
    mv a0 x10
    jal print_int

    # truyền 10 vào ecall sẽ kết thúc chương trình
    li a0 10
    ecall

# in ra một số nguyên
# giá trị đầu vào: a0: số nguyên cần in
# không trả về gì
print_int:
    # để in một số nguyên, chúng ta cần tạo ecall với a0 đặt bằng 1
    # thứ sẽ được in được lưu trong thanh ghi a1
    # dòng này sao chép số nguyên cần in vào a1
    mv a1 a0
    # đặt thanh ghi a0 bằng 1 để ecall sẽ in
    li a0 1
    # in số nguyên
    ecall
    # trả về hàm gọi
    jr ra
```

:::

## Rút gọn cấu trúc điều khiển

Chúng ta mô tả các cách "go-to" (heh) để rút gọn các vòng lặp phổ biến thành các cấu trúc có thể dịch trực tiếp hơn sang hợp ngữ.

### Điều kiện `if`

Xét mã C bên dưới. Rút gọn mã sử dụng ý tưởng được tận dụng trong [Ví dụ 1](#sec-branch-ex1) và [Ví dụ 2](#sec-branch-ex2) để đảo ngược điều kiện với câu lệnh `goto`.

```c
if(cond) {
    line1;
    line2;
}
line3;
```

:::{note} Hiển thị rút gọn
:class: dropdown

```c
if(!cond) goto AfterIf;
line1;
line2;

AfterIf:
    line3;
```
:::


### Vòng lặp `while`

```c
while(cond) {
    line1;
    line2;
} 
line3;
```

:::{note} Hiển thị rút gọn
:class: dropdown

```c
Loop:
    if(!cond) goto AfterLoop;
    line1;
    line2;
    goto Loop;

AfterLoop:
    line3;
```
:::

### Vòng lặp `while` với `break`

Trong mã bên dưới, `break` rút gọn thành câu lệnh `goto`:

```c
while(true) {
    line;
    break;
}
line;
```

:::{note} Hiển thị bản dịch
:class: dropdown

```c
while(true) {
    line;
    goto AfterWhile;
}
AfterWhile:
    line;
```

:::

### Vòng lặp `for`

```c
for(startline;cond;incline) {
    line1;
    line2;
} 
line3;
```

Chúng tôi khuyến nghị trước tiên dịch vòng lặp `for` thành vòng lặp `while` tương đương:

```c
startline;
while(cond) {
    line1;
    line2;
    incline;
} 
line3;
```

Cuối cùng, rút gọn với `goto`.

:::{note} Hiển thị rút gọn
:class: dropdown

```c
startline;

Loop:
    if(!cond) goto AfterLoop
    line1;
    line2;
    incline;
    goto Loop

AfterLoop:
    line3;
```
:::

### Vòng lặp `do-while`

Mã C:

```c
do {
    line1;
    line2;
} while(cond)
line3;
```

:::{note} Hiển thị rút gọn
:class: dropdown

```c
Loop:
    line1;
    line2;
    if(cond) goto Loop;
 
line3;
```
:::


(sec-appendix-goto)=
## Phụ lục: `goto`

Trong C, câu lệnh `goto Label;` đặt dòng tiếp theo được thực thi là dòng được [gắn nhãn](#sec-labels) với `Label`. Dòng được gắn nhãn này có thể ở bất kỳ đâu khác trong chương trình.


(sec-goto-warning-2)=
:::{warning} `goto` không phải là thực hành C tốt
Đừng viết `goto` trong mã C thực!

Việc sử dụng [goto](https://en.wikipedia.org/wiki/Goto) trong lịch sử đã phổ biến trong các ngôn ngữ lập trình trước đó, có lẽ vì nó phản ánh cách ngôn ngữ hợp ngữ thực thi các lệnh. Ngày nay, C hiện đại và ngôn ngữ bậc cao ủng hộ các câu lệnh luồng điều khiển có cấu trúc qua `if-else`, vòng lặp `for` và `while`, v.v.
:::

@fig-goto-xkcd cho chúng ta một lý do hài hước (nếu không phải hư cấu) tại sao chúng ta không nên sử dụng `goto` trong lập trình C.

:::{figure} images/goto-xkcd.png
:label: fig-goto-xkcd
:alt: "Truyện tranh xkcd trong bốn panel: một lập trình viên chọn goto main_sub3 thay vì tái cấu trúc, biên dịch, sau đó một velociraptor tấn công; trò đùa cảnh báo chống lại việc sử dụng goto bất cẩn."
:align: center
:width: 80%

Sử dụng `goto` có thể dẫn đến các cuộc tấn công của velociraptor. ([xkcd](https://xkcd.com/292/), [explainxkcd](https://www.explainxkcd.com/wiki/index.php/292:_goto))
:::

Tuy nhiên, chúng ta chia sẻ một số ví dụ `goto` trong C, cho những ai muốn thực hành viết mã thân thiện hơn với bản dịch hợp ngữ trực tiếp. Chúng tôi khuyến khích bạn xem các ví dụ này với một chút hoài nghi.

### Ví dụ `malloc`

Xét mã bên dưới. Chúng ta muốn viết lại mã này trong trường hợp bất kỳ cuộc gọi `malloc` nào thất bại và trả về `NULL`.

```c
int* a = malloc(sizeof(int)*1000);
int* b = malloc(sizeof(int)*1000000);
int* c = malloc(sizeof(int)*1000000000);
FILE* d = fopen(filename);
```

Mã bên dưới là một nỗ lực, mặc dù nó không giải phóng các khối `malloc` trước đó.

```c
int* a = malloc(sizeof(int)*1000);
if(a == NULL) allocation_failed();
int* b = malloc(sizeof(int)*1000000);
if(b == NULL) allocation_failed();
int* c = malloc(sizeof(int)*1000000000);
if(c == NULL) allocation_failed();
FILE* d = fopen(filename);
if(d == NULL) allocation_failed();
```

Mã cuối cùng sử dụng `goto` để giải phóng các khối đã cấp phát trước đó khi bất kỳ thất bại đơn lẻ nào xảy ra.

```c
int* a = malloc(sizeof(int)*1000);
if(a == NULL) goto ErrorA;
int* b = malloc(sizeof(int)*1000000);
if(b == NULL) goto ErrorB;
int* c = malloc(sizeof(int)*1000000000);
if(c == NULL) goto ErrorC;
FILE* d = fopen(filename);
if(d == NULL) {
           free(c);
ErrorC:    free(b);
ErrorB:    free(a);
ErrorA:    allocation_failed();
}
```
