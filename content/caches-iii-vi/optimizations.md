---
title: "Cache Optimizations"
---

(sec-cache-optimizations-vi)=
## Mục Tiêu Học Tập

* Xem xét cách giảm các thành phần miss rate và miss penalty trong công thức average memory access time.
* Xem xét các đánh đổi với cache optimizations: block size, cache size, associativity, và multilevel caches.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/DiH8xtQeCJA
:width: 100%
:title: "[CS61C FA20] Lecture 26.2 - Caches III: Writes, Block Sizes, Misses"
:::

::::

Công thức average memory access time cung cấp cho chúng ta một framework để trình bày các cache optimizations nhằm cải thiện hiệu suất cache:

:::{embed} #eq-amat
:::

Chúng ta có thể tổ chức các cache optimizations cơ bản theo cách nó ảnh hưởng đến từng trong ba thành phần trong phương trình này:

* Giảm miss rate
* Giảm miss penalty
* Giảm hit time

Các kỹ thuật cache optimization trong chương này cải thiện hiệu suất ở một trong các danh mục trên, nhưng thường phải trả giá bằng việc làm giảm các danh mục khác. @tab-cache-optimizations-vi tóm tắt độ phức tạp và lợi ích hiệu suất.

:::{table} Điều chỉnh từ Figure B.18 trong _Computer Organization: A Quantitative Approach_. Tóm tắt các cache optimizations cơ bản cho thấy ảnh hưởng đến hiệu suất cache và độ phức tạp. Thông thường một kỹ thuật chỉ giúp một yếu tố. (+) có nghĩa là kỹ thuật _cải thiện_ yếu tố đó (_giảm_ miss rate, hit time, hoặc miss penalty), (-) có nghĩa là nó làm giảm yếu tố đó, và ô trống có nghĩa là không có ảnh hưởng.
:label: tab-cache-optimizations-vi

| Kỹ Thuật | Hit Time | Miss Penalty | Miss rate |
| :-- | :-- | :-- | :-- |
| Larger block size | | – | + |
| Larger cache size | – | | + |
| Higher associativity | – | | + |
| Multilevel caches | | + | |
| Avoiding address translation during cache indexing[^later] | + | | |
:::

[^later]: Đọc về virtual memory. Sắp có.

Trước tiên chúng ta lưu ý rằng trong các [cache design policies](#sec-cache-design-policy), [replacement policy](#sec-cache-replacement-policy) và [write policy](#sec-cache-write-policy) không được coi là các cache optimizations phổ biến. Thay vào đó, trong thực tế các caches hầu như luôn được thiết kế với LRU (hoặc FIFO) và write-back policies; các lựa chọn thay thế khác không được coi là hợp lý. Do đó chúng ta tập trung thảo luận trong phần này vào các nguyên tắc thiết kế khác và cache size/capacity.

## Larger Block Size

Các blocks lớn hơn tận dụng spatial locality và do đó sẽ giảm compulsory misses. Đồng thời, các blocks lớn hơn có thể tốn kém vì một số lý do, như được hiển thị trong @fig-figurative-block-size-vi:

* **Miss penalty**: Với các blocks lớn hơn, có nhiều thứ cần đọc từ memory khi có compulsory miss (và ghi vào memory khi replacement). Các blocks lớn hơn sẽ tăng miss penalty.
* **Miss rate**: Ban đầu, các blocks lớn hơn sẽ giảm miss rate vì chúng cải thiện **spatial locality**. Các blocks lớn hơn giảm số lượng blocks trong cache, nên các blocks lớn hơn có thể tăng conflict misses và thậm chí capacity misses nếu cache nhỏ. Nói cách khác, có ít blocks hơn có thể làm giảm **temporal locality** và (ở một điểm uốn nào đó) tăng miss rate.
* **AMAT**: Vì AMAT kết hợp cả hai thành phần này, tăng block size quá một điểm nhất định có thể _tăng_ AMAT.

:::{figure} images/figurative-block-size.png
:label: fig-figurative-block-size-vi
:alt: "Three conceptual trend graphs compare miss penalty, miss rate, and AMAT versus block size. The miss-penalty graph rises with block size, the miss-rate graph decreases at first and then increases for larger blocks, and the AMAT graph decreases initially before rising again."

Mặc dù các blocks lớn hơn ban đầu có thể giảm miss rate vì chúng cải thiện spatial locality, chúng cũng giảm số lượng blocks trong cache và có thể làm giảm temporal locality. Quá một điểm nhất định, average memory access time sẽ tăng với block size lớn hơn.
:::

@fig-data-block-size-vi cho thấy sự đánh đổi miss rate chi tiết hơn, được đo trên một benchmark đầu những năm 2000.

:::{figure} images/data-block-size.png
:label: fig-data-block-size-vi
:alt: "Empirical miss-rate versus block-size plot for multiple cache sizes. The plot compares 1KB, 8KB, 16KB, 64KB, and 256KB caches. Each trend line decreases as block size increases from 16 bytes to 64 bytes, but then increases as the block size increases further from 64 bytes to 256 bytes."

Miss rate so với block size cho bốn caches khác nhau. Lưu ý rằng miss rate thực sự tăng nếu block size quá lớn so với cache size. Phiên bản điều chỉnh từ Figure B.10 trong _Computer Organization: A Quantitative Approach_.
:::

## Larger Cache Size

Từ _Computer Organization: A Quantitative Approach_, Appendix B.3:

> Cách rõ ràng để giảm capacity misses [trong @fig-data-associativity-vi] là tăng capacity của cache. Nhược điểm rõ ràng là có thể hit time dài hơn và chi phí và công suất cao hơn. Kỹ thuật này đặc biệt phổ biến trong off-chip caches.

Trong @fig-data-associativity-vi, chúng ta thấy rằng khi cache size tăng, miss rate giảm cho tất cả các cache associativities.

Như một ví dụ hiện đại, chip M1 có L2 cache kích thước 12 MiB—lớn hơn đáng kể so với L1 cache với 128 KiB.[^apple-m1]

(sec-higher-associativity-vi)=
## Higher Associativity

Như đã thảo luận [trước đó](#sec-cache-misses-vi), higher associativity giảm miss rate bằng cách giảm compulsory misses.

_Computer Organization: A Quantitative Approach_ đo miss rate trên các caches có kích thước và associativities khác nhau, như được hiển thị trong @fig-data-associativity-vi. Direct-mapped cache (1-way set associative) có miss rate cao nhất trên các cache sizes.

:::{figure} images/data-associativity.png
:label: fig-data-associativity-vi
:alt: "Total miss-rate versus cache size plot for varying cache associativities. All trend lines decrease in miss rate as cache size increases in KB."

Total miss rate cho mỗi cache size, theo associativity. Lưu ý rằng cùng nhau, compulsory misses và capacity misses (đường nét đứt) theo định nghĩa là miss rate của fully associative cache. Phiên bản điều chỉnh từ Figure B.9 trong _Computer Organization: A Quantitative Approach_.
:::

Cũng như đã thảo luận trước đó, [fully associative caches](#sec-fully-associative) làm phức tạp hardware và do đó có thể tăng hit time. Tuy nhiên, trong các processors hiện đại, L1 caches có associativity hợp lý, ví dụ, 4-way hoặc 8-way.[^associativity] [^apple-m1]

[^associativity]: Tôi không có nguồn chính xác cho điều này, nhưng đây là một số đoạn trích: _Computer Organization: A Quantitative Approach_ nói, "Trong nhiều processors gần đây, các designers đã chọn associativity cao hơn thay vì caches lớn hơn" (Section 2.2). Phần này tiếp tục đề cập rằng ARM Cortex-A8 (2005) sử dụng four-way set associative caches.

[^apple-m1]: Yu et al. (SEC 2023) Table 1 báo cáo rằng chip Apple M1 P(erformance)-core có L1 data cache 8-way, 128-KiB (64B blocks, 256 sets) và L2 caches 12-way, 12-MiB (128B blocks, 8192 sets) 16-way. [presentation](https://www.usenix.org/conference/usenixsecurity23/presentation/yu-jiyong), [paper](https://www.usenix.org/system/files/usenixsecurity23-yu-jiyong.pdf).

## Multilevel Caches

:::{note} Xem lại phần [AMAT](#sec-amat) của chúng ta

Trong phần trước này, chúng ta phân tích cách thêm L2 cache cải thiện AMAT bằng cách giảm average miss penalty.
:::

Chúng ta lưu ý rằng triển khai multilevel caches tốn không gian hardware (xem [phần trước](#sec-multi-level-caches) về không gian cache trên [Apple A14 Bionic Chip](#fig-apple-a14). Khi chúng ta giới thiệu [các thuật ngữ cache chính](#sec-cache-terminology), chúng ta cũng lưu ý trong một footnote rằng chip Apple M1 có các blocks kích thước khác nhau cho L1 cache và L2 cache. Điều này cũng làm phức tạp hardware. Tuy nhiên, multilevel caches được sử dụng rộng rãi trong các processors hiện đại xử lý nhiều loại ứng dụng khác nhau.

## Các Kỹ Thuật Khác

Các kỹ thuật khác chắc chắn tồn tại nhưng nằm ngoài phạm vi của khóa học này–hoặc ít nhất, phạm vi bao phủ hiện tại của chúng ta về chủ đề. Hai kỹ thuật bổ sung được đề cập trong _Computer Organization: A Quantitative Approach, B.3":

1. **Avoiding address translation when indexing the cache.** Chúng ta sẽ thảo luận sau[^later] khi giới thiệu virtual memory. Phương pháp này giảm hit time và được sử dụng rộng rãi.
2. (ngoài phạm vi) Ưu tiên read misses hơn writes. Nếu read miss xảy ra cũng gây ra memory write (tức là, dirty block phải được ghi lại vào cache), triển khai write buffer. Bằng cách này, khi có read miss, copy dirty block vào write buffer, sau đó đọc memory, _rồi_ ghi memory từ buffer.
