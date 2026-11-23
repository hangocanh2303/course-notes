---
title: "Cache Blocking"
---

(sec-cache-blocking-vi)=
## Mục Tiêu Học Tập

* Viết các chương trình tận dụng hiểu biết về thiết kế cache cơ bản.
* Định nghĩa cache blocking.

Chúng ta đã thấy trong [phần trước](#sec-cache-optimizations-vi) cách mà với tư cách là kiến trúc sư máy tính, chúng ta có thể giảm cache misses bằng cách tăng capacity của cache. Tuy nhiên, với tư cách là lập trình viên, chúng ta thường không có quyền kiểm soát hardware của máy tính. Việc thay thế cache không hề đơn giản. Thay vào đó, chúng ta phải giả định rằng chúng ta có một kiến trúc hardware cố định nào đó, sau đó xem cách chúng ta có thể viết lại chương trình để tối đa hóa việc sử dụng hardware.

**Cache blocking** là một kỹ thuật lập trình viên sắp xếp lại các data accesses để sử dụng tốt hơn data được nạp vào cache và giảm cache misses.

Trong phần này, chúng ta xem xét một program benchmark cụ thể: [matrix multiplication](#sec-dgemm). Sau khi thử một naive implementation ban đầu, chúng ta cho thấy cách hiểu về thiết kế cơ bản của cache thực sự có thể cải thiện cách chúng ta viết chương trình.

## Matrix Multiplication (DGEMM)

Trong phần này, chúng ta sử dụng một matrix multiplication benchmark.
Trong ví dụ này, chúng ta sẽ xem xét nhân matrix $A$ (4 hàng $\times$ 8 cột) với matrix $B$ (8 hàng $\times$ 4 cột) để tạo ra matrix $C$ (4 hàng $\times$ 4 cột).[^int-notation]

[^int-notation]: Sử dụng ký hiệu toán học đúng đắn, trong đó $\mathbb{Z}$ là tập hợp tất cả các số nguyên: $A \in \mathbb{Z}^{n \times d}, B \in \mathbb{Z}^{d \times m}, C \in \mathbb{Z}^{n \times m}$. Trong ví dụ của chúng ta, $n = m = 4, d = 8$.

```{note} Chi tiết Matrix Multiplication
Xem lại [phần này](#sec-dgemm) mô tả row-major order matrix multiplication benchmark trong phần này.
```

Giả sử các matrices $A$, $B$, và $C$ được lưu trữ theo **row-major order** dưới dạng `int A[]`, `int B[]`, và `int C[]`.

Code C, để tham khảo:

<!-- :::{figure} images/matmul-row-major.png
:label: fig-matmul-row-major
:alt: "Matrix memory-layout illustration showing row-major storage order for A, B, and C matrices in memory. The top rectangle shows the wide Nx8 matrix with elements labeled 1 through 16. The bottom rectangle shows the Nx8 elements of the matrix laid out in contiguous space in memory where the matrix is stored."

Assume that all matrices are stored in **row-major order**.
:::
 -->
:::{embed} #code-igemm-simple
:::

## Chi Tiết Kiến Trúc

Giả sử chúng ta chạy chương trình C trên một kiến trúc 32-bit có cache đơn tầng:

* Dung lượng 128B
* Fully associative
* Block size 16B (vậy 8 blocks)
* LRU replacement policy
* Write-back write policy với dirty bit

Cho ví dụ matrix multiplication của chúng ta, chúng ta sẽ giả định rằng trên kiến trúc này, `sizeof(int)` là `4`, nên mỗi block chứa bốn `int`s.

## Phương Pháp 1: Naive DGEMM Memory Access Pattern

Giả sử cache bắt đầu ở trạng thái cold.

Giả sử trước tiên chúng ta tính $C_{00}$, là dot-product của hàng thứ không (zero-indexed) của $A$ và cột thứ không (zero-indexed) của $B$.

<!-- ::::{figure}
:label: anim-matmul-00
:::{iframe} https://view.officeapps.live.com/op/embed.aspx?src=https://github.com/61c-teach/course-notes/raw/refs/heads/main/content/caches-iii/pptx/matmul-00.pptx
:width: 100%
:title: "C[0][0] Memory Access Pattern"
:::
Computing $C_00$ as vector multiplication of the zero-th row of $A$ and the zero-th column of $B$. Use the menu bar to trace through the animation or download a copy of the PDF/PPTX file.
:::: -->

::::{figure}
:label: anim-matmul-00-vi
:alt: "Embedded slides animating memory accesses while computing matrix element C zero zero from the zero-th row of A and zero-th column of B."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vTAFRr4VJ6o2brFhpjCS_edmdWM0j2cKViq6OjTX1XqDhRJWYa68FU07IyZw2oG6g/pubembed?start=false&loop=false
:width: 100%
:title: "Slides walking through a C[0][0] Memory Access Pattern."
:::
Tính $C_{00}$ như vector multiplication của hàng thứ không của $A$ và cột thứ không của $B$. Sử dụng menu bar để theo dõi animation hoặc truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1GJiXwZ8gGuZxLxSU5raiTPY_E0I4AQVU/edit?usp=sharing).
::::

:::{note} Hiển thị cache hits/misses
:class: dropdown

Giả sử `sum` được lưu trong register. Giả sử truy cập bất kỳ phần tử nào của `A`, `B`, hoặc `C` sẽ dẫn đến memory access.

1. `A[0][0] * B[0][0]`. Compulsory cache miss trên `A[0][0]`, compulsory cache miss trên `B[0][0]`.

    Nội dung cache, theo thứ tự most recently used:
    * Hàng $0$ của B
    * Hàng $0$ của A, nửa đầu (phần tử $0$ đến $3$)

1. `A[0][1] * B[1][0]`. Cache hit trên `A[0][1]`, compulsory cache miss trên `B[1][0]`.

    Nội dung cache:
    * Hàng $1$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $0$ của B

1. `A[0][2] * B[2][0]`. Cache hit trên `A[0][2]`, compulsory cache miss trên `B[2][0]`.

    Nội dung cache:
    * Hàng $2$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $1$ của B
    * Hàng $0$ của B

1. `A[0][3] * B[3][0]`. Cache hit trên `A[0][3]`, compulsory cache miss trên `B[3][0]`.

    Nội dung cache:
    * Hàng $3$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $2$ của B
    * Hàng $1$ của B
    * Hàng $0$ của B

1. `A[0][4] * B[4][0]`. Compulsory cache miss trên `A[0][4]`, compulsory cache miss trên `B[4][0]`.

    Nội dung cache:
    * Hàng $4$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $3$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $2$ của B
    * Hàng $1$ của B
    * Hàng $0$ của B

1. `A[0][5] * B[5][0]`. Cache hit trên `A[0][5]`, compulsory cache miss trên `B[5][0]`.

    Nội dung cache:
    * Hàng $5$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $4$ của B
    * Hàng $3$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $2$ của B
    * Hàng $1$ của B
    * Hàng $0$ của B

  Cache bây giờ đã đầy (at capacity).

1. `A[0][6] * B[6][0]`. Cache hit trên `A[0][6]`, compulsory cache miss trên `B[6][0]`. Cache đầy, nên thay thế block least recently used (hàng $0$ của B).

    Nội dung cache:
    * Hàng $6$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $5$ của B
    * Hàng $4$ của B
    * Hàng $3$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $2$ của B
    * Hàng $1$ của B
    * Hàng $0$ của B

1. `A[0][7] * B[7][0]`. Cache hit trên `A[0][6]`, compulsory cache miss trên `B[7][0]`. Cache đầy, nên thay thế block least recently used (hàng $1$ của B).

    Nội dung cache:
    * Hàng $7$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $6$ của B
    * Hàng $5$ của B
    * Hàng $4$ của B
    * Hàng $3$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $2$ của B
    * Hàng $1$ của B

1. `C[0][0] = sum`. Cache miss trên `C[0][0]`. Cache đầy, nên thay thế block least recently used (hàng $2$ của B).

    Nội dung cache:
    * Hàng $0$ của C
    * Hàng $7$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $6$ của B
    * Hàng $5$ của B
    * Hàng $4$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $3$ của B
:::

Nội dung cache sau khi tính $C_{00}$, theo thứ tự most recently used:

* Hàng $0$ của C
* Hàng $7$ của B
* Hàng $0$ của A, nửa sau
* Hàng $6$ của B
* Hàng $5$ của B
* Hàng $4$ của B
* Hàng $0$ của A, nửa đầu
* Hàng $3$ của B

Tiếp theo, giả sử chúng ta tính $C_{01}$, là dot-product của hàng thứ không (zero-indexed) của $A$ và cột thứ nhất (zero-indexed) của $B$.

<!-- ::::{figure}
:label: anim-matmul-ij
:::{iframe} https://view.officeapps.live.com/op/embed.aspx?src=https://github.com/61c-teach/course-notes/raw/refs/heads/main/content/caches-iii/pptx/matmul-00.pptx
:width: 100%
:title: "C[i][j] Memory Access Pattern"
:::
Computing $C_{ij}$ as vector multiplication of the i-th row of $A$ and the j-th column of $B$. Use the menu bar to trace through the animation or download a copy of the PDF/PPTX file.
:::: -->

::::{figure}
:label: anim-matmul-ij-vi
:alt: "Embedded slides animating memory accesses while computing a general inner product C i j from row i of A and column j of B."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vRWBkNhA5huAtKWqxfxruNlEUAqXRxVDGHzjT88Ov3ZJnfrupfQsbNZHSyXOyS3SQ/pubembed?start=false&loop=false
:width: 100%
:title: "Slides walking through a C[i][j] Memory Access Pattern."
:::
Tính $C_{ij}$ như vector multiplication của hàng thứ i của $A$ và cột thứ j của $B$. Sử dụng menu bar để theo dõi animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/1AekxfV7tcsUA0CcpvY4J9jg1YvnWKMfZ/edit?usp=sharing).
::::

:::{note} Hiển thị cache hits/misses
:class: dropdown

Giả sử `sum` được lưu trong register. Giả sử truy cập bất kỳ phần tử nào của `A`, `B`, hoặc `C` sẽ dẫn đến memory access.

1. `A[0][0] * B[1][0]`. Cache hit trên `A[0][0]`, **non-compulsory** cache miss trên `B[1][0]`. Cache đầy, nên thay thế block least recently used (hàng $3$ của B).

    Nội dung cache:
    * Hàng $0$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $0$ của C
    * Hàng $7$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $6$ của B
    * Hàng $5$ của B
    * Hàng $4$ của B

1. `A[0][1] * B[1][1]`. Cache hit trên `A[0][1]`, **non-compulsory** cache miss trên `B[1][1]`. Cache đầy, nên thay thế block least recently used (hàng $4$ của B).

    Nội dung cache:
    * Hàng $1$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $0$ của B
    * Hàng $0$ của C
    * Hàng $7$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $6$ của B
    * Hàng $5$ của B

1. `A[0][2] * B[1][2]`. Cache hit trên `A[0][2]`, **non-compulsory** cache miss trên `B[1][2]`. Cache đầy, nên thay thế block least recently used (hàng $5$ của B).

    Nội dung cache:
    * Hàng $2$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $1$ của B
    * Hàng $0$ của B
    * Hàng $0$ của C
    * Hàng $7$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $6$ của B

1. `A[0][3] * B[1][3]`. Cache hit trên `A[0][3]`, **non-compulsory** cache miss trên `B[1][3]`. Cache đầy, nên thay thế block least recently used (hàng $6$ của B).

    Nội dung cache:
    * Hàng $3$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $2$ của B
    * Hàng $1$ của B
    * Hàng $0$ của B
    * Hàng $0$ của C
    * Hàng $7$ của B
    * Hàng $0$ của A, nửa sau

1. `A[0][4] * B[1][4]`. Cache hit trên `A[0][4]`, **non-compulsory** cache miss trên `B[1][4]`. Cache đầy, nên thay thế block least recently used (hàng $7$ của B).

    Nội dung cache:
    * Hàng $4$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $3$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $2$ của B
    * Hàng $1$ của B
    * Hàng $0$ của B
    * Hàng $0$ của C

1. `A[0][5] * B[1][5]`. Cache hit trên `A[0][5]`, **non-compulsory** cache miss trên `B[1][5]`. Cache đầy, nên thay thế block least recently used (hàng $0$ của C).

    Nội dung cache:
    * Hàng $5$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $4$ của B
    * Hàng $3$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $2$ của B
    * Hàng $1$ của B
    * Hàng $0$ của B

1. `A[0][6] * B[1][6]`. Cache hit trên `A[0][6]`, **non-compulsory** cache miss trên `B[1][6]`. Cache đầy, nên thay thế block least recently used (hàng $0$ của B).

    Nội dung cache:
    * Hàng $6$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $5$ của B
    * Hàng $4$ của B
    * Hàng $3$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $2$ của B
    * Hàng $1$ của B

1. `A[0][7] * B[1][7]`. Cache hit trên `A[0][7]`, **non-compulsory** cache miss trên `B[1][7]`. Cache đầy, nên thay thế block least recently used (hàng $1$ của B).

    Nội dung cache:
    * Hàng $7$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $6$ của B
    * Hàng $5$ của B
    * Hàng $4$ của B
    * Hàng $3$ của B
    * Hàng $0$ của A, nửa đầu
    * Hàng $2$ của B

1. `C[0][0] = sum`. **Non-compulsory** cache miss trên `C[0][0]`. Cache đầy, nên thay thế block least recently used (hàng $2$ của B).

    Nội dung cache:
    * Hàng $0$ của C
    * Hàng $7$ của B
    * Hàng $0$ của A, nửa sau
    * Hàng $6$ của B
    * Hàng $5$ của B
    * Hàng $4$ của B
    * Hàng $3$ của B
    * Hàng $0$ của A, nửa đầu
:::

Nội dung cache sau khi tính $C_{01}$, theo thứ tự most recently used:

* Hàng $0$ của C
* Hàng $7$ của B
* Hàng $0$ của A, nửa sau
* Hàng $6$ của B
* Hàng $5$ của B
* Hàng $4$ của B
* Hàng $3$ của B
* Hàng $0$ của A, nửa đầu

:::{warning} Row-major order, revisited

Trong matrix multiplication, việc tính các phần tử tiếp theo của $C$ dẫn đến non-compulsory cache misses trên các phần tử của $B$ và hàng đích của $C$!

Layout row-major của B trong memory gây ra các memory accesses quá nhiều. Chúng ta liên tục thay thế các hàng của $B$!
:::

Trong ví dụ matmul của chúng ta, chúng ta biết rằng `B` được lưu trữ theo row-major-layout. Để truy cập một **cột** của $B$ như cần thiết trong matrix multiplication, chúng ta phải nạp vào tất cả 8 hàng của `B`.

Từ P&H 4.4 cho các matrices vuông (N-by-N):

> Nếu cache có thể chứa một matrix N-by-N và một hàng N, thì ít nhất hàng thứ `i` của `A` và toàn bộ matrix `B` có thể ở trong cache. Ít hơn thế và misses có thể xảy ra cho cả `B` và `C`. Trong trường hợp xấu nhất, sẽ có 2 N{sup}`3`+ N{sup}`2` memory words được truy cập cho N{sup}`3` operations.

## Phương Pháp 2: Cache Blocking với Transpose

Một quan sát là sẽ tốt hơn nhiều nếu chỉ nạp 8 phần tử trong _cột_ của `B`, và không nạp các phần tử trong các cột khác cần cho các matrix multiplications sau này.

Một kỹ thuật cache blocking có thể **transpose** B trước matrix multiplication. Transpose của $B$ được viết là $B^T$ và được định nghĩa trong đó $B^T_{jk} = B_{kj}$ cho tất cả các chỉ số $j$ và $k$, như được hiển thị trong @fig-matmul-transpose-vi.

:::{figure} images/matmul-transpose.png
:label: fig-matmul-transpose-vi
:alt: "Matrix transpose diagram illustrating B transpose used to improve contiguous-memory access in multiplication. The left rectangle shows the original tall matrix B with column 1 highlighted, and elements 0 and 4 highlighted further. The right rectangle shows B transpose with the same elements highlighted, but now in their new transpose locations. An arrow between the two rectangles shows the ability to transform between B and B transpose."
:width: 60%

$B^T$ là matrix transpose của $B$

:::

Nếu chúng ta duy trì một bản copy của `B_T` (toán học là $B^T$), chúng ta có thể định nghĩa lại matrix multiplication như sau:

```c
// cache-blocking code
// cho hàng i, cột j của C
int sum = 0; // sizeof(int) = 4
for (int k = 0; k < size; k++) {
  sum += A[i][k] * B_T[j][k];
}
C[i][j] = sum;
```

Ghi chú:

* `B_T` vẫn là matrix được lưu trữ theo row-major order. Tuy nhiên, bây giờ matrix gốc $B$ của chúng ta thực sự được lưu trữ theo column-major order.
* Với optimization ở trên, chúng ta ngăn việc liên tục thay thế và fetch cùng data từ main memory. Thay vào đó, chúng ta nạp mỗi cột của `B`, hai memory accesses mỗi lần.

Transposing khá chậm; nó cũng yêu cầu overhead N{sup}`2` để hoàn thành và kích hoạt cùng loại cache misses như chúng ta đã quan sát trong tính toán ban đầu. Chúng ta chỉ đơn giản là di chuyển hiệu suất cache kém từ matrix multiplication sang một phần khác của chương trình.

(sec-cache-blocking-tiling-vi)=
## Phương Pháp 3: Cache Blocking với Submatrix Computation (Tiling)

Phương pháp cache blocking thứ hai của chúng ta quan sát rằng matrix multiplication có thể được tính theo từng phần. Một **submatrix** (tức là **tile**) của $C$ có thể được tính như tổng của việc nhân các submatrices khác nhau của $A$ và $B$.

@anim-matmul-block-2-vi minh họa cache blocking với **blocking factor** hoặc `BLOCKSIZE` bằng 2. Chúng ta tính tile 2x2 của $C$ với các phần tử $C_{ij}$, trong đó $i \in {0, 1}$ và $j \in {0, 1}$. Tile này có thể được tính bằng bốn phép nhân submatrix.

::::{figure}
:label: anim-matmul-block-2-vi
:alt: "Embedded slides illustrating cache blocking with a two-by-two tile of C computed from smaller submatrix multiplications of A and B."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vQU5B3-twc1_pBY024sp72KsxJ6O8tieO_QwOYPj4VgLt7--ONJ0BbG-Lj1CDwdoQ/pubembed?start=false&loop=false
:width: 100%
:title: "Slides illustrating cache blocking with a C[i][j] Memory Access Pattern."
:::
Cache blocking. Sử dụng menu bar để theo dõi animation hoặc truy cập [Google slides gốc](https://docs.google.com/presentation/d/1HY9AE2z3eb1eWPx0VQvVusZ83caTvZ5s/edit?usp=drive_link&ouid=113745915748997113650&rtpof=true&sd=true).
::::

:::{note} Hiển Thị Giải Thích
:class: dropdown

1. Nhân hai phần tử đầu tiên trong Hàng 0 và 1 của A với hai phần tử đầu tiên trong Hàng 0 và 1 của B. Lưu bốn kết quả vào tile C đích.

    Nội dung cache[^not-lru]:
    * Hàng $0$ của A, nửa đầu
    * Hàng $1$ của A, nửa đầu
    * Hàng $0$ của B
    * Hàng $1$ của B
    * Hàng $0$ của C
    * Hàng $1$ của C

[^not-lru]: Không theo thứ tự LRU hiện tại. Đánh dấu đây là TODO cho các học kỳ tương lai.

1. Nhân hai phần tử tiếp theo của A với hai phần tử đầu tiên trong Hàng 2 và 3 của B. Cộng vào kết quả trong tile C đích.

    Nội dung cache[^not-lru]:
    * Hàng $0$ của A, nửa đầu
    * Hàng $1$ của A, nửa đầu
    * Hàng $0$ của B
    * Hàng $1$ của B
    * Hàng $0$ của C
    * Hàng $1$ của C
    * Hàng $2$ của B
    * Hàng $3$ của B

1. Nhân hai phần tử tiếp theo của A với hai phần tử đầu tiên trong Hàng 4 và 5 của B. Cộng vào kết quả trong tile C đích.

    Nội dung cache[^not-lru]:
    * Hàng $3$ của A, nửa đầu
    * Hàng $4$ của A, nửa đầu
    * Hàng $0$ của C
    * Hàng $1$ của C
    * Hàng $4$ của B
    * Hàng $5$ của B

1. Nhân hai phần tử cuối cùng của A với hai phần tử đầu tiên trong Hàng 6 và 7 của B. Cộng vào kết quả trong tile C đích.

    Nội dung cache[^not-lru]:
    * Hàng $3$ của A, nửa đầu
    * Hàng $4$ của A, nửa đầu
    * Hàng $0$ của C
    * Hàng $1$ của C
    * Hàng $6$ của B
    * Hàng $7$ của B

:::

## Phân Tích Hiệu Suất

Xem hai phần sau:

* [Phương pháp 1](#sec-dgemm-sisd): Naive DGEMM
* [Phương pháp 2](#sec-dgemm-sisd): DGEMM Transpose
* [Phương pháp 3](#sec-dgemm-simd): DGEMM Tiled (với SIMD)

Phương pháp tiled cho matrix multiplication hiệu quả hơn nhiều khi chúng ta có thể tận dụng parallelism trong hardware.

## Tóm Tắt: Cache Blocking

Lưu ý rằng cache blocking vẫn có thể thay thế các hàng của $B$ sẽ cần sau này; nó không tránh được tất cả capacity misses. Tuy nhiên, nó giảm **tổng số memory accesses**, do đó giảm tổng số compulsory misses.

Từ P&H 4.4 cho các matrices vuông (N-by-N):

> Chỉ xem xét capacity misses, tổng số memory words được truy cập là 2 N{sup}`3`/`BLOCKSIZE` + N{sup}`2`. Tổng này là cải thiện khoảng một hệ số `BLOCKSIZE`.

Blocking khai thác kết hợp spatial và temporal locality:

* $A$ được hưởng lợi từ spatial locality
* $B$ được hưởng lợi từ temporal locality
* $C$ được hưởng lợi từ spatial locality (nhiều kết quả hơn của $C$ được tính cho cùng memory accesses đến $A$ và $B$).
