---
title: "Tổng kết"
---

## Và để kết luận$\dots$

* Con trỏ và mảng C **gần như giống nhau**[^huge-caveat], ngoại trừ với các lời gọi hàm.
* C biết cách **tăng con trỏ**.
* C là ngôn ngữ hiệu quả, nhưng có **ít bảo vệ**:
  * Giới hạn mảng **không được kiểm tra**
  * Biến **không được tự động khởi tạo**
* Sử dụng handle để thay đổi con trỏ.
* Chuỗi là mảng các ký tự với ký tự kết thúc null. Độ dài là số ký tự, nhưng bộ nhớ cần thêm 1 cho \0
* **Cảnh báo**: Chi phí của hiệu quả là nhiều overhead hơn cho lập trình viên. "C cho bạn nhiều dây thừng, đừng tự treo cổ mình với nó!"

[^huge-caveat]: Sự khác biệt lớn nhất giữa mảng và con trỏ nằm ở vị trí của chúng trong bộ nhớ; sự khác biệt này dẫn đến nhiều chi tiết bạn đã thấy trong chương này. Xem [chương tiếp theo](#sec-mem-layout) để có khung bố cục bộ nhớ tổng quát sẽ giúp bạn hiểu sự phân biệt.

## Đọc sách giáo khoa

K&R: Chương 5-6

## Tài liệu tham khảo bổ sung

* Ghi chú về C của Giáo sư danh dự Brian Harvey [notes on C](https://inst.eecs.berkeley.edu/~cs61c/resources/HarveyNotesC1-3.pdf)

## Bài tập
Kiểm tra kiến thức của bạn!

### Ôn tập khái niệm

:::{exercise}
:label: c-ptrs-01
1. **Đúng/Sai**: Cách đúng để khai báo một mảng ký tự là `char[] array`.
:::

:::{solution} c-ptrs-01
:label: c-ptrs-01-sol
:class: dropdown

**Sai.** Cách đúng là `char array[]`.

<!--Xem: [Lecture 4 Slide 20](https://docs.google.com/presentation/d/1qSZZ1_rcPgtix08uJtxgkueccjWzwiCRrr4fhsGueJw/edit?slide=id.g32c0b5df322_0_692#slide=id.g32c0b5df322_0_692)-->
:::

:::{exercise}
:label: c-ptrs-02
2. **Đúng/Sai**: C là ngôn ngữ truyền theo giá trị (pass-by-value).
:::

:::{solution} c-ptrs-02
:label: c-ptrs-02-sol
:class: dropdown

**Đúng.** Nếu bạn muốn truyền một tham chiếu đến bất kỳ thứ gì, bạn nên sử dụng con trỏ.

<!--Xem: [Lecture 4 Slide 5](https://docs.google.com/presentation/d/1qSZZ1_rcPgtix08uJtxgkueccjWzwiCRrr4fhsGueJw/edit?slide=id.g32af6a99fd0_0_10#slide=id.g32af6a99fd0_0_10)-->
:::

:::{exercise}
:label: c-ptrs-03
3. Con trỏ là gì? Nó có điểm gì chung với biến mảng?
:::

:::{solution} c-ptrs-03
:label: c-ptrs-03-sol
:class: dropdown

Như chúng ta hay nói, "mọi thứ chỉ là bit." Một con trỏ chỉ là một chuỗi các bit, được diễn giải như một địa chỉ bộ nhớ. Một mảng hoạt động như một con trỏ đến phần tử đầu tiên trong bộ nhớ được cấp phát cho mảng đó. Tuy nhiên, tên mảng không phải là một biến; nghĩa là, `&arr` là `arr`, trong khi `&ptr` không phải là `ptr` trừ khi có điều gì đó kỳ diệu xảy ra (điều đó có nghĩa gì?).

<!--Xem: [Lecture 4 Slide 5](https://docs.google.com/presentation/d/1qSZZ1_rcPgtix08uJtxgkueccjWzwiCRrr4fhsGueJw/edit?slide=id.g32af6a99fd0_0_10#slide=id.g32af6a99fd0_0_10)-->
:::

:::{exercise}
:label: c-ptrs-04
4. Nếu bạn cố giải tham chiếu một biến không phải là con trỏ, điều gì sẽ xảy ra? Còn khi bạn free nó?
:::

:::{solution} c-ptrs-04
:label: c-ptrs-04-sol
:class: dropdown

Nó sẽ xử lý các bit cơ bản của biến đó như thể chúng là một con trỏ và cố gắng truy cập dữ liệu ở đó. C sẽ cho phép bạn làm hầu như bất cứ điều gì bạn muốn, mặc dù nếu bạn cố truy cập một địa chỉ bộ nhớ "bất hợp pháp", nó sẽ segfault vì những lý do chúng ta sẽ học sau trong khóa học. Đó là lý do tại sao C không được coi là "an toàn bộ nhớ": bạn có thể tự bắn vào chân mình nếu không cẩn thận. Nếu bạn free một biến đã được free trước đó hoặc không được malloc/calloc/realloc, những điều tồi tệ sẽ xảy ra. Hành vi là không xác định và kết thúc thực thi, dẫn đến lỗi "invalid free".

<!--Xem: [Lecture 4 Slide 18](https://docs.google.com/presentation/d/1qSZZ1_rcPgtix08uJtxgkueccjWzwiCRrr4fhsGueJw/edit?slide=id.g32a3dfb97c2_1_32#slide=id.g32a3dfb97c2_1_32)-->
:::
