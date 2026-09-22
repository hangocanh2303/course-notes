---
title: "Tóm tắt"
---

:::{warning} ⚠️⚠️⚠️ Bạn đang tìm Tiền tố IEC?
Các tiền tố IEC như MiB, GiB không phải là nội dung C về mặt kỹ thuật nhưng đã được đề cập trong bài giảng này. Chúng được liên kết ở thanh bên phía dưới: [Tiền tố IEC và Cơ số 10](#sec-iec-prefixes).
:::


## Và để kết luận$\dots$

* Các hàm generic (tức là generics), sử dụng con trỏ void * để thao tác trên bộ nhớ.
  * Generics có mặt rộng rãi trong thư viện chuẩn C! (`malloc`, ...)
  * Generics yêu cầu hiểu biết vững chắc về bộ nhớ; bằng việc thao tác các byte tùy ý, bạn có nguy cơ vi phạm ranh giới dữ liệu, ví dụ: "Frankenstein hóa" hai nửa của các int.

* Khi viết generics:
  * Con trỏ generic không hỗ trợ giải tham chiếu.
  * Thay vào đó, sử dụng các hàm xử lý byte (`memcpy`, `memmove`).
  * Số học con trỏ: đầu tiên ép kiểu sang mảng byte với (`char *`).

* Con trỏ hàm cho phép các hàm bậc cao trong C, ví dụ: map, filter, sắp xếp, v.v.

## Tài liệu đọc thêm

K&R 7.8.5, 8.7
