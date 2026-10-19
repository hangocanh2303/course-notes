---
title: "Tổng Kết"
---

## Và Kết Luận$\dots$

Ta đã dành một chương để đề cập [một hình](#fig-call-flow-vi):

:::{figure} images/call-flow.png
:width: 60%
:alt: "Sơ đồ luồng theo chiều dọc của quá trình dịch và nạp: mã nguồn C foo.c được biên dịch thành assembly foo.s, hợp dịch thành object file foo.o, liên kết với lib.o để tạo ra file thực thi a.out, rồi được nạp vào bộ nhớ."

Sơ đồ các bước biên dịch và chạy một chương trình C.
:::

* [Compiler](#sec-compiler-vi) chuyển đổi một file ngôn ngữ bậc cao thành một file ngôn ngữ assembly.
* [Assembler](#sec-assembler-vi) loại bỏ giả lệnh, chuyển đổi những gì có thể thành ngôn ngữ máy, tạo danh sách kiểm tra cho linker (bảng tái định vị).
  * Các chi tiết này được lưu trong [object file](#sec-assembler-details-vi).
  * Assembler thực hiện 2 lượt để phân giải địa chỉ trong text segment, xử lý các tham chiếu nội bộ đến mã độc lập vị trí.
* [Linker](#sec-linker-vi) kết hợp nhiều file `.o` và phân giải địa chỉ tuyệt đối.
  * Linker cho phép biên dịch riêng biệt, các thư viện không cần biên dịch, và phân giải các địa chỉ còn lại ([thêm chi tiết](#sec-linker-details-vi))
* [Loader](#sec-loader-vi) nạp file thực thi vào bộ nhớ và bắt đầu thực thi.

## Đọc Thêm Từ Sách Giáo Khoa

P&H 2.12

<!-- ## Tài Liệu Tham Khảo Bổ Sung -->

## Bài Tập
Kiểm tra kiến thức của bạn!

### Ôn Tập Khái Niệm

:::{exercise}
:label: call-01-vi
1. Assembler phải thực hiện bao nhiêu lượt qua mã? Tại sao?
:::

:::{solution} call-01-vi
:label: call-01-sol-vi
:class: dropdown
**Hai lượt**: Lượt đầu tìm tất cả địa chỉ nhãn, và lượt hai phân giải các tham chiếu tiến trong khi sử dụng các địa chỉ nhãn này.
:::

:::{exercise}
:label: call-02-vi
2. Bước nào trong CALL phân giải định địa chỉ tương đối? Định địa chỉ tuyệt đối?
:::

:::{solution} call-02-vi
:label: call-02-sol-vi
:class: dropdown

**Assembler** thường xử lý định địa chỉ tương đối. **Linker** xử lý định địa chỉ tuyệt đối, phân giải các tham chiếu đến vị trí bộ nhớ bên ngoài.
:::

:::{exercise}
:label: call-03-vi
3. Mô tả sáu phần chính của object file được xuất ra bởi Assembler (Header, Text, Data, Relocation Table, Symbol Table, Debugging Information).
:::

:::{solution} call-03-vi
:label: call-03-sol-vi
:class: dropdown
* **Header**: Kích thước và vị trí của các phần khác
* **Text**: Mã máy
* **Data**: Biểu diễn nhị phân của bất kỳ dữ liệu nào trong file nguồn
* **Relocation Table**: Xác định các dòng mã cần được "xử lý" bởi Linker (nhảy đến nhãn external (vd: file lib), tham chiếu đến dữ liệu tĩnh)
* **Symbol Table**: Danh sách các nhãn file và dữ liệu có thể được tham chiếu qua các file
* **Debugging Information**: Thông tin bổ sung cho debugger
:::
