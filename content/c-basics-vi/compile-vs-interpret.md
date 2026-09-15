---
title: "Biên dịch vs. Thông dịch"
---

(compile-vs-interpret-sec)=
## Mục tiêu học tập

* Làm quen với quy trình cấp cao của việc thực thi chương trình có thể biên dịch.
* Hiểu các đánh đổi giữa biên dịch và thông dịch.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/cFN0bX8mlmg?si=BRTt2o6rNEY9crMl
:width: 100%
:title: "[CS61C FA20] Lecture 03.2 - C Intro: Basics: Compile v. Interpret"
:::

::::

## Biên dịch vs. Thông dịch

Có hai cách chính để một chương trình được máy tính chạy: biên dịch và thông dịch.

C là ngôn ngữ biên dịch. **Compiler** C ánh xạ các chương trình C trực tiếp thành **machine code** cụ thể cho kiến trúc, hoặc các chuỗi bit gồm `1` và `0`.
Một **executable** là file gồm machine code nhị phân này có thể được thực thi trên máy tính của bạn. Các executable được tạo bằng cách biên dịch mã nguồn.

Các ngôn ngữ có thể biên dịch cho phép chúng ta chuyển chương trình dễ dàng hơn giữa các kiến trúc khác nhau. Ví dụ, năm 2020, Apple quyết định thay đổi kiến trúc cho dòng máy tính Mac của họ. Họ chuyển từ bộ xử lý x86 dựa trên Intel sang bộ xử lý ARM. Ngay cả với bước chuyển lớn này, các chương trình C không thay đổi _nhiều lắm_. Thay vào đó, sự thay đổi xảy ra trong chính các compiler, cũng là các chương trình. Chúng được viết lại để xử lý việc dịch từ ngôn ngữ C cấp cao sang các kiến trúc lệnh mới.

Các chương trình Python và Java so sánh như thế nào? Chúng khác nhau chủ yếu ở _khi nào_ một chương trình được chuyển đổi thành các lệnh máy cấp thấp.

* Java: Chuyển đổi thành bytecode độc lập với kiến trúc, sau đó được biên dịch bởi compiler just-in-time (JIT)
* Python: **Thông dịch**. Chuyển đổi thành byte code tại runtime.

### Biên dịch: Ưu điểm

**1. Thời gian biên dịch hợp lý.** Tưởng tượng bạn có hai chương trình, `foo.c` và `bar.c`. Thay đổi `foo.c` sẽ không ngụ ý rằng `bar.c` cần được biên dịch lại. Quy trình này được phối hợp thông qua `Makefile`, mà bạn sẽ thấy trong một khóa học tương lai.

**2. Hiệu năng runtime thường nhanh hơn nhiều.** C biên dịch thường sẽ chạy nhanh hơn so với code Java tương đương về chức năng. Rốt cuộc, quy trình biên dịch tối ưu hóa code cho một kiến trúc cho trước.

Lưu ý rằng tùy thuộc vào ứng dụng của bạn, bạn vẫn có thể thích Python hơn vì có các thư viện được viết cho Python được tối ưu hóa cho GPU; các thư viện có thể sử dụng tương đương có thể không tồn tại cho C. Python cũng có [Cython](https://en.wikipedia.org/wiki/Cython), mà bạn có thể thấy trong một lớp tương lai.

## Biên dịch: Nhược điểm

**1. Các file đã biên dịch, bao gồm executable, là cụ thể cho kiến trúc.** Executable phụ thuộc vào loại bộ xử lý (ví dụ, MIPS vs. x86 vs. RISC-V) và hệ điều hành (ví dụ, Windows vs. Linux vs. MacOS).
"**Porting** code của bạn" sang một kiến trúc mới có nghĩa là xây dựng lại executable: sao chép file `.c`, sau đó biên dịch lại bằng `gcc`.

**2. Chu kỳ phát triển chậm hơn**: Không giống Python, C không thực sự có vòng lặp "đọc-đánh giá-in" (REPL). Thay vào đó, chu kỳ là "chỉnh sửa file, biên dịch, liên kết, chạy, tìm lỗi", nghĩa là quá trình phát triển có thể chậm hơn nhiều.

**3. Liên kết là nút thắt cổ chai.** Một executable chương trình sẽ cần được biên dịch lại khi bất kỳ phần nào của chương trình thay đổi. Biên dịch là một quy trình dài! Trong khi một số phần của quy trình biên dịch có thể được tăng tốc—ví dụ, các phần con chương trình độc lập có thể được biên dịch song song—các phần khác vẫn tuần tự, như giai đoạn liên kết (mà chúng ta sẽ nói về sau). "Nút thắt tuần tự" này là một ví dụ của **Định luật Amdahl**; thêm sau.

## "Biên dịch" như Thuật ngữ Thông thường

**Biên dịch** một chương trình C theo cách nói thông thường đề cập đến toàn bộ quy trình sử dụng compiler để dịch các chương trình C thành executable. Chúng ta sẽ sử dụng thuật ngữ này bây giờ.

Trong thực tế, quy trình đầy đủ này có nhiều bước:

1. Biên dịch các file `.c` thành các file `.o`
1. Assemble tự động
1. Liên kết các file .o thành một executable.

Chúng ta sẽ thảo luận đây như một quy trình bốn giai đoạn ("CALL": Compile, Assemble, Link, Load) sau này trong khóa học.

## Lỗi Compile-time vs. Lỗi Runtime

Với quy trình hai bước này, khi code bằng C bạn có thể gặp hai loại lỗi.

**Lỗi compile-time** thường dựa trên cú pháp, ví dụ, bạn quên dấu chấm phẩy. Vì C là ngôn ngữ dựa trên kiểu, lỗi compile-time cũng sẽ phát sinh nếu bạn sử dụng một phép toán không hợp lệ trên một biến cụ thể. Ví dụ, toán tử chia (`/`) không được định nghĩa cho [kiểu biến con trỏ](#sec-pointers), lưu trữ địa chỉ, và một chương trình cố gắng làm như vậy sẽ kích hoạt lỗi tại compile time.

**Lỗi runtime** xảy ra trong quá trình thực thi chương trình. Lỗi runtime phổ biến nhất là **segfault**, hoặc lỗi phân đoạn. Segfault xảy ra khi bạn cố truy cập một phần bộ nhớ "không thuộc về bạn."

Khi lập trình bằng C, có nhiều cách mà segfault có thể xảy ra. Điều quan trọng cần lưu ý là tùy thuộc vào chương trình chính xác của bạn, không phải mọi trường hợp bên dưới đều có thể gây ra segfault!

1. Dereference một con trỏ null. Điều này sẽ _luôn_ kích hoạt segfault. Đọc thêm về [con trỏ](#sec-pointers).
1. Cố gắng ghi vào bộ nhớ chỉ đọc. Điều này sẽ _luôn_ kích hoạt segfault. Đọc thêm về [bố cục bộ nhớ](#sec-mem-layout), và xem ví dụ khi chúng ta thảo luận về [chuỗi](#sec-strings).
1. Truy cập chỉ số vượt giới hạn trên một mảng. Chỉ số mà segfault sẽ xảy ra có phần không thể đoán trước, do đó có rủi ro bảo mật của [buffer overflow](#sec-array).
1. Truy cập một con trỏ đến heap đã được `free` trước đó. Điều này phụ thuộc vào implementation; đọc thêm về [heap](#sec-heap).
1. Nhiều trường hợp tiềm năng khác!

Danh sách trên sẽ có vẻ như một danh sách các sự kiện rời rạc, đặc biệt đối với sinh viên không quen thuộc với C. Chúng tôi khuyên bạn quay lại danh sách này sau khi bạn đã đọc thêm về con trỏ, mảng, và bố cục bộ nhớ.
