---
title: "Tổng kết"
---

## Và để Kết luận$\dots$

Để thực hiện pipeline, chúng ta phân tách datapath thành 5 giai đoạn rời rạc, mỗi giai đoạn hoàn thành một chức năng khác nhau và truy cập các tài nguyên khác nhau trên đường thực thi toàn bộ lệnh.
Nhớ lại năm giai đoạn:

* Trong giai đoạn IF, chúng ta sử dụng Program Counter để truy cập lệnh của mình khi nó được lưu trữ trong IMEM.
* Sau đó, chúng ta phân tách các phần riêng biệt cần thiết từ các bit lệnh trong giai đoạn ID và tạo immediate, các giá trị thanh ghi từ RegFile, và các tín hiệu điều khiển khác.
* Tiếp theo, sử dụng các giá trị và tín hiệu này, chúng ta hoàn thành các phép toán ALU cần thiết trong giai đoạn EX.
* Kế đến, bất cứ điều gì chúng ta làm liên quan đến DMEM (không nhầm lẫn với RegFile hoặc IMEM) được thực hiện trong giai đoạn MEM,
* Cuối cùng, chúng ta đến giai đoạn WB, nơi chúng ta ghi giá trị được tính mà chúng ta muốn trở lại vào thanh ghi trả về trong RegFile.

5 giai đoạn này, được phân chia bởi thanh ghi, cho phép vận hành các giai đoạn khác nhau của datapath trong cùng chu kỳ xung clock.

* Các lệnh khác nhau có thể sử dụng các giai đoạn khác nhau cùng một lúc. Tại mỗi chu kỳ xung clock, các đầu vào cần thiết vào một giai đoạn cụ thể được lấy mẫu tại cạnh lên xung clock (và có sẵn sau độ trễ clk-to-q).
* Sau khi giai đoạn hoạt động trên các đầu vào, các đầu ra tương ứng được đưa vào **thanh ghi pipeline** cho giai đoạn tiếp theo. Thanh ghi pipeline cũng có thể được yêu cầu để truyền thông tin có thể không cần thiết cho giai đoạn ngay sau, nhưng một số giai đoạn tương lai.

:::{embed} #fig-five-stage-summary
:::

ISA RISC-V được thiết kế cho pipelining:

* Tất cả lệnh đều rộng 32 bit.
    * Dễ dàng fetch (`IF`) và decode (`ID`), mỗi thao tác trong một chu kỳ xung clock.
    * Đối lập với CISC (Complex) x86: lệnh rộng từ 1 đến 15 byte!
* Một tập nhỏ các định dạng lệnh tiêu chuẩn.
    * Có thể decode/đọc thanh ghi trong một giai đoạn (thứ 2, `ID`).
* Địa chỉ load/store về mặt khái niệm:
    * Tính địa chỉ trong giai đoạn thứ 3 (`EX`) với ALU; và
    * Truy cập bộ nhớ trong một giai đoạn (thứ 4, `MEM`).
    * Các toán hạng bộ nhớ đều được căn chỉnh
    * Truy cập bộ nhớ chỉ mất một chu kỳ.

## Tài liệu Đọc thêm

P&H 4.6, 4.7, 4.8

## Tài liệu Tham khảo Bổ sung

## Bài tập
Kiểm tra kiến thức của bạn!

### Bài tập Ngắn

:::{exercise}
:label: pl-01-vi
1. **Đúng/Sai**: Bằng cách pipeline datapath CPU, mỗi lệnh đơn sẽ thực thi nhanh hơn vì pipelining giảm độ trễ mỗi lệnh (dẫn đến tăng tốc hiệu năng).
:::

:::{solution} pl-01-vi
:label: pl-01-sol-vi
:class: dropdown
**Sai.** Vì chúng ta chèn thanh ghi giữa mỗi giai đoạn trong datapath, thời gian để một lệnh hoàn thành thực thi qua 5 giai đoạn sẽ dài hơn datapath single-cycle. Một lệnh đơn sẽ mất nhiều chu kỳ xung clock để đi qua tất cả các giai đoạn, với chu kỳ xung clock dựa trên đường tới hạn (giai đoạn có thời gian dài nhất).
::: 

:::{exercise}
:label: pl-02-vi
2. **Đúng/Sai**: Datapath CPU pipeline dẫn đến việc thực thi lệnh với thông lượng cao hơn so với CPU single-cycle.
:::

:::{solution} pl-02-vi
:label: pl-02-sol-vi
:class: dropdown
**Đúng.** Nhớ lại rằng thông lượng là số lệnh được xử lý trên đơn vị thời gian. Pipelining dẫn đến thông lượng cao hơn vì nhiều lệnh có thể ở giai đoạn khác nhau của datapath cùng một lúc.
:::
