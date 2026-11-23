---
title: "Memory Hierarchy, Xem Lại"
---

(sec-memory-hierarchy-vi)=
## Mục Tiêu Học Tập

* Định nghĩa các thành phần chính của memory hierarchy: processor, cache, memory, disk.

::::{note} 🎥 Video Bài Giảng: Memory Hierarchy
:class: dropdown

:::{iframe} https://www.youtube.com/embed/7Lk5UacJeYA
:width: 100%
:title: "[CS61C FA20] Lecture 24.3 - Caches I: Memory Hierarchy"
:::

::::


<!-- :::{figure} images/principle-of-locality-memory-hierarchy-pyramid.png
:label: fig-principle-of-locality-memory-hierarchy-pyramid-2
:width: 50%
:alt: "Pyramid-style memory hierarchy diagram from smallest fastest level near the CPU to largest slowest storage, illustrating temporal and spatial locality across registers, caches, DRAM, and disk."
The memory hierarchy.
::: -->

:::{warning} Xem lại memory hierarchy

Xem lại giả định hiện tại của chúng ta về [memory](#sec-memory-hierarchy-early): Giả sử processor, registers, và memory.

:::

> Lý tưởng là người ta mong muốn một dung lượng bộ nhớ lớn vô hạn sao cho bất kỳ ... word cụ thể nào cũng sẽ có sẵn ngay lập tức. ... Chúng ta ... buộc phải nhận ra khả năng xây dựng một hệ thống phân cấp các bộ nhớ, mỗi bộ nhớ có dung lượng lớn hơn bộ nhớ trước đó nhưng truy cập kém nhanh hơn.
> 
> -- Preliminary Discussion of the Logical Design of an Electronic Computing Instrument (1946)

## Memory Wall

Trong khi hiệu năng phần cứng tiếp tục cải thiện, có một khoảng cách dai dẳng và ngày càng tăng giữa những cải tiến trong phần cứng processor và các kết nối memory/thiết bị (**Processor-DRAM gap**, hay đơn giản là **memory gap**).[^att] "**Memory wall**"[^wulf-mckee] đặt ra một giới hạn đáng kể về hiệu năng cho nhiều workload hiện đại, đặc biệt trong AI.[^arxiv]

[^wulf-mckee]: Wm A. Wulf, Sally A. McKee. "Hitting the Memory Wall: Implications of the Obvious." ACM SIGARCH 1994. DOI: [10.1145/216585.216588](https://dl.acm.org/doi/10.1145/216585.216588)
[^att]:  Maurice Wilkes. "The memory gap and the future of high performance memories." ACM SIGARCH 2001. DOI: [10.1145/373574.373576](https://dl.acm.org/doi/abs/10.1145/373574.373576)
[^arxiv]: Amir Gholami et al. "AI and Memory Wall." IEEE Micro Journal 2024. Extended version on arXiV. DOI: [10.1109/MM.2024.3373763](https://doi.org/10.1109/MM.2024.3373763), [arXiV:2403.14123](https://arxiv.org/abs/2403.14123)

:::{figure} images/graph-processor-dram-gap.png
:label: fig-graph-processor-dram-gap-vi
:width: 100%
:alt: "Time-series graph showing the widening performance gap between processor speed and DRAM access speed over 30 years from 1980 to 2010."
Khoảng cách hiệu năng giữa DRAM và CPU.
:::

Bằng cách thiết kế một **memory hierarchy**, chúng ta có thể tận dụng lượng nhỏ phần cứng tốc độ cao mà không làm tăng vọt chi phí kiến trúc cũng như không hy sinh dung lượng dữ liệu và lưu trữ.

(sec-memory-hierarchy-revisited-vi)=
## Memory Hierarchy, Xem Lại

Trước đó, chúng ta giả định chỉ có **hai** tầng trong memory hierarchy: registers (trên CPU) và memory (DRAM gần, nhưng trên một chip riêng). Bây giờ chúng ta tiếp tục [thảo luận trước đó](#sec-memory-hierarchy-early) về memory hierarchy.

:::{embed} #fig-3-memory-hierarchy
:::

Sự không khớp giữa tốc độ processor và memory ("điệu tango cẩn thận" được mô tả [trước đó](#sec-memory-hierarchy)) dẫn chúng ta thêm một tầng mới: **Memory cache**, hay cache viết tắt. Cache thường nằm trên cùng chip với CPU và phù hợp với memory hierarchy như sau:

* **Kích thước**: Nhỏ hơn memory, nhưng chắc chắn lớn hơn 32 registers trên processor RISC-V của chúng ta.
* **Tốc độ**: Sử dụng phần cứng nhanh hơn nhiều so với DRAM (dùng cho main memory), nhưng chậm hơn registers.
* **Chi phí**: Sử dụng phần cứng đắt hơn DRAM.

Có các tầng bổ sung thấp hơn main memory: **disk** là một tầng lớn (theo nghĩa đen).

(sec-memory-hierarchy-copy-vi)=
:::{hint} Các tầng của memory hierarchy chứa các bản sao dữ liệu ở tầng thấp hơn
Giống như cache chứa **bản sao** của một tập con dữ liệu trong main memory, main memory chứa **các bản sao** dữ liệu trên disk. Các tầng khác nhau "đồng bộ hóa" các bản sao này bằng các phương pháp khác nhau.
:::

+++ {"label": "block-hierarchy-management-vi"}
Dữ liệu di chuyển khác nhau giữa các tầng khác nhau của memory hierarchy:

* **Registers và memory**: Được quản lý bởi compiler. Loads và stores di chuyển dữ liệu vào và ra.
* **Cache và memory**: Được quản lý bởi phần cứng cache controller. Chúng ta sẽ mô tả hoạt động cấp cao, nhưng để lại việc triển khai cho khóa học sau.
* **Memory và disk**: Được quản lý bởi hệ điều hành và phần cứng đặc biệt thông qua [virtual memory](#sec-virtual-memory), một khái niệm chúng ta thảo luận sau. Ngoài ra được quản lý bởi programmer/user thông qua files và file streams.
+++

Tóm lại, chúng ta hướng đến ảo giác của "bộ nhớ rất lớn và nhanh":

* Chúng ta làm cho bộ nhớ **nhanh** bằng cách sử dụng hierarchy, nơi các tầng cao hơn sử dụng phần cứng nhanh hơn, nhỏ hơn, và đắt hơn và nằm gần processor hơn về mặt vật lý.
* Chúng ta làm cho bộ nhớ **lớn** bằng cách tận dụng nguyên lý **locality** bằng cách "caching" dữ liệu "đúng" ở các tầng cao hơn, và ủy quyền các tầng thấp hơn để lưu trữ nhiều dữ liệu hơn. Tầng thấp nhất chứa tất cả dữ liệu có sẵn (mặc dù ngày nay chúng ta không đi đến magnetic disk và dừng lại ở SSD).

Nếu hữu ích, chúng ta xem lại [phép tương tự của Jim Gray](#sec-memory-hierarchy) về thời gian truy cập dữ liệu trên registers, trên cache, trong main memory, và trên disk.

:::{embed} #fig-3-locality
:::

(sec-multi-level-caches-vi)=
## Multi-Level Cache

Bạn có thể đã nhận thấy rằng [sơ đồ memory hierarchy](#fig-3-memory-hierarchy) chứa nhiều cache được đánh nhãn Level 1, Level 2, và Level 3. Một máy tính có thể có nhiều cache, trong đó mỗi cache là một **bản sao** dữ liệu từ tầng thấp hơn trong memory hierarchy.

Xem xét chip Apple A14 bionic, mà chúng ta đã giới thiệu [trước đó](#sec-intro-sds):

:::{embed} #fig-apple-a14
:::

L2 cache nằm trên mạch tích hợp, thường liền kề với CPU. System Level Cache được đánh nhãn trong sơ đồ có lẽ là Level 3 cache, được chia sẻ giữa nhiều lõi CPU.[^system-level-cache]

[^system-level-cache]: Chúng ta không thảo luận nhiều về L3 cache trong khóa học này. Xem [Wikipedia](https://en.wikipedia.org/wiki/CPU_cache).

:::{hint} IMEM và DMEM là các cache!

L1 cache thường được nhúng vào hai phần: **L1i** (instruction memory) và **L1d** (data memory). Đây **chính xác** là các khối IMEM và DMEM trên [RISC-V datapath](#sec-state-elements) của chúng ta!
:::

1. **L1 cache** (L1$[^cash-money]): Thường được nhúng trực tiếp vào CPU, do đó nó không được đánh nhãn trong sơ đồ trên.  
    * Kích thước: Hàng chục hoặc hàng trăm [KiB](#sec-iec-prefixes).
    * Hit Time (xem [thuật ngữ cache](#sec-cache-terminology)): Hoàn thành trong một clock cycle hoặc ít hơn.
    * Miss rate (xem [thuật ngữ cache](#sec-cache-terminology)): 1-5%
2. **L2 cache** (L2$): Nằm trên mạch tích hợp, thường liền kề với CPU.
    * Kích thước: Hàng chục hoặc hàng trăm MiB.
    * Hit Time: Vài clock cycles
    * Miss rate: 10-20%

[^cash-money]: Ký hiệu `$` cho cache là một sáng tạo của Berkeley. Không phải tôi :-)

### Demo

Để tìm ra kích thước của các thành phần khác nhau của memory hierarchy trên máy Linux, chúng ta có thể sử dụng `df` và `sysctl`. Các lệnh sau được chạy trên máy Mac OS X.

Để xác định **kích thước disk**, sử dụng `df`. Hiển thị mặc định là theo blocks (ví dụ, dòng); sử dụng tùy chọn `-h` cho tiền tố IEC (cơ số hai), và tùy chọn `-H` cho tiền tố cơ số 10.

```bash
$ df -h
Filesystem        Size    Used   Avail Capacity iused ifree %iused  Mounted on
/dev/disk3s1s1   460Gi    17Gi    38Gi    31%    427k  395M    0%   /
devfs            215Ki   215Ki     0Bi   100%     744     0  100%   /dev
...
$ df -H 
Filesystem        Size    Used   Avail Capacity iused ifree %iused  Mounted on
/dev/disk3s1s1    494G     18G     40G    31%    427k  395M    0%   /
devfs             220k    220k      0B   100%     744     0  100%   /dev
...
```

Để xác định **kích thước cache** và **kích thước memory**, sử dụng `sysctl`. Vì lệnh này liệt kê tất cả các thuộc tính của system kernel, chúng ta pipe output qua `grep` để lấy những gì chúng ta muốn. Đơn vị mặc định là bytes cho memory và cache.

```bash
$ sysctl -a | grep hw.memsize
hw.memsize: 25769803776
hw.memsize_usable: 25143640064
$ sysctl -a | grep "hw.l.*size"
hw.l1icachesize: 131072
hw.l1dcachesize: 65536
hw.l2cachesize: 4194304
```

:::{exercise} Ôn tập về [tiền tố nhị phân và cơ số 10](#sec-iec-prefixes)
:label: ex-cache-size-vi

Trong demo trên, kích thước L2 cache là bao nhiêu, tính bằng bytes?
* **A.** 2 MB
* **B.** 2 MiB
* **C.** 4 MB
* **D.** 4 MiB
* **E.** 4 GB
* **F.** 4 GiB
* **G.** Giá trị khác
:::

:::{solution} ex-cache-size-vi
:label: ex-cache-size-sol-vi
:class: dropdown

**D**.

```{math}
\begin{aligned}
4194304 \text{ B} &= 2^{(\log_2{4194304})} \text{ B} = 2^{22} \text{ B} \\
&= 4 \cdot 2^{20} \text{ B} = 4 \text{ MiB}
\end{aligned}
```
:::

(sec-storage-vi)=
## Storage

:::{warning} Nội dung này không được kiểm tra, nhưng...

Hiểu phần này hữu ích cho việc hiểu máy tính của bạn.
:::

::::{note} 🎥 Video Bài Giảng: Storage
:class: dropdown

:::{iframe} https://www.youtube.com/embed/wyIVDnsG8g0
:width: 100%
:title: "[CS61C FA20] Lecture 28.1 - OS & Virtual Memory Intro: Intro"
:::

::::

::::{note} 🎥 Video Bài Giảng: Actual CPUs
:class: dropdown

:::{iframe} https://www.youtube.com/embed/isEHXkkPtE4
:width: 100%
:title: "[CS61C FA20] Lecture 27.4 - Caches IV: Actual CPUs"
:::

::::
::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/MJwBmN8L2Lo
:width: 100%
:title: "[CS61C FA20] Lecture 29.2 - Virtual Memory I: Physical Memory and Storage"
:::

::::

:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vR4TRAAB71WlQqQUetxcAhTBdq7QfT0xqjMlLU-qT0OH5GTiGZEUPqLNrroMw6Dg2ERrOyPfnJHIu2y/pubembed?start=false&loop=false
:width: 100%
:title: "Slides trình bày về storage và memory hierarchy. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1dzVr8fWAnCVh8wSvONkBmx_bPnelngBnay_Mark2vT0/edit?usp=sharing)"
:::

Phiên bản viết sẽ sớm có, nhưng không quá sớm. Hiện tại, hãy biết các công nghệ sau:

* Cache sử dụng SRAM (static random access memory).
* Primary storage là memory. Công nghệ sử dụng là DRAM (dynamic random access memory), thường được gọi là RAM.
* Secondary storage là disk. Công nghệ sử dụng là SSD (solid-state drive), mặc dù trong quá khứ magnetic disk (hard drive) cũng phổ biến.
