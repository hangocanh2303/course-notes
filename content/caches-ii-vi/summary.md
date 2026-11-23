---
title: "Tổng kết"
---

## Và để kết luận$\dots$

**Replacement policy**:

* Đối với direct-mapped cache, mỗi block của memory ánh xạ đến một block cụ thể trong cache của chúng ta. Khi cache miss, nếu có dữ liệu hiện diện trong cache block đó, thì chúng ta phải loại bỏ block để nhường chỗ cho dữ liệu mới.
* Đối với non-direct-mapped cache, chúng ta có thể chọn một trong nhiều cache block để đặt dữ liệu mới. Khi cache đầy, chúng ta sẽ phải quyết định block nào cần loại bỏ để nhường chỗ cho dữ liệu mới. Block Replacement policy quyết định block nào nên được thay thế.
    * Least recently used (LRU)
    * First-in, first-out (FIFO)
    * Random

:::{embed} #tab-cache-replacement-vi
:::

**Write policy**:

:::{embed} #tab-cache-write-vi
:::

<!--
## Và để kết luận$\dots$

Chúng ta sử dụng cache để làm cho việc truy cập dữ liệu nhanh hơn. Khi làm việc với main memory (RAM), vấn đề
chính gặp phải là việc truy cập vào main memory rất chậm. Trên thực tế, các processor hiện đại
mất khoảng 100 instruction cycle hoặc hơn để truy cập main memory, nghĩa là
truy cập memory trở thành bottleneck của các chương trình của chúng ta. Cache giúp khắc phục vấn đề này cho chúng ta–
chúng giữ một phần dữ liệu trong main memory mà chúng ta có thể truy cập lại sau này. Chúng
gần processor hơn trong memory hierarchy, và do đó truy cập cache nhanh hơn nhiều so với
truy cập main memory.

:::{embed} #fig-von-neumann-cache-vi
:::

Như đã thấy ở trên, truy cập vào cache là bước trung gian giữa CPU yêu cầu một bit memory,
và truy cập main memory thực tế - nếu dữ liệu không được tìm thấy trong cache, chỉ khi đó main
memory mới được truy cập. Bằng cách này, các chuyến đi không cần thiết đến main memory được tránh. Một chi tiết quan trọng
là cache có kích thước nhỏ hơn nhiều so với main memory - đây là lý do chúng ta phải hiệu quả trong
những gì chúng ta giữ trong cache.

Khi chúng ta lưu dữ liệu trong cache, chúng ta cần hiệu quả nhất có thể. Để làm điều này, chúng
ta tận dụng locality. Chúng ta có hai loại locality khác nhau cần xem xét.

* **Temporal Locality**: Nếu chúng ta đã truy cập một thông tin gần đây, có thể chúng
ta sẽ truy cập lại. Vì vậy, chúng ta giữ dữ liệu này trong cache.
* **Spatial Locality**: Nếu chúng ta đã truy cập một vị trí memory gần đây, có khả năng chúng ta
cũng sẽ truy cập các địa chỉ lân cận. Vì vậy, chúng ta cũng giữ các địa chỉ lân cận trong
cache. Một ví dụ là truy cập mảng - nếu chúng ta truy cập phần tử thứ 0 của mảng, có khả năng
chúng ta cũng sẽ truy cập phần tử thứ 1.

### Hiểu T/I/O

Lưu ý rằng cache giữ dữ liệu trong các block có kích thước bằng block size của cache. Khi
làm việc với cache, chúng ta phải có thể phân tích các địa chỉ memory mà chúng ta làm việc để
hiểu chúng phù hợp với cache của chúng ta ở đâu. Có ba trường:
* **Tag**: Được sử dụng để phân biệt các block khác nhau sử dụng cùng index.

Số bit Tag = (# bit trong địa chỉ memory) - Index Bit - Offset Bit

* **Index**: Set mà phần memory này sẽ được đặt vào.
Số bit Index = log2
(# Index)
* **Offset**: Vị trí của byte trong block.
Số bit Offset = log2
(Block Size)

Với các định nghĩa này, điều sau đây là đúng:
log2
(kích thước memory) = # bit địa chỉ memory = # bit tag + # bit index + # bit offset
Một đẳng thức hữu ích khác cần nhớ là:
kích thước cache = block size ∗ num b

#TAG INDEX OFFSET LAYOUT

Như đã thấy ở trên, các bit tag ở bên trái (có trọng số cao nhất), các bit index ở giữa, và
các bit offset ở bên phải (có trọng số thấp nhất).
-->

## Tài liệu đọc thêm

P&H 5.1-5.4, 5.8, 5.9, 5.13

## Tài liệu tham khảo bổ sung

* [Cache Flowchart](https://inst.eecs.berkeley.edu/~cs61c/sp21/resources-pdfs/caches.pdf)

Minh họa tuyệt vời bởi Ketrina (Yim) Thompson: [CS Illustrated](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2009/EECS-2009-79.html) Cache Handouts

* [Cache Basics](https://csillustrated.berkeley.edu/PDFs/posters/cache-1-basics-poster.pdf)
* [Cache Associativity](https://csillustrated.berkeley.edu/PDFs/posters/cache-3-associativity-poster.pdf)
* [Cache Misses](https://csillustrated.berkeley.edu/PDFs/posters/cache-2-misses-poster.pdf)

## Bài tập

Kiểm tra kiến thức của bạn!

### Bài tập ngắn

:::{exercise}
:label: caches-02-vi
**Đúng/Sai**: Chúng ta không thể sử dụng cache 1KB trong hệ thống 32-bit vì nó quá nhỏ và không thể chứa tất cả các địa chỉ có thể.
:::

:::{solution} caches-02-vi
:label: caches-02-sol-vi
:class: dropdown
**Sai.**
:::

:::{exercise}
:label: caches-03-vi
1. **Đúng/Sai**: Với cùng cache size và block size, 4-way set associative cache sẽ có ít bit index hơn direct-mapped cache.
:::

:::{solution} caches-03-vi
:label: caches-03-sol-vi
:class: dropdown
**Đúng.** Direct-mapped cache cần index mọi block của cache, trong khi 4-way set associative cache cần index mọi set gồm 4 block. 4-way set associative cache sẽ có ít hơn 2 bit index so với direct-mapped cache.
:::

:::{exercise}
:label: caches-04-vi
2. **Đúng/Sai**: Giảm block size để tăng số block được giữ bởi cache cải thiện tốc độ chương trình cho tất cả các chương trình.
:::

:::{solution} caches-04-vi
:label: caches-04-sol-vi
:class: dropdown
**Sai.** Tương tự như câu hỏi trước, tác động phụ thuộc vào chương trình. Nếu một chương trình lặp qua bộ nhớ liên tiếp (như mảng), có block size lớn hơn với ít block hơn có thể có lợi vì mỗi block chứa nhiều dữ liệu liên tiếp hơn. Ví dụ, nếu Cache A có 10 block và block size 8 byte trong khi Cache B có 20 block và block size 4 byte, và chúng ta lặp qua mảng 80 ký tự, Cache A sẽ có 10 cache miss và 70 hit, trong khi Cache B sẽ có 20 miss và 60 hit.
:::
