---
title: "Set-Associative Cache"
---

(sec-set-associative-vi)=
## Mục tiêu học tập

* Sử dụng thuật ngữ địa chỉ, mô tả cách tìm một block trong set-associative cache: tag, index, và offset.
* Với một mẫu truy cập bộ nhớ cho trước đến set-associative cache, xác định mỗi truy cập bộ nhớ là cache hit hay cache miss.
* So sánh set-associative cache với direct-mapped và fully associative cache.

::::{note} 🎥 Video bài giảng: Set-Associative Caches
:class: dropdown

:::{iframe} https://www.youtube.com/embed/B4XiuH0kdEk
:width: 100%
:title: "[CS61C FA20] Lecture 27.1 - Caches IV: Set-Associative Caches"
:::

::::

::::{note} 🎥 Video bài giảng: Set-Associative, Line Replacement
:class: dropdown

:::{iframe} https://www.youtube.com/embed/a-ejTer1Ijg
:width: 100%
:title: "[CS61C FA20] Lecture 25.3 - Caches IV: Block Replacement with Example"
:::

Từ 7:00 trở đi

::::

## Placement Policy

(sec-set-associative-policy-vi)=
:::{note} Placement policy _Set Associative_

Một block có thể được đặt vào một tập hợp vị trí hạn chế trong cache.

:::

Một **set** là một nhóm các block trong cache. Một block đầu tiên được ánh xạ đến một set, và sau đó block có thể được đặt ở bất cứ đâu trong set đó.

Cache là **N-Way Set Associative** nếu các set gồm N cache block được gán một index duy nhất. **Associativity** của cache do đó là số slot (tức là **way**) được gán cho mỗi set được index. @fig-sa-2way-small-vi là cache 16-byte, 2-way set-associative với block 4B.

:::{figure} images/sa-2way-small.png
:label: fig-sa-2way-small-vi
:width: 80%
:alt: "Bảng cache 2-way set-associative nhỏ cho thấy set 0 là hai cache line đầu tiên và set 1 là hai cache line thứ hai."

Cache 16-byte, 2-way set-associative với block 4B.
:::

::::{tip} Kiểm tra nhanh

Trong @fig-sa-2way-choice-vi, ánh xạ nào đại diện cho 32B **2-way** SA Cache với block 4B?

:::{figure} images/sa-2way-choice.png
:label: fig-sa-2way-choice-vi
:width: 80%
:alt: "Bảng cache set-associative với 8 cache line và 2 set (được gán nhãn A). Ví dụ làm nổi bật hai way ứng viên trong một set được index cho lựa chọn vị trí đặt (được gán nhãn B)."

Cache 16-byte, 2-way set-associative với block 4B.
:::

* **A.** Hai set gồm bốn block
* **B.** Bốn set gồm hai block

::::

:::{note} Hiển thị đáp án
:class: dropdown

**B.** Bốn set gồm hai block.

* "2-way" có nghĩa là có **hai block** mỗi set.
* Nếu cache có dung lượng 32B với block 4B, thì có tổng cộng tám block.
* Nếu mỗi set có hai block, thì có tổng cộng **bốn set**.
:::

:::{tip} Kiểm tra nhanh
Các block trong set-associative cache có cần valid bit không? Các set trong set-associative cache có cần valid bit không?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Trong tất cả các loại cache, valid bit được sử dụng để kiểm tra xem **tag** của block có hợp lệ hay không (và do đó block có chứa dữ liệu hợp lệ cho chương trình này). Vì có một tag cho mỗi block, tất cả **block** cần valid bit. Đặt valid bit ở cấp set sẽ không cho chúng ta biết các block riêng lẻ (way) trong set có hợp lệ hay không.
:::

## Identification

Xác định cache hit trong set-associative cache hoạt động tương tự như quy trình trong direct-mapped cache, vì dữ liệu chỉ có thể được lưu tại một **index**. Tuy nhiên, bây giờ index có nhiều slot (tức là **way**).

Ví dụ, chúng ta có thể kết nối địa chỉ bộ nhớ 12-bit trong @fig-sa-2way-address-vi với set-associative cache trong @fig-sa-2way-valid-vi (bên dưới).

:::{figure} images/sa-2way-address.png
:label: fig-sa-2way-address-vi
:width: 60%
:alt: "Phân chia địa chỉ bộ nhớ cho cache 2-way set-associative thành các trường: tag ở bit 11 đến 4, index ở bit 3 đến 2, và block offset ở bit 1 đến 0."
Đối với set-associative cache, địa chỉ bộ nhớ được chia thành **ba** trường: tag, index, và offset. Đối với cache trong @fig-sa-2way-valid-vi, địa chỉ bộ nhớ 12-bit được chia thành tag 8-bit, index 2-bit, và offset 2-bit.

:::

Lưu ý:

* Trong set-associative cache, **index** được sử dụng để chọn **set**.
* Trong set-associative cache, **tag** là các bit trên của địa chỉ, loại trừ các bit cho index và offset. Tag được sử dụng để kiểm tra cache block.
* Như với tất cả cache, **offset** là phần của địa chỉ cần thiết để mô tả byte offset trong một block. Đây luôn là các bit thấp nhất của địa chỉ byte bộ nhớ.
* **Block address** là tag nối với index.
* Mặc dù địa chỉ bộ nhớ này có vẻ giống hệt với [hình địa chỉ](#fig-dm-address-vi) được thảo luận trong [phần direct-mapped cache](#sec-direct-mapped-vi), chúng ta lưu ý rằng cache trong @fig-sa-2way-valid-vi **lớn gấp đôi** so với [hình direct-mapped cache](#fig-dm-valid-vi).

## Replacement Policy

:::{tip} Kiểm tra nhanh

Đối với set-associative cache, những replacement policy nào có thể được triển khai? Chọn tất cả những cái đúng.

* **A.** Least Recently Used
* **B.** Most Recently Used
* **C.** FIFO
* **D.** Random
* **E.** Không có cái nào ở trên
:::

:::{note} Hiển thị đáp án
:class: dropdown

**A, B, C, D**.

Khi cache miss, set-associative cache phải xác định block nào cần thay thế **trong set**. Tất cả replacement policy bây giờ xảy ra trong mỗi set của block.

Giống như fully associative cache, hầu hết set-associative cache triển khai LRU, FIFO, hoặc random replacement (trong set). "Most recently used" có thể thực hiện được, nhưng vì nó mâu thuẫn với temporal locality, chính sách như vậy không hữu ích trong thực tế.
:::

## Write Policy

:::{tip} Kiểm tra nhanh

Đối với set-associative cache, những write policy nào có thể được triển khai?

* **A.** Write-through
* **B.** Write-back
* **C.** Cả A và B
* **D.** Không có cái nào ở trên
:::

:::{note} Hiển thị đáp án
:class: dropdown

**C**. cả A và B.

Tất cả các mức associativity của cache đều có thể hỗ trợ write-through hoặc write-back policy. Một lần nữa, _placement_ policy không ảnh hưởng đến lựa chọn của chúng ta về _khi nào_ ghi vào memory xảy ra.
:::

## Hướng dẫn

Animation sau đây theo dõi bốn lần truy cập bộ nhớ đến không gian địa chỉ 12-bit trên 32B, 2-way set-associative cache với block 4B của chúng ta. Giả sử write-back policy. Giả sử cache bắt đầu ở trạng thái [cold](#sec-cache-temperatures-vi), như trong @fig-sa-2way-valid-vi.

:::{figure} images/sa-2way-valid.png
:label: fig-sa-2way-valid-vi
:width: 60%
:alt: "Bảng cho thấy cache cold 2-way set-associative với 4 set. Mỗi cache line có valid và dirty bit."
Ảnh chụp [cold](#sec-cache-temperatures-vi) của 32B, 2-way set-associative cache với block 4B và dirty bit cho write-back.
:::

::::{figure}
:label: fig-sa-2way-warmup-vi
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vQwW0AI7yiWfn0L5EVQQOVNbs6ke7nQDHkKwM7CHQ-IRMSnOn-88QXC9LCTtORDQmWLXp9wK_Vd_U0b/pubembed?start=false&loop=false
:width: 100%
:title: "Các slide đi qua bốn lần truy cập bộ nhớ với 2-way set associative cache trong phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1Wi_d703PYdJgahH0bjxTKVvwT6KQ0dCFLZF7q6O0e9o/edit?usp=sharing)"
:::

::::

:::{note} 1. Load byte @ `0xFE2`. Cache miss.
:class: dropdown

Địa chỉ `0xFE2` trong nhị phân: `0b1111 1110 0010`

* Tag: `0b11111110`, hoặc `0xFE`
* Index: `0b00`, hoặc `0`
* Offset: `0b10`

1. **Cache Miss**. Không có tag hợp lệ nào trong set với index `0` khớp với `0xFE`.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Thay thế block ít được sử dụng gần đây nhất trong set với index `0` (ở đây, có thể là cái nào cũng được vì cache cold). Tải vào một block dữ liệu từ memory bắt đầu @ địa chỉ `0xFE0` (`0b1111 1110 0000`). Ghi tag `0xFE`. Đánh dấu valid bit. Bỏ đặt dirty bit.
1. **Đọc**. Đọc byte trong cache block tại offset `0b10` và trả về processor.

:::

:::{note} 2. Store byte @ `0x61C`. Cache miss.
:class: dropdown

Địa chỉ `0x61C` trong nhị phân: `0b0110 0001 1100`

* Tag: `0b01100001`, hoặc `0x61`
* Index: `0b11`, hoặc `3`
* Offset: `0b00`

1. **Cache Miss**. Không có tag hợp lệ nào trong set với index `3` khớp với `0x61`.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Thay thế block ít được sử dụng gần đây nhất trong set với index `3` (ở đây, có thể là cái nào cũng được vì cache cold). Tải vào cache entry `3` một block dữ liệu từ memory bắt đầu @ địa chỉ `0x61C` (`0b0110 0001 1100`). Ghi tag `0x61`. Đánh dấu valid bit. Bỏ đặt dirty bit.
1. **Ghi**. Ghi byte trong cache block tại offset `0b00`. Đặt dirty bit.
:::

:::{note} 3. Load byte @ `0x61B`. Cache miss.
:class: dropdown

Địa chỉ `0x61B` trong nhị phân: `0b0110 0001 1011`

* Tag: `0b01100001`, hoặc `0x61`
* Index: `0b10`, hoặc `2`
* Offset: `0b11`

1. **Cache Miss**. Không có tag hợp lệ nào trong set với index `2` khớp với `0x61`.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Thay thế block ít được sử dụng gần đây nhất trong set với index `2` (ở đây, có thể là cái nào cũng được vì cache cold). Tải vào cache entry `2` một block dữ liệu từ memory bắt đầu @ địa chỉ `0x618` (`0b0110 0001 1000`). Ghi tag `0x61`. Đánh dấu valid bit. Bỏ đặt dirty bit.
1. **Đọc**. Đọc byte trong cache block tại offset `0b11` và trả về processor.

:::

:::{note} 4. Load byte @ `0xCAD`. Cache miss.
:class: dropdown

Địa chỉ `0xCAD` trong nhị phân: `0b1100 1010 1101`

* Tag: `0b11001010`, hoặc `0xCA`
* Index: `0b11`, hoặc `3`
* Offset: `0b01`

1. **Cache Miss**. Không có tag hợp lệ nào trong set với index `3` khớp với `0xCA`.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Thay thế block ít được sử dụng gần đây nhất trong set với index `3`; ở đây, nó là block **không hợp lệ**. Thay thế block không hợp lệ bằng một block dữ liệu từ memory bắt đầu @ địa chỉ `0xCAC` (`0b1100 1010 1100`). Ghi tag `0xCA`. Đánh dấu valid bit. Bỏ đặt dirty bit.
1. **Đọc**. Đọc byte trong cache block tại offset `0b01` và trả về processor.

:::

So sánh hướng dẫn set-associative cache này với hướng dẫn cho [direct-mapped cache](#sec-dm-walkthrough-vi):

* Identification của cache hit xảy ra bằng cách kiểm tra **M tag** trong M-way set associative cache: mỗi tag cho M block trong set.
* Truy cập bộ nhớ 2 và 3 tạo cache entry trong cache entry `3` và `2`, tương ứng; các cache entry này chia sẻ cùng tag. Tuy nhiên, các block trong các entry này có **block address** khác nhau.
* Truy cập bộ nhớ 4 **không** gây ra block replacement/memory write. Vì có hai way trong một set, block hiện có trong set với index `3` không bị thay thế. Cuối truy cập bộ nhớ 4, set với index `3` đã đầy.

(sec-cache-associativity-vi)=
## Associativity: Thảo luận

Chúng ta minh họa trong @fig-tio-address-vi mối quan hệ giữa block address, tag, index, và offset.

:::{figure} images/tio-address.png
:label: fig-tio-address-vi
:width: 60%
:alt: "Bảng phân tách địa chỉ cho thấy địa chỉ bộ nhớ theo byte. Block address được chia thành tag và index, và phần còn lại của địa chỉ là block offset. Tag được sử dụng để kết nối cache với memory, index được sử dụng để chọn cache block cho vị trí đặt trong cache, và byte offset chỉ định byte trong cache block."
Địa chỉ bộ nhớ (định địa chỉ theo byte) có thể được phân tách thành **block address** và **block offset**. Đối với direct-mapped cache và set-associative cache, block address có thể được chia thêm thành tag và index. Fully associative cache không có trường index.
:::

Lưu ý:

* Fully associative cache không có trường index; block address _chính là_ tag.
* Trong direct-mapped cache và set-associative cache, **index** là các bit thấp của block address được sử dụng để chọn set.
* Tag được so sánh với tag của mọi block trong set.
* **Offset** là địa chỉ của dữ liệu mong muốn trong block.

:::{tip} Kiểm tra nhanh

Sử dụng thuật ngữ set-associativity, fully associative cache với M block là gì?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Fully associative cache có thể có block được đặt ở bất cứ đâu trong cache. Nói cách khác, fully associative cache M-block là set-associative cache **M-way**, trong đó M là số block trong cache.
:::

:::{tip} Kiểm tra nhanh

Sử dụng thuật ngữ set-associativity, direct-mapped cache với M block là gì?
:::

:::{note} Hiển thị đáp án
:class: dropdown

Direct-mapped cache có thể có mỗi block được đặt chính xác ở một vị trí trong cache. Nói cách khác, direct-mapped cache M-block là set-associative cache **one-way**, trong đó M là số set trong cache, và mỗi set có một block.
:::

:::{figure} images/placement-policies.png
:label: fig-placement-policies-vi
:width: 75%
:alt: "Biểu đồ phổ placement-policy từ fully-associative cache đến direct mapped cache với set-associative cache ở giữa. Fully associative cache được gắn nhãn là đặt block mới ở bất cứ đâu trong cache. Set-associative cache được gắn nhãn là đặt block mới vào một trong N vị trí, gọi là set. Direct mapped cache được gắn nhãn là đặt block mới vào một vị trí cụ thể."
Phổ của các placement policy, với set-associative là cách tiếp cận ở giữa.
:::

Chúng ta cuối cùng đã sẵn sàng để xem lại các đánh đổi của các lựa chọn thiết kế. Dưới đây, placement policy được xác định bởi lựa chọn associativity.

```{embed} #sec-cache-design-policy-vi
```

:::{hint} Set-associative cache

Set-associative cache là điểm cân bằng tốt cho các placement policy.

* Một mức độ associativity nhỏ cân bằng các phần tốt của fully associative cache (tức là hiệu suất tốt hơn bằng cách giảm [conflict miss](#sec-cache-misses)) với các phần tốt của direct-mapped cache (phần cứng đơn giản hơn). 8-way set associative cache được sử dụng trong first-level cache trong phần cứng hiện đại.
* Associativity cao hơn giảm [miss rate](#sec-cache-misses); xem các phép đo trong [phần sau](#sec-higher-associativity).

:::
