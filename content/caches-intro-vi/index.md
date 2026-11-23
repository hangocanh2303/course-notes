---
title: "Phép Tương Tự Thư Viện"
---

(sec-library-vi)=
## Mục Tiêu Học Tập

* Phát triển phép tương tự thư viện để hiểu về memory cache.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/Daj8eXa1mA0
:width: 100%
:title: "[CS61C FA20] Lecture 24.2 - Caches I: Library Analogy"
:::

::::

Trong chương này, chúng ta tập trung vào khái niệm **bộ nhớ** và thảo luận về cách thiết kế bộ nhớ ảnh hưởng quan trọng đến hiệu năng máy tính. Trong hình dưới đây, chúng ta tập trung vào cách hai thành phần, **cache** và **memory**, tương tác với nhau.

:::{embed} #fig-great-idea-new-school
:::

Để nhắc lại (hehe), chúng ta xem lại hình ảnh về các thành phần của máy tính từ [phần trước đó](#sec-architecture-elements). Chúng ta sẽ thấy trong chương này rằng sự thật phức tạp hơn hình ảnh này; chúng ta muốn chèn **cache** vào đâu đó. Nhưng như thế nào?

:::{embed} #fig-von-neumann
:enumerated: false
:::

## Thư Viện: Phép Tương Tự Về Bộ Nhớ

Để hiểu về cache, có một phép tương tự tốt sẽ rất hữu ích.[^ok-boomer] Hãy xem lại cách tìm sách kiểu cũ trong một thư viện lớn.

[^ok-boomer]: Chúng ta biết rằng hầu hết thư viện ngày nay có danh mục điện tử, và tìm kiếm đơn giản chỉ là một số tìm kiếm internet với các key và bảng được ánh xạ phù hợp. Phép tương tự của chúng ta xem xét thời kỳ trước danh mục điện tử—danh mục thẻ, nơi bạn cuộn một bánh xe khổng lồ các thẻ để tra cứu vị trí sách của mình (theo thời gian tuyến tính).

Giả sử bạn là sinh viên Cal và bạn muốn tìm một cuốn sách trong thư viện chính, hay [Main Stacks](https://www.lib.berkeley.edu/visit/main-stacks). Có hai giai đoạn trong quá trình tìm kiếm của bạn:

1. **Tìm kiếm trong danh mục thẻ lớn**. Danh mục thư viện[^ok-boomer] là một danh mục khổng lồ ánh xạ tiêu đề/tác giả đến số chỉ mục (ví dụ, [Hệ thống thập phân Dewey](https://en.wikipedia.org/wiki/Dewey_Decimal_Classification)).
2. **Mượn sách**. Thực hiện chuyến đi khứ hồi để đi đến Main Stacks, lấy cuốn sách mong muốn, mượn sách với thủ thư, và đi bộ trở lại bàn của bạn.

### Tại sao bộ nhớ lớn lại chậm?

Thư viện lớn hơn làm tăng cả hai độ trễ: Thư viện lớn sẽ có danh mục mở rộng, làm cho việc tìm kiếm lâu hơn. Hơn nữa, thư viện lớn về mặt vật lý có nhiều sách hơn, vì vậy bạn sẽ phải đi xa hơn trong chuyến đi khứ hồi để lấy sách.

Giống như thư viện là một "bộ nhớ" mở rộng của tất cả các sách trong khuôn viên trường, bộ nhớ điện tử cũng có vấn đề tương tự. Độ trễ trong bộ nhớ được nhân lên bởi công nghệ được sử dụng (ví dụ, SRAM vs. DRAM vs. disk, được thảo luận trong [phần phụ bonus bên dưới](#sec-storage)).

:::{hint} Mục tiêu của kiến trúc bộ nhớ.
Điều chúng ta muốn là một bộ nhớ **lớn** nhưng **nhanh**. Mặc dù về lý thuyết là không thể, nhưng trong thực tế chúng ta muốn tạo ra **ảo giác** này về một bộ nhớ lớn mà chúng ta có thể truy cập nhanh như một bộ nhớ rất nhỏ. Bề ngoài, chúng ta phải xuất hiện như đang truy cập một bộ nhớ thực sự lớn—lớn nhất có thể về mặt vật lý—nhưng nhanh như phần tử nhỏ nhất—một thanh ghi. Liệu chúng ta có thể đạt được điều đó? Hãy cùng tìm hiểu.
:::

## Thư Viện, tiếp tục: Bài Báo Cáo Sách và Một Cái Bàn

Làm thế nào bạn viết một báo cáo sử dụng sách thư viện? Trong thực tế[^ok-boomer-2], bạn sẽ cố gắng giảm thiểu các chuyến đi xuống main stacks như sau:

[^ok-boomer-2]: Trong thực tế ngày nay, bạn có thể cũng sẽ không đến thư viện vật lý và thay vào đó sử dụng các tài nguyên thư viện điện tử có sẵn. Nhưng hãy làm việc với chúng tôi ở đây. Giả sử bước đầu tiên của bạn là bạn đi đến thư viện.

1. Tìm kiếm danh mục và tra cứu một cuốn sách.
1. Mượn sách từ Main Stacks (tức là, thực hiện chuyến đi khứ hồi)
1. Đặt sách lên bàn. Bạn tìm một cái bàn ở trong thư viện, gần Main Stacks.
1. Nếu bạn cần thêm, lặp lại quy trình và thêm sách mới vào bàn của bạn. Đây là quy trình của **locality**: chưa trả lại những cuốn sách trước đó, vì bạn có thể cần chúng sớm.

Giả sử có một giới hạn về số lượng sách–giả sử, mười cuốn sách–mà bạn có thể mở và trải ra trên bàn.[^librarian] Trong trường hợp hiếm hoi bạn cần cuốn sách thứ 11 hoặc 12, bạn có thể đi lấy nó, nhưng phần lớn, "working set" dữ liệu của bạn rất hiệu quả. Thời gian để bạn lấy một cuốn sách trên bàn ngắn hơn nhiều so với thời gian để bạn đi đến main stacks và đọc sách. Nếu bàn của bạn nhỏ hơn (giả sử, như những cái trong HP Auditorium hoặc Wheeler auditorium), bạn sẽ không thể lưu nhiều sách cục bộ trên bàn, và bạn sẽ phải thực hiện nhiều chuyến đi hơn đến Main Stacks.

[^librarian]: Giả sử thủ thư đến và nói rằng bạn không thể chồng sách lên nhau, hoặc điều gì đó tương tự.

Phần quan trọng của phép tương tự này là **bàn** của bạn ở trong thư viện, gần Main Stacks nhưng không ở đó. Bàn là **memory cache** của bạn: nó là quyền truy cập cục bộ, nhanh, và _nhỏ hơn_ đến những cuốn sách mà bạn đang sử dụng ngay lập tức. Main Stacks là **main memory**–nó có mọi thứ có thể, nhưng lấy sách từ đó phát sinh thêm chi phí. Bộ sưu tập mười cuốn sách trên bàn của bạn đủ hợp lý để viết phần lớn báo cáo của bạn, mặc dù mười cuốn sách chỉ là 0.00001% số sách trong Main Stacks (và tất cả các thư viện UC Berkeley).

:::{warning} Cache chứa các bản sao của dữ liệu từ main memory

Có một lưu ý cho phép tương tự thư viện này: không giống như sách trong thư viện, memory cache hoạt động trên **các bản sao** dữ liệu từ main memory. Chúng ta sẽ xem lại lưu ý này khi chúng ta định nghĩa chính thức memory cache.
:::
