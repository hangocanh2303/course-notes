---
title: "C vs. Java"
---


## Mục tiêu học tập

* Sử dụng chương trình "Hello World" để hiểu cấu trúc chương trình C.
* Làm so sánh sơ bộ giữa C và Java.

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/A6ELzsvVEnE?si=PjlUZ4Aisa0PJjle
:width: 100%
:title: "[CS61C FA20] Lecture 03.3 - C Intro: C v. Java and C Syntax"
:::

::::

## Hello World

(hello_world_c)=
:::{card}
Chương trình C: `hello_world.c`
^^^

```c
#include <stdio.h>
int main(int argc, char *argv[]) {
  printf("Hello World!\n");
  return 0;
}
```

:::

(hello_world_java)=
:::{card}
Chương trình Java: `hello.java`
^^^

```java
public class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello world!");
  }
}
```

:::

### Những điểm nổi bật

* Trong C, chúng ta import thư viện bằng `#include`. Ở đây, chúng ta include `stdio` cho `printf()`, in ra stdout (ở đây, dòng lệnh).
* Có một hàm `main`.
  * C là **hướng hàm**; không giống Java, đây không phải là method của object.
  * Kiểu trả về của `main` không phải `void`; nó là số nguyên.
  * Theo quy ước, các chương trình C trả về `0` khi thành công. (Lý do chính là dễ dàng hơn để kiểm tra bằng không; chúng ta sẽ quay lại điều này sớm)

### Demo Chạy

Các hướng dẫn bên dưới chủ yếu để tham khảo. Tham khảo [phần này](#compile-vs-interpret-sec) để hiểu chi tiết.

:::{note} Biên dịch và chạy C
:class: dropdown

Để chạy chương trình này, sử dụng chương trình dòng lệnh, `gcc`, để biên dịch chương trình. Điều này tạo ra một chương trình nhị phân với tên mặc định `a.out` ([tại sao lại đặt tên như vậy?](https://en.wikipedia.org/wiki/A.out#:~:text=out%20is%20a%20file%20format,'s%20PDP%2D7%20assembler.)). Sau đó, chạy chương trình nhị phân.

```bash
$ gcc hello_world.c
$ ./a.out
Hello World!
```

Trong thực tế, đổi tên nhị phân thành thứ gì đó có ý nghĩa hơn, như `hello_world`:

```bash
$ gcc -o hello_world hello_world.c
$ ./hello_world
Hello World!
```

Bạn cũng sẽ thấy hữu ích khi tạo các ký hiệu debug cho `gdb`, debugger của chúng ta.

```bash
$ gcc -d -o hello_world hello_world.c
$ gdb hello_world
```

:::

(c-vs-java-sec)=
## C vs. Java

@tab-c-vs-java bên dưới được điều chỉnh từ bảng [C Programming vs. Java Programming](https://introcs.cs.princeton.edu/java/faq/c2java.html), được tạo cho chuỗi CS nhập môn của Đại học Princeton. Di chuột qua chú thích để biết thêm thông tin về mỗi hàng.

:::{table} (a) C vs. Java; (b) các toán tử tương tự
:label: tab-c-vs-java
:align: center

| Tính năng | C | Java |
| :--- | :--- | :--- |
| Mô hình Ngôn ngữ[^language-paradigm] | Hướng Hàm (đơn vị lập trình: hàm) | Hướng Đối tượng (đơn vị lập trình: Class = Kiểu Dữ liệu Trừu tượng) |
| Biên dịch[^compile-vs-interpret] | `gcc hello.c` tạo code ngôn ngữ máy | `javac Hello.java` tạo bytecode ngôn ngữ máy ảo Java |
| Thực thi[^compile-vs-interpret] | `./a.out` nạp, thực thi chương trình | `java Hello` thông dịch bytecodes |
| Quản lý Bộ nhớ Động[^dmm] | Thủ công (`malloc`, `free`) (thêm sau) | Garbage collection tự động; `new` vừa cấp phát vừa khởi tạo |
| Khai báo biến | Khai báo có kiểu; khai báo trước khi dùng | (giống) |
| Khai báo hàm | Dùng dấu ngoặc nhọn. `void` nghĩa là không có giá trị trả về | (giống) |
| Truy cập thư viện | `#include <stdio.h>` | `import java.util.*` |
| Chú thích | Nhiều dòng: `/* ... */`, cuối dòng: `// ...` | (giống) |

| Toán tử | C và Java |
| :--- | :--- |
| số học | `+`, `-`, `*`, `/`, `%` |
| gán | `=` |
| gán mở rộng | `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `\|=`, `^=`, `<<=`, `>>=` |
| logic bit | `~`, `&`, `\|`, `^` |
| dịch bit | `<<`, `>>` |
| logic boolean | `!`, `&&`, `\|\|` |
| kiểm tra bằng | `==`, `!=` |
| nhóm biểu thức con | `()` |
| quan hệ thứ tự | `<`, `<=`, `>`, `>=` |
| tăng và giảm | `++`, `--` |
| chọn thành viên[^member-selection] | `.`, `->` |
| đánh giá điều kiện[^conditional-evaluation] | `? :` |
:::

[^language-paradigm]:
    Java là ngôn ngữ hướng đối tượng; C chủ yếu là ngôn ngữ chức năng. Trong C, ý tưởng cốt lõi là hàm, trong khi ở Java, đó là class hoặc kiểu dữ liệu trừu tượng. Mặc dù có thể viết code giống object trong C, nếu chương trình yêu cầu object thì bạn thực sự nên sử dụng C++.
  
[^compile-vs-interpret]:
    Chúng ta đã thảo luận chi tiết điều này trong [phần trước](#compile-vs-interpret-sec).

[^dmm]:
    Java quản lý bộ nhớ cho bạn với garbage collection. Trong C, tất cả dây an toàn và phòng đệm đều không còn; bạn tự mình quản lý tất cả bộ nhớ, và bạn có thể gặp rắc rối rất nhanh. Thêm lần sau.

[^member-selection]: Hơi khác với Java vì có cả structure và con trỏ đến structure, thêm lần sau

[^conditional-evaluation]: `cond ? body_true : body_false`

## Thêm Điểm Nổi bật

**1. Quy ước đặt tên biến**: Trong C, dùng `snake_case`[^snake-case], KHÔNG PHẢI `camelCase` [^camelcase].

[^snake-case]: [Wikipedia](https://en.wikipedia.org/wiki/Snake_case)

[^camelcase]: [Wikipedia](https://en.wikipedia.org/wiki/Camel_case)

**2. Đối số dòng lệnh**: Trong chương trình [`hello_world.c`](#hello_world_c), hàm `main` có thể nhận đối số dòng lệnh với hai tham số:

* `argc` là số đếm nguyên có bao nhiêu đối số bạn có. Chính executable cũng được tính là một đối số. Nếu bạn chạy thứ gì đó như `./hello_world my_file`, `argc` là `2`.
* `argv`: là con trỏ đến mảng các đối số, dưới dạng chuỗi C. Chúng ta thảo luận về con trỏ, mảng, và chuỗi chi tiết hơn lần sau. Bây giờ, nếu bạn chạy `./hello_world my_file`, đối số đầu tiên[^zero-index] là đường dẫn của chính chương trình (`./hello_world`) và đối số thứ hai là chuỗi `my_file`.

[^zero-index]: Giống Python, mảng và chuỗi C được đánh chỉ số từ không.

:::{card}
**Đối số dòng lệnh**: Cú pháp mảng trông quen thuộc như thế nào?
^^^

```c
#include <stdio.h>
int main(int argc, char *argv[]) {
  printf("Received %d args\n", argc);
  for (int i = 0; i < argc; i++) {
    printf("arg %d: %s\n", i, argv[i]);
  }
  return 0;
}
```

:::

**3. Dấu ngoặc nhọn**: Ngôn ngữ C cho phép bỏ qua dấu ngoặc nhọn cho các câu lệnh một dòng—thậm chí cho các cấu trúc điều khiển như if-else và for. Điều này giống như trong Java, nhưng chúng tôi không nói với bạn. :-)

Nhưng chỉ vì bạn có thể, không có nghĩa là bạn nên. Vì các dòng tiếp theo của cấu trúc điều khiển được coi là ngoài body, bỏ qua dấu ngoặc nhọn dẫn đến nhiều lỗi debug[^curly-braces]:

[^curly-braces]: Stack Overflow: [Is it a bad practice to use an if-statement without curly braces?](https://stackoverflow.com/questions/2125066/is-it-a-bad-practice-to-use-an-if-statement-without-curly-braces)

:::{card}
**Bỏ qua dấu ngoặc nhọn**: Những dòng nào được in?
^^^

```c
#include <stdio.h>
int main(int argc, char *argv[]) {
    int x = 0;
    if (x == 0)
        printf("x is 0\n");
    if (x != 0) // cẩn thận!
        printf("x not 0 line 1\n");
        printf("x not 0 line 2\n");
    return 0;
}
```

:::
