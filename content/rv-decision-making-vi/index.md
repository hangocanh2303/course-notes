---
title: "Bộ đếm chương trình"
---

(sec-rv-pc)=
## Mục tiêu học tập

* Hiểu cách thanh ghi bộ đếm chương trình (PC) cập nhật giữa các lệnh.
* Hiểu cách bộ xử lý sử dụng PC để xác định lệnh nào cần load và thực thi.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/X6SbnHmeN6w
:width: 100%
:title: "[CS61C FA20] Lecture 09.2 - RISC-V Decisions II: A Bit About Machine Program"
:::

::::

Cho đến nay, chúng ta đã thấy rằng trong quá trình thực thi, các giá trị được lưu trong **thanh ghi**. **Các lệnh hợp ngữ** hoạt động trên thanh ghi và load/store giá trị giữa thanh ghi và bộ nhớ.

Chúng ta thảo luận về những điều cơ bản của mô hình bộ nhớ RISC-V trong hai chương tiếp theo.

## Chương trình được lưu trữ, xem lại

Trong [phần trước](#sec-stored-program) chúng ta đã thảo luận về khái niệm máy tính **chương trình được lưu trữ**, được sử dụng hiệu quả cho tất cả các máy tính đa năng ngày nay:

> Dữ liệu không chỉ phải đại diện cho số; nó có thể đại diện cho chính chương trình.

Nhớ lại rằng ngôn ngữ hợp ngữ thường được tạo ra bởi compiler.[^call] Một assembler sau đó tạo ra mã máy có thể đọc được. Thông thường, điều này được lưu như một file thực thi, sau đó được load vào phân đoạn **text** của bộ nhớ:

:::{figure} #fig-c-mem-layout
:width: 50%
:alt: "Sơ đồ không gian địa chỉ C với text tại các địa chỉ thấp nhất, data phía trên text, heap phía trên data phát triển lên trên, stack phát triển xuống dưới từ các địa chỉ cao nhất, và không gian không sử dụng giữa heap và stack."

Bố cục bộ nhớ C (in lại từ @fig-c-mem-layout từ [phần này](#sec-mem-layout)).
:::

[^call]: Chúng ta mở rộng về quy trình compiler-assembler-linker-loader trong [phần sau](#sec-call).

RISC-V có mô hình bộ nhớ tương tự. Mỗi lệnh hợp ngữ RISC-V được lưu như **mã máy**, tức là bit. Mỗi lệnh hợp ngữ dịch sang một lệnh máy 32-bit (trong RV32I, kích thước từ của chúng ta là 32 bit). Ví dụ, lệnh `slli x12 x10 0x10` dịch sang các bit `0x01051613`.[^instruction-formats]. Các lệnh máy 32-bit này sau đó tạo thành file thực thi mã máy.

File thực thi mã máy quá lớn để vừa vào thanh ghi, nên nó nằm trong **bộ nhớ** (trong C, đây sẽ là phân đoạn text). Bản thân chương trình về cơ bản là một chuỗi các lệnh RISC-V, mỗi lệnh rộng 32 bit, thường được thực thi theo thứ tự cho đến khi bộ xử lý gặp một [branch hoặc jump](#sec-branches).

[^instruction-formats]: Chúng ta sẽ xem cách thực hiện bản dịch này sau.

(sec-program-counter)=
## Bộ đếm chương trình (PC)

Làm thế nào máy tính biết lệnh nào cần thực thi? Bộ xử lý cũng theo dõi giá trị này trong một **thanh ghi**! Từ [Đặc tả RV32I](https://docs.riscv.org/reference/isa/unpriv/rv32.html#2-2-programmers-model-for-base-integer-isa):

> Có một thanh ghi unprivileged bổ sung: bộ đếm chương trình `PC` giữ địa chỉ của lệnh hiện tại.

**Bộ đếm chương trình** (PC[^pc]) hiệu quả là một con trỏ đến bộ nhớ và là một thanh ghi có tên `pc`.[^pc-name] Thanh ghi `pc` **không** phải là một trong 32 thanh ghi được đánh số `x0` đến `x31`. Nó là một thanh ghi _riêng biệt_ thường không được chỉ định rõ ràng như đích đọc/ghi cho các lệnh.

[^pc]: Program Counter, không phải Personal Computer.

[^pc-name]: Intel gọi bộ đếm chương trình là Instruction Pointer (IP). Cú pháp Verilog là PC, mặc dù [RISC-V Unprivileged Manual](https://docs.riscv.org/reference/isa/unpriv/rv32.html) gọi nó là `pc`.


Chúng ta xem lại [bố cục máy tính khái niệm](#fig-von-neumann) từ [trước đó](#sec-architecture-elements) và tập trung vào bộ đếm chương trình trong @fig-program-counter.

:::{figure} images/program-counter.png
:label: fig-program-counter
:width: 80%
:alt: "Sơ đồ khối bộ xử lý và bộ nhớ: đường dữ liệu chứa bộ đếm chương trình, thanh ghi, và ALU; một mũi tên cam có nhãn địa chỉ lệnh chạy từ PC đến vùng chương trình trong bộ nhớ, và đọc lệnh trả về đơn vị điều khiển."

Bộ đếm chương trình giữ địa chỉ của lệnh hiện tại.
:::

Bộ xử lý bên trái bao gồm đơn vị điều khiển và đường dữ liệu. Bộ nhớ nằm bên phải. Bên trong đường dữ liệu, chúng ta có 32 thanh ghi và PC, là một thanh ghi bên trong bộ xử lý giữ **địa chỉ byte** của lệnh tiếp theo sẽ được thực thi.

Đơn vị điều khiển[^control] sử dụng PC như sau:

1. Đọc PC,
2. Lấy một lệnh từ bộ nhớ,
3. Thực thi lệnh sử dụng đường dữ liệu, và
4. Cập nhật PC để trỏ đến lệnh tiếp theo.

(sec-rv32i-pc-4)=
:::{warning} Các lệnh có kích thước từ
Mỗi lệnh RV32I rộng một từ, nên các lệnh liên tiếp nằm cách nhau 4 byte. Lựa chọn thiết kế này tuân theo sự đơn giản của RISC-V bằng cách giữ hầu hết mọi thứ ở độ rộng từ (ví dụ: độ rộng thanh ghi và truy cập bộ nhớ với `lw` và `sw`).

Theo mặc định, PC được tăng thêm 4 byte, tương ứng với lệnh tuần tự tiếp theo.
:::

[^control]: Chúng ta thảo luận về đơn vị điều khiển sau khi chúng ta thiết kế bộ xử lý. Bây giờ, chúng ta giới thiệu bộ đếm chương trình để giải thích tập hợp đầy đủ các lệnh hợp ngữ.

### Ví dụ: Lệnh số học

Animation bên dưới theo dõi qua một ví dụ đơn giản về cách thực thi lệnh số học cập nhật **cả** thanh ghi đích **và** thanh ghi bộ đếm chương trình.

:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vRzEG3hI-o7XL7oL1njxPvQq0jr7uR3pVlBTtBX6KM82YUC1wROduPqaLwCiS7iU_y9p0hbTTiooPYn/pubembed?start=false&loop=false
:width: 100%
:title: "Animation theo dõi qua một ví dụ về cách thực thi các lệnh số học sẽ cập nhật cả thanh ghi đích và thanh ghi bộ đếm chương trình, như được chi tiết trong phần này. Truy cập [Google Slides gốc](https://docs.google.com/presentation/d/1nt1Qum-w_TcAtcdsT9iVIBhmDbTvjEuiFnK4bv7NeRE/edit?usp=sharing)"
:::

Ở trên, bộ xử lý thực thi một lệnh như sau:

1. Bộ xử lý đọc `pc`, hiện đang giữ `0x00000008`.
2. Bộ xử lý đọc lệnh trong bộ nhớ tại địa chỉ `0x00000008`, đó là `slli x12 x10 0x10`.
3. Bộ xử lý thực thi lệnh này bằng cách đọc và ghi thanh ghi. Nếu `x10` giữ `0x0000 34FF`, thì `x12` được cập nhật thành `0x34FF0000`.
4. Bộ xử lý cập nhật `pc` để giữ `0x00000008 + 4`, hay `0x0000000c`.

Một số lệnh sẽ yêu cầu PC được cập nhật khác đi. Các lệnh này được gọi là **branch** và khiến một địa chỉ hoàn toàn mới được load vào PC. Đây là chủ đề của hai chương tiếp theo.
