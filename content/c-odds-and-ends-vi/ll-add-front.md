---
title: "Linked List: add_front"
---

(sec-ll-add-front)=
## Mục tiêu học tập

* Viết code sử dụng struct và typedef trong bộ nhớ động
* Thực hành cấp phát bộ nhớ trên heap với `malloc`
* Viết code sử dụng hàm với con trỏ đôi.

Qua phần này và [phần tiếp theo](#sec-ll-full), chúng ta sẽ thấy một ví dụ về việc sử dụng structure, con trỏ, và bộ nhớ động (heap) để triển khai một linked list của các chuỗi C.

:::{warning} Code khác nhau, mục tiêu học tập khác nhau

Code trong phần này và [phần tiếp theo](#sec-ll-full) khác nhau trong khai báo head của linked list. Điều này có chủ đích để tập trung vào các mục tiêu học tập khác nhau:

* Phần này là thực hành với con trỏ đôi.
* [Phần tiếp theo](#sec-ll-full) cố ý trừu tượng hóa định nghĩa giao diện linked list khỏi triển khai của nó.

:::

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/sJaSIlZzBPk?si=4Ar6Alh95EZZrdg
:width: 100%
:enumerated: false
:title: "[CS61C FA20] Lecture 05.2 - C Memory Management: Linked List Example"
:::

::::

Code được thảo luận trong course note này hơi khác với video bài giảng:

* Quy ước đặt tên đã được cập nhật để giống C hơn (ví dụ: `snake_case`; typedef có hậu tố `_t`)
* Code course note trình bày một kịch bản linked list tương đối thực tế, nơi con trỏ `head` được khai báo trong `main` và sau đó được cập nhật thông qua hàm `add_to_front` (do đó, tham số con trỏ đôi)

## Struct `node_t`


**Linked list** là một cấu trúc dữ liệu có thể được mô tả đệ quy, như được định nghĩa [bên dưới](#code-ll-def):

(code-ll-def)=
```{code} c
:linenos:
typedef struct _node node_t;
struct _node {
    char *data;
    node_t *next;
};
```

Mỗi `node_t` có hai trường: `data`, có một con trỏ (ví dụ: node "lưu" một chuỗi), và `next`, là một con trỏ đến một `node_t` khác. Cấu trúc đệ quy có nghĩa là con trỏ `next` của một node trỏ đến một `node_t` khác, mà sau đó trỏ đến `node_t` tiếp theo, và cứ tiếp tục, cho đến khi thành viên `next` của node cuối cùng là `NULL`, báo hiệu kết thúc danh sách.

Để làm code sạch hơn, chúng ta sử dụng `typedef`. Dòng 1 khai báo `node_t` như một bí danh của `struct _node`, đã được khai báo trước nhưng chưa định nghĩa. Các dòng từ 2 trở đi sau đó định nghĩa các trường của `struct _node`.[^typedef-struct]

[^typedef-struct]: Đọc về quy ước `typedef` và `struct` trên [StackOverflow](https://stackoverflow.com/questions/71270954/how-to-properly-use-typedef-for-structs-in-c).

## Code Linked List: `add_to_front`

Hãy xem code thêm một chuỗi vào linked list hiện có.

Trong code [bên dưới](#code-ll-add), `head` được định nghĩa như một con trỏ đến node đầu tiên trong linked list. Danh sách bắt đầu rỗng, tức là `head` trỏ đến không gì (`NULL`). Sau đó, Dòng 4 gọi `add_to_front`. Khi chúng ta trả về từ lời gọi này, chúng ta mong đợi rằng `head` bây giờ nên trỏ đến một node có *bản sao* của chuỗi được truyền vào.[^ll-str-malloc]

[^ll-str-malloc]: Sinh viên thường hỏi tại sao `add_to_front` tạo một node trỏ đến _bản sao_ của chuỗi. Sau cùng, chuỗi đã tồn tại, vậy tại sao không trỏ đến nó? Đây là quan điểm của tôi: `add_to_front` đang tạo một node mới, và lý tưởng data của node đó là của riêng nó để kiểm soát. Nếu chúng ta đẩy việc cấp phát/giải phóng bộ nhớ node sang các hàm node, thì chúng ta giảm nguy cơ data node sẽ chứa các con trỏ treo. Cuối cùng, hầu hết các ứng dụng linked list liên quan đến việc thay đổi data node; chúng ta sao chép các hằng chuỗi vào bộ nhớ động trước để sửa đổi chúng.

(code-ll-add)=

```{code} c
:linenos:

# include <string.h>
int main() {
  node_t *head = NULL;
  add_to_front(&head, "abc");
  …  // giải phóng các node, chuỗi ở đây…
}

void add_to_front(node_t **head_ptr, char *data) {
  node_t *node = (node_t *) malloc(sizeof(node_t));
  node->data = (char *) malloc(strlen(data) + 1); // thêm một byte
  strcpy(node->data, data);  // strcpy cũng sao chép null terminator
  node->next = *head_ptr;
  *head_ptr = node;
}
```

Xem xét hàm `add_to_front` ở trên. Hàm nhận hai con trỏ: một đến linked list (một **con trỏ đôi** `node_t **head_ptr`) và một chuỗi, `char * data`.
Nhớ lại rằng con trỏ là cách nhẹ để truyền dữ liệu vào một hàm, ngay cả khi bản thân linked list hoặc chuỗi khá lớn.

Nhưng tại sao lại là con trỏ đôi? Chúng ta sẽ thảo luận điều này khi chúng ta theo dõi qua code, từng dòng một.

### [Dòng 4](#code-ll-add): `main`

Đối số đầu tiên được truyền vào là một địa chỉ (`&head`); nói cách khác, `head_ptr` là một **con trỏ** đến `head`. Đối số thứ hai được truyền vào là một hằng chuỗi (tức là một mảng chỉ đọc). Trong khi C là truyền theo giá trị, các đối số mảng suy biến thành con trỏ đến phần tử đầu tiên, vì vậy `data` là một con trỏ đến ký tự đầu tiên trong hằng chuỗi.

:::{figure} images/ll-line04.png
:label: fig-ll-line04
:width: 100%
:alt: "Gán đối số cho add_to_front: head_ptr nhận địa chỉ của head, có giá trị hiện tại là NULL, và data trỏ đến hằng chuỗi abc với byte null kết thúc."

Gán đối số `add_to_front`
:::

### [Dòng 9](#code-ll-add): Cấp phát không gian heap cho node mới

Lời gọi `malloc` này tạo một struct `node_t` mới trong bộ nhớ động (tức là heap). Trong sơ đồ đơn giản, `node` trỏ đến một `node_t` mới được cấp phát tại địa chỉ heap `0x300`. Tại thời điểm này, nội dung của node đó — cả trường `data` và `next` — chỉ là rác vì C không khởi tạo chúng cho bạn.

:::{figure} images/ll-line09.png
:label: fig-ll-line-09
:width: 100%
:alt: "Sau khi cấp phát node, con trỏ cục bộ node lưu địa chỉ heap 0x300 cho một node_t mới có các trường data và next vẫn chưa khởi tạo."

Dòng 9
:::

### [Dòng 10](#code-ll-add): Cấp phát không gian heap cho chuỗi của node mới

Phía bên phải: Lời gọi `malloc` này tạo một mảng ký tự mới trong bộ nhớ động. Trước khi sao chép chuỗi, chúng ta tạo chỗ cho nó[^ll-str-malloc] trên heap.

Chuỗi được định nghĩa là mảng ký tự kết thúc bằng null; do đó `malloc` không gian cho chuỗi luôn liên quan đến `strlen(string) + 1`. Chúng ta sử dụng `strlen(string)` để tìm ra chuỗi dài bao nhiêu, nhưng nhớ rằng `strlen` không bao gồm null terminator. Nếu chuỗi là `"abc"`, `strlen` nói ba, nhưng bạn thực sự cần bốn để bao gồm `'\0'`.

Phía bên trái: Con trỏ được trả về bởi `malloc` sau đó được đặt làm trường `data` của `node` sử dụng ký hiệu mũi tên (`node->data`), cách dựa trên con trỏ để đi theo `node` đến trường `data` của nó.

:::{figure} images/ll-line10.png
:label: fig-ll-line-10
:width: 100%
:alt: "Sau khi cấp phát lưu trữ chuỗi, node->data được đặt thành địa chỉ heap 0x350, trỏ đến bốn byte chưa khởi tạo dành cho abc và null terminator."

Dòng 10
:::

### [Dòng 11](#code-ll-add): `strcpy` data chuỗi

Khi chúng ta có không gian chưa khởi tạo đó được dành riêng, chúng ta gọi `strcpy` (sao chép chuỗi) để mang giá trị qua. Điều này sao chép các ký tự `'a'`, `'b'`, `'c'`, và null terminator vào không gian mới được cấp phát của chúng ta.[^ll-str-strcpy]

[^ll-str-strcpy]: `strcpy` là một hàm đáng sợ. Từ trang `man`: "`strcpy(dst, src)` sao chép chuỗi được trỏ bởi `src`, vào một chuỗi tại buffer được trỏ bởi `dst`. Lập trình viên chịu trách nhiệm cấp phát buffer đích đủ lớn, tức là `strlen(src) + 1`."

:::{figure} images/ll-line11.png
:label: fig-ll-line-11
:width: 100%
:alt: "Sau strcpy, các byte heap tại node->data chứa a, b, c, và null terminator, trong khi node->next vẫn chưa khởi tạo."

Dòng 11
:::

### [Dòng 12](#code-ll-add): Cập nhật con trỏ next của node mới

Tiếp theo, chúng ta đặt trường `next` của `node`. Đây là một ví dụ tuyệt vời về chia sẻ; con trỏ next của node mới bây giờ trỏ đến head ban đầu của danh sách. Lưu ý rằng chúng ta giải tham chiếu với `*head_ptr` (một con trỏ đôi đến node). Trong trường hợp này, `head_ptr` _trỏ đến_ `NULL`, vì vậy giải tham chiếu `head_ptr` cho chúng ta địa chỉ `NULL`, mà chúng ta sao chép vào struct.

:::{figure} images/ll-line12.png
:label: fig-ll-line-12
:width: 100%
:alt: "Đặt node->next thành *head_ptr lưu NULL vào trường next, làm node mới trở thành đuôi của danh sách một node."

Dòng 12
:::

### [Dòng 13](#code-ll-add): Cập nhật head của linked list

Cuối cùng, chúng ta phải cập nhật head của danh sách, được định nghĩa như một con trỏ đến node đầu tiên trong linked list. "head" hiện đang là `NULL` nhưng nên được cập nhật thành `0x300`, là địa chỉ của struct chúng ta vừa tạo, mà theo định nghĩa là cái mà `node` trỏ đến. Do đó đủ để đặt `*head_ptr` (giá trị mà `head_ptr` trỏ đến) thành `node` (0x300).

:::{figure} images/ll-line13.png
:label: fig-ll-line-13
:width: 100%
:alt: "Cập nhật *head_ptr thành node ghi 0x300 vào head, vì vậy head danh sách bây giờ trỏ đến node mới được cấp phát có data trỏ đến abc."

Dòng 13
:::

### [Dòng 4](#code-ll-add) Trở về `main` sau lời gọi hàm

Nhớ lại rằng các biến được khai báo trong một hàm được thu hồi bởi stack khi hàm kết thúc. Struct `node_t` và chuỗi được cấp phát trên heap vẫn tồn tại, nhưng con trỏ cục bộ `node` biến mất. Tuy nhiên, chúng ta vẫn có thể theo dõi node chúng ta đã tạo vì con trỏ đôi cho phép chúng ta cập nhật trực tiếp giá trị của biến `head`.

:::{figure} images/ll-line05.png
:label: fig-ll-line-05
:width: 100%
:alt: "Trạng thái sau khi trở về main: biến cục bộ node đã biến mất, nhưng head bây giờ lưu 0x300 và vẫn đến được heap node và chuỗi abc đã sao chép."

Trở về từ lời gọi hàm tại Dòng 4
:::

## Kết luận

Ví dụ này cho thấy cách làm việc thành thạo với struct và con trỏ trong C. Chúng tôi sẽ tiếp tục cho bạn xem thêm các ví dụ để bạn thoải mái hơn với cách quản lý bộ nhớ hoạt động. Chúc may mắn, và hẹn gặp lại trong phần tiếp theo!
