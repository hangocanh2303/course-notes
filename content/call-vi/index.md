---
title: "Tổng Quan về CALL"
---

(sec-call-vi)=
## Mục Tiêu Học Tập

* Nắm được bốn bước để dịch và chạy một chương trình: Compile (Biên dịch), Assemble (Hợp dịch), Link (Liên kết), và Load (Nạp) — viết tắt là CALL.

Trong bài này, chúng ta sẽ tìm hiểu cách chạy một chương trình. Ta sẽ đi sâu hơn vào định nghĩa **thông thường** về việc biên dịch mã C thành file thực thi nhị phân:

:::{figure} images/compile-high-level.png
:label: fig-compile-high-level-vi
:width: 100%
:alt: "Hình hai phần so sánh cách nhìn thông thường và cách nhìn cụ thể về quá trình dịch. Bên trái: sơ đồ từ file nguồn foo.c qua bước biên dịch đến file thực thi a.out, rồi nạp vào bộ nhớ. Bên phải: danh sách theo chiều dọc gồm mã nguồn C, assembly, và mã máy — các dạng biểu diễn văn bản liên tiếp của cùng một chương trình."

Nói chung, "biên dịch mã C" nghĩa là dịch chương trình `foo.c` thành file thực thi `a.out`. Nhưng assembly liên quan như thế nào và ở đâu?
:::

Hãy quay lại với [Ý Tưởng Lớn về Trừu Tượng Hóa](#fig-great-idea-1). Hôm nay, chúng ta sẽ thảo luận mọi thứ **phía trên** đường ISA màu cam — cách đi từ ngôn ngữ bậc cao như C xuống ngôn ngữ assembly bậc thấp hơn, và cuối cùng đến mã máy ở mức thấp nhất.

## Dịch và Chạy Chương Trình

Chúng ta sẽ dành bài học hôm nay để thảo luận về @fig-call-flow.

Quá trình **dịch** một chương trình C `foo.c` từ mã C bậc cao thành file thực thi `a.out` thực ra gồm ba bước (Compile, Assemble, Link), cộng thêm một bước để chạy chương trình (Load). Đáng chú ý, ba bước đầu viết tắt là CAL — Go Bears!!!

:::{figure} images/call-flow.png
:label: fig-call-flow-vi
:width: 60%
:alt: "Sơ đồ luồng theo chiều dọc của quá trình dịch và nạp: mã nguồn C foo.c được biên dịch thành assembly foo.s, hợp dịch thành object file foo.o, liên kết với lib.o để tạo ra file thực thi a.out, rồi được nạp vào bộ nhớ."

Sơ đồ các bước biên dịch và chạy một chương trình C.
:::

Chi tiết tổng quan được thể hiện trong @tab-call-flow:

:::{table} Đầu vào/đầu ra của mỗi bước trong CALL.
:label: tab-call-flow-vi

| Bước | Đầu vào | Đầu ra | Ghi chú |
| :-- | :-- | :-- | :-- |
| [**Compiler**](#sec-compiler-vi) | Mã ngôn ngữ bậc cao (vd: `foo.c`) | Mã Assembly (vd: `foo.s`) | Đầu ra có thể chứa giả lệnh (`mv`, `li`, `j`, v.v.) |
| [**Assembler**](#sec-assembler-vi) | Mã Assembly (vd: `foo.s`) | Object file — module ngôn ngữ máy (vd: `foo.o`) | Thay giả lệnh bằng lệnh thật; tạo ra object file (module). |
| [**Linker**](#sec-linker-vi) | Các object file (vd: `foo.o`, `lib.o`) | Mã máy thực thi (vd: `a.out`) | Linker cho phép biên dịch riêng từng file. Thay đổi một file không cần biên dịch lại toàn bộ chương trình. |
| [**Loader**](#sec-loader-vi) | Mã thực thi (vd: `a.out`) | (chương trình được chạy) | Khi chạy file thực thi, loader đầu tiên nạp file từ đĩa vào bộ nhớ rồi thực thi nó. |

:::

(sec-compiler-vi)=
### Compiler (Trình Biên Dịch)

::::{note} 🎥 Video Bài Giảng: Compiler
:class: dropdown

:::{iframe} https://www.youtube.com/embed/QrRbiNPz-Kw
:width: 100%
:title: "[CS61C FA20] Lecture 13.2 - Compilation, Assembly, Linking, Loading: Compiler"

Phần lớn video này thảo luận về tổng quan CALL ở trên.
:::

::::

Compiler dịch mã C thành mã assembly. Thông thường, ta dùng chương trình như `gcc` để **biên dịch** chương trình. Trong lớp này, ta đã thực hành **viết tay** RISC-V với trình mô phỏng Venus.

Quan trọng là, đầu ra assembly của compiler có thể chứa **giả lệnh**. Như đã đề cập trong [phần trước](#sec-pseudoinstructions), giả lệnh (vd: `mv t1 t2`) giúp viết và suy nghĩ về chương trình ở mức assembly dễ dàng hơn nhiều. Ở bước tiếp theo, [assembler](#sec-assembler-vi) sẽ dịch các giả lệnh này thành lệnh thật (vd: `addi t1 t2 0`) trong ISA.

(sec-directives-vi)=
:::{hint} Chỉ Thị Assembly (Directives)

Chỉ thị (directives) đưa ra hướng dẫn cho assembler và được tạo bởi compiler. Chỉ thị **không** tạo ra **lệnh máy**! Thay vào đó, chúng cho các bước sau biết cách xây dựng các phần của object file đầu ra. Xem [ví dụ](#code-hello-s-vi) của chúng ta.

:::

(sec-assembler-vi)=
### Assembler (Trình Hợp Dịch)

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/87YbZ9x8QEY
:width: 100%
:title: "[CS61C FA20] Lecture 13.3 - Compilation, Assembly, Linking, Loading: Assembler"
:::

đến 1:26

::::

Assembler dịch mã assembly thành các module máy. Nó dịch **giả lệnh** thành lệnh thật và tạo ra **object file**. Assembler sử dụng các **chỉ thị** assembly để tạo object file, chứa các phần của **text segment** (phân đoạn mã), **data segment** (phân đoạn dữ liệu) của file thực thi, và nhiều thứ khác.

:::{hint} Đọc thêm về cách assembler tạo object file

Còn nhớ [nhãn](#sec-labels) trong lệnh assembly không? Assembler có thể phân giải một số nhãn (tức là dịch thành mã máy) nếu chúng liên quan đến **định địa chỉ tương đối PC** trong cùng module.

Tuy nhiên, assembler không biết các header nằm ở đâu (vd: thư viện chuẩn C); nó giao việc phân giải này cho Linker ở bước sau. Do đó, đầu ra của assembler thường không phải là file thực thi hoàn chỉnh; thay vào đó, **object file** là một **module ngôn ngữ máy** chứa cả các lệnh của module này và thông tin cho các bước sau (vd: liên kết) và debug.

Chúng ta cung cấp thêm chi tiết về việc tạo object file trong [phần sau](#sec-assembler-details-vi).
:::

(sec-linker-vi)=
### Linker (Trình Liên Kết)

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/2lVYrK7spaw
:width: 100%
:title: "[CS61C FA20] Lecture 13.4 - Compilation, Assembly, Linking, Loading: Linker"
:::

đến 1:38
::::

Linker ghép nối nhiều object module lại để tạo ra file thực thi. Nó giải quyết tất cả các "việc cần làm" mà assembler để lại, bao gồm **tái định vị** mọi thứ cho file thực thi cuối cùng:

1. Ghép các **text segment** từ mỗi file `.o` lại với nhau.
2. Ghép các data segment từ mỗi file `.o`, rồi nối tiếp vào cuối segment của Bước 1.
3. Phân giải các tham chiếu — tức là các địa chỉ mà assembler chưa thể giải quyết.

:::{figure} images/linker-flow.png
:width: 40%
:alt: "Sơ đồ kết hợp của Linker: các object module file1.o và file2.o được vẽ như các hình chữ nhật, mỗi cái chia thành các segment text, data, và thông tin phụ trợ; các mũi tên từ cả hai module đi vào khối Linker trung tâm."
:::

Linker cho phép **biên dịch riêng biệt** các phần khác nhau của chương trình. Quan trọng là, nó hỗ trợ việc _không phải biên dịch lại_ các thư viện lớn. Ví dụ, các thư viện chuẩn C (vd: `stdio`) là một phần của mã nguồn Linux, có hơn 20 triệu dòng code. Nhờ có linker, biên dịch lại một file `foo.c` đơn giản không cần biên dịch lại `stdio` :-)

:::{hint} Đọc thêm về cách linker phân giải tham chiếu

Linker cập nhật **địa chỉ tuyệt đối** vì nó biết kích thước đầy đủ của text và data segment của file thực thi. Đọc thêm trong [phần sau](#sec-linker-details-vi). Phần này cũng nói về thư viện liên kết tĩnh và liên kết động.

:::

:::{hint} File thực thi chứa gì?

File thực thi `a.out` chứa cả mã máy (segment `text`) và dữ liệu tĩnh (segment `data`). File thực thi cũng bao gồm header (để chỉ định vị trí các segment trong file) và thông tin debug.

:::

(sec-loader-vi)=
### Loader (Trình Nạp)

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/E9zh8pobPNw
:width: 100%
:title: "[CS61C FA20] Lecture 13.5 - Compilation, Assembly, Linking, Loading: Loader"
:::

::::

Loader thực hiện nhiều việc để chạy chương trình. Để tìm hiểu thêm, chúng tôi rất khuyến khích học lớp cao cấp như CS 162: Hệ Điều Hành!

**1. Nạp chương trình vào không gian địa chỉ mới** trong bộ nhớ.[^vm]

[^vm]: Chúng ta thảo luận về bộ nhớ ảo trong [chương sau](#sec-virtual-memory).

* Đọc header của file `a.out` để biết kích thước các segment text, data.
* Tạo không gian địa chỉ mới cho chương trình, đủ lớn để chứa **text và data segment**, cùng với stack segment.
  * Sao chép các lệnh và dữ liệu từ file thực thi vào không gian địa chỉ mới.
  * Sao chép các tham số truyền cho chương trình lên stack.

**2. Khởi tạo các thanh ghi máy**. Xóa các thanh ghi, và gán con trỏ stack `sp` vào địa chỉ được chỉ định của vị trí stack trống đầu tiên.

**3. Nhảy đến routine khởi động**:
Sao chép các tham số chương trình từ stack vào các thanh ghi, đặt program counter `pc`.

**4. Chạy chương trình**. Nếu hàm main trả về, kết thúc chương trình bằng system call exit.
