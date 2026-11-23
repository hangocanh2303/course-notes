---
title: "Tóm Tắt"
---

## Và Kết Luận$\dots$

### Cache Misses
Để đánh giá hiệu suất cache và hit rate, đặc biệt là xác định hiệu quả của cấu hình cache hiện tại, việc phân tích các misses xảy ra và điều chỉnh phù hợp là rất hữu ích. Dưới đây, chúng ta phân loại cache misses thành hai loại:
1. Compulsory: Một miss phải xảy ra khi bạn nạp một block nhất định lần đầu tiên, do đó gọi là "compulsory". Compulsory misses là các cache attempts sẽ không bao giờ là hit bất kể thiết kế cache.
2. Noncompulsory: Một cache miss xảy ra sau khi data đã được nạp vào cache và sau đó bị loại bỏ. Nếu miss có thể được giảm bớt bằng cách tăng cache size hoặc associativity, thì miss được coi là noncompulsory.

### Cache Associativity
Direct-Mapped caches, nơi mỗi block của memory ánh xạ đến cụ thể một slot trong cache của chúng ta, tốt cho tìm kiếm nhanh, hardware đơn giản, và thay thế nhanh, nhưng không tốt cho spatial locality! Đây là lúc chúng ta đưa associativity vào vấn đề. Associativity là số slots mà một memory block có thể ánh xạ đến trong cache của chúng ta. Do đó, Fully-Associative cache có associativity cao nhất, nghĩa là một memory block có thể ánh xạ đến bất kỳ cache block nào. Direct-Mapped cache của chúng ta, mặt khác, có ít nhất (chỉ là 1-way set associative) vì một memory block chỉ có thể ánh xạ đến một cache block duy nhất.

Với N-way set associative cache, các mối quan hệ sau đây đúng:
* Number of Blocks = 𝑁 × Number of Sets
* Index bits = log2(Number of sets)

:::{note} Một 2-way set associative cache với 4 index bits có bao nhiêu sets và blocks?
:class: dropdown

Với 2-way set associative cache có 4 index bits, sẽ có $2^4 = 16$ sets cho $2 \times 16 = 32$ blocks trong cache. Một địa chỉ đơn sẽ ánh xạ đến một trong 16 sets và sẽ được đặt vào một trong hai blocks.
:::

## Tài Liệu Tham Khảo Bổ Sung

* [Cache Flowchart](https://inst.eecs.berkeley.edu/~cs61c/sp21/resources-pdfs/caches.pdf)

Minh họa tuyệt vời bởi Ketrina (Yim) Thompson: [CS Illustrated](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2009/EECS-2009-79.html) Cache Handouts

* [Cache Basics](https://csillustrated.berkeley.edu/PDFs/posters/cache-1-basics-poster.pdf)
* [Cache Associativity](https://csillustrated.berkeley.edu/PDFs/posters/cache-3-associativity-poster.pdf)
* [Cache Misses](https://csillustrated.berkeley.edu/PDFs/posters/cache-2-misses-poster.pdf)

### Tham khảo Benchmark

Dữ liệu trong phần này từ Patterson & Hennessy, _Computer Organization: A General Approach_, Fifth Edition.

* Miss rate theo cache size và associativity là Figure B.8 và được tổng hợp từ dữ liệu đo trên SPEC2000 benchmarks trên kiến trúc Alpha, bởi Cantin và Hill.[^cantin-hill] Block sizes cho tất cả caches là 64 bytes.
* Miss rate theo cache size và block size là Figure B.11 từ dữ liệu đo trên SPEC92 benchmark trên DECstation 5000, bởi Gee et al.[^gee-et-al].

[^cantin-hill]: J.F. Cantin và M.D. Hill [2001]. "Cache PErformance for Selected SPEC CPU2000 Benchmarks." https://doi.org/10.1145/563519.563522 [website](https://research.cs.wisc.edu/multifacet/misc/spec2000cache-data/) Version 2.0.

[^gee-et-al]: J.D. Gee, M.D. Hill, D.N. Pnevmatikatos, và A.J. Smith [1993]. "Cache performance of the SPEC92 benchmark suite." _IEEE Micro_ 13:4 (August), 17-27.https://doi.org/10.1109/40.229711

### Bài Tập Ngắn

:::{exercise}
:label: caches-05-vi
1. **Đúng/Sai**: Giảm block size để tăng số blocks được giữ bởi cache cải thiện tốc độ chương trình cho tất cả chương trình.
:::

:::{solution} caches-05-vi
:label: caches-05-sol-vi
:class: dropdown
**Sai.** Tương tự như câu hỏi trước, ảnh hưởng phụ thuộc vào chương trình. Nếu chương trình lặp qua bộ nhớ liên tục (như một mảng), có block sizes lớn hơn với ít blocks hơn có thể có lợi vì mỗi block chứa nhiều data liên tục hơn. Ví dụ, nếu Cache A có 10 blocks và block size 8 bytes trong khi Cache B có 20 blocks và block size 4 bytes, và chúng ta lặp qua một mảng 80 characters, Cache A sẽ gặp 10 cache misses và 70 hits, trong khi Cache B sẽ có 20 misses và 60 hits.
:::

:::{exercise}
:label: caches-06-vi
1. **Đúng/Sai**: Caches thấy cải thiện ngay lập tức trong memory access time khi chương trình bắt đầu thực thi.
:::

:::{solution} caches-06-vi
:label: caches-06-sol-vi
:class: dropdown
**Sai.** Cache bắt đầu ở trạng thái "cold" và yêu cầu nạp giá trị vào blocks đầu tiên trực tiếp từ memory, buộc phải có compulsory misses. Điều này có thể được giảm bớt phần nào bằng cách sử dụng hardware prefetcher, sử dụng pattern hiện tại của misses để dự đoán và prefetch data có thể được truy cập sau. Prefetchers nằm ngoài phạm vi của khóa học này.
:::

:::{exercise}
:label: caches-07-vi
2. **Đúng/Sai**: Tăng cache size bằng cách thêm blocks luôn cải thiện (tăng) hit rate cho tất cả chương trình.
:::

:::{solution} caches-07-vi
:label: caches-07-sol-vi
:class: dropdown
**Sai.** Liệu điều này có cải thiện hit rate cho một chương trình nhất định phụ thuộc vào đặc điểm của chương trình. Ví dụ, một chương trình lặp qua mảng một lần có thể có mỗi lần truy cập cách nhau hơn một block (ví dụ, nếu block size là 8B nhưng chúng ta truy cập mỗi phần tử thứ tư của integer array, các lần truy cập cách nhau 16B). Điều này dẫn đến compulsory misses, không thể giảm chỉ bằng cách thêm blocks vào cache.
:::
