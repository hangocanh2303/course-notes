---
title: "Average Memory Access Time (AMAT)"
short_title: "Average Memory Access Time"
---

## Mục Tiêu Học Tập

* Định nghĩa hit rate, hit time, miss rate, và miss penalty.
* Sử dụng công thức average memory access time (AMAT) để so sánh các thiết kế multi-level cache.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/pCZUG0TzwjE
:width: 100%
:title: "[CS61C FA20] Lecture 27.3 - Caches IV: Average Memory Access Time (AMAT)"
:::

::::

Vì hiệu năng là lý do chính cho memory hierarchy, điều quan trọng là đo lường thời gian để phục vụ hit hoặc miss. Do đó chúng ta định nghĩa các thuật ngữ sau trong @tab-cache-terminology-vi:

:::{table} Thuật ngữ cache quan trọng
:label: tab-cache-terminology-vi

| Kết Quả Request | Tỷ Lệ | Thời Gian |
| :-- | :-- | :-- |
| **Cache Hit** | **Hit rate**: tỷ lệ truy cập hit trong cache. | **Hit time**: thời gian (latency) để truy cập cache memory, bao gồm thời gian cần để xác định liệu truy cập là hit hay miss. |
| **Cache miss** | **Miss rate**: 1 - hit rate. | **Miss penalty**: Thời gian để thay thế một line bằng line tương ứng từ tầng thấp hơn của memory hierarchy.|
:::

Vì cache nhỏ hơn và được xây dựng bằng các bộ phận bộ nhớ nhanh hơn, hit time sẽ nhỏ hơn nhiều so với miss penalty, bao gồm thời gian truy cập tầng tiếp theo trong hierarchy.

(sec-amat-vi)=
## Average Memory Access Time

Thời gian truy cập dữ liệu cho cả hit và miss đều ảnh hưởng đến hiệu năng. Các nhà thiết kế đôi khi sử dụng **average memory access time** (AMAT) như một cách để so sánh các thiết kế cache. Từ P&H 5.4:

> Average memory access time là thời gian trung bình để truy cập bộ nhớ xem xét cả hit và miss và tần suất của các loại truy cập khác nhau.

```{math}
:label: eq-amat-vi
:enumerated: true

\text{AMAT} = \text{Hit Time} + \text{Miss Rate} \times \text{Miss Penalty}
```

Chúng ta sẽ sử dụng các giả định sau trong khóa học này:

* Khi **cache miss**, tổng thời gian để lấy dữ liệu là tổng của hit time **cộng** miss penalty.
* Miss rate của lower-level cache (ví dụ, L2) là tỷ lệ miss từ higher-level cache (ví dụ, L1) mà _cũng_ miss trong lower-level cache này.

::::{exercise} Chỉ có L1 cache
:label: ex-amat-l1-only-vi

Xem xét thiết kế memory hierarchy trong @fig-amat-l1-only-vi:

:::{figure} images/amat-l1-only.png
:label: fig-amat-l1-only-vi
:width: 80%
:alt: "Rectangle representing CPU with L1 cache and processor. Blue arrows show the processor accessing L1 cache and the return from L1 cache on a hit. A second rectangle on the right shows DRAM, connected to the L1 cache inside the CPU with a green access arrow when there is a miss from L1 cache, including the return arrow incurring the miss penalty for accessing outside the CPU box."

Memory hierarchy chỉ với một L1 cache.
:::

Các thuộc tính hiệu năng cache:

* L1 Hit Time = 1 cycle
* L1 Miss rate = 5%
* L1 Miss penalty = 200 cycles

Average Memory Access Time là bao nhiêu, tính bằng cycles?
::::

::::{solution} ex-amat-l1-only-vi
:label: ex-amat-l1-only-sol-vi

Sử dụng Phương trình @eq-amat-vi:

:::{math}
\begin{aligned}
\text{AMAT} &= \text{L1 Hit Time} + \text{L1 Miss Rate} \times \text{L1 Miss Penalty} \\
&= 1 + 0.05 \cdot 200 \\
&= 11 \text{ cycles} \\
\end{aligned}
:::

Khi miss penalty xảy ra, chúng ta _vẫn chịu hit time khứ hồi_. @fig-amat-l1-only-tree-vi minh họa hai trường hợp.

:::{figure} images/amat-l1-only-tree.png
:label: fig-amat-l1-only-tree-vi
:width: 40%
:alt: "Probability tree for L1-only cache with AMAT calculation for both hit and miss branches and their cycle costs."

Phân tích hiệu năng cache một tầng. 95% thời gian, chúng ta chịu 1 cycle delay để truy cập L1 cache. 5% thời gian, chúng ta chịu 201 cycles delay (để truy cập L1 cache và truy cập memory).
:::

::::

::::{exercise} L1 và L2 cache
:label: ex-amat-l1-l2-vi

Bây giờ, xem xét việc chèn L2 cache vào hierarchy, như được hiển thị trong @fig-amat-l1-l2-vi:

:::{figure} images/amat-l1-l2.png
:label: fig-amat-l1-l2-vi
:width: 100%
:alt: "Two-level memory hierarchy diagram with rectangles for CPU with internal L1 cache, L2 cache, and main memory (DRAM). Access arrows show an access of L1 cache with L1 hit time between processor and L1 cache, an access of L2 cache with L2 hit time between L1 and L2 caches, and an access of DRAM with miss penalty between DRAM and L2 cache."
Memory hierarchy với L1 cache và L2 cache.
:::

Các thuộc tính hiệu năng cache:

* L1 Hit Time = 1 cycle
* L1 Miss rate = 5%
* L2 Hit Time = 5 cycles
* L2 Miss rate = 15%
* L2 Miss penalty = 200 cycles

Average Memory Access Time là bao nhiêu, tính bằng cycles?

::::

::::{solution} ex-amat-l1-l2-vi
:label: ex-amat-l1-l2-sol-vi

Dựa trên [các giả định AMAT](#sec-amat-vi), miss rate của L2 cache là tỷ lệ miss từ L1 cache mà _cũng_ miss trong L2 cache.

Chúng ta có thể sử dụng Phương trình @eq-amat-vi đệ quy:

```{math}
\begin{aligned}
\text{AMAT} &= \text{L1 Hit Time} + \text{L1 Miss Rate} \times \text{L1 Average Miss Penalty} \\
&= \text{L1 Hit Time} + \text{L1 Miss Rate} \times \bigl(\text{L2 Hit Time} + \text{L2 Miss Rate} \times \text{L2 Miss Penalty}\bigr) \\
&= \text{L1 Hit Time} + \text{L1 Miss Rate} \times \bigl(5 + 0.15 \cdot 200\bigr) \\
&= 1 + 0.05 \cdot 35 \\
&= 2.75 \text{ cycles} \\
\end{aligned}
```

Bây giờ, L1 miss penalty bao gồm cả trường hợp L2-hit và trường hợp L2-miss, như được hiển thị trong @fig-amat-l1-l2-tree-vi.

:::{figure} images/amat-l1-l2-tree.png
:label: fig-amat-l1-l2-tree-vi
:width: 65%
:alt: "Two-level AMAT probability tree splitting on L1 hit-miss with 95% hit rate, and then L2 hit-miss outcomes with 85% hit rate. The tree also calculates delay times from accessing caches."

Phân tích hiệu năng cache hai tầng. 95% thời gian, chúng ta chịu 1 cycle delay để truy cập L1 cache. 5% thời gian, chúng ta miss L1 cache. Trong kịch bản L1 miss này, 85% thời gian chúng ta chịu 6 cycles delay (để truy cập cả L1 và L2 cache). 15% thời gian chúng ta chịu 206 cycles delay (để truy cập L1 cache, L2 cache, và memory).
:::
::::

Thiết kế L1 và L2 cache **nhanh gấp 4 lần** so với thiết kế chỉ có L1!

:::{tip} Tại sao L2\$ miss rate (thường) cao hơn L1\$ miss rate?

L2 cache có xu hướng chỉ nhận các truy cập bộ nhớ "khó" (những truy cập miss trong L1 cache).[^mem-access-patterns] Nói cách khác, L1 cache xử lý hầu hết các pattern truy cập bộ nhớ cho dữ liệu có temporal locality. L2 cache cung cấp spatial locality tốt hơn để giảm miss rate. Tuy nhiên, vì dữ liệu L1 cache là tập con của dữ liệu L2 cache, L2 cache vẫn sẽ miss nếu các pattern truy cập bộ nhớ nhảy giữa nhiều địa chỉ khác nhau.

[^mem-access-patterns]: Hashemi et al. "Learning Memory Access Patterns." 2018 [arXiV:1803.02329](https://arxiv.org/abs/1803.02329)

:::

## Preview: Tối Ưu Hóa Cache

Chúng ta đã đề cập rằng AMAT được sử dụng để so sánh các thiết kế cache. Điểm ảnh hưởng hiệu năng chính đến AMAT là **miss rate**. Điều này có thể được đo lường qua nhiều benchmark chương trình, mỗi benchmark có các pattern truy cập bộ nhớ khác nhau.

Trong phần này, chúng ta đã thấy một cách để tối ưu hóa hiệu năng cache bằng cách giới thiệu **multilevel cache** để giảm miss penalty.

Trong chương này, chúng ta sẽ giới thiệu các nguyên lý chính của thiết kế cache. Sau đó, với các nguyên lý thiết kế này trong đầu, chúng ta xem lại các kỹ thuật tối ưu hóa cơ bản để cải thiện hiệu năng cache.

<!--

To optimize cache performance:

1. Introduce multilevel caches to reduce miss penalty. We just saw this.
1. Get a larger cache. This is limited by cost and physical technology capabilities. Furthermore, bigger caches are slower. We would love for higher caches (like L1 cache) to have a hit time of less than the cycle time.
1. Place lines of the cache in a way that maximizes temporal and spatial locality as needed for the average program.
    1. Larger cache line size to reduce miss rate, but can increase the miss penalty.
    1. Higher associativity to reduce miss rate, but can increase hit time.¡ 

The last group of techniques is the core of **cache design** and placement policies. Up next!
-->
