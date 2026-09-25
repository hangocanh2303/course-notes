---
title: "Floating Point: Thảo luận thêm"
subtitle: "Nội dung này không được kiểm tra"
---

(sec-float-discussion)=
## Mục tiêu học tập

* Hiểu các định dạng floating point nào được sử dụng trong thực tế

::::{note} 🎥 Video bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/VkLcogCQAho
:width: 100%
:title: "[CS61C FA20] Lecture 06.5 - Floating Point: Floating Point Discussion"
:::

::::

Trong phiên bản trước của khóa học, chúng tôi đã đề cập floating point chi tiết hơn nhiều qua nhiều bài giảng. Trong các học kỳ gần đây, chúng tôi đã giảm các chủ đề floating point để tập trung vào cốt lõi của chuẩn, và chúng tôi không đề cập các chủ đề nâng cao hơn như số học, ép kiểu, và các biểu diễn floating point khác. Hiện tại, chúng tôi để nội dung ngoài phạm vi này dưới đây làm tham khảo chung.

## Phép cộng Floating Point

Hãy xem xét số học với số floating point.

Phép cộng floating point phức tạp hơn phép cộng số nguyên. Chúng ta không thể chỉ cộng các significand mà không xem xét giá trị số mũ. Nói chung:

* Không chuẩn hóa để khớp số mũ
* Cộng các significand với nhau
* Giữ số mũ đã khớp
* Chuẩn hóa, có thể thay đổi số mũ
* (Lưu ý: Nếu dấu khác nhau, chỉ cần thực hiện phép trừ thay thế.)

Vì cách số floating point được lưu trữ, các phép toán đơn giản như phép cộng không phải lúc nào cũng có tính kết hợp.

Định nghĩa `x`, `y`, và `z` lần lượt là $-1.5 \times 10^{38}$, $1.5 \times 10^{38}$, và $1.0$.

$$
\begin{align}
\texttt{x + (y + z)} &= -1.5 \times 10^{38} + (1.5 \times 10^{38} + 1.0) \\
&= -1.5 \times 10^{38} + (1.5 \times 10^{38}) \\
&= 0.0
\end{align}
$$

$$
\begin{align}
\texttt{(x + y) + z} &= (-1.5 \times 10^{38} + 1.5 \times 10^{38}) + 1.0 \\
&= 0.0 + 1.0\\
&= 1.0
\end{align}
$$


Nhớ rằng, floating point thực tế **xấp xỉ** kết quả thực. Với số mũ lớn hơn, bước nhảy giữa các float cũng lớn hơn. Trong ví dụ này, $1.5 \times 10^{38}$ lớn hơn nhiều so với $1.0$ đến mức $1.5 \times 10^{38} + 1.0$ trong biểu diễn floating point được làm tròn thành $1.5 \times 10^{38}$.

## Các chế độ làm tròn Floating Point

Khi chúng ta thực hiện phép toán trên số thực, chúng ta phải lo lắng về việc làm tròn để đưa kết quả vào trường significand. Phần cứng floating point mang thêm hai bit độ chính xác, sau đó làm tròn để có giá trị phù hợp.

Có bốn chế độ làm tròn chính:

* **Làm tròn về $+\infty$**. LUÔN làm tròn "lên": 2.001 → 3, -2.001 → -2
* **Làm tròn về $-\infty$**. LUÔN làm tròn "xuống": 1.999 →  1, -1.999 →  -2
* **Cắt ngắn**. Chỉ bỏ các bit cuối (làm tròn về 0)
* **Không thiên lệch**. Nếu ở giữa, làm tròn đến số chẵn.

Chế độ không thiên lệch là mặc định, mặc dù các chế độ khác có thể được chỉ định. Không thiên lệch hoạt động _gần như_ giống làm tròn bình thường. Nói chung, chúng ta làm tròn đến số có thể biểu diễn gần nhất, ví dụ: 2.4 làm tròn thành 2, 2.6 thành 3, 2.5 thành 2, 3.5 thành 4, v.v. Nếu giá trị ở ranh giới, chúng ta làm tròn đến số chẵn gần nhất. Nói cách khác, nếu có "hòa", một nửa thời gian chúng ta làm tròn lên; nửa còn lại chúng ta làm tròn xuống. Bản chất "không thiên lệch" này đảm bảo sự công bằng trong tính toán bằng cách cân bằng các sai số.

## Ép kiểu và chuyển đổi

Làm tròn cũng xảy ra khi chuyển đổi giữa các kiểu số. Trong C:

* **`int` sang `float`**: Có các số nguyên lớn mà `float` không thể xử lý chính xác vì nó thiếu đủ bit trong significand. Ví dụ, $2^{24} + 1$ sẽ "bám" vào float chẵn gần nhất.
* **`float` sang `int`**: Floating point với thành phần phân số đơn giản là không có biểu diễn số nguyên. C sử dụng **cắt ngắn** để ép và chuyển đổi floating point sang số nguyên gần nhất. Ví dụ, `(int) 1.5` bị cắt thành `1`.

Do đó ép kiểu kép không hoạt động như mong đợi. Code A và Code B dưới đây có thể không luôn in `"true"`:

```c
/* Code A */
int i = …;
if (i == (int)((float) i)) {
   printf("true\n");
}

/* Code B */
float f = …;
if (f == (float)((int) f)) {
   printf("true\n");
}
```

## Các biểu diễn Floating Point khác

### Precision vs. Accuracy

Nhớ lại từ trước:

* **Precision (Độ chính xác bit)** là số bit được sử dụng để biểu diễn một giá trị.
* **Accuracy (Độ chính xác giá trị)** là sự khác biệt giữa giá trị thực của một số và biểu diễn máy tính của nó.

Precision cao cho phép accuracy cao nhưng không đảm bảo điều đó.
Có thể có precision cao nhưng accuracy thấp.

Ví dụ, xem xét `float pi = 3.14;`. `pi` sẽ được biểu diễn sử dụng tất cả 23 bit của significand ("precision cao"), nhưng nó chỉ là một xấp xỉ của $\pi$ ("không chính xác").

Dưới đây, chúng ta thảo luận các biểu diễn floating point khác có thể cho số chính xác hơn trong một số trường hợp. Tuy nhiên, vì tất cả các biểu diễn này đều có precision cố định (tức là độ rộng bit cố định), chúng ta không thể biểu diễn mọi thứ một cách hoàn hảo.

### Còn nhiều biểu diễn Floating Point nữa

Vẫn còn nhiều biểu diễn khác tồn tại. Đây là một số từ chuẩn IEEE 754:

* **Quad-precision**, hay định dạng IEEE 754 quadruple-precision binary128. Được định nghĩa là 128 bit (15 bit số mũ, 112 bit significand) với phạm vi và độ chính xác không thể tin được.
* **Oct-Precision**, hay định dạng IEEE 754 octuple-precision binary256. Được định nghĩa là 256 bit (19 bit số mũ, 236 bit significand).
* **Half-Precision**, hay định dạng IEEE 754 half-precision binary16. Được định nghĩa là 16 bit (5 bit số mũ, 10 bit significand).

Các kiến trúc chuyên dụng cho miền đòi hỏi các định dạng số khác nhau (@tab-float-types). Ví dụ, bfloat16[^bf16] trên Tensor Processing Unit (TPU) của Google được định nghĩa trên 16 bit (8 bit số mũ, 7 bit significand); vì trường số mũ rộng hơn, nó bao phủ cùng phạm vi như định dạng IEEE 754 độ chính xác đơn với chi phí độ chính xác significand. Sự đánh đổi này được ưu tiên do gradient biến mất về không cho việc huấn luyện mạng neural.

:::{table} Các bộ tăng tốc miền khác nhau hỗ trợ các định dạng số nguyên và floating point khác nhau.
:label: tab-float-types
:align: center

| Bộ tăng tốc | int4 | int8 | int16 | fp16 | bf16[^bf16] | fp32 | tf32[^tf32] |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Google TPU v1 | | x | | | | | |
| Google TPU v2 | | | | | x | | |
| Google TPU v3 | | | | | x | | |
| Nvidia Volta TensorCore | | x | | x | | x | |
| Nvidia Ampere TensorCore | x | x | x | x | x | x | x |
| Nvidia DLA | | x | x | x | | | |
| Intel AMX | | x | | | x | | |
| Amazon AWS Inferentia | | x | | x | x | | |
| Qualcomm Hexagon | | x | | | | | |
| Huawei Da Vinci | | x | | x | | | |
| MediaTek APU 3.0 | | x | x | x | | | |
| Samsung NPU | | x | | | | | |
| Tesla NPU | | x | | | | | |

:::

[^tf32]: Xem [TensorFloat-32 của Nvidia](https://en.wikipedia.org/wiki/TensorFloat-32).
[^bf16]: Xem [bfloat16 của Google](https://docs.cloud.google.com/tpu/docs/bfloat16).

Với những ai quan tâm, chúng tôi khuyến nghị đọc về [định dạng Unum](https://en.wikipedia.org/wiki/Unum_%28number_format%29) được đề xuất, gợi ý sử dụng độ rộng trường _thay đổi_ cho số mũ và significand. Định dạng này thêm một "u-bit" để cho biết số là chính xác hay nằm giữa các unum.
