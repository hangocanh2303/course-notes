---
title: "Triển khai bộ nhớ Heap"
subtitle: "Nội dung này không được kiểm tra"
---

(sec-heap-allocator)=
## Mục tiêu học tập

Phần này được bao gồm như nội dung bonus và không được kiểm tra. Nếu bạn tò mò về việc triển khai heap allocator của riêng mình, hãy học CS 162: Operating Systems!

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/Sq5tSeWfnGY?si=lhSSc2EofyEeO4ar
:width: 100%
:enumerated: false
:title: "[CS61C FA20] Lecture 05.4 - C Memory Management: Memory Management"
:::

::::

```{embed} #sec-os-overview
```

## Thiết kế Heap Allocator

Quản lý heap rất khó. Trong khi bạn không cần lo lắng về việc OS di chuyển mọi thứ xung quanh, bạn phải quản lý các yêu cầu kích thước của riêng mình và cách bạn làm việc với bộ nhớ đó.

Một thiết kế lý tưởng:

* Có triển khai nhanh của `malloc` và `free`
* Tạo ra overhead tối thiểu, tức là "bookkeeping" cần thiết để theo dõi bộ nhớ.
* Tránh **phân mảnh (fragmentation)**, tức là nơi hầu hết bộ nhớ trống nằm trong nhiều chunk nhỏ. Điều này ngụ ý nhiều byte trống nhưng không có khả năng đáp ứng một yêu cầu lớn vì các byte trống không liền kề trong bộ nhớ.

**Phân mảnh ngoài (External fragmentation)**: Nếu bạn yêu cầu 100 byte (R1) rồi 1 byte (R2), và rồi bạn free R1, bộ nhớ của bạn bây giờ bị phân mảnh thành hai vùng riêng biệt. Khi yêu cầu thứ ba (R3) đến, hệ thống phải thông minh về nơi đặt nó dựa trên các mẫu trước đây.

## Triển khai Malloc/Free của K&R

Một triển khai của heap allocator là từ Phần 8.7 của K&R. Mã này sử dụng một số tính năng ngôn ngữ C mà chúng ta chưa thảo luận và được viết theo phong cách rất ngắn gọn; đừng lo lắng nếu bạn không thể giải mã mã.

Bookkeeping: Mỗi khối bộ nhớ có một header chứa **kích thước** của khối và một **con trỏ đến khối tiếp theo**. Điều này tạo ra một **danh sách liên kết vòng** của tất cả các khối trống.

`malloc()`: Khi bạn đưa ra yêu cầu, hệ thống tìm kiếm danh sách trống này để xem liệu nó có thể phục vụ bộ nhớ bạn muốn không. Nếu nó duyệt toàn bộ danh sách và không tìm thấy gì, thêm bộ nhớ được yêu cầu từ hệ điều hành. Nếu OS không thể đáp ứng yêu cầu, nó trả về `NULL`, báo hiệu rằng yêu cầu thất bại.

`free()`: Kiểm tra xem các khối liền kề với khối được freed cũng trống không. Nếu có, các khối trống liền kề được hợp nhất (**coalesced**) thành một khối trống lớn hơn duy nhất. Nếu không, khối được freed chỉ được thêm vào danh sách trống.

Điều quan trọng cần nhận ra là malloc không phải "tức thì" như cấp phát mảng stack, mất khoảng một chu kỳ đồng hồ. malloc là một lời gọi hàm, nghĩa là stack phải tăng chỉ để gọi nó, và rồi malloc phải thực hiện cơ chế nội bộ của việc duyệt danh sách vòng đó. Trong trường hợp xấu nhất, có thể mất nhiều thời gian để tìm kiếm toàn bộ danh sách chỉ để trả về `NULL`, làm chương trình của bạn bị đình trệ.

## Chọn khối trong `malloc`

Nếu có nhiều khối bộ nhớ trống đủ lớn cho một yêu cầu nào đó, làm thế nào chúng ta chọn cái nào để sử dụng? Có ba chiến lược phổ biến:

1. **Best Fit**: Điều này tìm kiếm toàn bộ danh sách để tìm khối có kích thước gần nhất với những gì bạn yêu cầu. Trong khi điều này cung cấp sự phù hợp chặt chẽ nhất, nó thường để lại những "mảnh vụn" bộ nhớ nhỏ xíu (như khoảng trống 4-byte nếu bạn yêu cầu 100 byte từ một khối 104-byte), làm danh sách trống dài hơn và dài hơn.
2. **First Fit**: Bạn lấy khối đầu tiên bạn tìm thấy đủ lớn. Nó nhanh, nhưng có xu hướng tạo ra nhiều "viên sỏi" nhỏ hoặc mảnh vụn ở đầu danh sách mà bạn phải bỏ qua sau này.
3. **Next Fit**: Điều này giống First Fit, nhưng thay vì bắt đầu từ đầu danh sách mỗi lần, nó tiếp tục tìm kiếm từ nơi nó dừng lần trước. Điều này giúp phân phối "viên sỏi" đều hơn trong suốt danh sách.
