---
title: "Thông Dịch vs. Biên Dịch"
subtitle: "Nội dung này không kiểm tra"
---

## Mục Tiêu Học Tập

Phần này được bao gồm như nội dung bổ sung và không được kiểm tra. Nếu bạn tò mò về các khái niệm này, hãy học CS 164: Compilers and Programming Languages!

::::{note} 🎥 Video Bài Giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/DFEe0uRAm7o
:width: 100%
:title: "[CS61C FA20] Lecture 13.1 - Compilation, Assembly, Linking, Loading: Interpretation vs Translation"
:::

::::

## Phổ Thực Thi Ngôn Ngữ

Thông dịch và biên dịch là hai cách chạy chương trình được viết bằng một ngôn ngữ nguồn cụ thể.

* **Thông dịch (Interpretation)**: Thực thi trực tiếp chương trình bằng ngôn ngữ nguồn. Trình thông dịch là một chương trình thực thi các chương trình khác.
* **Biên dịch (Translation)**: Chuyển đổi chương trình từ ngôn ngữ nguồn sang chương trình tương đương bằng ngôn ngữ khác. Biên dịch sang ngôn ngữ bậc thấp hơn hầu như luôn có nghĩa là hiệu quả cao hơn, hiệu năng tốt hơn.

Nói chung, ta **thông dịch** ngôn ngữ bậc cao khi hiệu quả không quan trọng. Ta **biên dịch** sang ngôn ngữ bậc thấp hơn để tăng hiệu năng. Cách sau là những gì ta thường làm với mã C; ta dịch nó thành mã máy và lưu dưới dạng file thực thi.

Xem xét @fig-language-continuum. Bên trái là các ngôn ngữ bậc cao hơn; bên phải là các ngôn ngữ bậc thấp hơn.

:::{figure} images/language-continuum.png
:label: fig-language-continuum-vi
:width: 100%
:alt: "Phổ ngang đặt các ngôn ngữ từ trái sang phải là Python, Java, C++, C, Java bytecode, Assembly, và Machine code. Bên trái của phổ được gắn nhãn dễ lập trình nhưng thông dịch không hiệu quả, và bên phải được gắn nhãn khó lập trình nhưng thông dịch hiệu quả."

Phổ thực thi ngôn ngữ.
:::

* **Python**: Ở phía xa bên trái, Python cực kỳ dễ viết nhưng được thông dịch hoàn toàn nên chậm nhất.
* **Java**: Hơi khó viết hơn Python. Java không phải ngôn ngữ thông dịch hoàn toàn; bạn biên dịch nó xuống bytecode rồi thông dịch bytecode đó, cho phép nó chạy trên nhiều máy.
* **C**: Ở đâu đó ở giữa. C được coi là ngôn ngữ bậc cao nhưng không dễ làm việc như các ngôn ngữ khác (như ta đã thấy); tuy nhiên, khi được biên dịch xuống nó nhanh hơn nhiều.
* **Assembly và Mã Máy**: Ở phía xa bên phải là mã máy — thứ mà người ta phải làm việc trong những ngày đầu của máy tính (xem [EDSAC](#sec-stored-program)). [Như ta đã thấy](#sec-machine-instructions), mã máy khó viết bằng tay nhưng rất dễ thông dịch — vì hầu như không có thông dịch nào! Mã máy có thể được thực thi trực tiếp trên phần cứng.

### Thông Dịch

Bạn có thể quen thuộc với Python, là ngôn ngữ thông dịch bậc cao. Trình thông dịch Python chỉ là một chương trình đọc chương trình Python `foo.py` và thực hiện các chức năng của chương trình Python đó, tạo ra đầu ra khi cần.

:::{figure} images/python-interpreter.png
:label: fig-python-interpreter-vi
:width: 50%
:alt: "Sơ đồ pipeline đơn giản nơi chương trình Python foo.py đi vào hộp có nhãn Python Interpreter, và mũi tên từ hộp đó chỉ đến Memory."

Trình thông dịch Python.
:::

Tại sao bạn lại thông dịch mã máy bằng phần mềm? Các chương trình C và thậm chí RISC-V cũng được thông dịch trong các trường hợp cụ thể.

Một lý do là **mô phỏng**; ví dụ, trong Venus, ta có trình mô phỏng RISC-V nơi bạn có thể lấy assembly và thông dịch nó để có nhiều thông tin debug hơn. Trình mô phỏng Venus giúp ta học RISC-V vì ta có thể bước qua mã máy.

Một lý do khác là **chuyển đổi** chương trình từ một ISA sang ISA khác. Xem xét việc chuyển đổi ISA của Apple Macintosh ngày xưa. Apple chuyển từ ISA 680x0 sang PowerPC, rồi sang x86 (và giờ sang ARM). Thay vì yêu cầu tất cả chương trình phải được dịch lại từ ngôn ngữ bậc cao, thay vào đó các nhà thiết kế cho phép file thực thi chứa mã máy cũ và/hoặc mới, rồi **giả lập** — bằng cách thông dịch mã cũ bằng phần mềm nếu cần. Quá trình chậm hơn này hỗ trợ tương thích ngược bằng cách sử dụng phần mềm để thông dịch ISA cũ trên phần cứng mới.

Năm 2020, Apple lại chuyển sang Apple silicon dựa trên ARM. Để hỗ trợ quá trình chuyển đổi này, Apple phát hành [Rosetta](https://support.apple.com/en-us/102527), một ứng dụng chạy các app dựa trên Intel trên chip Apple. Liên kết trong câu trước sử dụng từ "translation" khá tự do; giờ ta biết rằng quá trình này thực sự là thông dịch.

## Thông Dịch vs. Biên Dịch

Từ góc nhìn của chương trình, nó không biết — và không quan tâm — liệu nó đang chạy trên trình thông dịch (phần mềm) hay silicon thô (phần cứng). Khi bạn đưa cho nó lệnh "add", nó thực hiện phép cộng. Tuy nhiên, có những ưu và nhược điểm khi chọn thông dịch vs. biên dịch:

* **Triển khai**: Thường dễ viết trình thông dịch hơn; bạn có thể đã làm điều đó trong CS 61A (Structure and Interpretation of Computer Programs).
* **Debug**: Trình thông dịch gần với mức cao, nên bạn có thông báo lỗi và hỗ trợ debug tuyệt vời (xem Venus). Trình biên dịch cũng có thể hỗ trợ một số debug nhưng cần mã hóa số dòng và tên biến để sử dụng debug; ta sẽ thấy điều này sau.
* **Hiệu năng**: Trình thông dịch chậm hơn khoảng 10x hoặc hơn. Biên dịch hầu như luôn nhanh hơn vì nó nhắm vào phần cứng cụ thể.
* **Tính di động**: Mã có thể được thông dịch nhỏ hơn và cung cấp độc lập tập lệnh vì nó có thể được thông dịch (và do đó chạy) trên bất kỳ máy nào.
* **Sở hữu trí tuệ**: Biên dịch giúp "ẩn" "nguồn" chương trình khỏi người dùng.
