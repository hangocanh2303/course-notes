---
title: "Thuật ngữ Cache"
---

(sec-cache-terminology-vi)=
## Mục tiêu học tập

* Giải thích cách cache tận dụng temporal locality và spatial locality.
* Theo dõi quá trình truy cập bộ nhớ với cache.
* Làm quen với các thuật ngữ quan trọng về cache: cache hit, cache miss, block (cache line), tag.

::::{note} 🎥 Video bài giảng: Locality, Design, and Management
:class: dropdown

:::{iframe} https://www.youtube.com/embed/xrCp6DKazuk
:width: 100%
:title: "[CS61C FA20] Lecture 24.4 - Caches I: Locality, Design, Management"
:::

::::


::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/lGp8FK1NW_k
:width: 100%
:title: "[CS61C FA20] Lecture 25.3 - Caches II: Cache Terminology"
:::

::::

## Nguyên lý Locality


Làm thế nào để tạo ra ảo giác về một bộ nhớ lớn mà chúng ta có thể truy cập nhanh? Từ P&H 5.1:

> Cũng như bạn không cần truy cập tất cả các sách trong thư viện cùng một lúc với xác suất bằng nhau, một chương trình không truy cập tất cả code hoặc dữ liệu của nó cùng một lúc với xác suất bằng nhau. Nếu không, sẽ không thể làm cho hầu hết các truy cập bộ nhớ nhanh trong khi vẫn có bộ nhớ lớn trong máy tính, cũng như bạn không thể đặt tất cả sách thư viện trên bàn và vẫn tìm được những gì bạn muốn một cách nhanh chóng.

Cache là nền tảng của memory hierarchy. Chúng chứa **bản sao của một tập con dữ liệu** từ main memory.[^earlier]

[^earlier]: Chi tiết này đã được thảo luận [trước đó](#sec-library) nhưng luôn đáng được nhắc lại.

:::{hint} Nguyên lý locality

Cache hoạt động dựa trên nguyên lý **temporal và spatial locality**.

* **Temporal locality**: Nếu một mục được tham chiếu, nó sẽ có xu hướng được tham chiếu lại sớm.
* **Spatial locality**: Nếu một mục được tham chiếu, các mục có địa chỉ gần đó sẽ có xu hướng được tham chiếu sớm.

:::

:::{table} Nguyên lý temporal và spatial locality.
:label: tab-locality-vi

| Thuộc tính | Temporal Locality | Spatial Locality |
| :--- | :--- | :--- |
| Ý tưởng | Nếu chúng ta sử dụng nó bây giờ, khả năng cao là chúng ta sẽ muốn sử dụng lại sớm. | Nếu chúng ta sử dụng một vùng bộ nhớ, khả năng cao là chúng ta sẽ sử dụng các vùng lân cận sớm. |
| Phép so sánh thư viện | Chúng ta giữ một cuốn sách trên bàn trong khi mượn thêm một cuốn khác. | Nếu chúng ta mượn tập 1 của một bộ sách tham khảo, nhân tiện, chúng ta cũng sẽ mượn tập 2. Thư viện đặt các sách cùng chủ đề gần nhau trên cùng các kệ để tăng spatial locality. |
| Bộ nhớ | Nếu một vị trí bộ nhớ được tham chiếu, nó sẽ có xu hướng được tham chiếu lại sớm. Do đó, giữ các mục dữ liệu được truy cập gần đây nhất gần processor hơn. | Nếu một vị trí bộ nhớ được tham chiếu, các vị trí có địa chỉ gần đó sẽ có xu hướng được tham chiếu sớm. Di chuyển các **block** bao gồm các word liên tiếp đến gần processor hơn. |

:::


## Thuật ngữ quan trọng về Cache

Từ [Wikipedia](https://en.wikipedia.org/wiki/CPU_cache):

> Dữ liệu được truyền giữa memory và cache trong các block có kích thước cố định, gọi là cache line hoặc cache block. Khi một cache line được sao chép từ memory vào cache, một cache entry được tạo ra. Cache entry sẽ bao gồm dữ liệu được sao chép cũng như vị trí bộ nhớ được yêu cầu (gọi là tag).

Bộ nhớ có thể **định địa chỉ theo byte**, nghĩa là mỗi byte trong bộ nhớ có một địa chỉ **address**. Điều này giống hệt với khái niệm bộ nhớ từ [trước đó](#sec-address-space). Giống như bộ nhớ, cache cần tra cứu dữ liệu theo địa chỉ bộ nhớ (xem [bên dưới](#sec-cache-memory-access-vi)). Tuy nhiên, bây giờ cache không còn có quyền truy cập vào toàn bộ không gian địa chỉ bộ nhớ vì dung lượng lưu trữ hạn chế của nó.

Do đó, mỗi entry trong cache cần theo dõi (ít nhất) **hai** thông tin:

1. **Cache block** (còn gọi là **block**, hoặc **cache line**)[^block-vs-line] là đơn vị dữ liệu được sao chép từ memory vào cache. Block là đơn vị bộ nhớ nhỏ nhất có thể được truyền giữa main memory và cache. Việc sao chép một _line_ dữ liệu (thay vì chỉ một word, hoặc một byte) giúp chúng ta tận dụng **spatial locality**.

    Mỗi block có entry riêng trong cache.

1. **Tag**: Các địa chỉ liên kết với dữ liệu trong một block.

    Từ P&H 5.3: "**Tag** là một trường trong bảng được sử dụng cho memory hierarchy, chứa thông tin địa chỉ cần thiết để xác định liệu [line] liên kết trong hierarchy có tương ứng với [word hoặc byte] được yêu cầu hay không."

    Mỗi cache entry có tag riêng. Do đó, mỗi block được liên kết với một tag.

[^block-vs-line]: Tài liệu không nhất quán về việc gọi đơn vị dữ liệu được truyền giữa cache và main memory là "block" hay "line". Bạn sẽ thấy cả hai. Chúng ta sẽ cố gắng sử dụng "block" khi có thể, trừ khi trích dẫn nguồn.

Thuật ngữ liên quan đến kích thước:

* **Block size** (còn gọi là **line size**) là số byte dữ liệu được lưu trong block này. Mỗi block trong cache có cùng block size. Để tận dụng spatial locality, cache thường có block size lớn hơn một word.
* **Capacity** là kích thước của cache, tính bằng byte.

:::{warning} Kích thước/dung lượng Cache
:label: block-cache-size-vi

Từ [Wikipedia](https://en.wikipedia.org/wiki/CPU_cache):
> "Kích thước" của cache là lượng dữ liệu main memory mà nó có thể chứa. Kích thước này có thể được tính bằng số byte được lưu trong mỗi data block nhân với số block được lưu trong cache. (Các bit tag, [và metadata khác] không được tính vào kích thước[^practice], mặc dù chúng ảnh hưởng đến diện tích vật lý của cache.)

Đối với chip Apple M1[^m1-line]:

* L1 cache là 128 KiB, 8-way [set-associative](#sec-set-associative-vi) với 64-byte block.
* L2 cache là 12 MiB, 12-way [set-associative](#sec-set-associative-vi) với 128-byte block.

[^m1-line]: Nguồn: [GoFetch](https://gofetch.fail/).

[^practice]: Xem so sánh kích thước trong Sadler et al., ICCD 2006. DOI: [10.1109/ICCD.2006.4380862](https://doi.org/10.1109/ICCD.2006.4380862)

Trong khóa học này, khi chúng ta nói cache 32B, chúng ta muốn nói cache có thể lưu 32 byte **dữ liệu** từ memory, tức là 32 = (số block) x (line size).

[^metadata]: Tag, valid bit, dirty bit, v.v. Được thảo luận trong [chương tiếp theo](#sec-fully-associative-vi). 

:::

(sec-cache-memory-access-vi)=
## Truy cập bộ nhớ có/không có Cache

Khi một lệnh load hoặc store được truy cập, processor **yêu cầu** dữ liệu tại một địa chỉ cụ thể từ memory hierarchy. Trong phần này, chúng ta so sánh cách truy cập bộ nhớ hoạt động—có và không có cache. Chuyển đổi giữa hai thẻ bên dưới.

:::::{tab-set}
::::{tab-item} Bố cục máy tính có cache
:sync: tab-with-cache

:::{figure} images/von-neumann-cache.jpg
:label: fig-von-neumann-cache-vi
:width: 100%
:alt: "Bố cục máy tính kiểu Von Neumann với khối processor kết nối thông qua tín hiệu read/write và address đến cache rồi đến bộ nhớ đầy đủ. Khối memory cũng được kết nối ở bên phải với input và output riêng biệt thông qua các interface I/O memory."

Cache được chèn vào bố cục máy tính cơ bản từ [phần trước](#sec-architecture-elements).
:::

::::

::::{tab-item} Bố cục máy tính không có cache
:sync: tab-without-cache

:::{embed} #fig-von-neumann
:::

::::
:::::

Xem xét lệnh load word `lw t0 0(t1)`. Giả sử thanh ghi `t1` chứa `0x12F0`, và word bắt đầu tại địa chỉ bộ nhớ `0x12F0` là `1234`.

:::::{tab-set}
::::{tab-item} Truy cập bộ nhớ có cache
:sync: tab-with-cache

Truy cập bộ nhớ **có cache**:

1. Processor gửi địa chỉ `0x12F0` đến cache
1. Cache kiểm tra bản sao của dữ liệu với địa chỉ `0x12F0`

    1. (2a) Nếu **cache hit** (tìm thấy khớp): cache đọc `1234`
    2. (2b) Nếu **cache miss** (không khớp): cache gửi địa chỉ `0x12F0` đến Memory

        1. (2b(i)) Memory đọc block chứa `1234` (tức là block chứa dữ liệu tại địa chỉ `0x12F0`)
        1. (2b(ii)) Memory gửi block chứa `1234` đến cache
        1. (2b(iii)) Cache thay thế một block nào đó để lưu block mới chứa `1234`
        1. (2b(iv)) Cache đọc `1234`
1. Cache gửi `1234` đến Processor
1. Processor nạp `1234` vào thanh ghi `t0`
::::

::::{tab-item} Truy cập bộ nhớ không có cache
:sync: tab-without-cache

Truy cập bộ nhớ **không có cache**:

1. Processor gửi địa chỉ `0x12F0` đến memory
1. Memory đọc `1234` @ địa chỉ `0x12F0`
1. Memory gửi `1234` đến Processor
1. Processor nạp `1234` vào thanh ghi `t0`
::::
::::: 

Khi có cache trong hệ thống, có hai tình huống có thể xảy ra khi truy cập bộ nhớ:

* **Cache hit**: Dữ liệu bạn đang tìm kiếm nằm trong cache. Lấy dữ liệu từ cache và mang đến processor.
* **Cache miss**: Dữ liệu bạn đang tìm kiếm không có trong cache. Đi đến tầng thấp hơn trong memory hierarchy để tìm dữ liệu, đặt dữ liệu vào cache. Sau đó, mang dữ liệu đến processor.

(sec-cache-temperatures-vi)=
## Nhiệt độ Cache

Mục tiêu của chúng ta trong thiết kế cache là temporal và spatial locality cho nhiều loại workload khác nhau. Chúng ta mượn thuật ngữ khí hậu để mô tả hiệu suất cache:

* **Cold**: Cache "trống".[^empty-analogy]
* **Warming**: Cache đang được lấp đầy với các giá trị mà chúng ta hy vọng sẽ truy cập lại.
* **Warm**: Cache đang hoạt động tốt, với tỷ lệ hit khá.
* **Hot**: Cache đang hoạt động rất tốt với tỷ lệ hit cao.

[^empty-analogy]: Cache không bao giờ thực sự "trống". Thay vào đó, các block đôi khi có thể chứa dữ liệu rác đối với chương trình đang chạy. Chúng ta thảo luận điều này trong [phần tiếp theo](#sec-valid-bit-vi).

## Bốn câu hỏi về Memory Hierarchy

Phần này được chuyển thể từ Patterson và Hennessy. _Computer Architecture: A Quantitative Approach_, Fifth Edition. 2012. Phụ lục B.

(sec-cache-design-policy-vi)=
:::{note} Các chính sách thiết kế Cache

Có bốn câu hỏi cấp cao cho thiết kế cache. Chúng ta sẽ gọi ba trong số này là **chính sách**:

1. **Placement policy**: Block (tức là line) có thể được đặt ở đâu trong cache? Cache entry nào có thể liên kết với block (tức là line) này?
1. **Identification**: Làm thế nào để tìm block (tức là line) nếu nó có trong cache? Identification liên quan chặt chẽ với placement policy.
1. **Replacement policy**: Block (tức là line) nào nên được thay thế khi miss?
1. **Write policy**: Điều gì xảy ra khi ghi?
:::

Câu trả lời cho những câu hỏi này giúp chúng ta hiểu các đánh đổi khác nhau của cache (và thậm chí của các tầng khác trong memory hierarchy, như chúng ta sẽ thấy trong phần sau). Chúng ta sẽ hỏi bốn câu hỏi này với mọi ví dụ. Chúng ta bắt đầu bằng cách giới thiệu các placement policy:

* [Fully Associative Cache](#sec-fully-associative-vi)
* [Set-Associative Cache](#sec-set-associative-vi)
* [Direct Mapped Cache](#sec-direct-mapped-vi)
