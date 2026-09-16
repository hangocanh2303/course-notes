---
title: "Heap"
---

(sec-heap)=
## Mục tiêu học tập

* So sánh bộ nhớ động của heap với bộ nhớ stack.
* `free` mọi khối được cấp phát với `malloc`, `realloc`, hoặc `calloc`
* Thực hành đọc trang `man` Linux để xác định hành vi cụ thể của các hàm C `stdlib`.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/J6mhHw7UTPM?si=uK2zpU_N6wkfs5zv
:width: 100%
:enumerated: false
:title: "[CS61C FA20] Lecture 05.1 - C Memory Management: Dynamic Memory Allocation"
:::

Đến 9:36
::::

## Cấp phát bộ nhớ động trên heap

Lưu trữ động trên heap rất hữu ích khi chúng ta phải duy trì bộ nhớ bền vững, có thể thay đổi kích thước qua các lời gọi hàm, chẳng hạn như khi chúng ta viết chương trình để xây dựng cấu trúc dữ liệu thông qua các lời gọi hàm khác nhau.

Phép so sánh gần nhất trong Java là sử dụng từ khóa Java `new`, cấp phát bộ nhớ động cho một đối tượng. Tuy nhiên, Java có garbage collector chạy nền giải phóng các đối tượng được cấp phát động khi chúng không được sử dụng. Trong C, chúng ta cần giải phóng không gian trên heap thủ công; chúng ta thậm chí cần thay đổi kích thước không gian thủ công nếu hết. Những khác biệt chính này dẫn đến nguồn rò rỉ bộ nhớ và lỗi C lớn nhất trong các chương trình phức tạp hơn ngoài phạm vi của khóa học này.

Bộ nhớ trên heap được quản lý với sự trợ giúp của các hàm thư viện chuẩn C trong `stdlib.h`:

* `malloc`: **m**emory **alloc**ation (cấp phát bộ nhớ), tức là cấp phát một khối bộ nhớ trên heap
* `free`: giải phóng bộ nhớ trên heap
* `realloc`: tái cấp phát các khối bộ nhớ đã cấp phát trước đó thành khối bộ nhớ lớn hơn hoặc nhỏ hơn trên heap.

Hãy trước tiên mô tả hoạt động heap ở cấp độ cao, rồi đi sâu vào chữ ký hàm cho mỗi hàm này. Điều này sẽ thông tin về các cạm bẫy tiềm ẩn và chi tiết mà chúng ta không cần xem xét với bộ nhớ stack.

## Hoạt động Heap

Hãy trước tiên mô tả heap so với stack. Bộ nhớ động (tức là lưu trữ trong heap) có thể được cấp phát (`malloc`), thay đổi kích thước (`realloc`), và giải phóng (`free`) trong thời gian chạy chương trình. Do đó chúng ta không nhất thiết cần biết tất cả kích thước của các khối bộ nhớ trên heap tại thời điểm biên dịch; so sánh điều này với stack, nơi kích thước mỗi biến cục bộ phải được xác định trước để xác định kích thước của mỗi stack frame.

Stack là một pool bộ nhớ tương đối nhỏ, và các khối bộ nhớ được cấp phát như các stack frame, liền kề nhau. Ngược lại, heap thường **khổng lồ** — lớn hơn stack nhiều — và các khối bộ nhớ không nhất thiết được cấp phát theo thứ tự liền kề.

Vị trí chính xác của các khối bộ nhớ heap được ủy quyền cho một heap allocator tích hợp được triển khai trong thư viện chuẩn C (đọc thêm trong [phần tùy chọn](#sec-heap-allocator) của chương này). @fig-c-heap minh họa cách một heap allocator có thể xử lý bốn yêu cầu bộ nhớ.

1. Yêu cầu R1: `malloc` 100 byte không gian. Heap allocator tìm một khối thấp trong heap.
1. Yêu cầu R2: `malloc` 10 byte không gian. Heap allocator tìm một khối gần R1.
1. `free` 100 byte không gian từ yêu cầu R1. Heap allocator đánh dấu khối gốc đã giải phóng và có sẵn cho các yêu cầu tương lai.
1. Yêu cầu R3: `malloc` 50 byte không gian. Heap allocator có thể chọn cấp phát một phần của khối ban đầu được cấp phát cho R1, hoặc nó có thể chọn đặt nó ở một nơi hoàn toàn mới. Cả hai đều có nguy cơ **phân mảnh (fragmenting)** heap, ngăn chặn các khối liền kề cho các yêu cầu tương lai.

:::{figure} images/c-heap.png
:label: fig-c-heap
:width: 100%
:alt: "Dòng thời gian cấp phát heap bốn bước: yêu cầu R1 cấp phát 100 byte, yêu cầu R2 cấp phát 10 byte, R1 được giải phóng, rồi yêu cầu R3 cho 50 byte có thể được đặt trong một lỗ trống thấp hơn được tái sử dụng hoặc một vùng cao hơn riêng biệt. Hình minh họa sự lựa chọn của allocator và phân mảnh tiềm ẩn."

Một heap allocator xử lý các yêu cầu R1, R2, R3 và một `free`, có thể phân mảnh heap.
:::

Chúng tôi không mong đợi bạn biết cách một heap allocator quyết định cấp phát bộ nhớ ở đâu (xem [phần tùy chọn](#sec-heap-allocator) của chương này nếu bạn tò mò). Thay vào đó, biết rằng chúng ta phải cẩn thận: Đừng giả định *bất cứ điều gì* về các vị trí bộ nhớ chúng ta nhận lại từ các hàm heap. Chúng ta chỉ có thể tin tưởng rằng một yêu cầu đơn lẻ sẽ trả về một khối bộ nhớ liền kề; tuy nhiên, các yêu cầu bộ nhớ heap liên tiếp có thể dẫn đến các khối khá xa nhau.

## Các hàm C `stdlib` để quản lý heap

### `void *malloc(size_t n)`

`malloc` là một hàm nhận số byte bạn muốn và trả về một con trỏ đến **không gian chưa khởi tạo**. Nói cách khác, `n` byte bắt đầu từ con trỏ đó ban đầu chứa rác.

* **Tham số** `size_t n`: Một kiểu số nguyên không dấu đủ lớn để "đếm" byte bộ nhớ.
* **Trả về**: Con trỏ `void *`, tức là con trỏ đến không gian tổng quát (đọc thêm trong [phần sau](#sec-generics)). Giá trị trả về `NULL` cho biết không còn bộ nhớ khả dụng trên heap.

Giả định kích thước của đối tượng có thể dẫn đến mã gây hiểu lầm, không portable, vì vậy chúng ta sử dụng `sizeof` trong các ví dụ bên dưới[^explicit-typecast].

:::{card} Cấp phát một struct
^^^
```{code} c
:linenos:
typedef struct { ... } treenode_t; 
treenode_t *tp = malloc(sizeof(treenode_t)); 
if (!tp) { ... }
```
:::

:::{card} Cấp phát một mảng 20 `uint32_t`:
^^^
```c
uint32_t *ptr = malloc(20*sizeof(uint32_t));
if (!ptr) { ... }
```
:::

[^explicit-typecast]: **Typecast ngầm** được hiển thị trong phần này chuyển đổi giá trị trả về của `malloc` từ kiểu `(void *)` sang, giả sử, `(treenode_t *)` và giả định nó hoạt động. Trong **C hiện đại, typecast ngầm là ổn**. Tuy nhiên, trong pre-ANSI C, typecast con trỏ ngầm tạo ra cảnh báo, vì vậy cú pháp typecast **rõ ràng** như `treenode_t *tp = (treenode_t *) malloc(sizeof(treenode_t))` được ưa thích. Cuối cùng, C++ là một ngôn ngữ hoàn toàn khác, và các typecast con trỏ ngầm như vậy sẽ tạo ra lỗi. Những khác biệt này quan trọng cần nhớ khi bạn chuyển mã C giữa các hệ thống. Đọc thêm trên [StackOverflow](https://stackoverflow.com/questions/605845/should-i-cast-the-result-of-malloc-in-c/33047365#33047365).

**Kiểm tra `NULL`**: Chúng ta luôn muốn kiểm tra xem `malloc` có thành công không, để chúng ta có thể thoát an toàn thay vì crash. So sánh chế độ thất bại này với chế độ của `main`, nơi trả về số không là thành công. Ở đây, chúng ta biết có đúng một giá trị sẽ không bao giờ là địa chỉ bộ nhớ hợp lệ: `NULL`, mà khi truy cập sẽ crash chương trình của bạn.

**Typecast giá trị trả về của `malloc`**. Thông thường, chúng ta sẽ **typecast** con trỏ tổng quát được trả về thành con trỏ có kiểu. Điều này có nghĩa là chúng ta sẽ diễn giải địa chỉ được trả về bởi `malloc` như vị trí của một kiểu biến cụ thể. Ví dụ, nếu chúng ta đang cấp phát không gian heap cho một mảng `uint32_t`, typecast thành con trỏ `uint32_t *` sẽ thuận tiện cho số học con trỏ, truy cập mảng, và các phép toán số nguyên.

:::{tip} Kiểm tra nhanh
Xem xét mã cấp phát struct ở trên. Cái gì được cấp phát trên heap, nếu có? Còn trên stack, nếu có?
:::

:::{note} Hiển thị đáp án
:class: dropdown

* Dòng 1: `typedef struct` không cấp phát bộ nhớ. Nó chỉ định nghĩa `treenode_t`.
* Dòng 2, RHS (bên phải): `malloc(sizeof(treenode_t))` cấp phát `sizeof(treenode_t)` byte bộ nhớ trên heap.
* Dòng 2, LHS (bên trái): `treenode_t *tp` là một **biến cục bộ**. Khai báo này cấp phát ít nhất `sizeof(tp *)` (tức là kích thước của một con trỏ) lên stack frame. Chỉ dựa trên mã C, chúng ta không thể chỉ định chính xác bao nhiêu dữ liệu được cấp phát cho toàn bộ stack frame, nhưng nó phải ít nhất `sizeof(tp *)` trừ khi trình biên dịch C quyết định tối ưu với các thanh ghi phần cứng (thêm sau).
:::

### `void free(void *ptr)`

`free` là một hàm nhận một con trỏ trên heap để giải phóng.

* **Tham số `void * ptr`**: Một con trỏ chứa địa chỉ ban đầu được trả về bởi `malloc`/`realloc`.
* **Trả về**: Không có giá trị trả về.

Vì heap C không thực hiện garbage collection tự động, là lập trình viên C chúng ta phải luôn `free` bộ nhớ mà chúng ta cấp phát trên heap. Một phép so sánh hay được rút ra từ phim [_Godfather_ (1972)](https://www.imdb.com/title/tt0068646/)[^godfather]:

> ... Nó gần giống như đến gặp Bố Già và xin một ân huệ.
> 
> Bạn: [_chắp tay_] Bố Già...con muốn xin ngài một chút bộ nhớ...
> 
> Bố Già: [_vuốt mèo cưng_] Một ngày nào đó — và ngày đó có thể không bao giờ đến — ta có thể gọi con đến để làm một ân huệ cho ta. Nhưng cho đến ngày đó, ta sẽ cho con cái này với `malloc` theo hợp đồng rằng con phải free nó khi con xong...

[^godfather]: Xem video bài giảng để có ấn tượng [_Godfather_ (1972)](https://www.imdb.com/title/tt0068646/) hay. Timestamp 3:24

:::{card} Cấp phát/giải phóng bộ nhớ heap
^^^
```c
uint32_t *ptr = malloc(20*sizeof(uint32_t));
...
free(ptr); // typecast ngầm sang (void *)
```
:::

Nếu bạn không `free` bộ nhớ, bạn có nguy cơ **rò rỉ bộ nhớ** — nghĩa là bộ nhớ được cấp phát nhưng không bao giờ được truy cập trên heap, và bạn cuối cùng sẽ hết bộ nhớ. Đối với các chương trình chạy lâu như server, lỗi rò rỉ bộ nhớ có thể dẫn đến crash khó debug rất, rất lâu trong runtime.

Nếu bạn không `free` bộ nhớ _đúng cách_, chương trình của bạn sẽ crash hoặc hoạt động rất kỳ lạ sau đó, gây ra lỗi RẤT khó tìm ra. Đây là phân đoạn tương ứng của trang hướng dẫn Linux (gõ lệnh `man free`):

```
Hàm free() sẽ làm cho không gian được trỏ bởi ptr được
giải phóng; tức là, được làm cho khả dụng để cấp phát thêm. Nếu
ptr là con trỏ null, không có hành động nào xảy ra. Nếu không, nếu
đối số không khớp với con trỏ trước đó được trả về bởi một hàm
trong POSIX.1-2008 cấp phát bộ nhớ như bởi malloc(), hoặc nếu
không gian đã được giải phóng bởi lời gọi free() hoặc realloc(),
hành vi là không xác định.
```

Nói cách khác, khi bạn free bộ nhớ: truyền vào địa chỉ gốc được trả về từ `malloc`, và không "double free". Những đối số này sẽ tạo ra hành vi không xác định. Ở trên, đầu của khối heap được `malloc` là địa chỉ được lưu trong `ptr`. Trong khi truyền `ptr+1` vào `free` về mặt kỹ thuật vẫn trỏ đến đâu đó trong khối bộ nhớ này, tùy thuộc vào cách `free` được triển khai, `free(ptr+1)` có thể crash chương trình...hoặc tệ hơn...

Tại sao heap không kiểm tra những sai lầm này trong runtime? Trong C, cấp phát bộ nhớ đơn giản là quá quan trọng về hiệu năng đến mức không có thời gian để làm điều này. Kết quả thông thường là bạn bằng cách nào đó làm hỏng cấu trúc nội bộ của memory allocator, và bạn sẽ không phát hiện ra cho đến rất lâu sau đó trong một phần hoàn toàn không liên quan của mã. Nó giống như không đánh răng thường xuyên; bạn sẽ trả giá nhiều năm sau, và qua các triệu chứng không liên quan đến răng...

### `void *realloc(void *ptr, size_t size)`

`realloc` là một hàm thay đổi kích thước một khối đã cấp phát trước đó tại ptr thành kích thước mới. Khi làm điều đó, nó _có thể_ cần sao chép tất cả dữ liệu đến vị trí mới.

* **Tham số `void *ptr`**: Một con trỏ chứa địa chỉ ban đầu được trả về bởi `malloc`/`realloc`, HOẶC giá trị `NULL`.
* **Tham số `size_t size`**: Một kiểu số nguyên không dấu đủ lớn để "đếm" byte bộ nhớ.
* **Trả về**: Con trỏ `void *`, tức là con trỏ đến không gian tổng quát. Giá trị trả về `NULL` cho biết không còn bộ nhớ khả dụng trên heap.

Từ trang `man` Linux:

```
Nếu ptr là con trỏ null, realloc() sẽ tương đương với
malloc() cho kích thước được chỉ định.

Nếu ptr không khớp với con trỏ trước đó được trả về bởi calloc(),
malloc(), hoặc realloc() hoặc nếu không gian trước đó đã được
giải phóng bởi lời gọi free() hoặc realloc(), hành vi là
không xác định.
```

Mã bên dưới thảo luận từng điểm này:

```{code} c
:linenos:
uint32_t *ip;

/* 1 */
ip = realloc(NULL, 10*sizeof(uint32_t));
if(!ip) { ... } // kiểm tra NULL
...

/* 2 */
ip = realloc(ip, 20*sizeof(uint32_t));
if(!ip) { ... } // kiểm tra NULL
...

/* 3 */
realloc(ip, 0);
```

:::{note} Giải thích

Như với `malloc`, chúng ta luôn muốn kiểm tra xem con trỏ được trả về từ `realloc` có phải `NULL` không, nghĩa là chúng ta đã hết bộ nhớ. Chúng ta làm điều này cho cả Trường hợp 1 và 2.

1. Dòng 4: Tương đương với `ip = malloc(10*sizeof(uint32_t));`
1. Dòng 9: Tái cấp phát khối kích thước 10-`uint32_t` gốc thành một khối có thể lưu 20 `uint32_t`. Khi làm điều đó, khối được cập nhật giữ nguyên nội dung của 10 phần tử đầu tiên, hoặc bất cứ thứ gì trong khối gốc. Vì địa chỉ của khối được cập nhật có thể khác với khối gốc, cập nhật `ip` thành địa chỉ mới. Heap allocator sẽ free khối gốc nếu xảy ra relocation, vì vậy lập trình viên chỉ đơn giản chịu trách nhiệm free khối mới.
1. Dòng 14: Tương đương với `free(ip);` cho một số triển khai của `free`.
:::

### `void *calloc(size_t nelem, size_t elsize);`

Nhiều lập trình viên C thích sử dụng `calloc` để cấp phát bộ nhớ vì, không giống `malloc`, nó **khởi tạo** tất cả các bit trong khối được cấp phát thành không. Từ trang `man`:

```
Hàm calloc() sẽ cấp phát không gian chưa sử dụng cho một mảng
nelem phần tử, mỗi phần tử có kích thước tính bằng byte là elsize.
Không gian sẽ được khởi tạo thành tất cả bit 0.
```

Như với `malloc` và `realloc`, bất kỳ con trỏ nào được trả về bởi `calloc` luôn nên được kiểm tra `NULL` trước.

## Khi bộ nhớ Heap trở nên tồi tệ

Làm việc với heap là khó khăn. Bộ nhớ có thể được cấp phát / giải phóng bất cứ lúc nào! Một số cạm bẫy:

* **Rò rỉ bộ nhớ**: Bạn quên giải phóng bộ nhớ heap không sử dụng.
* **Sử dụng sau khi free**: Bạn free một khối bộ nhớ nhưng vẫn sử dụng con trỏ đó ở đâu đó sau này.
* **Double free**: Bạn free cùng bộ nhớ hai lần.

:::{tip} Kiểm tra nhanh

Có bao nhiêu lỗi quản lý bộ nhớ trong đoạn mã này? 0, 1, 2, 3, hoặc khác?

```{code}c
:linenos:
void free_mem_x() {
  int fnh[3];
  ...
  free(fnh); 
}

void free_mem_y() {
  int *fum = malloc(4*sizeof(int));
  free(fum+1);
  ...
  free(fum);
  ...
  free(fum);
  }
```
:::

:::{note} Hiển thị đáp án
:class: dropdown

Có 3 lỗi:

* Dòng 4: `free()` trên bộ nhớ được cấp phát trên stack
* Dòng 9: `free()` trên bộ nhớ không phải con trỏ từ `malloc()`
* Dòng 13: Double `free()`
:::

### Rò rỉ bộ nhớ / Thất bại `free()`

Runtime không kiểm tra việc lập trình viên thất bại trong việc quản lý bộ nhớ. Bộ nhớ quá quan trọng về hiệu năng đến mức các nhà thiết kế ngôn ngữ C ban đầu thực sự quyết định không có thời gian để kiểm tra và quản lý bộ nhớ trong nền. Kết quả thông thường của việc thất bại trong việc quản lý bộ nhớ của bạn là bạn làm hỏng cấu trúc nội bộ của heap allocator, và bạn phát hiện ra rất lâu sau đó trong một phần hoàn toàn không liên quan của mã!

**Rò rỉ bộ nhớ** là thất bại `free()` bộ nhớ đã cấp phát.

* Triệu chứng ban đầu. Không có gì. Cho đến khi bạn đạt đến điểm tới hạn, rò rỉ bộ nhớ thực sự không phải là vấn đề.
* Triệu chứng sau: Hiệu năng bí ẩn rơi xuống vực. Hành vi phân cấp bộ nhớ có xu hướng tuyệt vời cho đến khi nó không còn tuyệt vời, rồi nó đụng nhiều vách đá.
* ...và rồi chương trình của bạn bị giết! Vì hệ điều hành (OS) nói "không" khi bạn yêu cầu thêm bộ nhớ. Chúng ta thảo luận điều này khi nói về bộ nhớ ảo.

### Sử dụng sau khi free

Nhớ lại rằng **tham chiếu treo (dangling reference)** là khi bạn tiếp tục sử dụng một con trỏ, ngay cả sau khi nó đã được giải phóng. Trong mã bên dưới, Dòng 7 có một tham chiếu treo vì nó sử dụng `foo` sau khi nó đã được freed ở Dòng 5.

```{code} c
:linenos:
int *foo;
… 
foo = malloc(sizeof(int));
…
free(foo);
…
bar(foo); // !!!
```

Đọc sau khi free có thể bị hỏng! Nếu một thứ gì đó khác chiếm bộ nhớ đó, chương trình của bạn có thể sẽ đọc thông tin sai. Một tham chiếu treo đến bộ nhớ đã freed cũng có thể làm hỏng dữ liệu khác, làm chương trình của bạn crash _rất lâu_ sau đó.

### Double Free

Từ trang `man` (`man malloc`):

```
Hàm free() giải phóng không gian bộ nhớ được trỏ bởi ptr,
phải được trả về bởi lời gọi trước đó đến malloc(),
calloc(), hoặc realloc(). Nếu không, hoặc nếu free(ptr) đã
được gọi trước đó, hành vi không xác định xảy ra. Nếu
ptr là NULL, không có thao tác nào được thực hiện.
```

Bản chất chính xác của cái gọi là "hành vi không xác định" phụ thuộc vào chương trình và kiến trúc của bạn nhưng có thể bao gồm use-after-free (như trước) hoặc làm hỏng dữ liệu heap nội bộ (tùy thuộc vào cách heap allocator được triển khai trên máy của bạn).

### Quên realloc() có thể di chuyển dữ liệu

Từ trang `man` (`man malloc`):

```
Hàm realloc() trả về một con trỏ đến bộ nhớ mới được cấp phát,
được căn chỉnh phù hợp cho bất kỳ kiểu tích hợp nào, hoặc
NULL nếu yêu cầu thất bại. Con trỏ được trả về có thể là
giống như ptr nếu việc cấp phát không được di chuyển (ví dụ: có
chỗ để mở rộng cấp phát tại chỗ), hoặc khác với ptr
nếu việc cấp phát được di chuyển đến địa chỉ mới.
```

**Ví dụ**: Mã sau gọi `realloc` ở Dòng 3 mà không gán lại con trỏ `nums`. Trong trường hợp này, nếu `realloc` gán lại khối, thì `nums` bây giờ có thể trỏ đến bộ nhớ không hợp lệ; hơn nữa, chúng ta cũng sẽ không còn có con trỏ đến khối mới được cấp phát.

```{code} c
:linenos:
int *nums = malloc(10*sizeof(int));
int *g = nums;
realloc(nums, 20*sizeof(int));
```

Mã sau cố gắng sửa vấn đề trên bằng cách gán lại `nums` đúng cách ở Dòng 3. Tuy nhiên, vì `g` vẫn có giá trị cũ của `nums`, bây giờ `g` có thể trỏ đến bộ nhớ không hợp lệ, vì bây giờ không có đảm bảo rằng `g == nums`.

```{code} c
:linenos:
int *nums = malloc(10*sizeof(int));
int *g = nums;
nums = realloc(nums, 20*sizeof(int));
```

## Valgrind

Nói chung, để bắt các lỗi quản lý bộ nhớ, sử dụng các công cụ như [Valgrind](https://valgrind.org/). Valgrind làm chậm chương trình của bạn đi một bậc độ lớn, nhưng vô giá để kiểm thử và debug mã C. Nó thêm nhiều kiểm tra để bắt hầu hết các lỗi (nhưng không phải tất cả), bao gồm các trường hợp phổ biến nhất được mô tả trong chương này: rò rỉ bộ nhớ, lạm dụng `free()`, và ghi qua cuối các mảng.
