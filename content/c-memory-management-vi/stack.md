---
title: "Stack"
---

(sec-stack)=
## Mục tiêu học tập

* Hiểu cách stack pointer tự động cấp phát và giải phóng stack frame.
* Hiểu khi nào an toàn để truyền con trỏ vào stack giữa các hàm.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/Keducx5bp-g
:width: 100%
:enumerated: false
:title: "[CS61C FA20] Lecture 05.3 - C Memory Management: Memory Locations"
:::

Từ 8:37 trở đi
::::


## Cách Stack hoạt động

Stack là một khối bộ nhớ liền kề bắt đầu từ địa chỉ cao và tăng trưởng xuống dưới. Mỗi khi một hàm được gọi, một **stack frame** mới được cấp phát trên stack như một khối bộ nhớ liền kề. Stack frame này bao gồm không gian cho những thứ sau:

* các biến cục bộ được khai báo trong hàm
* một **địa chỉ trả về**, tức là địa chỉ lệnh trong phân đoạn text cần được truy cập tiếp theo sau khi hàm này kết thúc thực thi [^stack-info]
* các đối số hàm [^stack-info]

[^stack-info]: (cùng chú thích như trong [phần trước](#sec-mem-layout)) Vì các tham số và địa chỉ trả về rất quan trọng đối với việc gọi và trả về hàm, chúng được lưu trữ trực tiếp trong CPU nếu có thể — trên phần cứng đặc biệt gọi là thanh ghi (mà chúng ta nói sau). Vì chỉ có số lượng thanh ghi hạn chế, các tham số và địa chỉ trả về bổ sung được lưu trong bộ nhớ trên stack cho đến khi cần.

Cấp phát và giải phóng cực kỳ nhanh trên stack nhờ **stack pointer**. **Stack pointer** là một giá trị được theo dõi nội bộ[^sp-reg] cho chúng ta biết địa chỉ của "đỉnh stack", tức là điểm bắt đầu của frame hiện tại, và từ đó xác định việc cấp phát và giải phóng trên stack.

[^sp-reg]: Bản thân stack pointer phải sống ở đâu đó. Thay vì sống trong bộ nhớ, nó sống trên CPU trong một thanh ghi phần cứng đặc biệt, để có thể đọc và cập nhật nhanh chóng. Đây là chi tiết chúng ta bỏ qua bây giờ và thảo luận chi tiết sau.

Phân đoạn Stack quản lý các stack frame như cấu trúc dữ liệu stack: **Last In, First Out (LIFO)** - Vào sau, Ra trước.
"_Tăng_" stack bằng cách di chuyển stack pointer xuống địa chỉ thấp hơn, từ đó đẩy vào một "stack frame mới" (tức là một khối bộ nhớ mới). "_Thu nhỏ_" stack bằng cách di chuyển stack pointer lên địa chỉ cao hơn, từ đó lấy ra stack frame hiện tại.

Trong @fig-c-stack, lưu ý sự tăng trưởng **xuống dưới** của stack có nghĩa là các biến cục bộ của `fooB()` có địa chỉ byte thấp hơn các biến trong hàm gọi nó `fooA()`, và cứ tiếp tục như vậy.

:::{figure} images/c-stack.png
:label: fig-c-stack
:width: 80%
:alt: "Chuỗi gọi fooA đến fooB đến fooC được hiển thị bên cạnh các frame xếp chồng. Stack được xây dựng xuống dưới khi chuỗi gọi thực thi, có fooA ở trên cùng, tiếp theo là fooB rồi fooC, và cuối cùng là stack pointer sp tại ranh giới frame hiện tại thấp nhất. Mũi tên xuống cho thấy các stack frame mới được cấp phát về phía địa chỉ thấp hơn."

Stack tăng trưởng xuống dưới. Stack pointer (`sp`) trỏ đến đỉnh của stack, tức là địa chỉ của stack frame hiện tại.
:::

Bộ slide trong @fig-c-stack-anim minh họa động việc cấp phát và giải phóng trên stack qua stack pointer.

::::{figure}
:label: fig-c-stack-anim
:alt: "Bộ slide nhúng minh họa chuyển động của stack pointer khi các hàm C lồng nhau cấp phát và lấy ra các stack frame trên stack tăng trưởng xuống dưới."
:::{iframe} https://docs.google.com/presentation/d/e/2PACX-1vT4VF2QQM8HQDb84y2sg6Bie_PURuJfOZ5yyrFh3AWCJY2lay45Vqy33iN7XIrV1fO2tIb3G7590KcW/pubembed?start=false&loop=false
:width: 100%
:enumerated: false
:title: "Animation that steps through the enumerated text in this section about how the memory stack works. Access [original Google Slides](https://docs.google.com/presentation/d/12ZVT3XK4WGY7nm_Z_9FKaTT1h6YxtwCu-sZMBnAROCc/edit?usp=sharing)"
:::
Minh họa mở rộng về quản lý bộ nhớ stack trong C.
::::

:::{note} Giải thích @fig-c-stack-anim
:class: dropdown

1. `main()` được gọi ngay khi chương trình tải. Một stack frame cho `main` được cấp phát bằng cách di chuyển stack pointer `sp` xuống đầu frame mới này (nhớ lại: các khối bộ nhớ được tham chiếu bằng địa chỉ thấp nhất của chúng).
2. `a(0)` được gọi bởi `main`. `sp` di chuyển xuống. Các biến cục bộ cho `a` được khởi tạo trong bộ nhớ stack bắt đầu từ `sp` và đi lên trên. `sp` tạo đủ không gian để lưu các biến cục bộ, có kích thước được biết tại thời điểm biên dịch. Vì vậy không có nguy cơ việc khởi tạo các biến này sẽ tràn vào frame của `main`.
3. `b(1)` được gọi bởi `a`. Stack frame cho `b` được cấp phát ngay bên dưới stack frame của `a`. Stack pointer `sp` di chuyển xuống đầu frame mới này.
4. `c(2)` được gọi bởi `b`, v.v.
5. `d(3)` được gọi bởi `c`, v.v.
6. Khi `d` kết thúc thực thi, "trả về" quyền điều khiển cho hàm gọi `c` bằng cách (1) đặt lệnh tiếp theo để thực thi thành địa chỉ trả về[^stack-info], và (2) lấy ra stack frame, tức là cập nhật `sp` thành cuối stack frame của `d`, cũng là đầu stack frame của `c`. `c` bây giờ là hàm tiếp tục thực thi.
7. Khi `c` kết thúc thực thi, trả về quyền điều khiển cho `b`, v.v.
8. Khi `b` kết thúc thực thi, trả về quyền điều khiển cho `a`, v.v.
9. Khi `a` kết thúc thực thi, trả về quyền điều khiển cho `main`, v.v.
10. Khi `main` kết thúc thực thi, `sp` bây giờ trỏ đến địa chỉ cao nhất của stack, và không còn stack frame nào trên stack. Kết thúc chương trình.

:::

:::{warning} Bí mật của stack để quản lý bộ nhớ nhanh

Khi một hàm trả về và stack pointer di chuyển lên, hệ thống _**không** xóa bộ nhớ_. "Không có thời gian" để đặt về không; C quá bận.
:::

Xem xét @fig-c-stack-anim. Nếu bạn có một mật khẩu bí mật như `"Bosco"` được lưu trong một biến cục bộ trong hàm `d()`, và hàm `d()` trả về, chuỗi `"Bosco"` đó vẫn còn trong bộ nhớ mặc dù bạn không được phép truy cập nó. Bạn có thể thấy điều này nếu bạn đi theo một con trỏ đến một biến cục bộ đã trả về; lần đầu tiên bạn in nó, nó có thể vẫn hiển thị giá trị cũ (như `3`), nhưng lần tiếp theo bạn gọi một hàm như `printf`, lời gọi hàm mới đó tạo stack frame riêng của nó và ghi đè dữ liệu cũ.

## Truyền con trỏ vào stack

Tại thời điểm này trong cuộc đời lập trình của bạn, bạn có thể đã thấy hữu ích khi truyền dữ liệu từ một hàm này sang một hàm khác. Bây giờ chúng ta cũng biết rằng các con trỏ vào stack có thể trỏ đến _dữ liệu cũ_, nếu hàm của biến cục bộ tương ứng đã kết thúc thực thi! Trong những trường hợp này, C sẽ không báo lỗi, nhưng chương trình của bạn có thể có hành vi không xác định.

:::{warning} Tên mảng đã khai báo chỉ tham chiếu đến mảng trong phạm vi cục bộ của chúng.

Nhớ lại rằng [mảng trong C](#sec-array) là một cách khai báo cục bộ một khối bộ nhớ lớn. Bây giờ chúng ta biết rằng khai báo cục bộ này có nghĩa là khai báo mảng cấp phát không gian trên stack, và do đó phải tính vào kích thước cố định của stack frame của hàm. Mảng được khai báo cục bộ phải có kích thước đã biết tại thời điểm biên dịch, như trong [ví dụ `sizeof` này](#sec-array-sizeof).
 
Khi tên mảng được truyền như đối số cho các hàm khác, chúng **suy biến** thành con trỏ đến bộ nhớ. Bây giờ, chúng ta biết rằng mảng sẽ suy biến thành con trỏ đến bộ nhớ trên _stack_.
:::

:::{hint} Địa chỉ cao hơn trong stack

Sử dụng con trỏ vào stack như _đối số hàm_ là an toàn.
:::

Đoạn code bên dưới tương ứng với bố cục bộ nhớ stack trong @fig-c-stack-buf-ok. Cấu trúc code này phổ biến khi `load_buf` tải dữ liệu từ, giả sử, một file vào một **buffer** bộ nhớ cụ thể. `main` trước tiên cấp phát `len` byte không gian[^size_t-info] cho buffer trong biến cục bộ `buf`, rồi gọi `load_buf`, rồi xử lý dữ liệu trong `buf`.

[^size_t-info]: kiểu số nguyên không dấu đủ lớn để "đếm" byte bộ nhớ.

```{code} c
:linenos:
void load_buf(char *ptr, size_t len) {
  ...
}

int main() {
  ...
  char buf[...];
  load_buf(buf, BUFLEN);
  ...
}
```

:::{figure} images/c-stack-buf-ok.png
:label: fig-c-stack-buf-ok
:width: 70%
:alt: "Mẫu stack-buffer an toàn: main cấp phát mảng cục bộ buf trong frame riêng của nó và truyền một con trỏ đến load_buf trong frame callee bên dưới. Vì main vẫn hoạt động trong suốt cuộc gọi, load_buf có thể ghi vào buf một cách an toàn."

`main` truyền biến cục bộ `buf` của nó vào lời gọi hàm `load_buf`.
:::

Tất cả các stack frame hiện đang trên stack tương ứng với các lời gọi hàm chưa kết thúc thực thi. Trong @fig-c-stack-buf-ok, stack frame `main` tồn tại cho đến khi `load_buf` (mà `main` gọi) trả về, và sau đó nữa. `load_buf` có thể "tin tưởng" rằng `len` byte bộ nhớ stack bắt đầu từ `ptr` sẽ tồn tại trong suốt quá trình thực thi của hàm và có thể tự tin ghi vào vùng stack này.

:::{danger} Địa chỉ thấp hơn trong stack

Trả về một con trỏ vào stack là thảm họa!
:::

Đoạn code bên dưới tương ứng với bố cục bộ nhớ stack nguy hiểm trong @fig-c-stack-buf-bad. Ở đây, `main` vẫn muốn tải và xử lý dữ liệu từ một file, nhưng nó làm điều đó bằng cách trước tiên tạo buffer trong lời gọi hàm `make_buf`, rồi gọi `foo` để xử lý dữ liệu.

```{code} c
:linenos:
char *make_buf(size_t len) {
    char buf[len];
    return buf;
}
void foo(char *ptr2, size_t len) { ... }
int main(){
   char *ptr = make_buf(BUFLEN); 
   foo(ptr, BUFLEN);
   ...
}
```

:::{figure} images/c-stack-buf-bad.png
:label: fig-c-stack-buf-bad
:width: 70%
:alt: "Mẫu con trỏ stack trả về không an toàn được hiển thị trong hai phần: bên trái, make_buf tạo buf cục bộ và trả về địa chỉ của nó cho main; bên phải, một lời gọi sau đó đến foo tái sử dụng vùng stack thấp hơn đó. Con trỏ ptr trong main bây giờ treo và có thể tham chiếu đến dữ liệu đã bị ghi đè."

Trái: Bố cục stack khi `make_buf` trả về. Phải: Bố cục stack khi `foo` đang thực thi.
:::

`main` có một biến cục bộ con trỏ `ptr`. Ở Dòng 7, `ptr` được cập nhật thành một địa chỉ trong `make_buf`, mà _đã trả về rồi_. Chúng ta gọi đây là **tham chiếu treo (dangling reference)**, vì `ptr` trỏ đến bộ nhớ đã được giải phóng.

Địa chỉ được lưu trong `ptr` là một vị trí của stack _bên dưới_ stack frame `main`. Khi `foo` được gọi ở Dòng 8, stack frame `foo` sẽ tham chiếu bộ nhớ bên dưới stack frame `main`, đúng nơi `ptr` được cho là trỏ đến! Không có đảm bảo rằng `len` byte tại địa chỉ `ptr` sẽ không được sử dụng cho dữ liệu khác.

Trong khi chúng ta _có thể_ hack được một cái gì đó hoạt động cho ví dụ đơn giản này, hãy xem xét rằng có thể có nhiều hàm bổ sung được gọi giữa `main` và `foo`. Nếu bản thân `foo` gọi một hàm hệ thống, giả sử `printf`, thì càng ít cơ hội `ptr` không bị hỏng bởi các lời gọi hàm khác.

Trên hết, nhớ rằng bộ nhớ thấp hơn trong stack bị ghi đè khi các hàm khác được gọi.
