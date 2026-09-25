---
title: "Load từ, Store vào"
---

(sec-load-store)=
## Mục tiêu học tập

* Nhớ câu châm ngôn: Load _từ_ bộ nhớ, Store _vào_ bộ nhớ.
* Giải thích tại sao một ISA cần định nghĩa các lệnh để truy cập bộ nhớ.

::::{note} 🎥 Video bài giảng - Load từ, Store vào
:class: dropdown

:::{iframe} https://www.youtube.com/embed/wXGhuhLKkqg
:width: 100%
:title: "[CS61C FA20] Lecture 08.1 - RISC-V lw, sw, Decisions I: Storing in Memory"
:::

Đến 7:33[^egg-test]. Ngoài ra, có một phần ôn tập hữu ích về endianness, mà chúng ta đã đề cập trong [phần trước](#sec-endianness).

[^egg-test]: Video bài giảng này bắt đầu với một bài kiểm tra trứng thú vị. Cá nhân tôi tin rằng nó sẽ giúp bạn nắm bắt các khái niệm của sách giáo khoa P&H, nhưng tôi chưa thử.

::::

Vì thanh ghi khan hiếm, việc cẩn thận và tái sử dụng chúng như bộ nhớ tạm thời là quan trọng. Nhìn chung, việc tối thiểu hóa footprint thanh ghi là công việc của compiler tối ưu hóa. Tuy nhiên, khi chúng ta làm việc với lượng dữ liệu lớn hơn, chúng ta phải "tràn" ra khỏi thanh ghi và vào bộ nhớ.

RISC-V định nghĩa các lệnh để truy cập bộ nhớ.
Nhớ lại bố cục máy tính cơ bản của chúng ta (@fig-von-neumann). Bộ xử lý giao tiếp với bộ nhớ bằng cách phát hành **địa chỉ** để đọc hoặc ghi dữ liệu:

* **Load từ** bộ nhớ: Đọc dữ liệu từ bộ nhớ vào bộ xử lý.
* **Store vào** bộ nhớ: Ghi dữ liệu từ bộ xử lý vào bộ nhớ.

:::{warning} Hướng nào cho truy cập bộ nhớ?

Với tư cách là kiến trúc sư máy tính, thế giới của chúng ta lấy bộ xử lý làm trung tâm–vì đó là nơi tất cả hoạt động diễn ra. Vì vậy hướng của phép toán này là theo quan điểm của phép toán: load _từ_, store _vào_.

Việc thực hành sử dụng thuật ngữ đúng là quan trọng; làm như vậy sẽ giúp bạn nội hóa tốt hơn các khái niệm cốt lõi!
:::

## RISC-V là Load-Store

RISC-V được gọi là [kiến trúc load-store](https://en.wikipedia.org/wiki/Load%E2%80%93store_architecture)[^load-store]—ám chỉ cách nó định nghĩa truy cập bộ nhớ và các phép toán trên dữ liệu (@fig-rv-load-store).

:::{figure} images/load-store.png
:label: fig-rv-load-store
:width: 100%
:alt: "Sơ đồ lấy bộ xử lý làm trung tâm: thanh ghi x4 đối mặt với một từ bốn byte trong bộ nhớ tại 0x100 với các offset byte cộng zero đến cộng ba. Một mũi tên tím có nhãn load từ chỉ từ bộ nhớ đến thanh ghi; một mũi tên xanh có nhãn store chỉ từ thanh ghi đến bộ nhớ, minh họa cách các phép toán load và store hoạt động."

Load và store giữa thanh ghi và bộ nhớ.
:::

RISC-V chỉ cho phép các phép toán trên dữ liệu trong **thanh ghi**. Bất kỳ phép toán nào trên dữ liệu từ bộ nhớ phải liên quan đến nhiều lệnh:

1. **Load** dữ liệu từ bộ nhớ vào thanh ghi
2. **Thực thi** phép toán trên thanh ghi trong bộ xử lý.
3. (nếu cần) **Store** dữ liệu trở lại bộ nhớ.

[^load-store]: Các mô hình khác tồn tại, thường ở đất CISC. ISA x86 cho phép các phép toán số học trong đó một toán hạng ở trong bộ nhớ và toán hạng còn lại ở trong thanh ghi.

RISC-V định nghĩa một tập các lệnh `load` và `store` để di chuyển dữ liệu giữa bộ nhớ và thanh ghi. Hãy xem các lệnh này!
