---
title: "Tóm tắt"
---

## Và Kết luận$\dots$

* C được chọn để khai thác các tính năng cơ bản của phần cứng.
  * Chúng ta sẽ bắt đầu thảo luận về quản lý bộ nhớ chi tiết hơn lần sau với con trỏ và mảng.
* C được biên dịch và liên kết để tạo executable.
  * Ưu điểm: Tốc độ
  * Nhược điểm: Chu kỳ chỉnh sửa-biên dịch chậm
* C trông chủ yếu giống Java, ngoại trừ không có lập trình hướng đối tượng
  * Các Kiểu Dữ liệu Trừu tượng được định nghĩa thông qua struct; thêm lần sau
  * `bool`: `0` và `NULL` là false. Mọi thứ khác là true.
  * Sử dụng `intN_t` và `uintN_t` cho code di động!
* Các biến chưa khởi tạo chứa **rác**.
  * "Bohrbugs" (có thể lặp lại) vs "Heisenbugs" (ngẫu nhiên)

## Đọc thêm từ Textbook

K&R Ch. 1-5

## Tài liệu Tham khảo Bổ sung

* [C reference Slides](https://inst.eecs.berkeley.edu/~cs61c/sp21/resources-pdfs/garcia_c_reference_slides.pdf)
* [Brian Harvey's Intro to C](https://inst.eecs.berkeley.edu/~cs61c/sp21/resources-pdfs/HarveyNotesC1-3.pdf)

## Bài tập
Kiểm tra kiến thức!

### Ôn tập Khái niệm

:::{exercise}
:label: c-basics-01
1. **Đúng/Sai**: Trong các ngôn ngữ biên dịch, thời gian biên dịch thường khá nhanh, tuy nhiên thời gian chạy chậm hơn đáng kể so với các ngôn ngữ thông dịch.
:::

:::{solution} c-basics-01
:label: c-basics-01-sol
:class: dropdown

**Sai.** Thời gian biên dịch hợp lý, hiệu năng runtime xuất sắc. Nó tối ưu hóa cho một loại bộ xử lý và hệ điều hành cụ thể.

<!--Xem: [Lecture 3 Slide 18](https://docs.google.com/presentation/d/1Wx65MzIzJa-dJvrE2IwTm7EGNP0DmiXi1haFtBj69No/edit?slide=id.g32e1ad37bb7_0_314#slide=id.g32e1ad37bb7_0_314)-->
:::
