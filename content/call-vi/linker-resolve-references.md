---
title: "Linker: Phân Giải Tham Chiếu"
---

(sec-linker-details-vi)=
## Mục Tiêu Học Tập

* Giải thích tại sao linker có thể phân giải địa chỉ tuyệt đối.
* Xác định khi nào địa chỉ lệnh nhánh/nhảy được phân giải bởi assembler hoặc linker.
* So sánh thư viện liên kết tĩnh và liên kết động.

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/2lVYrK7spaw
:width: 100%
:title: "[CS61C FA20] Lecture 13.4 - Compilation, Assembly, Linking, Loading: Linker"
:::

từ 1:38 trở đi
::::

Đến lúc này, compiler đã hoàn thành và cho bạn file `.s`, và assembler đã hoàn thành để tạo ra file `.o` (object file). Bạn có thể có nhiều file `.o`, cộng thêm các file thư viện như file `.a` — đó chỉ là các gói lớn chứa nhiều file `.o` gói lại với nhau (xem [hình trước](#fig-call-flow-vi)). Công việc của linker[^link-editor] là tập hợp tất cả những thứ này lại và tạo ra file thực thi `a.out`.

[^link-editor]: Linker còn được gọi là "link editor" vì nó "chỉnh sửa" hoặc sửa tất cả các liên kết trong bảng tái định vị.

Một lần nữa, ta thích việc linker cho phép biên dịch riêng biệt các phần khác nhau của chương trình. Ta sẽ thảo luận một số đánh đổi ở phần sau.

Từ [phần tổng quan](#sec-linker-vi):

> Linker ghép nối nhiều object module lại để tạo ra file thực thi. Nó giải quyết tất cả các "việc cần làm" mà assembler để lại, bao gồm **tái định vị** mọi thứ cho file thực thi cuối cùng:
>
> 1. Ghép các **text segment** từ mỗi file `.o` lại với nhau.
> 2. Ghép các data segment từ mỗi file `.o`, rồi nối tiếp vào cuối segment của Bước 1.
> 3. Phân giải các tham chiếu — tức là các địa chỉ mà assembler chưa thể giải quyết.

:::{figure} images/linker-flow.png
:label: fig-linker-flow-vi
:width: 40%
:alt: "Hai object module, file1.o và file2.o, mỗi cái được vẽ như hình chữ nhật với các segment text, data, và info; các mũi tên đưa cả hai vào khối Linker trung tâm. Sơ đồ cho thấy file thực thi đầu ra duy nhất a.out gồm các segment text và data đã tái định vị từ cả hai module cùng thông tin tái định vị tổng hợp."

Sơ đồ luồng cho các thành phần của linker.
:::

## Phân Giải Tham Chiếu

:::{hint} Những Địa Chỉ Nào Cần Tái Định Vị?

Ta có ba loại địa chỉ:

1. Định địa chỉ Tương đối PC: `beq`, `bne`, `jal`, v.v.

2. Tham chiếu Hàm External, vd: `jal` hoặc `auipc`/`jalr`

3. Tham chiếu Dữ liệu Tĩnh

Loại đầu tiên, định địa chỉ tương đối PC trong object module, đã được assembler hoàn thành. Linker phân giải hai loại sau, vì:

* Assembler không biết các hàm external ở đâu vì tại thời điểm hợp dịch, địa chỉ đến các object module khác là chưa biết.
* Tại thời điểm hợp dịch, vị trí cuối cùng của dữ liệu tĩnh là chưa biết vì data segment có thể được **tái định vị** do có nhiều object module.
:::

::::{hint} Những lệnh nào cần được **chỉnh sửa để tái định vị**?

Xem @fig-linker-instructions:

* Nhảy External dạng J-Format (vd: đến thư viện C)
* Các lệnh với địa chỉ tuyệt đối, vd: `auipc`/`jalr` hoặc `auipc`/`addi`
* Các lệnh chỉ định tham chiếu dữ liệu tĩnh, vd: `lw`/`sw` (của dữ liệu tĩnh), `auipc`/`lui`/`addi` (vd: khi là một phần của giả lệnh `la`[^la] hoặc `li`).
* Các lệnh load/store truy cập biến `.data`[^gp]

[^gp]: Global pointer (`gp`) là con trỏ đến data (static) segment. Nằm ngoài phạm vi khóa học này.
[^la]: Xem [ví dụ](#sec-call-example-vi) để biết thêm chi tiết về giả lệnh load address `la`.

(một lần nữa, B-Type không cần chỉnh sửa, vì các nhảy có điều kiện như vậy nằm trong mỗi module)

:::{figure} images/linker-instructions.png
:label: fig-linker-instructions-vi
:width: 100%
:alt: "Sơ đồ định dạng lệnh so sánh các trường bit J/U-type, I-type, và S-type, với các vùng placeholder được tô sáng trong các định dạng J/U và S nơi các cập nhật tái định vị được ghi."

Linker "phân giải tham chiếu" bằng cách thay thế các placeholder thích hợp trong các lệnh mã máy.
:::

::::

Chúng tôi có thể sẽ không kiểm tra chi tiết về phân giải tham chiếu, nhưng với những ai tò mò:

1. Với RV32, linker giả định text segment đầu tiên bắt đầu tại địa chỉ `0x10000`.

2. Linker biết **độ dài** của mỗi text và data segment và **thứ tự** của các text và data segment. Linker sau đó **tính địa chỉ tuyệt đối** của (1) mỗi nhãn cần nhảy đến, và (2) mỗi mảnh dữ liệu tĩnh được tham chiếu.

3. Linker sau đó **phân giải các tham chiếu**. Nó tìm kiếm mỗi tham chiếu (dữ liệu hoặc nhãn) trong tất cả các symbol table "người dùng". Nếu không tìm thấy, nó sau đó tìm trong các file thư viện (vd: cho `printf`). Một khi địa chỉ tuyệt đối được xác định, nó điền vào mã máy một cách phù hợp bằng cách chỉnh sửa các lệnh cụ thể để tái định vị.

Cuối cùng, linker xuất ra một file thực thi chứa text và data cùng với header/thông tin debug.

## [Ngoài Lề] Liên Kết Tĩnh vs. Động

Cho đến giờ, ta đã mô tả cách truyền thống: **thư viện liên kết tĩnh** (statically-linked libraries). File thực thi bao gồm toàn bộ thư viện, ngay cả khi không phải tất cả đều được chương trình người dùng sử dụng. Thư viện giờ là một phần của file thực thi — nếu thư viện cập nhật, ta sẽ không nhận được bản sửa trừ khi biên dịch lại chương trình người dùng.

Một lựa chọn khác là **thư viện liên kết động** (dynamically-linked libraries, DLL), phổ biến trên các nền tảng Windows và UNIX. Cách tiếp cận chủ đạo trong mô hình này là liên kết ở mức mã máy _tại thời điểm nạp_: loader dành chỗ cho thư viện, nhét nó vào, và chạy nó.

Đánh đổi:

* **Không gian vs. Thời gian**:
  * Với thư viện liên kết tĩnh, ta có file thực thi độc lập nhưng cuối cùng rất lớn.
  * Với DLL, lưu trữ chương trình cần ít không gian đĩa hơn; ít thời gian để gửi chương trình hơn. Hơn nữa, hai chương trình cần ít bộ nhớ hơn (nếu chúng chia sẻ thư viện). Tuy nhiên, giờ có **chi phí thời gian chạy**; cần thêm thời gian để liên kết động các thư viện.
* **Xử lý nâng cấp hợp lý**:
  * Với thư viện liên kết tĩnh, nếu thư viện có lỗ hổng bảo mật hoặc bản sửa lỗi, mọi nhà phát triển phải biên dịch lại và gửi lại bản cập nhật cho người dùng vì thư viện cũ, bị lỗi đã được "nướng" vào file `a.out` của họ.
  * Với DLL, thay thế libXYZ.so nâng cấp mọi chương trình sử dụng thư viện XYZ. Tuy nhiên, DLL giới thiệu sự phức tạp: giờ ta cần **nhiều file** để chạy một chương trình. Chỉ có file thực thi thôi là không đủ nữa!

Nhìn chung, liên kết động thêm sự phức tạp cho compiler, linker và OS. Tuy nhiên, lợi ích của nó vượt trội so với sự phức tạp.

:::{figure} images/compile-xkcd.jpg
:label: fig-compile-xkcd-vi
:width: 50%
:alt: "Truyện tranh xkcd cho thấy hai nhân viên văn phòng gần cửa: một người kêu lên họ đang biên dịch như một cái cớ để tránh làm việc, khiến một giọng khác đáp lại hãy tiếp tục đi. Trò đùa nhấn mạnh thời gian biên dịch dài như một sự chậm trễ quen thuộc trong lập trình."

Biên dịch là một quá trình chậm. [xkcd](https://xkcd.com/303/), [explainxkcd](https://www.explainxkcd.com/wiki/index.php/303:_Compiling)
:::
