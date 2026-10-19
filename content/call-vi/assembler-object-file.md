---
title: "Assembler: Object File"
---

(sec-assembler-details-vi)=
## Mục Tiêu Học Tập

* Nhận biết các thành phần cơ bản của object file.
* Giải thích tại sao assembler có thể phân giải địa chỉ tương đối PC nhưng không thể phân giải địa chỉ tuyệt đối.
* Giải thích tại sao việc phân giải địa chỉ tương đối PC cần quá trình hai lượt (two-pass).

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/87YbZ9x8QEY
:width: 100%
:title: "[CS61C FA20] Lecture 13.3 - Compilation, Assembly, Linking, Loading: Assembler"
:::

từ 1:26 trở đi

::::

Từ [phần tổng quan](#sec-assembler-vi):

> Assembler dịch mã assembly thành các module máy. Nó dịch **giả lệnh** thành lệnh thật và tạo ra **object file**. Assembler sử dụng các **chỉ thị** assembly để tạo object file, chứa các phần của **text segment**, **data segment** của file thực thi, và nhiều thứ khác.


## Object File

File `.o` object cuối cùng là một module máy, ở dạng **nhị phân**:

1. **Object File Header**: kích thước và vị trí của các phần khác trong object file. Đây giống như "mục lục".
2. **Text Segment**: mã máy.
3. **Data Segment**: biểu diễn nhị phân của dữ liệu tĩnh trong file nguồn.
4. **Symbol Table** (Bảng ký hiệu): Danh sách các nhãn của file, dữ liệu tĩnh có thể được tham chiếu bởi các chương trình khác.
5. **Relocation Table** (Bảng tái định vị): Các dòng mã cần sửa sau (bởi Linker).
6. Thông tin Debug.

Text Segment và Data Segment (nhớ lại [bố cục bộ nhớ chương trình](#fig-c-mem-layout)) được dịch thành mã máy ở những chỗ có thể. Ba mục cuối (Symbol Table, Relocation Table, Thông tin Debug) được dùng cho Linker ở bước sau để phân giải mọi thứ và tạo ra một file thực thi duy nhất.

Chúng tôi rất khuyến khích đọc phần này rồi xem [ví dụ](#sec-call-example-vi) ở cuối chương.

## Text Segment

Một [ví dụ text segment](#code-hello-o-vi) được thảo luận trong [ví dụ](#sec-call-example-vi) ở cuối chương.

**Các lệnh Số học, Logic**: Với các trường hợp đơn giản như `add` hoặc `sub`, lệnh 32 bit chứa tất cả thông tin cần thiết để tạo mã máy. Điều này bao gồm các lệnh R-Format và một số lệnh I-Format.

**Các lệnh Nhánh và Nhảy Tương đối PC**, như `beq`/`bne`/v.v. và `jal`. Sau khi giả lệnh được thay bằng lệnh thật, tất cả **định địa chỉ tương đối PC** đã biết _trong object file_ có thể được tính toán. Xác định offset để mã hóa bằng cách đếm số lệnh half-word giữa lệnh hiện tại và lệnh đích.

Sau khi thay thế giả lệnh, assembler thực hiện **hai lượt** qua chương trình để tính tất cả các offset.

* **Ghi lại tất cả nhãn vào bảng ký hiệu**: Assembler ghi lại vị trí các nhãn bằng cách lưu chúng trong **symbol table** (bảng ký hiệu).
* **Sau đó, phân giải các tham chiếu**: Sử dụng vị trí nhãn để tạo mã máy và mã hóa cứng các offset nhánh/nhảy ở những chỗ có thể.

Nếu assembler chỉ thực hiện một lượt (từ các lệnh trước đến sau), các "tham chiếu tiến" (nhãn đến vị trí sau trong chương trình) sẽ có offset tương đối PC chưa biết.

:::{warning} Assembler không thể phân giải tất cả địa chỉ

Assembler **chỉ hợp dịch mã assembly cụ thể** và tạo ra object file, _không phải_ file thực thi cuối cùng. Do đó nó không biết cách phân giải:

* Tham chiếu đến **các file khác**, vd: gọi `strlen` từ thư viện chuỗi C bằng `jalr`
* Tham chiếu đến **dữ liệu tĩnh**, vd: lệnh load address `la` được tách thành `lui` và `addi`. Phân giải hai immediate này cần biết **địa chỉ tuyệt đối** 32 bit đầy đủ của dữ liệu _trong file thực thi_.

Vì **địa chỉ tuyệt đối** chưa thể xác định được, assembler giao lại cho bước sau (Linker) mọi tham chiếu phụ thuộc vào việc biết các object module khác. Assembler ghi lại các tham chiếu chưa phân giải trong Relocation Table và Symbol Table.
:::

## Symbol Table (Bảng Ký Hiệu)

**Symbol table** là danh sách các nhãn (thủ tục) và dữ liệu (như mảng global) trong file của bạn mà có thể được sử dụng bởi các file khác. Symbol Table cũng được sử dụng bởi debugger `gdb`.

* **Nhãn lệnh**: Dùng để tính mã máy cho định địa chỉ tương đối PC trong nhánh, gọi hàm, v.v.
  * Nếu bạn muốn gọi hàm như `printf` từ thư viện, linker cuối cùng sẽ cần ký hiệu này (thêm vào đó, xem bảng tái định vị bên dưới).
  * Có thể dùng [chỉ thị](#sec-directives-vi) `.globl` để cho phép nhãn được tham chiếu bởi các file khác.
* **Data segment**: bất cứ thứ gì trong phần được đánh dấu bởi chỉ thị `.data`. Nhớ rằng data segment có các biến global có thể được truy cập/sử dụng bởi các file khác.

Một [ví dụ symbol table](#tab-hello-symbol-table-vi) được thảo luận trong [ví dụ](#sec-call-example-vi) ở cuối chương.

## Relocation Table (Bảng Tái Định Vị)

**Relocation table** là "danh sách việc cần làm" để sửa sau (bởi **linker** ở bước sau). Nó chứa các placeholder cho:

* **Địa chỉ tuyệt đối cho bất kỳ nhãn external nào** được sử dụng bởi lệnh nhảy, vd: trong các file lib, `jal ext_label`
* **Địa chỉ tuyệt đối cho bất kỳ dữ liệu nào nằm trong data segment**, vd: biến tĩnh từ lệnh load address `la`. Assembler không biết phần tĩnh hay "vị trí cuối cùng" ở đâu; nó giao việc phân giải "vị trí cuối cùng" này cho **linker**.

Một [ví dụ relocation table](#tab-hello-relocation-table-vi) được thảo luận trong [ví dụ](#sec-call-example-vi) ở cuối chương.
