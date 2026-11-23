---
title: "Tổng Kết"
---

## Và Để Kết Luận$\dots$

### AMAT (Average Memory Access Time)
Nhắc lại rằng AMAT viết tắt của Average Memory Access Time. Đây là cách đo lường hiệu năng của hệ thống cache. Công thức cho AMAT là:

$\text{AMAT} = (\text{Hit Time}) + (\text{Miss Rate}) * (\text{Miss Penalty})$

<!--
In a multi-level memory hierarchies (e.g. multi-level caches), we can separate miss rates into two types that we consider for each level.
* **Global**: Calculated as the number of accesses that missed at that level divided by the total number of accesses **to the memory system**.
* **Local**: Calculated as the number of accesses that missed at that level divided by the total number of accesses **to that memory level**.
-->

## Đọc Thêm Từ Giáo Trình

P&H 5.1-5.4, 5.8, 5.9, 5.13

## Tài Liệu Tham Khảo Bổ Sung

* [Cache Flowchart](https://inst.eecs.berkeley.edu/~cs61c/sp21/resources-pdfs/caches.pdf)

Amazing Illustrations by Ketrina (Yim) Thompson: [CS Illustrated](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2009/EECS-2009-79.html) Cache Handouts

* [Cache Basics](https://csillustrated.berkeley.edu/PDFs/posters/cache-1-basics-poster.pdf)
* [Cache Associativity](https://csillustrated.berkeley.edu/PDFs/posters/cache-3-associativity-poster.pdf)
* [Cache Misses](https://csillustrated.berkeley.edu/PDFs/posters/cache-2-misses-poster.pdf)

## Bài Tập

:::{exercise}
:label: caches-01-vi
**Đúng/Sai**: Nếu một phần dữ liệu nằm cả trong cache và trong memory, đọc nó từ cache nhanh hơn đọc từ memory.
:::

:::{solution} caches-01-vi
:label: caches-01-sol-vi
:class: dropdown
**Đúng.** Cache nhỏ hơn và nhanh hơn memory.
:::
