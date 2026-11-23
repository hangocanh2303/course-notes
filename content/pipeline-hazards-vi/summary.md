---
title: "Tổng kết"
---

## Và để Kết luận$\dots$

Một trong những chi phí của pipelining là nó đưa vào pipeline hazard. **Hazard**, nói chung, là các vấn đề với thứ gì đó trong pipeline lệnh của CPU có thể khiến lệnh tiếp theo thực thi không chính xác.

Chúng ta thảo luận ba loại hazard:

* Structural hazard (phần cứng không đủ)
* Data hazard (sử dụng giá trị lỗi thời trong tính toán)
* Control hazard (thực thi các lệnh sai).

## Tài liệu Đọc thêm

P&H 4.8, 4.10

## Tài liệu Tham khảo Bổ sung

## Bài tập

### Bài tập Ngắn

:::{exercise}
:label: haz-01-vi
1. **Đúng/Sai**: Có hai cổng 'read' cũng như một cổng 'write' đến Register File giải quyết hazard của hai lệnh đọc và ghi vào cùng thanh ghi đồng thời.
:::

:::{solution} haz-01-vi
:label: haz-01-sol-vi
:class: dropdown
**Sai.** Việc bổ sung các cổng độc lập vào RegFile cho phép nhiều lệnh truy cập RegFile cùng lúc (chẳng hạn như một lệnh đọc giá trị của hai toán hạng, trong khi lệnh khác đang ghi vào thanh ghi trả về). Tuy nhiên, điều này không hoạt động nếu hai lệnh đang đọc và ghi vào cùng thanh ghi cùng lúc. Một số giải pháp cho data hazard này có thể là stall lệnh sau hoặc forward giá trị đọc từ lệnh trước, bypass hoàn toàn RegFile.
::: 

:::{exercise}
:label: haz-02-vi
2. **Đúng/Sai**: Không có forwarding hoặc double-pumping, data hazard thường sẽ dẫn đến 3 stall.
:::

:::{solution} haz-02-vi
:label: haz-02-sol-vi
:class: dropdown
**Đúng.** Lệnh tiếp theo phải chờ lệnh trước hoàn thành EX, MEM, và WB, trước khi nó có thể bắt đầu EX của mình.
::: 

:::{exercise}
:label: haz-03-vi
3. **Đúng/Sai**: Tất cả data hazard có thể được giải quyết với forwarding.
:::

:::{solution} haz-03-vi
:label: haz-03-sol-vi
:class: dropdown
**Sai.** Hazard sau lw không thể được giải quyết hoàn toàn với forwarding vì đầu ra không được biết cho đến sau giai đoạn MEM. Chúng ta vẫn cần stall ngoài forwarding từ thanh ghi pipeline MEM/WB đến giai đoạn EX.
::: 

:::{exercise}
:label: haz-04-vi
4. **Đúng/Sai**: Stalling là cách duy nhất để giải quyết control hazard.
:::

:::{solution} haz-04-vi
:label: haz-04-sol-vi
:class: dropdown
**Sai.** Có các kỹ thuật tiên tiến hơn như dự đoán branch, dự đoán con đường nào branch sẽ đi và flush pipeline nếu dự đoán sai.
:::
