---
title: "Giới thiệu"
---

## Mục tiêu học tập

* Nắm được thuật ngữ: RISC, CISC, ISA.
* Hiểu rằng ở mức độ cao, kiến trúc RISC hỗ trợ tập lệnh đơn giản với phần cứng nhanh.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/w7efr8-MRPQ
:width: 100%
:title: "[CS61C FA20] Lecture 07.1 - RISC-V Intro: RISC-V Assembly Language"
:::

::::

Chúng ta sẽ đi sâu xuống mức trừu tượng tiếp theo và tìm hiểu về kiến trúc tập lệnh RISC-V và ngôn ngữ hợp ngữ RISC-V. Ở đầu khóa học, chúng ta đã thảo luận về Ý tưởng lớn về Trừu tượng hóa (@fig-great-idea-1): phân lớp các mức trừu tượng khác nhau để biểu diễn các hệ thống tính toán phức tạp.

:::{figure} ../great-ideas/images/1-abstraction.png
:label: fig-great-idea-1
:width: 100%
:alt: "Sơ đồ trừu tượng phân lớp: compiler, assembler, sau đó là mã máy phía trên đường ISA, với kiến trúc phần cứng và mạch logic phía dưới. Phía bên phải cho thấy các ví dụ tương ứng từ C và hợp ngữ RISC-V qua nhị phân, sơ đồ khối bộ xử lý, và logic cổng NAND."

Ý tưởng lớn #1: Trừu tượng hóa.
:::

Ở trên cùng, có ngôn ngữ bậc cao (như C). Bên dưới là ngôn ngữ hợp ngữ; bên dưới nữa là mã máy (phiên bản máy có thể đọc được); và bên dưới đó là các triển khai như sơ đồ khối. Thậm chí sâu hơn nữa, chúng ta tìm thấy các cổng logic, được xây dựng từ transistor.

### Kiến trúc tập lệnh

Luôn có các *giao diện được định nghĩa rõ ràng* giữa các lớp này. Trong phần này, chúng ta tập trung vào **kiến trúc tập lệnh (ISA)**, định nghĩa cách phần mềm giao tiếp với phần cứng.

:::{figure} ../great-ideas/images/old-school-machine-structures.png
:label: fig-old-school-machine-structures
:width: 100%
:alt: "Ngăn xếp phần mềm từ ứng dụng và hệ điều hành qua compiler và assembler xuống thanh Kiến trúc Tập lệnh, với các lớp phần cứng bên dưới bao gồm bộ xử lý, bộ nhớ, I/O, datapath, thiết kế số và mạch, transistor, và chế tạo. Vùng trung tâm, từ Thiết kế Số đến Hệ điều hành được đánh dấu là trọng tâm của khóa học này."

Ý tưởng lớn #1: Trừu tượng hóa.
:::

Từ [Wikipedia](https://en.wikipedia.org/wiki/Instruction_set_architecture):

> Nhìn chung, một ISA định nghĩa các lệnh, kiểu dữ liệu, thanh ghi, và giao diện lập trình để quản lý bộ nhớ chính như các chế độ địa chỉ hóa, bộ nhớ ảo, và cơ chế nhất quán bộ nhớ. ISA cũng bao gồm mô hình input/output của giao diện lập trình được.

(sec-isa-note)=
:::{note} Các ISA hiện đại

Các máy tính hiện đại tuân thủ các đặc tả ISA–nghĩa là, một ISA định nghĩa các hoạt động mà một máy tính cụ thể hỗ trợ, và cách nó triển khai chúng. Nó chỉ định một số thành phần:

* **Ngôn ngữ hợp ngữ**: các lệnh máy tính cấp thấp.
* **Ngôn ngữ máy**: cách các lệnh được biểu diễn dưới dạng bit.
* Các tính năng và thiết kế kiến trúc cơ bản.

:::

Ở một phía, ISA được sử dụng làm tiêu chuẩn để chúng ta lập kế hoạch và viết các lệnh máy tính. Ở phía còn lại, ISA được sử dụng làm tiêu chuẩn để chúng ta thiết kế *máy tính và phần cứng* (ví dụ: CPU). Các lệnh được viết theo ISA sau đó có thể được thực thi bởi các máy tính được thiết kế theo cùng ISA đó.

### Tại sao học hợp ngữ?

Trong thực tế, hiếm khi chúng ta tự viết mã ngôn ngữ hợp ngữ. Phổ biến nhất, ngôn ngữ hợp ngữ được tạo ra bởi compiler. Một assembler sau đó tạo ra mã máy có thể đọc được.

Vậy tại sao phải học hợp ngữ?

Xem xét đoạn trích này từ một bài đăng năm 2004 trên trang diễn đàn [slashdot.org](https://developers.slashdot.org/story/04/02/05/228200/learning-computer-science-via-assembly-language):

> Một [cuốn sách mới](https://archive.org/details/programming-from-the-ground-up) vừa được phát hành dựa trên một khái niệm mới - dạy khoa học máy tính thông qua ngôn ngữ hợp ngữ (ngôn ngữ hợp ngữ Linux x86, cụ thể là). Cuốn sách này dạy cách máy tính hoạt động, chứ không chỉ ngôn ngữ. Tôi nhận thấy rằng sự khác biệt chính giữa lập trình viên tầm thường và xuất sắc là liệu họ có biết ngôn ngữ hợp ngữ hay không. Những người biết có xu hướng hiểu bản thân máy tính ở mức độ sâu hơn nhiều. Mặc dù chưa từng nghe đến ngày nay, khái niệm này thực sự không mới – trước đây không có nhiều lựa chọn. Máy tính Apple chỉ đi kèm với BASIC và ngôn ngữ hợp ngữ, và có sách về ngôn ngữ hợp ngữ cho trẻ em. Đây là lý do tại sao những người cũ thường được xem như 'phù thủy': họ phải biết lập trình ngôn ngữ hợp ngữ. Có lẽ nỗi ám ảnh hiện tại về việc học bằng ngôn ngữ 'dễ' là cách làm sai ..."

Hiểu hợp ngữ có nghĩa là hiểu cách máy tính thực thi các lệnh. Bằng cách đó, chúng ta có thể viết các chương trình bằng ngôn ngữ bậc cao có hiệu suất cao, hiệu quả tài nguyên và tiết kiệm chi phí.

## RISC vs. CISC

Các CPU khác nhau triển khai các ISA khác nhau:

* [x86](https://en.wikipedia.org/wiki/X86): Intel i9, i7, i5, i3, và nhiều bộ xử lý AMD
* [ARM](https://en.wikipedia.org/wiki/ARM_architecture_family): Được sử dụng trong nhiều điện thoại di động; cũng là cơ sở của [dòng Apple silicon](https://en.wikipedia.org/wiki/Apple_M1)
* [MIPS](https://en.wikipedia.org/wiki/MIPS_architecture), kể từ năm 2020 đã bị ngừng sử dụng và chuyển sang
* [RISC-V](https://en.wikipedia.org/wiki/RISC-V): Trọng tâm của khóa học này.

Vào đầu những năm 1970 và 1980, có xu hướng xây dựng các lệnh ngày càng phức tạp cho máy tính. Rốt cuộc, thế giới là phần mềm! Mỗi lệnh sẽ thực hiện nhiều tác vụ, như truy cập bộ nhớ và thực hiện số học đồng thời. Các tập lệnh phức tạp này có thể giảm kích thước chương trình và thậm chí số lần truy cập bộ nhớ trên mỗi chương trình. Tuy nhiên, phần cứng cần thiết để triển khai các tập lệnh này thường phức tạp hơn để thiết kế và tốn kém để triển khai. Về sau, các kiến trúc này được gọi là Máy tính Tập lệnh Phức tạp (CISC).

Vào đầu những năm 80, một ý tưởng khác xuất hiện từ [John Cocke](https://en.wikipedia.org/wiki/John_Cocke_(computer_scientist)), người đã thiết kế IBM 801. Đây là **máy tính tập lệnh rút gọn** (RISC) đầu tiên: Giữ tập lệnh nhỏ và đơn giản. Độ phức tạp được đẩy vào phần mềm và compiler để tổng hợp các hoạt động phức tạp bằng các lệnh đơn giản này. Lớp đơn giản hơn này giúp việc xây dựng phần cứng nhanh hơn *dễ dàng hơn nhiều*!

Ý tưởng RISC[^turing] nhanh chóng được chấp nhận: Một chương trình nhất định bây giờ sẽ cần nhiều lệnh hợp ngữ hơn nhưng vẫn thực thi nhanh hơn trước nếu phần cứng tương ứng có thể được thiết kế để thực thi nhiều lệnh hơn mỗi giây.

Ý tưởng này sau đó được phát triển đến mức độ đầy đủ bởi Giáo sư Dave Patterson tại UC Berkeley và John Hennessy tại Stanford. Hai phòng thí nghiệm này phát triển hai dự án rất tương tự đồng thời: dự án RISC (tại UC Berkeley) và dự án MIPS (tại Stanford). Các bộ xử lý RISC đã được sử dụng cho vô số kiến trúc vi xử lý.

Cuối cùng, cả RISC và CISC (tức là kiến trúc "không RISC") vẫn tương đối phổ biến. Cả hai mô hình kiến trúc đều đang được phát triển nhanh chóng, và các thiết kế công nghiệp vẫn đang chạy đua để cải thiện hiệu suất. Trên thị trường, nhiều máy tính dựa trên Microsoft mua kiến trúc Intel; gần đây, Apple đã phát triển silicon dựa trên ARM của riêng mình.

[^turing]: Ba người tiên phong RISC ban đầu này cuối cùng đã giành được Giải thưởng Turing của ACM cho những đóng góp của họ cho kiến trúc máy tính: [Cocke](https://amturing.acm.org/award_winners/cocke_2083115.cfm) năm 1987, và [Patterson](https://amturing.acm.org/award_winners/patterson_2316693.cfm) và [Hennessy](https://amturing.acm.org/award_winners/hennessy_1426931.cfm) năm 2017.

## Tại sao RISC-V?

Khi giảng dạy, chúng ta phải chọn một ISA. x86 rất phức tạp, trong khi các ISA cũ hơn hoặc "tự chế" thường thiếu ngăn xếp phần mềm thực sự hoặc compiler.

ISA RISC-V[^isa] được phát triển năm 2010 tại UC Berkeley. Sau khi xuất hiện khoảng 10 năm trước, nó đã được sử dụng để phát triển tất cả các cấp độ hệ thống máy tính, từ vi điều khiển trong hệ thống nhúng đến siêu máy tính quy mô kho dữ liệu. Nó hỗ trợ nhiều biến thể bộ xử lý–phổ biến nhất là 32-bit, 64-bit và 128-bit.

RISC-V cực kỳ phổ biến vì hai lý do chính: Nó là **mã nguồn mở** và **miễn phí giấy phép**. Bất kỳ ai cũng có thể sử dụng nó mà không phải trả bản quyền, làm cho nó phổ biến cho giảng dạy[^teaching], nghiên cứu *và* sử dụng thương mại. Sự phát triển RISC-V được hỗ trợ bởi hệ sinh thái chia sẻ, quốc tế đang phát triển[^rv-international] của các nhà lãnh đạo học thuật và công nghiệp. Toàn bộ định nghĩa của kiến trúc RISC-V vừa trên một trang duy nhất gọi là "Green Card,"[^green-card] được đặt tên theo green card IBM 360 nổi tiếng từ những năm 1960.

Chúng ta dạy RISC-V trong lớp vì nó đơn giản và thanh lịch–cả cho việc hiểu ngôn ngữ hợp ngữ *và* cho việc thiết kế kiến trúc máy tính. Go Bears :-)

[^isa]: [RISC-V Unprivileged Instruction Set Architecture Specification](https://docs.riscv.org/reference/isa/unpriv/unpriv-index.html)

[^rv-international]: [RISC-V International](https://riscv.org/)

[^teaching]: Giáo sư Patterson và Krste Asanović bắt đầu dự án RISC-V năm 2010 như một phần của Par Lab để phát triển nghiên cứu và giảng dạy mở tại UC Berkeley. Một trong những đầu ra của dự án là chuỗi bốn khóa học cho sinh viên đại học và sau đại học, cuối cùng phát triển thành chương trình giảng dạy kiến trúc máy tính của UC Berkeley ngày nay. Đọc thêm trên [trang About của RISC-V](https://riscv.org/about/) và trong [Báo cáo Phả hệ RISC-V](https://riscv.org/about/genealogy/).

[^green-card]: ["RISC-V green card"](#sec-green-card) trong các ghi chú khóa học này dài hơn một trang, do định dạng web dễ truy cập. Để có thẻ tham khảo hai mặt với cùng thông tin, hãy xem thẻ tham khảo PDF trên trang web khóa học của chúng ta.
