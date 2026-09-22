---
title: "Tổng kết"
---

## Và để kết luận$\dots$

## Bài tập
Kiểm tra kiến thức của bạn!

### Bài tập ngắn

:::{exercise}
:label: c-odds-01
1. Điền nội dung bộ nhớ cho mỗi hệ thống sau khi khởi tạo `arr`. Giả sử `arr` bắt đầu tại địa chỉ bộ nhớ `0x1000`.

```
uint32_t arr[2] = {0xD3ADB33F, 0x61C0FFEE};
```

**(a)** Hệ thống Little-Endian

|        | +0    | +1    | +2    | +3    |
|--------|-------|-------|-------|-------|
|        | ...                   |
| `0x1000` |       |       |       |       |
| `0x1004` |       |       |       |       |
|        | ...                   |

**(b)** Hệ thống Big-Endian

|        | +0    | +1    | +2    | +3    |
|--------|-------|-------|-------|-------|
|        | ...                   |
| `0x1000` |       |       |       |       |
| `0x1004` |       |       |       |       |
|        | ...                   |
:::

:::{solution} c-odds-01
:label: c-odds-01-sol
:class: dropdown

**(a)** Hệ thống Little-Endian

|        | +0    | +1    | +2    | +3    |
|--------|-------|-------|-------|-------|
|        | ...                   |
| `0x1000` | `0x3F` | `0xB3` | `0xAD` | `0xD3` |
| `0x1004` | `0xEE` | `0xFF` | `0xC0` | `0x61` |
|        | ...                   |

**(b)** Hệ thống Big-Endian

|        | +0    | +1    | +2    | +3    |
|--------|-------|-------|-------|-------|
|        | ...                   |
| `0x1000` | `0xD3` | `0xAD` | `0xB3` | `0x3F` |
| `0x1004` | `0x61` | `0xC0` | `0xFF` | `0xEE` |
|        | ...                   |

:::
