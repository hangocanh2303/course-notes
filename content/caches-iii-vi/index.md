---
title: "Cache Misses"
---

(sec-cache-misses-vi)=
## Mục Tiêu Học Tập

* Định nghĩa ba loại cache misses: compulsory miss, capacity miss, và conflict miss.
* Giải thích cách conflict miss giúp đánh giá cache associativity.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/DiH8xtQeCJA
:width: 100%
:title: "[CS61C FA20] Lecture 26.2 - Caches III: Writes, Block Sizes, Misses"
:::

::::

Phương pháp cổ điển để cải thiện hiệu suất cache là giảm **miss rates**.

Để hiểu rõ hơn về nguyên nhân gây ra misses, trước tiên chúng ta phân loại misses thành ba danh mục:

* **Compulsory Miss**: Xảy ra do lần truy cập đầu tiên vào một line chưa từng có trong cache. Lần truy cập đầu tiên vào một block _không thể_ có trong cache, nên block phải được nạp vào. Còn được gọi là cold-start misses hoặc first-reference misses.
* **Capacity Miss**: Xảy ra khi cache không thể chứa tất cả các lines cần thiết trong quá trình thực thi chương trình. Xuất hiện khi các lines đã ở trong cache, bị thay thế, và sau đó được truy xuất lại.
* **Conflict Miss**: Nhiều lines cạnh tranh cùng một vị trí trong cache, ngay cả khi cache chưa đầy hoàn toàn. Những misses này còn được gọi là _collision_ misses.

## Phân Loại Misses

Để phân loại misses, chúng ta có thể chạy một address trace trên một tập các caches, sau đó thực hiện các bước sau[^thanks-kubi]:

1. Đầu tiên, xem xét một cache fully-associative kích thước vô hạn. Mọi miss xảy ra lúc này được coi là **compulsory miss**.

1. Tiếp theo, xem xét một cache kích thước hữu hạn (kích thước bạn muốn kiểm tra) với full-associativity. Mọi miss không thuộc #1 là **capacity miss**.

1. Cuối cùng, xem xét một cache kích thước hữu hạn với finite-associativity. Tất cả các misses còn lại không thuộc #1 hoặc #2 là **conflict misses**.

    Conflict misses chỉ xảy ra với cache **set-associative** hoặc **direct-mapped**. Conflict misses là những miss xảy ra khi chuyển từ fully associative sang, ví dụ, 8-way associative, 4-way associative, và cứ tiếp tục như vậy.

[^thanks-kubi]: Cảm ơn Giáo sư Kubiatowicz về thuật toán này.

:::{warning} Non-compulsory misses

Trong khóa học này, chúng ta chỉ phân biệt giữa compulsory và non-compulsory misses. **Non-compulsory misses** được định nghĩa là các misses không phải compulsory, tức là **capacity miss** hoặc **conflict miss**.

Bạn vẫn cần biết định nghĩa của capacity/conflict misses, nhưng bạn sẽ không bị yêu cầu phân loại bất kỳ non-compulsory miss cụ thể nào vào một trong hai danh mục con này. Chúng ta sẽ thấy những sắc thái về lý do tại sao ở cuối phần này.

:::

:::{hint} Kiểm Tra Nhanh 1

Trong ba loại miss, loại nào có thể xảy ra với fully associative caches?
:::

:::{note} Hiển Thị Đáp Án
:class: dropdown

* Compulsory misses
* Capacity misses
:::

:::{hint} Kiểm Tra Nhanh 2

Trong ba loại miss, loại nào có thể xảy ra với direct-mapped caches?
:::

:::{note} Hiển Thị Đáp Án
:class: dropdown

* Compulsory misses
* Capacity misses
* Conflict misses

:::

Định nghĩa chính xác hơn một chút về **conflict misses** do đó là:

> Các misses sẽ không xảy ra nếu cache là fully-associative và có LRU replacement, (với tất cả các noncompulsory misses khác là capacity).

[^wrl]: Norman P. Jouppi. "WRL Technical Note TN-53: Reducing Compulsory and Capacity Misses." August 1990. [Source](https://bitsavers.org/pdf/dec/tech_reports/WRL-TN-53.pdf)

Chúng ta sẽ thấy trong phần tiếp theo rằng định nghĩa này không hoàn toàn chính xác cho các memory accesses cụ thể, nhưng nó đủ để phân loại conflict miss _rates_ nói chung.

## Conflict Misses, Chi Tiết

Phần này hữu ích nếu bạn vẫn còn bối rối về sự khác biệt giữa hai loại non-compulsory misses: capacity misses và conflict misses. Tuy nhiên, chi tiết chính xác của ví dụ nằm ngoài phạm vi.

### Ví Dụ

Hãy so sánh các hit patterns của một vài loại cache.
Sơ đồ dưới đây cho thấy hit/miss pattern của các caches khác nhau khi chạy trên Ví dụ [Matrix Multiply](#sec-cache-blocking-vi).

1. FA 1M (Fully Associative, 1 triệu blocks. Đây là cache "vô hạn" của chúng ta với số hits tối đa có thể)
1. FA 4 blocks (Fully Associative)
1. 2SA 4 blocks (Set Associative)
1. DM 4 blocks (Direct Mapped)

@fig-cache-pedantic-misses-vi minh họa mỗi memory access như các cột từ trái sang phải. Mỗi cột là hit hoặc miss trong mỗi cache trong bốn caches.

:::{figure} images/cache-pedantic-misses.png
:label: fig-cache-pedantic-misses-vi
:alt: "Cache-access sequence graphic marking misses in red and hits in white to contrast miss categories for fully associative cache with 1M blocks, fully associative cache with 4 blocks, 2-way-set-associative cache with 4 blocks, and direct mapped cache with 4 blocks."

Cache misses được đánh dấu màu đỏ, và cache hits trong suốt (màu trắng).
:::

Khi chúng ta đi qua từng loại trong ba cache misses theo định nghĩa, chúng ta nhận ra có một danh mục "thứ tư" của misses mà chúng ta chưa đề cập. Xem animation bên dưới.

::::{figure}
:label: anim-cache-pedantic-misses-vi
:alt: "Embedded slides walking through conflict misses and equivalent fully associative versus lower-associativity cache behavior for the pedantic example in this section."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vR259Lvwi0mq_JUcQbeLD7oLLF-ZvFn1PBihG3YMV09mwtIhw0U08XBnmQXHDced7-vGWrc9RTXOc8B/pubembed?start=false&loop=false
:width: 100%
:title: "Slides walking through conflict misses for cache in this section. Access [original Google Slides](https://docs.google.com/presentation/d/1nL8eIuc16BVBKdSvs-z3B5huPgI5AHZy3s6wT7nfu4A/edit?usp=sharing)"
:::
::::

Định nghĩa hiện tại của conflict misses giả định rằng tất cả misses trong FA cache tương đương cũng gây ra misses trong các associativities thấp hơn. Như chúng ta thấy trong ví dụ, đôi khi chúng ta nhận được "conflict hits" vì một block lẽ ra đã bị thay thế lại ở trong cache lâu hơn. Hành vi này phụ thuộc vào replacement policy được chọn và thứ tự chính xác của các memory accesses.

### Trong Thực Tế: Conflict Miss Rates

Trong thực tế, các kiến trúc sư máy tính **không** xem xét từng instruction để phân tích loại miss. Điều này tốn kém và có vẻ không cần thiết. Thay vào đó, hiệu suất memory access được phân tích dưới dạng **miss rates** tổng hợp so với các program benchmarks.

Định nghĩa **conflict miss rate** là tổng miss rate đo được, trừ miss rate do compulsory misses, trừ fully associative LRU non-compulsory miss rate.

Đây là định nghĩa mà hầu hết các benchmarkers sử dụng vì một số lý do:

* Dễ tính toán.
* Định nghĩa không phụ thuộc vào misses của bất kỳ memory access cụ thể nào, mà là các misses tổng hợp phát sinh trên program benchmark của chúng ta trên các thiết kế cache khác nhau.
* Định nghĩa này thực sự đặc trưng cho những gì chúng ta quan tâm: **cách associativity ảnh hưởng đến hiệu suất cache** thông qua cache misses: không phải miss do cold caches, cũng không phải miss vì chúng ta đã tối đa hóa associativity.

Để xem một số ví dụ, hãy xem phần [cache optimizations](#sec-cache-optimizations-vi) của chúng ta.

### Khả Năng Định Nghĩa Lại

Phần này nằm ngoài phạm vi. Nếu chúng ta thực sự muốn chính xác về misses, chúng ta _có thể_ thảo luận về định nghĩa thứ hai của conflict misses theo memory access:

Định nghĩa conflict miss là miss sẽ không xảy ra nếu cache là fully associative theo ít nhất một replacement policy "nhất quán". Định nghĩa replacement policy "nhất quán" là policy giữ tất cả data trong cache lower-associativity, tức là bất kỳ replacement policy nào sao cho data trong fully associative cache là superset của data trong target cache của chúng ta. Định nghĩa này đảm bảo chúng ta không có "conflict hits", nhưng cũng có xu hướng phân loại misses nhiều hơn là capacity misses hơn là conflict misses.

Trong thực tế, định nghĩa thứ hai này không được sử dụng. Một lần nữa, trong performance chúng ta quan tâm nhiều hơn đến runtime **trung bình** và các danh mục misses **tổng thể**. Vì vậy định nghĩa đầu tiên là định nghĩa bạn nên nhớ nếu bạn muốn đánh giá hiệu suất memory access.
