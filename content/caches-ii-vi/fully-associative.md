---
title: "Fully Associative Cache"
---

(sec-fully-associative-vi)=
## Mục tiêu học tập

* Sử dụng thuật ngữ, mô tả cách tìm một block trong fully associative cache: address, tag, và offset.
* Giải thích tại sao cần valid bit trong thiết kế cache.
* So sánh các replacement policy khác nhau: LRU, FIFO, và random.
* So sánh hai write policy: write-through và write-back. Tối ưu hóa chính sách sau với dirty bit.
* Với một mẫu truy cập bộ nhớ cho trước, xác định mỗi truy cập bộ nhớ là cache hit hay cache miss.

::::{note} 🎥 Video bài giảng: Fully Associative
:class: dropdown

:::{iframe} https://www.youtube.com/embed/-7TxNYUeFng
:width: 100%
:title: "[CS61C FA20] Lecture 26.3 - Caches III: Fully Associative Caches"
:::

::::

::::{note} 🎥 Video bài giảng: Block Replacement
:class: dropdown

:::{iframe} https://www.youtube.com/embed/a-ejTer1Ijg
:width: 100%
:title: "[CS61C FA20] Lecture 25.3 - Caches IV: Block Replacement with Example"
:::

Đến 7:00. Ví dụ giả định đã quen thuộc với set associative cache, một placement policy chúng ta thảo luận trong [phần sau](#sec-set-associative-vi)

::::

::::{note} 🎥 Video bài giảng: Writes
:class: dropdown

:::{iframe} https://www.youtube.com/embed/DiH8xtQeCJA
:width: 100%
:title: "[CS61C FA20] Lecture 25.3 - Caches III: Writes, Block Sizes, Misses"
:::

Đến 13:00

::::

::::{note} 🎥 Video bài giảng: Block Size
:class: dropdown

:::{iframe} https://www.youtube.com/embed/DiH8xtQeCJA
:width: 100%
:title: "[CS61C FA20] Lecture 25.3 - Caches III: Writes, Block Sizes, Misses"
:::

Đến 22:45

::::


Làm thế nào để thiết kế cache? [Từ phần trước](#sec-cache-terminology-vi):

:::{embed} #sec-cache-design-policy-vi
:::

Trong phần này, chúng ta giới thiệu **fully associative cache** và sử dụng nó như phương tiện để thảo luận ví dụ chi tiết và các đánh đổi giữa replacement policy và write policy.

## Placement policy

(sec-fully-associative-policy-vi)=
:::{note} Placement policy _Fully Associative_

Một block có thể được đặt vào bất kỳ entry nào của cache.

:::

**Associativity** đề cập đến các entry có thể mà một block dữ liệu cụ thể có thể được liên kết.

## Identification

Làm thế nào để xác định **cache hit** trên một địa chỉ bộ nhớ? Nói cách khác, làm thế nào để biết dữ liệu tại một địa chỉ bộ nhớ cụ thể có thể được truy cập từ một block trong cache? Từ _Computer Organization: A Quantitative Approach_ Phụ lục B.1:

> Cache có address tag trên mỗi block frame cho biết block address. Tag của mọi cache block có thể chứa thông tin mong muốn được kiểm tra để xem nó có khớp với block address từ processor hay không. Theo quy tắc, tất cả các tag có thể được tìm kiếm song song vì tốc độ là rất quan trọng.

Có rất nhiều trong đoạn văn này.[^block-description] Trước tiên chúng ta khám phá mối quan hệ giữa địa chỉ bộ nhớ và tag của cache entry. Sau đó chúng ta giải thích cách xác định cache hit.

[^block-description]: Ở đây, "block frame" có nghĩa là chính cache entry, "block" là đơn vị dữ liệu, và "block address" là thứ chỉ ra địa chỉ bộ nhớ của byte có trọng số thấp nhất của block này. Chúng ta sẽ định nghĩa chính thức "block address" trong [phần tiếp theo](#sec-block-address-vi).

### Tag và Offset

Chúng ta muốn kết nối các block trong cache như trong @fig-fa-intro-vi, là fully associative cache 16B với block 4B, với địa chỉ bộ nhớ 12-bit trong @fig-fa-address-vi. Chúng ta làm điều này bằng cách chia địa chỉ thành hai phần: **tag** và **offset**.

:::{figure} images/fa-intro.png
:label: fig-fa-intro-vi
:width: 50%
:alt: "Bảng cache hiển thị tag và data cho fully associative cache 16 byte. Mỗi trong bốn cache line có tag duy nhất. Byte 3 của cache line đầu tiên được tô sáng, hiển thị kết quả truy cập tag đó với offset là 3."
Cache tag và offset trong fully associative cache 16B cho địa chỉ bộ nhớ 12-bit. Các byte trong block của entry đầu tiên chia sẻ cùng 10 bit trên của địa chỉ bộ nhớ: `0b0100001111`, hoặc `0x10F`, đó là tag. Địa chỉ của byte có trọng số cao nhất trong block đầu tiên do đó là `0x43F`.
:::

:::{figure} images/fa-address.png
:label: fig-fa-address-vi
:width: 60%
:alt: "Phân chia trường địa chỉ bộ nhớ cho fully associative cache thành chỉ trường tag và block offset. Tag chiếm bit 11 đến 2, và offset chiếm bit 1 đến 0."
Đối với fully associative cache, địa chỉ bộ nhớ được chia thành hai trường: tag và offset. Đối với các block trong @fig-fa-intro-vi, địa chỉ bộ nhớ 12-bit được chia thành tag 10-bit và offset 2-bit.

:::

Vậy **tag** là gì? Nhớ lại rằng tất cả các byte trong mỗi block dữ liệu đều từ cùng một vùng bộ nhớ. Địa chỉ của chúng sẽ chia sẻ một tập hợp chung các bit trên. Trong **fully associative cache** như trong @fig-fa-intro-vi, tất cả các bit trên này được đặt vào **tag** liên kết với mỗi block.

Còn **offset** thì sao? Bộ nhớ có thể định địa chỉ theo byte, vì vậy mỗi byte trong một block cho trước sẽ có địa chỉ bộ nhớ khác nhau. Các địa chỉ bộ nhớ của các byte trong một block cho trước sẽ không thay đổi ở các bit trên (tag) mà thay đổi ở các bit thấp nhất. **Offset** là phần của địa chỉ cần thiết để mô tả sự biến đổi này.


:::{tip} Kiểm tra nhanh

Xem lại mối quan hệ giữa các phần địa chỉ trong @fig-fa-address-vi và fully associative cache 16B (với block 4B) trong @fig-fa-intro-vi.

1. Block size là bao nhiêu, tính bằng byte?
1. Tổng dung lượng dữ liệu trên tất cả các block là bao nhiêu, tính bằng byte?
1. Nếu không gian bộ nhớ của chúng ta là $2^{12}$ byte, chúng ta có địa chỉ 12-bit. Bao nhiêu bit của địa chỉ này nên được dành cho offset?
1. Vẫn giả sử địa chỉ 12-bit, bao nhiêu bit của địa chỉ này nên được dành cho tag?
:::

:::{note} Lời giải

1. **4 byte**. Block size (hay còn gọi là line size) là số byte trên mỗi block. Trong @fig-fa-intro-vi, mỗi cache entry có "hàng" dữ liệu 4 byte trong block của nó.
1. **16 byte.** Dung lượng dữ liệu là số byte trên tất cả các block. @fig-fa-intro-vi cho thấy 4 block, mỗi block có kích thước 4 byte.
1. Offset xác định byte offset để truy cập dữ liệu từ một block cho trước. Để "index" vào mỗi trong 4 byte trong một block cho trước, chúng ta cần $\log_2{(\text{block size})}$ = **2 bit**.
1. Tag của mỗi cache entry liên kết dữ liệu trong block đó với một (tập hợp) địa chỉ cụ thể. Tập hợp địa chỉ này có thể thay đổi về offset (2 bit thấp) nhưng sẽ chia sẻ cùng tag. Vì chúng ta sử dụng địa chỉ bộ nhớ 12-bit trong ví dụ nhỏ của mình, đối với fully associative cache này tag của chúng ta là (# bit địa chỉ) - (# bit offset) = **10 bit**.
:::

:::{note} Giải thích thêm về @fig-fa-intro-vi
:class: dropdown

Xem xét địa chỉ của mỗi trong bốn byte của cache block đầu tiên (với tag `0x10F`):

* Byte có trọng số thấp nhất (ngoài cùng bên phải trong block đầu tiên của @fig-fa-intro-vi cache) có byte offset `0b00`. Chúng ta tái tạo địa chỉ bộ nhớ bằng cách thêm tag 10-bit `0x10F` vào trước offset để có `0b01 0000 1111 00`, hoặc `0x43C`.
* Byte có trọng số thấp thứ hai có offset `0b01`. Thêm cùng tag 10-bit `0x10F` vào trước để có `0b01 0000 1111 01`, hoặc `0x43D`.
* Byte có trọng số cao thứ hai có offset `0b10`. Địa chỉ nhị phân `0b01 0000 1111 10`, hoặc `0x43E`.
* Byte có trọng số cao nhất (ngoài cùng bên trái và được tô sáng trong block đầu tiên của @fig-fa-intro-vi cache) có offset `0b11`. Địa chỉ nhị phân `0b01 0000 1111 11`, hoặc `0x43F`.

::::

(sec-valid-bit-vi)=
### Valid bit

Phải có cách để biết rằng một cache block không có thông tin hợp lệ. Ví dụ, khi khởi động một chương trình, cache nhất thiết không có thông tin hợp lệ cho chương trình. Quy trình phổ biến nhất là thêm một chỉ báo (tức là **flag**) vào tag để cho biết mỗi entry trong cache có hợp lệ cho chương trình cụ thể này hay không.

**Valid bit** chỉ ra tag cho block có hợp lệ hay không. Nếu valid bit được đặt (`1`), tag tham chiếu đến địa chỉ bộ nhớ hợp lệ. Nếu nó không được đặt (`0`), chúng ta không nên khớp với tag này (ngay cả khi các bit tag khớp ngẫu nhiên).

:::{figure} images/fa-valid.png
:label: fig-fa-valid-vi
:width: 50%
:alt: "Bảng fully associative cache cold với tất cả valid bit không được đặt và tất cả 16 byte dữ liệu trống."
Ảnh chụp [cold](#sec-cache-temperatures-vi) của fully associative cache trong @fig-fa-intro-vi, nơi valid bit cho tất cả block không được đặt (tức là đặt thành `0`). Chúng ta minh họa valid bit trong hình ảnh dạng bảng như một cột metadata bổ sung.[^valid-hardware]
:::

[^valid-hardware]: Từ _Computer Organization: A Quantitative Approach_, Phụ lục B.1: "...thêm _valid bit_ vào tag để cho biết entry này có chứa địa chỉ hợp lệ hay không."

(sec-fa-walkthrough-vi)=
## Hướng dẫn: Làm nóng Fully Associative Cache

Animation sau đây theo dõi năm lần truy cập bộ nhớ đến không gian địa chỉ 12-bit trên fully associative cache 16B với block 4B của chúng ta. Giả sử cache bắt đầu ở trạng thái [cold](#sec-cache-temperatures-vi), như trong @fig-fa-valid-vi.

::::{figure}
:label: fig-fa-warmup-vi
:alt: "Các slide nhúng đi qua năm lần truy cập bộ nhớ trên fully associative cache cold địa chỉ 12-bit với dung lượng 16 byte và block 4 byte."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vQU9l8AzNd5IkR1OUoqNtUygI4anobQJsrXgAXObfGH_eoSDy3iCIVqHMtZE8p-TvSI06dUoLIb8Y-z/pubembed?start=false&loop=false
:width: 100%
:title: "Các slide đi qua quá trình làm nóng fully associative cache từ phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1NxTminubfgSHzH2S_N7SxTyquI5B4QidA8b6W4mnRlU/edit?usp=sharing)"
:::

Làm nóng fully associative cache.
::::

Để đơn giản hóa cho bây giờ, nếu chúng ta gặp cache miss, chúng ta tải block mới từ memory vào một cache entry không hợp lệ. Chúng ta thảo luận _block replacement policy_ trong [phần tiếp theo](#sec-cache-replacement-policy-vi).

:::{note} 1. Load byte @ `0x43F`. Cache miss.
:class: dropdown

Địa chỉ `0x43F` trong nhị phân: `0b0100 0011 1111`

* Tag: `0b0100001111`, hoặc `0x10F`
* Offset: `0b11`

1. **Cache Miss**. Không có tag hợp lệ nào trong cache khớp với `0x10F`.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Tải vào cache entry được chọn một block dữ liệu từ memory bắt đầu @ địa chỉ @ `0x43C`. Ghi tag `0x10F`. Đánh dấu valid bit.

    Spatial locality: Ngay cả khi chúng ta chỉ đọc một byte, việc tải từ memory sẽ tải toàn bộ block (ở đây, 4B), nơi tất cả byte dữ liệu trong block chia sẻ cùng tag vì chúng từ cùng vùng bộ nhớ:

    * Byte có trọng số thấp nhất trong block (offset `0b00`) ở @ địa chỉ bộ nhớ `0x43C` (`0b0100 0011 1100`)
    * Byte có trọng số cao nhất trong block (offset `0b11`) ở @ địa chỉ bộ nhớ `0x43F` (`0b0100 0011 1111`)

1. **Đọc**. Đọc byte trong cache block tại offset `0b11` (tức là byte có trọng số cao nhất trong block) và trả về processor.

:::

:::{note} 2. Load byte @ `0x5E2`. Cache miss.
:class: dropdown

Địa chỉ `0x5E2` trong nhị phân: `0b0101 1110 0010`

* Tag: `0b0101111000`, hoặc `0x178`
* Offset: `0b10`

1. **Cache Miss**. Không có tag hợp lệ nào trong cache khớp với `0x178`.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Tải vào cache entry được chọn một block dữ liệu từ memory bắt đầu @ địa chỉ `0x5E0` (`0b0101 1110 0000`). Ghi tag `0x178`. Đánh dấu valid bit.
1. **Đọc**. Đọc byte trong block tại offset `0b10` và trả về processor.
:::

:::{note} 3. Load word @ `0x824`. Cache miss.
:class: dropdown

Địa chỉ `0x824` trong nhị phân: `0b1000 0010 0100`

* Tag: `0b1000001001`, hoặc `0x209`
* Offset: `0b00`

1. **Cache Miss**. Không có tag hợp lệ nào trong cache khớp với `0x209`.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Tải vào cache entry bốn byte dữ liệu bắt đầu @ `0x824` (`0b1000 0010 0100`). Ghi tag `0x209`. Đánh dấu valid bit.
1. **Đọc**. Đọc **word** trong cache block tại offset `0b00` và trả về processor.
:::

:::{note} 4. Load byte @ `0x5E0`. Cache hit.
:class: dropdown

Địa chỉ `0x5E0` trong nhị phân: `0b0101 1110 0000`

* Tag: `0b0101111000`, hoặc `0x178`
* Offset: `0b00`

1. **Cache Hit**. Tag được yêu cầu `0x178` khớp với cache tag **hợp lệ**.
1. **Đọc**. Đọc byte trong cache block tại offset `0b00` và trả về processor.
:::

:::{note} 5. Load byte @ `0x524`. Cache miss.
:class: dropdown

Địa chỉ `0x524` trong nhị phân: `0b0101 0010 0100`

* Tag: `0b0101001001`, hoặc `0x149`
* Offset: `0b00`

1. **Cache Miss**. Không có tag hợp lệ nào trong cache khớp với `0x149`.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Tải vào cache entry được chọn một block dữ liệu từ memory bắt đầu @ địa chỉ `0x524` (`0b0101 0010 0100`). Ghi tag `0x149`. Đánh dấu valid bit.
1. **Đọc**. Đọc byte trong cache block tại offset `0b00` và trả về processor.
:::

Trong **năm lần truy cập bộ nhớ** này:

* Ba lần truy cập bộ nhớ đầu tiên là cache miss, phải chịu độ trễ đắt đỏ của việc truy cập main memory.
* Lần truy cập bộ nhớ thứ tư là cache hit, vì vậy không có truy cập main memory xảy ra.
* Lần truy cập bộ nhớ cuối cùng cũng là cache miss.

(sec-cache-replacement-policy-vi)=
## Replacement Policy

Sau năm lần truy cập bộ nhớ trước đó, fully associative cache của chúng ta đã đầy dung lượng (tức là "full", vì tất cả cache entry đều hợp lệ), như trong @fig-fa-full-vi.

:::{figure} images/fa-full.png
:label: fig-fa-full-vi
:width: 60%
:alt: "Trạng thái fully associative cache sau một chuỗi truy cập cho thấy tất cả entry đều bị chiếm. Tất cả bốn hàng có valid bit được bật và tag duy nhất."

Sau năm lần truy cập bộ nhớ được mô tả [ở trên](#sec-fa-walkthrough-vi), fully associative cache nhỏ của chúng ta đã đầy.
:::

**Replacement policy** định nghĩa cách cache controller xác định block nào bị thay thế khi cache miss. Đối với fully associative cache, có nhiều tùy chọn cho replacement policy.

Một replacement policy tự nhiên được gọi là **least recently used**, hoặc viết tắt là **LRU**. Từ _Computer Organization: A Quantitative Approach_ Phụ lục B.1:

> Để giảm khả năng loại bỏ thông tin sẽ cần sớm, các lần truy cập vào block được ghi lại. Dựa vào quá khứ để dự đoán tương lai, block bị thay thế là block không được sử dụng lâu nhất. LRU dựa vào hệ quả của locality: Nếu các block được sử dụng gần đây có khả năng được sử dụng lại, thì ứng cử viên tốt để loại bỏ là block ít được sử dụng gần đây nhất.

:::{hint} Kiểm tra nhanh

Dựa trên năm lần truy cập bộ nhớ được mô tả [ở trên](#sec-fa-walkthrough-vi), block nào là ít được sử dụng gần đây nhất trong @fig-fa-full-vi?

:::

(sec-lru-implementation-vi)=
:::{note} Lời giải: Triển khai LRU có thể
:class: dropdown

Đáp án: Block với tag `0x10F`.

Một cách tiếp cận là nhìn vào các lần truy cập bộ nhớ cho đến khi bạn tìm ra. Một thuật toán khả thi khác là gán một số cho mỗi block để theo dõi lịch sử truy cập. Mỗi lần truy cập bộ nhớ, đặt lại số về 0 nếu tag hợp lệ và khớp (hoặc sau cache miss, tag được cập nhật khớp), và tăng số nếu tag hợp lệ nhưng không khớp. Entry có số _cao nhất_ như vậy[^ties] là block **ít** được sử dụng gần đây nhất.

[^ties]: Phá vỡ hòa theo cách hợp lý, ví dụ: ngẫu nhiên.

1. Load byte @ `0x43F`. Tag `0x10F` là 0.
2. Load byte @ `0x5E2`. Tag `0x10F` là 1, `0x178` là 0.
3. Load **word** @ `0x824`. Tag `0x10F` là 2, `0x178` là 1, `0x209` là 0.
4. Load byte @ `0x5E0`. Tag `0x10F` là 3, `0x178` là 0, `0x209` là 1.
5. Load byte @ `0x524`. Tag `0x10F` là 4, `0x178` là 1, `0x209` là 2, `0x149` là 0.

Tag có số cao nhất là `0x10F`.
:::

::::{figure}
:label: fig-fa-lru-vi
:alt: "Các slide nhúng minh họa LRU replacement trong fully associative cache khi các truy cập bổ sung lấp đầy các entry hợp lệ và loại bỏ tag ít được sử dụng gần đây nhất."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vQZlQhRUK7d3f5-2SHQnPPSsyHFmG2xZ5hgEr5p8PyNZLKBl7zVK8UojiqD6OipYsRVa5v7t-ChQjT7/pubembed?start=false&loop=false
:width: 100%
:title: "Các slide đi qua replacement policy cho fully associative cache trong phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1vHYBxCYjtFT8vSbVFioz_o1K4T2e0mNykhJ89-yactQ/edit?usp=sharing)"
:::

Fully associative cache với least recently used (LRU) replacement policy.
::::

:::{note} 6. Load byte @ `0x972`. Cache miss.
:class: dropdown

Địa chỉ `0x972` trong nhị phân: `0b1001 0111 0010`

* Tag: `0b1001011100`, hoặc `0x25C`
* Offset: `0b10`

1. **Cache Miss**. Không có tag hợp lệ nào trong cache khớp với `0x25C`.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Chọn entry ít được sử dụng gần đây nhất (tag `0x10F`). Thay thế block của nó bằng một block dữ liệu từ memory bắt đầu @ địa chỉ `0x970` (`0b1001 0111 0000`). Ghi tag `0x25C`. Đánh dấu valid bit.
1. **Đọc**. Đọc byte trong cache block tại offset `0b10` và trả về processor.
:::

Các replacement policy phổ biến:[^lifo]

1. **Least recently used (LRU)**: Chọn block ít được sử dụng gần đây nhất để thay thế.
1. **Random**: Chọn block ngẫu nhiên để thay thế.
1. **First in, first out (FIFO)**: Chọn block _cũ nhất_ để thay thế (ngay cả khi block cũ nhất được sử dụng gần đây nhất). Một queue (sử dụng thuật ngữ từ Data Structures).

Mặc dù việc triển khai replacement policy nằm ngoài phạm vi khóa học này, chúng ta lưu ý những điều sau:

* LRU gây ra chi phí phần cứng đáng kể vì cache phải theo dõi lịch sử truy cập.
* FIFO là một xấp xỉ hợp lý của LRU mà không thêm quá nhiều phần cứng bổ sung.[^fifo]
* Random replacement phân bổ việc thay thế đều trên toàn cache, vì vậy chính sách này hoạt động tốt đáng ngạc nhiên khi workload có temporal locality thấp.

[^lifo]: Một số bạn có thể thắc mắc: Tại sao không dùng Most Recently Used (MRU)? Tại sao không dùng last in, first out (LIFO)? Mặc dù LIFO xấp xỉ MRU, cả hai chính sách này đều hoàn toàn đi ngược lại ý tưởng của temporal locality và do đó hơi ngớ ngẩn.

[^fifo]: Một lần nữa, việc triển khai FIFO replacement policy nằm ngoài phạm vi. Đọc _Computer Organization: A Quantitative Approach_ Phụ lục B.1 để biết thêm chi tiết.

:::{table} So sánh các cache replacement policy.
:label: tab-cache-replacement-vi

| Tính năng | LRU | FIFO | Random |
| :-- | :-- | :-- | :-- |
| **Mô tả** | Thay thế entry không được sử dụng lâu nhất | Thay thế block cũ nhất trong set (queue) | Thay thế block ngẫu nhiên |
| **Triển khai** | Bộ đếm bit cho mỗi cache block; xem [kiểm tra nhanh](#sec-lru-implementation-vi) | FIFO queue hoặc xấp xỉ tương tự | Tung đồng xu phần cứng tượng trưng |
| **Ưu điểm** | Temporal locality | Xấp xỉ hợp lý của LRU | Dễ triển khai |
| **Nhược điểm** | Phần cứng phức tạp để theo dõi lịch sử truy cập | – | Tệ nếu workload tận dụng temporal locality cao |
:::

(sec-cache-write-policy-vi)=
## Write Policy

Cho đến nay, chúng ta chỉ tập trung vào **đọc** bộ nhớ với các lệnh load. Nhưng còn các lệnh store, **ghi** dữ liệu vào bộ nhớ thì sao?

Nhớ lại rằng cache block là **bản sao** của dữ liệu ở các tầng thấp hơn:

:::{embed} #sec-memory-hierarchy-copy
:::

Với cache, chúng ta cần đảm bảo rằng main memory của chúng ta sẽ (cuối cùng) đồng bộ với cache nếu chúng ta sửa đổi dữ liệu.
Có hai tùy chọn cơ bản khi ghi vào cache:

1. **Write-through**: Thông tin được ghi vào cả block trong cache _và_ đến vị trí tương ứng trong main memory tầng thấp hơn.
1. **Write-back**: Thông tin chỉ được ghi vào cache block. Cache block bị sửa đổi chỉ được ghi vào main memory tầng thấp hơn **khi** nó bị thay thế.

Để triển khai write-back, chúng ta lưu ý thêm rằng chúng ta chỉ cần ghi lại các block _bị sửa đổi_ vào main memory khi thay thế. Điều này giảm miss penalty. Để giảm tần suất ghi lại block khi thay thế, sử dụng **dirty bit** cho mỗi cache entry.

* Nếu dirty bit được đặt (`1`), block đã bị sửa đổi khi ở trong cache. Khi block này bị thay thế, khi miss, ghi lại block này vào main memory.
* Nếu dirty bit không được đặt (`0`), block chưa bị sửa đổi ("clean"). Khi block này bị thay thế khi miss, không cần ghi lại, vì thông tin giống hệt với cache được tìm thấy trong main memory tầng thấp hơn.

@fig-fa-wb-vi animate fully associative cache của chúng ta với write-back policy, sử dụng dirty bit

::::{figure}
:label: fig-fa-wb-vi
:alt: "Các slide nhúng animate write-back policy và hành vi dirty-bit trên fully associative cache trong quá trình hit, miss, và replacement."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vSBzBoel_J30XSTEfEQJDsk1OjVQ-Jnii6At7ZyqDKed1wADBar24FT5vhoT92bKNfIsKh5XVwz5rqc/pubembed?start=false&loop=false
:width: 100%
:title: "Các slide chi tiết write policy cho fully associative cache trong phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/15Nz0bRbUMH1EW45UmeayDK5D_D8eLSKCBL-CvlEjvls/edit?usp=sharing)"
:::

Animation Write back, với dirty bit.
::::

:::{note} 7. Store byte @ `0x524`. Cache hit.
:class: dropdown

Địa chỉ `0x524` trong nhị phân: `0b0101 0010 0100`

* Tag: `0b0101001001`, hoặc `0x149`
* Offset: `0b00`

1. **Cache Hit**. Tag được yêu cầu `0x149` khớp với cache tag **hợp lệ**.
1. **Ghi**. Ghi byte trong cache block tại offset `0b00`. Đặt dirty bit.

Trong write-back cache này, truy cập bộ nhớ này không gây ra block replacement (vì đó là cache hit). Do đó không có ghi vào memory.
:::

:::{table} So sánh các cache write policy.
:label: tab-cache-write-vi

| Tính năng | Write-through | Write-back |
| :-- | :-- | :-- |
| **Mô tả** | Ghi vào cache và memory cùng một lúc | Ghi vào cache. Chỉ ghi lại vào memory khi block bị thay thế. |
| **Triển khai** | Dễ | Phức tạp hơn |
| **"Đồng bộ hóa" với memory?**[^coherency] | Có | Không. Đôi khi cache sẽ có bản sao dữ liệu mới nhất. |
| **Tối ưu hóa** | – | Bao gồm **dirty bit** để chỉ ghi lại các block bị sửa đổi. |
| **Hit time cho ghi** | Mỗi lần ghi vào cache block cũng cần ghi vào memory. Dài hơn. | Nhiều lần ghi trong một cache block chỉ cần một lần ghi vào memory (việc sau chỉ xảy ra khi replacement). Nhanh hơn. |
| **Miss penalty** | "Đọc" miss nhanh và không bao giờ dẫn đến ghi vào memory. Ngắn hơn. | "Đọc" miss có thời gian thay đổi, vì cũng có thể cần ghi vào memory nếu block bị thay thế là dirty. Dài hơn trung bình. |
| **AMAT** | Dài hơn | Ngắn hơn |

[^coherency]: Còn được gọi là data coherency, hoặc cache coherency.
:::

Chúng ta nên sử dụng write policy nào? Nó phụ thuộc vào tầng của memory hierarchy. Chúng ta thảo luận điều này thêm sau với virtual memory.

Lưu ý cuối: May mắn thay, hầu hết các truy cập cache là **đọc**, không phải **ghi**. Đọc chiếm ưu thế trong các truy cập cache (tất cả các truy cập instruction đều là đọc; hơn nữa, hầu hết các instruction ghi vào register, không phải memory). Một block có thể được đọc từ cache cùng lúc với việc tag được đọc và so sánh. Thật không may, đối với ghi, việc sửa đổi block không thể bắt đầu cho đến khi tag được kiểm tra để xem địa chỉ có hit hay không. Tuy nhiên, processor không phải đợi ghi hoàn thành và có thể bắt đầu thực thi các instruction khác trong thời gian này (xem xét giai đoạn `MEM` của pipeline năm giai đoạn của chúng ta).

## Tổng kết Hướng dẫn

Trong phần này, chúng ta đã theo dõi thiết kế cache cho không gian địa chỉ 12-bit với các tính năng sau:

* Block size: 4B
* Capacity: 16B
* Placement policy: Fully associative
* Replacement policy: Least Recently Used
* Write policy: Write-back

:::{figure} images/fa-all.png
:label: fig-fa-all-vi
:width: 70%
:alt: "Bảng thiết kế hoàn chỉnh cho thấy ví dụ về fully associative cache. Mỗi trong bốn hàng trong cache có valid bit, dirty bit, giá trị LRU, tag, và bốn data byte."

Thiết kế của fully associative cache được mô tả trong ví dụ của phần này.
:::

"Metadata" của Cache:

* Tag: 10 bit
* Valid bit
* Phần cứng LRU (ví dụ: bộ đếm)
* Dirty bit (để tối ưu write-back)

### Block size và hiệu suất

Trong cache nhỏ của chúng ta với block kích thước 4B, đọc một word tương đương với đọc toàn bộ block, nhưng trong thực tế block được tạo thành từ nhiều word (ví dụ: 16 hoặc 32 word trên mỗi block).

Từ P&H 5.3: "Việc sử dụng block lớn hơn tận dụng spatial locality: nó giảm miss rate và cải thiện hiệu quả của phần cứng cache bằng cách giảm lượng lưu trữ tag so với lượng lưu trữ dữ liệu trong cache. Mặc dù block size lớn hơn giảm miss rate, nó cũng có thể tăng miss penalty. Nếu miss penalty tăng tuyến tính với block size, block lớn hơn có thể dễ dàng dẫn đến hiệu suất thấp hơn.

## Fully Associative: Phần cứng và Hiệu suất

Fully associative cache sử dụng placement policy linh hoạt nhất, cũng là tốn kém nhất để triển khai trong phần cứng. Từ P&H 5.4:

> Để làm cho việc tìm kiếm [block trong fully associative cache] thực tế, nó được thực hiện song song với một bộ so sánh được liên kết với mỗi entry. Các bộ so sánh này làm tăng đáng kể chi phí phần cứng, hiệu quả làm cho fully associative placement chỉ thực tế cho các cache có số lượng block nhỏ.

Như trong @fig-fa-hardware-vi, phần cứng tương đối đơn giản: Lấy tag từ địa chỉ (tức là xây dựng bundle dây cho các bit trên), sau đó sử dụng một bộ so sánh cho mỗi cache entry để so sánh tag của cache entry với tag được cung cấp. OR các kết quả so sánh này lại với nhau để xác định cache hit.

:::{figure} images/fa-hardware.png
:label: fig-fa-hardware-vi
:width: 100%
:alt: "Góc nhìn phần cứng nhấn mạnh các so sánh tag song song bổ sung cần thiết cho fully associative placement. Các hình chữ nhật mô tả cache tag, valid bit, và data."
Fully associative placement tăng chi phí phần cứng.
:::

Fully associative cache không phổ biến trong các processor hiện đại. Vì phần cứng bổ sung cần thiết, associativity cao hơn không chỉ tăng phần cứng mà còn tăng công suất.[^power] Chúng ta sẽ thấy các placement policy với associativity thấp hơn trong phần tiếp theo giảm số lượng bộ so sánh cần thiết _và_ giảm độ phức tạp của mỗi bộ so sánh bằng cách giảm độ rộng của cache tag.

[^power]: Đọc thêm trong _Computer Organization: A Quantitative Appproach_, 5th edition, Section 2.1.
<!--
## Visuals

:::{figure} images/fully-associative-cache-lru.png
:label: fig-fully-associative-cache-lru-vi
:width: 60%
:alt: "Fully associative cache table with least-recently-used replacement metadata."
A fully associative cache with LRU replacement policy.
:::

:::{figure} images/warmed-up-cache-can-still-miss.png
:label: fig-warmed-up-cache-can-still-miss-vi
:width: 60%
:alt: "Example showing that a warmed-up cache can still miss due to replacement and access pattern."
Even a fully warmed-up cache can still produce a miss.
:::

:::{figure} images/fully-associative-lru-write-back.png
:label: fig-fully-associative-lru-write-back-vi
:width: 60%
:alt: "Fully associative cache state using LRU replacement and write-back dirty tracking."
A fully associative cache using LRU replacement and a write-back policy.
:::

:::{figure} images/placement-policies.png
:label: fig-placement-policies-vi
:width: 75%
:alt: "Placement-policy spectrum from fully associative through set-associative to direct-mapped."
The spectrum of cache placement policies from fully associative to direct mapped.
:::
1-->
