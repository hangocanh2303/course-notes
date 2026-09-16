---
title: "Tổng kết"
---

## Và để kết luận$\dots$

C không tự động xử lý bộ nhớ cho bạn, vì vậy tùy thuộc vào bạn, lập trình viên, để cấp phát,
sử dụng, và giải phóng bộ nhớ đúng cách. Trong mỗi chương trình, một không gian địa chỉ được dành riêng, được tách thành 2
vùng thay đổi động và 2 vùng 'tĩnh'.

* **Stack**: Lưu trữ các biến cục bộ bên trong các hàm. Dữ liệu trên stack được thu hồi tự động —
ngay lập tức sau khi hàm mà nó được định nghĩa trong đó trả về. Mỗi lời gọi hàm tạo một stack
frame giữ các đối số và biến cục bộ của hàm. Stack tăng trưởng xuống dưới với
các lời gọi hàm lồng nhau (cấu trúc LIFO), và thu nhỏ lên trên khi các hàm trả về.
* **Heap**: Lưu trữ bộ nhớ được cấp phát thủ công bởi lập trình viên với malloc, calloc, hoặc
realloc. Được sử dụng cho dữ liệu cần tồn tại sau khi hàm trả về. Tăng trưởng lên trên trong
bộ nhớ để 'gặp' stack. Bộ nhớ trên heap chỉ được giải phóng khi lập trình viên rõ ràng
giải phóng nó. Quản lý heap cẩn thận là cần thiết để tránh các lỗi khó khăn, khó tái tạo (Heisenbug).
* **Data (hoặc Static)**: Lưu trữ dữ liệu có kích thước cố định, như biến toàn cục và chuỗi ký tự. Không
tăng hoặc giảm qua quá trình thực thi hàm.
* **Text (hoặc Code)**: Được tải khi bắt đầu chương trình và không thay đổi sau đó, chứa
các lệnh thực thi và bất kỳ macro tiền xử lý nào.

Có một số hàm trong C có thể được sử dụng để cấp phát bộ nhớ động trên
heap. Sau đây là những hàm chúng ta sử dụng trong lớp này:

* `malloc(size_t size)` cấp phát một khối size byte và trả về đầu khối. Thời gian mất để tìm kiếm một khối thường không phụ thuộc vào size.
* `calloc(size_t count, size_t size)` cấp phát một khối `count * size` byte, **đặt mọi giá trị trong khối thành không**, rồi trả về đầu khối.
* `realloc(void *ptr, size_t size)` "thay đổi kích thước" một khối bộ nhớ đã cấp phát trước đó thành
`size` byte, trả về đầu khối đã thay đổi kích thước.
* `free(void *ptr)` giải phóng một khối bộ nhớ bắt đầu từ `ptr` đã được
cấp phát trước đó bởi ba hàm trước.

Hãy cẩn thận khi cấp phát buffer trên stack và heap! Heap là nguồn lỗi tinh vi lớn nhất trong mã C.

Học CS 162 để biết thêm!

## Đọc sách giáo khoa

K&R 7.8.5, 8.7

## Tài liệu tham khảo bổ sung

* Ghi chú về C của Giáo sư danh dự Brian Harvey [notes on C](https://inst.eecs.berkeley.edu/~cs61c/resources/HarveyNotesC1-3.pdf)
* Ghi chú về quản lý bộ nhớ của Giáo sư danh dự Paul Hilfinger [notes on memory management](https://inst.eecs.berkeley.edu/~cs61c/sp21/resources-pdfs/pnh.stg.mgmt.pdf)

## Bài tập
Kiểm tra kiến thức của bạn!

### Ôn tập khái niệm

:::{exercise}
:label: c-mm-01
1. **Đúng/Sai**: Các phân đoạn bộ nhớ được định nghĩa bởi phần cứng, và không thể thay đổi.
:::

:::{solution} c-mm-01
:label: c-mm-01-sol
:class: dropdown

**Sai.** Bốn phân đoạn bộ nhớ chính, stack, heap, static/data, và text/code cho bất kỳ tiến trình (ứng dụng) nào được định nghĩa bởi hệ điều hành và có thể khác nhau tùy thuộc vào loại bộ nhớ cần thiết để nó chạy.
:::

:::{exercise}
:label: c-mm-02
2. Ví dụ về tiến trình này có thể cần không gian stack đáng kể, nhưng rất ít text, static data, và không gian heap?
:::

:::{solution} c-mm-02
:label: c-mm-02-sol
:class: dropdown

(Hầu như bất kỳ scheme đệ quy sâu cơ bản nào, vì bạn đang thực hiện nhiều lời gọi hàm mới chồng lên nhau mà không đóng các lời gọi trước đó, và do đó, stack frame.)
:::

:::{exercise}
:label: c-mm-03
3. Ví dụ về tiến trình nặng text- và static data-?
:::

:::{solution} c-mm-03
:label: c-mm-03-sol
:class: dropdown
(Có lẽ một tiến trình cực kỳ phức tạp nhưng có sử dụng stack hiệu quả và không cấp phát bộ nhớ động.)
:::

:::{exercise}
:label: c-mm-04
4. Ví dụ về tiến trình nặng heap?
:::

:::{solution} c-mm-04
:label: c-mm-04-sol
:class: dropdown
(Có lẽ nếu bạn đang sử dụng nhiều bộ nhớ động mà người dùng cố gắng truy cập.)
:::
