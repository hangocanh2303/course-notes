---
title: "Tổng kết"
---

## Và để Kết luận$\dots$

::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/1WTP44UAtp4
:width: 100%
:title: "[CS61C FA20] Lecture 19.7 - Single-Cycle CPU Datapath II: Summary"
:::

::::


::::{note} 🎥 Video Bài giảng
:class: dropdown

:::{iframe} https://www.youtube.com/embed/qgGy_Ra9hr0
:width: 100%
:title: "[CS61C FA20] Lecture 20.5 - Single-Cycle CPU Control: Summary"
:::

::::


**Datapath single-cycle** của chúng ta là một hệ thống số đồng bộ có khả năng thực thi các lệnh RISC-V trong **mỗi một chu kỳ**. Nó được chia thành nhiều giai đoạn thực thi, trong đó mỗi giai đoạn chịu trách nhiệm hoàn thành một nhiệm vụ nhất định.

1. **IF** Instruction Fetch:
    * Gửi địa chỉ đến bộ nhớ lệnh (IMEM), và đọc IMEM tại địa chỉ đó.
    * Các đơn vị phần cứng: Thanh ghi PC, bộ cộng +4, mux PCSel, IMEM
1. **ID** Instruction Decode:
    * Tạo các tín hiệu điều khiển từ các bit lệnh, tạo immediate, và đọc thanh ghi từ RegFile.
    * Các đơn vị phần cứng: RegFile, ImmGen
1. **EX** Execute:
    * Thực hiện các phép toán ALU, và thực hiện so sánh branch.
    * Các đơn vị phần cứng: mux ASel, mux BSel, branch comparator, ALU
1. **MEM** Memory
    * Đọc từ hoặc ghi vào bộ nhớ dữ liệu (DMEM).
    * Các đơn vị phần cứng: DMEM
1. **WB** Writeback
    * Ghi lại PC + 4, kết quả của phép toán ALU, hoặc dữ liệu từ bộ nhớ vào RegFile.
    * Các đơn vị phần cứng: mux WBSel, RegFile

**Đường tới hạn** thay đổi dựa trên lệnh. Không phải tất cả các lệnh sử dụng tất cả các đơn vị phần cứng, và do đó không phải tất cả các lệnh hoạt động trong cả năm giai đoạn thực thi ("stages" là thuật ngữ chúng ta sử dụng cho các bộ xử lý pipeline).

**Bộ điều khiển** (ví dụ, mạch con logic điều khiển) chỉ định cách thực thi các lệnh và nó được triển khai như ROM (bộ nhớ chỉ đọc) hoặc như các cổng logic.

## Tài liệu Đọc thêm

P&H 4.1, 4.3, 4.4, 4.5

## Bài tập

:::{exercise}
:label: dp-01-vi
1. **Đúng/Sai**: Nếu độ trễ logic của việc đọc từ IMEM được giảm, thì bất kỳ chương trình (không rỗng) nào sử dụng datapath single cycle sẽ nhanh hơn.
:::

:::{solution} dp-01-vi
:label: dp-01-sol-vi
:class: dropdown
**Đúng** Vì mọi lệnh phải đọc từ IMEM trong giai đoạn instruction fetch, làm IMEM nhanh hơn sẽ tăng tốc mọi lệnh đơn lẻ.
:::

:::{exercise}
:label: dp-02-vi
2. **Đúng/Sai**: Có thể đưa cả đầu ra của immediate generator và giá trị trong rs2 vào ALU trong một lệnh đơn.
:::

:::{solution} dp-02-vi
:label: dp-02-sol-vi
:class: dropdown
**Sai** Bạn chỉ có thể sử dụng hoặc immediate generator hoặc giá trị trong thanh ghi rs2. Lưu ý trong datapath của chúng ta, có một mux với tín hiệu (BSel) quyết định liệu chúng ta sử dụng đầu ra của immediate generator hay giá trị trong rs2.
:::

:::{exercise}
:label: dp-03-vi
3. **Đúng/Sai**: Datapath single cycle sử dụng đầu ra của tất cả các đơn vị phần cứng cho mỗi lệnh.
:::

:::{solution} dp-03-vi
:label: dp-03-sol-vi
:class: dropdown
**Sai** Tất cả các đơn vị hoạt động trong mỗi chu kỳ, nhưng đầu ra của chúng có thể bị bỏ qua (gated) bởi các tín hiệu điều khiển.
::: 

:::{exercise}
:label: dp-04-vi
4. **Đúng/Sai**: Có thể thực thi các giai đoạn của datapath single cycle song song để tăng tốc thực thi một lệnh đơn.
:::

:::{solution} dp-04-vi
:label: dp-04-sol-vi
:class: dropdown
**Sai** Mỗi giai đoạn phụ thuộc vào giá trị được tạo ra bởi giai đoạn trước nó (ví dụ, instruction decode phụ thuộc vào lệnh được fetch).
:::

```{exercise}
:label: dp-05-vi
5. **Đúng/Sai**: Store và load là các lệnh duy nhất yêu cầu đầu vào/đầu ra từ DMEM.
```

:::{solution} dp-05-vi
:label: dp-05-sol-vi
:class: dropdown
**Đúng** Đối với tất cả các lệnh khác, chúng ta không cần đọc dữ liệu được đọc ra từ DMEM, và do đó không cần chờ đầu ra của giai đoạn MEM.
:::
