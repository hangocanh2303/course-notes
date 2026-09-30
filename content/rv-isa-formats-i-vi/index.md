---
title: "Ngôn ngữ máy"
---

(sec-machine-instructions)=
## Mục tiêu học tập

* Biết rằng các lệnh máy tuân theo các định dạng lệnh.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/DsDnNqL4gCo
:width: 100%
:title: "[CS61C FA20] Lecture 11.1 - RISC-V Instruction Formats I: Intro"
:::

::::

Nhớ lại từ [trước đó](#sec-isa-note) rằng một ISA chỉ định [các lệnh hợp ngữ](#sec-assembly-language), các lệnh máy, và các tính năng kiến trúc cơ bản như [thanh ghi](#sec-rv32i-registers), [truy cập bộ nhớ](#sec-load-store), và [thực thi lệnh](#sec-rv-pc).

Để tận dụng đầy đủ ISA, chúng ta phải viết [các lệnh máy](#sec-machine-language), là các biểu diễn bit của các lệnh hợp ngữ.

:::{warning} Các chương trình nhị phân phụ thuộc vào ISA

Các chương trình được phân phối dưới dạng nhị phân, tức là mã máy đã được assemble. Các chương trình đều gắn liền với một tập lệnh cụ thể, và có các tập lệnh khác nhau cho các kiến trúc khác nhau (ví dụ: điện thoại vs. PC). Một chương trình thực thi RISC-V sẽ không chạy trên máy Intel![^backwards-compatible]

[^backwards-compatible]: Tuy nhiên, nhiều tập lệnh tương thích ngược và phát triển theo thời gian. Ví dụ: PC mới nhất với ISA x86 ngày nay vẫn có thể chạy các chương trình từ Intel 8088 (1981)!

:::

## Các định dạng lệnh

Trong thảo luận về [thực thi lệnh](#sec-rv-pc) và bộ đếm chương trình, chúng ta đã học rằng tất cả các lệnh RV32I có [kích thước từ](#sec-rv32i-pc-4).[^instruction-word] Đơn giản hóa hoạt động cho RISC-V: Các lệnh có cùng kích thước với một từ dữ liệu (32 bit) để chúng có thể tận dụng cùng phần cứng cho truy cập bộ nhớ.

Làm thế nào để chúng ta dịch `add x1 x2 x3` thành từ 32-bit `00000000001100010000000010110011`?

[^instruction-word]: Chúng ta bao gồm RV32I (các lệnh số nguyên 32-bit). Các lệnh 32-bit tương tự được sử dụng cho RV32, RV64, RV128.

RISC-V định nghĩa sáu **định dạng lệnh** cơ bản, trong đó các lệnh tương tự sử dụng cùng định dạng. Mỗi định dạng lệnh chia một từ lệnh thành các **trường**; mỗi trường cho bộ xử lý biết điều gì đó về lệnh.

Chúng tôi hy vọng bạn coi hai chương này như một cuộc săn đố, nơi bạn học cách giải mã các cột ngoài cùng bên phải của [green card RISC-V](#sec-green-card) và bảng [Các loại lệnh](#tab-rv32i-types).

* Chương này: R-Type, I-Type, S-Type
* Chương tiếp theo: B-Type, U-Type, J-Type
