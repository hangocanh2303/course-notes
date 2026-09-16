---
title: "Giới thiệu"
subtitle: "Mô hình bộ nhớ C là gì?"
---

(sec-mem-layout)=
## Mục tiêu học tập

* Học các đặc điểm của bố cục bộ nhớ C.
* Phân biệt giữa cấp phát lưu trữ và khai báo biến.
* Biết nơi các biến được lưu trữ trong C, tức là phân đoạn bộ nhớ nào dữ liệu được cấp phát.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/Keducx5bp-g?si=tat-NaUsgv7fdlRy
:width: 100%
:title: "[CS61C FA20] Lecture 05.3 - C Memory Management: Memory Locations"
:enumerated: false
:::

đến 8:37
::::

(sec-memory-layout)=
## Bố cục bộ nhớ C

Cho đến nay, chúng ta đã thảo luận về cách các kiểu dữ liệu khác nhau chiếm không gian (tức là cách toán tử thời gian biên dịch `sizeof` hoạt động với các khai báo khác nhau của biến cục bộ), nhưng chúng ta chưa thảo luận về nơi tất cả các thứ trong C sống trong bộ nhớ.

### Cấp phát bộ nhớ

Trong C, có ba cách cấp phát không gian lưu trữ trong bộ nhớ:

* **Khai báo biến cục bộ**: Để chỉ định một biến cục bộ, khai báo nó bên trong một hàm, ví dụ: trong `main()`. Các biến cục bộ có thể truy cập được trong hàm chúng được khai báo; nói cách khác, trình biên dịch C sẽ chỉ nhận ra một biến cục bộ trong **namespace** của hàm đó. Chúng được khai báo và cấp phát tại thời điểm biên dịch (do đó `sizeof` hoạt động).
* **Khai báo biến toàn cục**: Các biến toàn cục được khai báo bên ngoài hàm. Các biến được khai báo trong **namespace** toàn cục có thể truy cập được bởi mọi chương trình con (`main()`, `foo()`, v.v.). Cảnh báo: Không lạm dụng biến toàn cục! Mặc dù chúng hữu ích để truy cập các hằng số dùng chung hoặc bảng lớn, bạn có thể viết code tệ nhất thế giới bằng cách có hàng tỷ biến toàn cục: bạn có nguy cơ bị chỉnh sửa ngẫu nhiên bởi các routine khác nhau, và bạn loại bỏ sức mạnh của trừu tượng hóa giữa các hàm. Biến toàn cục được khai báo và cấp phát tại thời điểm biên dịch.
* **Cấp phát bộ nhớ động**: Đôi khi chúng ta sẽ không biết cần bao nhiêu không gian cho đến khi chúng ta bắt đầu chạy chương trình. Ví dụ, chúng ta có thể cần tải dữ liệu từ một file, và chúng ta chỉ biết có bao nhiêu dữ liệu trong file cho đến khi đọc tất cả các byte của nó. Trong trường hợp này, chúng ta có thể cấp phát lưu trữ bộ nhớ động tại thời gian chạy bằng cách gọi các hàm cấp phát: `malloc` là một hàm phổ biến. Đọc thêm [khi chúng ta thảo luận về heap](#sec-heap).

Dựa trên các mô tả ở trên, lưu ý rằng **cấp phát lưu trữ** liên quan đến việc dành riêng một khối bộ nhớ để đặt dữ liệu vào. **Khai báo biến** có nghĩa là cấp phát một khối bộ nhớ và cũng chỉ định một tên để tham chiếu đến bộ nhớ đó. Vì các biến có kiểu, khai báo biến ngầm chỉ định kích thước của khối bộ nhớ cần cấp phát. Trong khi khai báo biến luôn ngụ ý cấp phát lưu trữ, điều ngược lại không đúng (ví dụ: với cấp phát bộ nhớ động).

### Bốn vùng của bố cục bộ nhớ

Không gian địa chỉ của chương trình C chứa 4 vùng như trong @fig-c-mem-layout.

:::{figure} images/c-mem-layout.png
:label: fig-c-mem-layout
:width: 50%
:alt: "Sơ đồ không gian địa chỉ C với text ở địa chỉ thấp nhất, data phía trên text, heap phía trên data tăng lên trên, và stack ở địa chỉ cao tăng xuống dưới. Khoảng trống được tô màu giữa heap và stack cho thấy không gian trống có sẵn cho tăng trưởng runtime."

Bố cục bộ nhớ chương trình C.
:::

| Phân đoạn bộ nhớ | Nội dung | Quản lý bộ nhớ |
| :--- | :--- | :--- |
| **Stack** | biến cục bộ, tham số, và địa chỉ trả về [^stack-info] | tự động; tăng trưởng _xuống dưới_[^stack-sec] |
| **Heap** | lưu trữ được cấp phát động | thay đổi kích thước theo yêu cầu (ví dụ: `malloc` để cấp phát lưu trữ, `free` để giải phóng lưu trữ); tăng trưởng _lên trên_[^heap-sec] |
| **Data** (hay **Static**)[^data-rodata] | biến toàn cục | kích thước cố định ("static") trong suốt chương trình |
| **Text** (hay **Code**)[^text] | mã chương trình | kích thước cố định trong suốt chương trình; dữ liệu được tải khi chương trình bắt đầu |

Tên của bốn phân đoạn bộ nhớ này khó nhớ lúc đầu, vì vậy chúng tôi xin lỗi thay mặt cho tất cả các nhà khoa học máy tính. Trong khi **stack** hoạt động rất giống cấu trúc dữ liệu stack bạn học trong khóa Cấu trúc dữ liệu, **heap** KHÔNG phải là cấu trúc dữ liệu heap; nó chỉ là một "đống bộ nhớ." Ngoài ra, mọi thứ về mặt kỹ thuật đều là dữ liệu, nhưng phân đoạn **data** cụ thể đề cập đến dữ liệu toàn cục[^data-rodata]. Cuối cùng, **text** có thể được nhớ vì mã chương trình nên là dữ liệu chỉ đọc, giống như nhiều văn bản bạn đọc trong đời thực là chỉ đọc.

Lập trình trong C đòi hỏi phải biết dữ liệu ở đâu trong bộ nhớ[^java-python-mem]. Nếu không, mọi thứ không hoạt động như mong đợi. Đặc biệt, bộ nhớ trong mỗi vùng trong bốn vùng được **quản lý** khác nhau. Nguồn lỗi lớn nhất trong C là từ các giả định không chính xác về bộ nhớ. Bài giảng này hoàn toàn về việc học cách bộ nhớ được quản lý để tránh các lỗi phổ biến. Chúng ta tập trung thảo luận vào stack và heap.

[^stack-info]: Vì các tham số và địa chỉ trả về rất quan trọng đối với việc gọi và trả về hàm, chúng được lưu trữ trực tiếp trong CPU nếu có thể — trên phần cứng đặc biệt gọi là thanh ghi (mà chúng ta nói sau). Vì chỉ có số lượng thanh ghi hạn chế, các tham số và địa chỉ trả về bổ sung được lưu trong bộ nhớ trên stack cho đến khi cần.
[^stack-sec]: Đọc thêm về [stack](#sec-stack).

[^heap-sec]: Đọc thêm về [heap](#sec-heap).

[^data-rodata]: Chúng ta chỉ đề cập đến phân đoạn `.data` trong khóa học này và bỏ qua dữ liệu chỉ đọc. Đọc thêm trên [Wikipedia](https://en.wikipedia.org/wiki/Data_segment).

[^text]: Đọc thêm trên [Wikipedia](https://en.wikipedia.org/wiki/Code_segment).

[^java-python-mem]: Ngược lại, Java và Python đều ẩn vị trí của các đối tượng.
