---
title: "Direct Mapped Cache"
---

(sec-direct-mapped-vi)=
## Mục tiêu học tập

* Sử dụng thuật ngữ địa chỉ, mô tả cách tìm một block trong direct-mapped cache: tag, index, và offset.
* Với một mẫu truy cập bộ nhớ cho trước đến direct-mapped cache, xác định mỗi truy cập bộ nhớ là cache hit hay cache miss.
* So sánh direct-mapped cache với fully associative cache.

::::{note} 🎥 Video bài giảng: Direct Mapped Caches
:class: dropdown

:::{iframe} https://www.youtube.com/embed/wF3Ekt-ZHdg
:width: 100%
:title: "[CS61C FA20] Lecture 26.1 - Caches II: Direct Mapped"
:::

::::

::::{note} 🎥 Video bài giảng: Direct Mapped Example
:class: dropdown

:::{iframe} https://www.youtube.com/embed/-OtUM6j6_xE
:width: 100%
:title: "[CS61C FA20] Lecture 26.2 - Caches II: Direct Mapped Example"
:::

::::

Trong [phần trước](#sec-fully-associative-vi), chúng ta đã giải thích tại sao chi phí phần cứng khiến fully associative cache khá không phổ biến trong các processor hiện đại. Bây giờ chúng ta giới thiệu chính sách đầu kia của phổ: **direct mapped cache**. Với cache mới này, chúng ta xem xét lại các chính sách thiết kế cache và đi qua một ví dụ.

:::{embed} #sec-cache-design-policy-vi
:::

## Placement Policy

(sec-direct-mapped-policy-vi)=
:::{note} Placement policy _Direct Mapped_

Một block chỉ có thể được đặt vào đúng một vị trí của cache.

:::

## Identification

Xem xét hình ảnh hóa của chúng ta cho direct-mapped cache 16B với block 4B trong @fig-dm-valid-vi.

:::{figure} images/dm-valid.png
:label: fig-dm-valid-vi
:width: 60%
:alt: "Bảng direct-mapped cache cold với valid và dirty bit và nội dung dữ liệu trống."
Ảnh chụp [cold](#sec-cache-temperatures-vi) của direct-mapped cache 16B với block 4B và dirty bit cho write-back.
:::

Trên bề mặt, direct mapped cache trông rất giống với [fully associative cache](#fig-fa-valid-vi) của chúng ta. Chúng ta thảo luận cách direct mapped placement policy rút ngắn độ rộng tag và tác động đến quy trình identification để xác định cache hit.

<!-- ## TODO: gotta put in the crazy coloring with indices, etc.-->

### Tag, Index, và Offset

Ánh xạ của hầu như tất cả direct-mapped cache là đơn giản:

```{math}
\text{(Block address) modulo (số block trong cache)}
```

Giống như trước, direct-mapped cache sao chép dữ liệu từ memory theo đơn vị **block**. Chúng ta có thể chuyển đổi từ **byte address** sang **block address**.

(sec-block-address-vi)=
:::{note} Block address vs. byte address

Theo định nghĩa, chúng ta nói rằng **block address** là "index" của một block trong memory, nếu chúng ta chia memory thành các block, không phải byte. Nói cách khác, **block address** của một cache block là các bit trên của _byte address_ của block này trong memory, loại trừ các bit thấp dành cho byte offset.

Chúng ta thảo luận điều này thêm trong [phần sau](#fig-tio-address-vi) sau khi đã đề cập đến [set associative cache](#sec-set-associative-vi).
:::

Ví dụ, chúng ta có thể kết nối direct-mapped cache trong @fig-dm-valid-vi với địa chỉ bộ nhớ 12-bit trong @fig-dm-address-vi.

:::{figure} images/dm-address.png
:label: fig-dm-address-vi
:width: 60%
:alt: "Phân tách địa chỉ direct-mapped thành các trường: tag ở bit 11 đến 4, index ở bit 3 đến 2, và block-offset ở bit 1 đến 0."
Đối với direct-mapped cache, địa chỉ bộ nhớ được chia thành **ba** trường: tag, index, và offset. Đối với cache trong @fig-dm-valid-vi, địa chỉ bộ nhớ 12-bit được chia thành tag 8-bit, index 2-bit, và offset 2-bit.

:::

* Trong direct-mapped cache, **index** được sử dụng để chọn block.
* Trong direct-mapped cache, **tag** là các bit trên của địa chỉ, loại trừ các bit cho index và offset. Tag được sử dụng để kiểm tra cache block.
* Như với tất cả cache, **offset** là phần của địa chỉ cần thiết để mô tả byte offset trong một block. Đây luôn là các bit thấp nhất của địa chỉ byte bộ nhớ.
* **Block address** là tag nối với index.

:::{tip} Kiểm tra nhanh

Xem lại mối quan hệ giữa các phần địa chỉ trong @fig-dm-address-vi và direct-mapped cache 16B (với block 4B) trong @fig-dm-valid-vi.

1. Block size là bao nhiêu, tính bằng byte?
1. Tổng dung lượng dữ liệu trên tất cả các block là bao nhiêu, tính bằng byte?
1. Nếu không gian bộ nhớ của chúng ta là $2^{12}$ byte, chúng ta có địa chỉ 12-bit. Bao nhiêu bit của địa chỉ này nên được dành cho offset?
1. Vẫn giả sử địa chỉ 12-bit, bao nhiêu bit của địa chỉ này nên được dành cho index?
1. Vẫn giả sử địa chỉ 12-bit, bao nhiêu bit của địa chỉ này nên được dành cho tag?
:::

:::{note} Lời giải
:class: dropdown

1. **4 byte**.
1. **16 byte.** Nhớ lại, dung lượng dữ liệu.
1. Offset vẫn liên quan đến block size. $\log_2{(\text{block size})}$ = **2 bit**.
1. Index chọn entry của cache. Để index vào mỗi trong 4 entry của cache này, chúng ta cần $\log_2{(\text{số block})} = **2 bit**.
1. Tag, như trước, là các bit trên của địa chỉ—nhưng bây giờ, nó là các bit trên _không_ được nắm bắt bởi index và offset. Vì chúng ta sử dụng địa chỉ bộ nhớ 12-bit trong ví dụ nhỏ của mình, đối với direct-mapped cache này tag của chúng ta là (# bit địa chỉ) - (# bit index) - (# bit offset) = **8 bit**.
:::

:::{warning} Direct Mapped: Tag, Index, Offset

Direct-mapped cache sử dụng địa chỉ để xác định **index** của _cache entry duy nhất_ có thể được liên kết với địa chỉ bộ nhớ này. Fully associative cache _không_ chia địa chỉ thành phần index vì block có thể được đặt ở bất cứ đâu.
:::

## Replacement Policy

:::{tip} Kiểm tra nhanh

Đối với direct-mapped cache, những replacement policy nào có thể được triển khai? Chọn tất cả những cái đúng.

* **A.** Least Recently Used
* **B.** Most Recently Used
* **C.** FIFO
* **D.** Random
* **E.** Không có cái nào ở trên
:::

:::{note} Hiển thị đáp án
:class: dropdown

**E.** Không có cái nào ở trên.

Trong direct mapped cache, chỉ có một block để thay thế—block hiện có với index khớp. LRU, most recently used, FIFO, và Random replacement sẽ vi phạm placement policy của direct-mapped cache.
:::

## Write Policy

:::{tip} Kiểm tra nhanh

Đối với direct-mapped cache, những write policy nào có thể được triển khai?

* **A.** Write-through
* **B.** Write-back
* **C.** Cả A và B
* **D.** Không có cái nào ở trên
:::

:::{note} Hiển thị đáp án
:class: dropdown

**C**. cả A và B.

Direct-mapped _placement_ policy không ảnh hưởng đến lựa chọn của chúng ta về _khi nào_ ghi vào memory xảy ra. Cả write-through và write-back policy đều có thể thực hiện được.
:::

(sec-dm-walkthrough-vi)=
## Hướng dẫn

Animation sau đây theo dõi bốn lần truy cập bộ nhớ đến không gian địa chỉ 12-bit trên direct-mapped cache 16B với block 4B của chúng ta. Giả sử write-back policy. Giả sử cache bắt đầu ở trạng thái [cold](#sec-cache-temperatures-vi), như trong @fig-dm-valid-vi.

::::{figure}
:label: fig-dm-warmup-vi
:alt: "Các slide nhúng đi qua bốn lần truy cập bộ nhớ trên direct-mapped cache cold địa chỉ 12-bit với dung lượng 16 byte, block 4 byte, và write-back policy."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vSfQ89lj8_qzjh6yX5lGIOVEzC3909wUDUGC5FRbyT060KMbeTBKLFvUIlPyHIkna968TML6yoBnGbe/pubembed?start=false&loop=false
:width: 100%
:title: "Các slide đi qua write policy của direct-mapped cache từ phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1SIM8fNAVuRooMmNGsFzuiB2MwlKYi_ZPWQrQYEh0dmE/edit?usp=sharing)"
:::

Làm nóng direct-mapped cache.
::::

:::{note} 1. Load byte @ `0xFE2`. Cache miss.
:class: dropdown

Địa chỉ `0xFE2` trong nhị phân: `0b1111 1110 0010`

* Tag: `0b11111110`, hoặc `0xFE`
* Index: `0b00`, hoặc `0`
* Offset: `0b10`

1. **Cache Miss**. Entry tại index `0` có tag không hợp lệ.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Tải vào cache entry `0` một block dữ liệu từ memory bắt đầu @ địa chỉ `0xFE0` (`0b1111 1110 0000`). Ghi tag `0xFE`. Đánh dấu valid bit. Bỏ đặt dirty bit.
1. **Đọc**. Đọc byte trong cache block tại offset `0b10` và trả về processor.

:::

:::{note} 2. Store byte @ `0x61C`. Cache miss.
:class: dropdown

Địa chỉ `0x61C` trong nhị phân: `0b0110 0001 1100`

* Tag: `0b01100001`, hoặc `0x61`
* Index: `0b11`, hoặc `3`
* Offset: `0b00`

1. **Cache Miss**. Entry tại index `3` có tag không hợp lệ.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Tải vào cache entry `3` một block dữ liệu từ memory bắt đầu @ địa chỉ `0x61C` (`0b0110 0001 1100`). Ghi tag `0x61`. Đánh dấu valid bit. Bỏ đặt dirty bit.
1. **Ghi**. Ghi byte trong cache block tại offset `0b00`. Đặt dirty bit.
:::

:::{note} 3. Load byte @ `0x61B`. Cache miss.
:class: dropdown

Địa chỉ `0x61B` trong nhị phân: `0b0110 0001 1011`

* Tag: `0b01100001`, hoặc `0x61`
* Index: `0b10`, hoặc `2`
* Offset: `0b11`

1. **Cache Miss**. Entry tại index `2` có tag không hợp lệ.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Tải vào cache entry `2` một block dữ liệu từ memory bắt đầu @ địa chỉ `0x618` (`0b0110 0001 1000`). Ghi tag `0x61`. Đánh dấu valid bit. Bỏ đặt dirty bit.
1. **Đọc**. Đọc byte trong cache block tại offset `0b11` và trả về processor.

:::

:::{note} 4. Load byte @ `0xCAD`. Cache miss.
:class: dropdown

Địa chỉ `0xCAD` trong nhị phân: `0b1100 1010 1101`

* Tag: `0b11001010`, hoặc `0xCA`
* Index: `0b11`, hoặc `3`
* Offset: `0b01`

1. **Cache Miss**. Mặc dù entry tại index `3` hợp lệ, tag `0x61` của nó **không** khớp với tag được cung cấp `0xCA`.
1. **Truy cập tầng thấp hơn của memory hierarchy**. Block hợp lệ hiện có tại cache entry `3` phải được thay thế. Dirty bit của nó được đặt, vì vậy ghi block này vào memory và thay thế nó bằng một block dữ liệu từ memory bắt đầu @ địa chỉ `0xCAC` (`0b1100 1010 1100`). Ghi tag `0xCA`. Đánh dấu valid bit. Bỏ đặt dirty bit.
1. **Đọc**. Đọc byte trong cache block tại offset `0b01` và trả về processor.

:::

So sánh hướng dẫn direct-mapped cache này với hướng dẫn cho [fully associative cache](#sec-fa-walkthrough-vi):

* Identification của cache hit xảy ra bằng cách kiểm tra **đúng một tag**: tag tại cache entry được index.
* Truy cập bộ nhớ 2 và 3 tạo cache entry trong cache entry `3` và `2`, tương ứng; các cache entry này chia sẻ cùng tag. Tuy nhiên, các block trong các entry này có **block address** khác nhau.
* Truy cập bộ nhớ 4 vẫn gây ra block replacement (và cache write-back) mặc dù cache chưa đầy. Cache entry tại index `3` bị chiếm bởi block với tag khác.

## Direct Mapped: Phần cứng và Hiệu suất

Triển khai direct-mapped cache trong phần cứng đơn giản hơn nhiều so với fully associative cache.

* Vì block chỉ có thể ở một vị trí, khi cache hit chúng ta chỉ cần một bộ so sánh để kiểm tra tag trong line mục tiêu (và một mux để lấy tag đó). Chúng ta không còn cần một bộ so sánh cho mỗi cache block/entry.
* Vì index bây giờ được sử dụng để chọn vị trí của block trong cache, nó không được mã hóa vào các bit của tag. Điều này giảm độ rộng của mỗi tag, do đó giảm overhead trên mỗi cache entry.

:::{figure} images/hardware-direct-mapped-cache.png
:label: fig-hardware-direct-mapped-cache-vi
:width: 70%
:alt: "Sơ đồ khối phần cứng của direct-mapped cache. Địa chỉ 32-bit được chia thành tag, index, và offset. Các mũi tên kết nối ba trường của địa chỉ đến nơi chúng được sử dụng trong sơ đồ không gian bộ nhớ để mô tả lựa chọn index, kiểm tra tag, và đường dẫn đầu ra dữ liệu."
Triển khai phần cứng của direct-mapped cache.
:::
