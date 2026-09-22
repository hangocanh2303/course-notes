---
title: "Linked List: Triển khai đầy đủ"
---

(sec-ll-full)=
## Mục tiêu học tập

* Triển khai cấu trúc dữ liệu linked list trong C.
* Sử dụng file header (`.h`) để trừu tượng hóa giao diện linked list khỏi triển khai linked list.

Chúng ta xây dựng trên [phần trước](#sec-ll-add-front) để triển khai một bộ tính năng đầy đủ cho linked list.

:::{warning} Code khác nhau, mục tiêu học tập khác nhau

Code trong phần này và [phần trước](#sec-ll-add-front) khác nhau trong khai báo head của linked list. Điều này có chủ đích để tập trung vào các mục tiêu học tập khác nhau:

* [Phần trước](#sec-ll-add-front) là thực hành với con trỏ đôi.
* Phần này cố ý trừu tượng hóa định nghĩa linked list khỏi triển khai của nó.

Điểm chính của phần này là tách biệt file header và file nguồn. Ít nhất cho đến mùa Xuân 2026, chúng tôi không mong đợi trong khóa học này rằng bạn sẽ cần định nghĩa header của riêng mình.[^temperature]

[^temperature]: Nếu bạn tò mò, [Wikibooks](https://en.wikibooks.org/wiki/C_Programming/Headers_and_libraries) có một ví dụ cập nhật toàn diện về thư viện nhiệt độ đơn giản với nhiều giải thích hơn những gì được cung cấp ở đây.
:::

## Thư viện Linked List

Trong phần này, chúng ta muốn triển khai một linked list trong C để người khác sử dụng.

* **tạo** một linked list.
* **thêm** một node mới (với bản sao của chuỗi được cung cấp) vào đầu linked list.
* **in** tất cả các chuỗi trong linked list.
* **giải phóng** và xóa linked list.

### Header và File nguồn

Lý tưởng, chúng ta muốn ẩn các chi tiết nội bộ của triển khai khỏi cách chúng ta quảng cáo linked list. C cung cấp một quy ước đơn giản để làm điều đó: **header** (file header `.h`) và **file nguồn** (file `.c`).

* Header: Khai báo tên `struct` hoặc `typedef`, khai báo chữ ký hàm, v.v.
* Nguồn: Triển khai hàm, v.v.

Bằng cách làm như vậy, chúng ta có thể biên dịch file nguồn và phân phối linked list của chúng ta như file header và một binary nguồn đã biên dịch. Một lập trình viên khác sau đó sẽ cần include file header (ví dụ: với `#include <linked_list.h>`) nhưng sẽ không cần biên dịch lại binary nguồn.

Chúng ta sẽ thảo luận quá trình biên dịch và liên kết này trong một chương sau.

## Đặc tả: `linkedlist.h`

Để tránh làm ô nhiễm namespace toàn cục, chúng ta sẽ thêm tiền tố `list_` vào tên hàm của chúng ta. File header bên dưới định nghĩa giao diện linked list:

(card-ll-header)=
:::{card}
File header: `linkedlist.h`
^^^

```{code} c
:linenos:
#ifndef LINKEDLIST_H
#define LINKEDLIST_H

typedef struct _node node_t;           // khai báo trước
typedef struct _linked_list linked_list_t;  // kiểu list ẩn

// Tạo một list rỗng mới
linked_list_t* list_create(void);

// Thêm một node vào đầu list
void list_add_front(linked_list_t *list, const char* data);

// In list
void list_print(const linked_list_t *list);

// Giải phóng list và tất cả bộ nhớ liên quan
void list_free(linked_list_t *list);

#endif
```

:::

Ghi chú:

* Dòng 1-2: Các macro này ngăn include kép, ví dụ: nếu header này được include bởi nhiều file C trong project.
* Dòng 4: Đây là khai báo **trước**. Nó là "trước" vì chúng ta chưa định nghĩa `struct _node` (chúng ta định nghĩa nó trong file nguồn cùng với triển khai của nó).
* Dòng 4-5: Tên struct `struct _node` và `struct _linked_list` là ẩn vì người dùng thư viện sẽ không cần sử dụng `_node` và `_linked_list` một cách rõ ràng, do đó chúng ta thêm tiền tố `_` vào tên.

## Triển khai: `linkedlist.c`


(card-ll-source)=
:::{card}
File nguồn: `linkedlist.c`
^^^

```{code} c
:linenos:

#include "linkedlist.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct _node {
   char *data;
   node_t *next;
};

struct _linked_list {
   node_t *head;
};

// Tạo một list rỗng mới
linked_list_t *list_create(void) {
   linked_list_t *list = malloc(sizeof(linked_list_t));
   if (!list) return NULL;
   list->head = NULL;
   return list;
}

// Thêm một node vào đầu, giả sử list không NULL
void list_add_front(linked_list_t* list, const char* data) {
   node_t* node = malloc(sizeof(node_t));
   if (NULL == node) { printf("Lỗi trong list_add_front: Không thể cấp phát node_t\n"); exit(1);}
   node->data = (char *) malloc(strlen(data) + 1); // thêm một byte
   if (NULL == node->data) { printf("Lỗi trong list_add_front: Không thể cấp phát chuỗi\n"); exit(1);}
   strcpy(node->data, data);
   node->next = list->head;
   list->head = node; // khác với trước
}

// In list
void list_print(const linked_list_t *list) {
   node_t *current = list->head;
   while (current) {
       printf("%s -> ", current->data);
       current = current->next;
   }
   printf("NULL\n");
}

// Giải phóng lưu trữ list và data
void list_free(linked_list_t *list) {
   node_t *current = list->head;
   while (current) {
       node_t *next = current->next;
       free(current->data);
       free(current);
       current = next;
   }
   free(list);
}
```

:::

Ghi chú:

* Dòng 6 - 13: Vì header đã định nghĩa `typedef`, chúng ta có thể trực tiếp sử dụng `typedef` trong khai báo hàm. Tuy nhiên, định nghĩa `struct _node` và `struct _linked_list` vẫn phải sử dụng tên `struct` ẩn của chúng.
* Dòng 31: Như đã lưu ý trước đó, triển khai này hơi khác với `add_to_front` trong [phần trước](#sec-ll-add-front).
* Dòng 49-50: Chúng ta phải giải phóng chuỗi `data` được cấp phát trên heap trước khi giải phóng chính node `current`, không phải ngược lại. Làm như vậy tránh chúng ta sử dụng `current` sau khi giải phóng.

## Sử dụng: `list_demo.c`

Hãy sử dụng giao diện linked list này trong chương trình bên dưới.


(card-ll-demo)=
:::{card}
Chương trình ví dụ `list_demo.c`
^^^

```{code} c
:linenos:
#include "linkedlist.h"
#include <stdio.h>

int main(void) {
   linked_list_t *list = list_create();
   if (NULL == list) { printf("Lỗi trong main: Không thể cấp phát linked_list_t`\n"); return(1);}

   list_add_front(list, "Alice");
   list_add_front(list, "Bob");
   list_add_front(list, "Cara");

   list_print(list);  // Cara -> Bob -> Alice -> NULL

   list_free(list);
   return 0;
}
```

:::

Ghi chú:

* Dòng 1: Chúng ta không include file nguồn `linkedlist.c`! Include file header `linkedlist.h` là đủ để khai báo trước các struct và tên hàm cần thiết để sử dụng trong `main`.
* Dòng 4: `void` trong tham số của `main` chỉ định rằng `main` không nhận bất kỳ đối số dòng lệnh nào.[^main-void]

[^main-void]: Liệu khai báo như vậy có được ưa thích hơn, giả sử, `int main()` hay không là điều tranh cãi. Xem [StackOverflow](https://stackoverflow.com/questions/12225171/difference-between-int-main-and-int-mainvoid).

### Chạy Demo

Demo bên dưới là để tham khảo.

```bash
$ ls
linkedlist.c linkedlist.h list_demo.c Makefile
$ make
gcc -Wall -Wextra -std=c17 -c list_demo.c -o list_demo.o
gcc -Wall -Wextra -std=c17 -c linkedlist.c -o linkedlist.o
gcc -Wall -Wextra -std=c17 -o list_demo list_demo.o linkedlist.o
$ ./list_demo 
Cara -> Bob -> Alice -> NULL
```

Chúng tôi sẽ không mong đợi bạn viết Makefile trong lớp này, vì vậy nhấp bên dưới nếu bạn tò mò.

:::{note} Mở rộng để xem nội dung `Makefile`
:class: dropdown

```bash
# Trình biên dịch
CC = gcc

# Cờ biên dịch
CFLAGS = -Wall -Wextra -std=c17

# Tên chương trình thực thi
TARGET = list_demo

# File nguồn
SRC = list_demo.c linkedlist.c

# File object (thay .c bằng .o)
OBJ = $(SRC:.c=.o)

# Quy tắc mặc định
all: $(TARGET)

# Liên kết các file object thành chương trình thực thi
$(TARGET): $(OBJ)
	$(CC) $(CFLAGS) -o $(TARGET) $(OBJ)

# Biên dịch file .c thành .o và tạo dependencies
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# Include các file dependency (nếu tồn tại)
-include $(DEP)

# Dọn dẹp các file build
clean:
	rm -f $(OBJ) $(TARGET) $(DEP)
```
:::

## Kết luận

Chúng ta có thể triển khai các API thư viện (giao diện lập trình ứng dụng) sử dụng C.

* Header trong file `.h` đặc tả API. Chú ý đặt tên để tránh làm rối namespace toàn cục!
* Triển khai API trong file `.c` (nên `#include` file `.h`)
* Sử dụng chúng trong các file `.c` khác (cũng nên `#include` file `.h`)
