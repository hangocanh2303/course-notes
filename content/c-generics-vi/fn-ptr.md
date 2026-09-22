---
title: "Con trỏ hàm"
---

:::{warning} ⚠️⚠️⚠️ Bạn đang tìm Tiền tố IEC?
Các tiền tố IEC như MiB, GiB không phải là nội dung C về mặt kỹ thuật nhưng đã được đề cập trong bài giảng này. Chúng được liên kết ở thanh bên phía dưới: [Tiền tố IEC và Cơ số 10](#sec-iec-prefixes).
:::


## Mục tiêu học tập

* Hiểu cú pháp con trỏ hàm.
* Khai báo và khởi tạo con trỏ hàm, và gọi các hàm được trỏ bởi con trỏ hàm.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/PlIYbp0fgY4
:width: 100%
:enumerated: false
:title: "Lecture 04.4 - C Intro: Pointers, Arrays, Strings: Function Pointer Example"
:::
::::


### Con trỏ hàm

Trong [chương trước](#sec-pointers) chúng ta đã đề cập ngắn gọn về **con trỏ hàm**.

```c
int (*fn) (void *, void *) = &foo;
(*fn)(x, y);
```

Ở dòng đầu tiên, `fn` là một hàm nhận hai con trỏ `void *` và trả về một `int`. Với khai báo này, chúng ta đặt nó trỏ đến hàm `foo`. Dòng thứ hai sau đó gọi hàm với các đối số `x` và `y`.

Con trỏ hàm cho phép chúng ta định nghĩa các **hàm bậc cao**, như map, filter, và sắp xếp generic.

Thông thường một con trỏ chỉ có thể trỏ đến một kiểu. Trong [chương sau](#sec-generics) chúng ta thảo luận về con trỏ `void *`, một **con trỏ generic** có thể trỏ đến bất cứ thứ gì. Trong khóa học này chúng ta sẽ sử dụng con trỏ generic một cách tiết kiệm để giúp tránh lỗi chương trình...và vấn đề bảo mật...và những thứ khác... Tuy nhiên, chúng ta sẽ gặp con trỏ generic khi làm việc với các hàm quản lý bộ nhớ trong thư viện chuẩn C (`stdlib`).

## Code ví dụ và Kết quả

Chương trình này triển khai `mutate_map`, hàm áp dụng (map) một hàm cho trước lên mỗi phần tử của một mảng `int`. Đáng chú ý, `mutate_map` định nghĩa một tham số con trỏ hàm.

(card-code-fn-ptr)=
:::{card}
Chương trình ví dụ `map_func.c`
^^^
(code-fn-ptr)=
```{code} c
:linenos:
#include <stdio.h>

/* áp dụng hàm lên mảng int */
void mutate_map(int arr[], int n, int(*fp)(int)) {
    for (int i = 0; i < n; i++)  
        arr[i] = (*fp)(arr[i]);
}

/* in mảng int */
void print_array(int arr[], int n) {
    for (int i = 0; i < n; i++)  
        printf("%d ", arr[i]);
    printf("\n");
}

int multiply2 (int x) {  return  2 * x;    }
int multiply10(int x) {  return 10 * x;   }

int main() {
    int arr[] = {3,1,4}, n = sizeof(arr)/sizeof(arr[0]);
    print_array(arr, n);

    mutate_map(arr, n, &multiply2);
    print_array(arr, n);

    mutate_map(arr, n, &multiply10);
    print_array(arr, n);

    return 0;
}
```

:::

Code này nhân mảng số nguyên `arr` với hai, sau đó nhân `arr` lần nữa với mười, với hai lời gọi liên tiếp đến `mutate_map`. Hai lời gọi `mutate_map` truyền vào các con trỏ hàm khác nhau đến `multiply2` và `multiply10` tương ứng.

Chạy file thực thi `map_func` đã biên dịch tạo ra kết quả sau:

```bash
$ ./map_func
3 1 4 
6 2 8 
60 20 80 
```

:::{note} Giải thích thêm
:class: dropdown

* **Dòng 4: Khai báo tham số `fp`.** Tham số `fp` là một con trỏ hàm. Nó nhận một đối số `int` và trả về một `int`.
* **Dòng 23, 26: Truyền đối số con trỏ hàm**. Các lời gọi hàm ở Dòng 23 và 26 truyền vào `multiply2` và `multiply10` tương ứng làm đối số con trỏ hàm. Toán tử địa chỉ (`&`) được sử dụng để dễ đọc nhưng về mặt kỹ thuật không cần thiết.[^fn-reference]
* **Dòng 6: Gọi `fp`**. Cú pháp `(*fp)(arr[i])` gọi hàm được trỏ bởi `fp` và truyền vào đối số `arr[i]`. Giống như trước, toán tử giải tham chiếu (`*`) được sử dụng để dễ đọc nhưng về mặt kỹ thuật không cần thiết cho con trỏ hàm.[^fn-reference]

:::

[^fn-reference]: `(*fp)(arg)` và
`fp = &fname` là các lựa chọn phong cách và không bắt buộc theo chuẩn C. Tuy nhiên, việc sử dụng chúng được khuyến khích mạnh mẽ để dễ đọc. StackOverflow có [nhiều](https://stackoverflow.com/questions/7518815/function-pointer-automatic-dereferencing) [bài viết](https://stackoverflow.com/questions/7518815/function-pointer-automatic-dereferencing) về chủ đề này.
