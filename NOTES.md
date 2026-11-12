# Ghi chú phân tích · Day 04

Họ và tên: Vũ Nam Khánh     
MSSV: 2A202604112

## 1. Case được chọn

- ID case: H7
- Câu trả lời của agent: Agent ghi nhận Cloudflare Spectrum bị timeout, nhưng sau đó kết luận dịch vụ "đang hoạt động bình thường".
- Điểm ở lần chạy 1: 65/100

## 2. Bằng chứng trong trace

```text
{"event":"tool","step":1,"name":"get_status","arguments":{"provider":"cloudflare","component":"Spectrum"},"result":{"ok":false,"error":"TimeoutError","message":"Cloudflare status timeout (eval giả lập)."}}
```

Nguyên nhân có thể xảy ra: agent đã diễn giải việc không có dữ liệu do timeout thành bằng chứng dịch vụ đang hoạt động. Trace không có kết quả status Cloudflare để hỗ trợ kết luận đó.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Trong `agent.py`, bổ sung chỉ dẫn rằng timeout/lỗi tool nghĩa là trạng thái chưa xác minh và không được suy ra dịch vụ đang operational. Thêm kiểm tra cuối để yêu cầu model bổ sung URL từ các incident detail thành công nếu câu trả lời bỏ sót nguồn.
- Vì sao chọn thay đổi này: Trace H7 cho thấy rõ kết luận operational xuất hiện sau `TimeoutError`, không có evidence hỗ trợ. Kiểm tra URL cũng hoàn thiện contract validation nguồn bắt buộc của lab.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 82.05/100 (22/22 case) | 76.95/100 (22/22 case), giảm 5.10 điểm |
| Điểm case đã chọn | H7: 65/100 | H7: 65/100 |
| Số model calls | 49 agent + 22 judge = 71 | 53 agent + 22 judge = 75 |
| Token, chi phí nếu có | ~78,500 tokens | ~95,000 tokens |

- Lỗi đã giải quyết: Trong lần chạy đầu, H7 kết luận Cloudflare operational sau timeout. Ở lần chạy sau, câu trả lời nói rõ trạng thái Cloudflare chưa được xác minh. Điểm judge vẫn là 65; lý do judge nói agent xác nhận Cloudflare operational, trái với câu trả lời thực tế, nên cần đối chiếu trace thay vì chỉ dựa vào điểm.
- Lỗi còn lại hoặc mới xuất hiện: H1 và H6 vẫn gọi tools trong một model step dù case yêu cầu ít nhất hai step; H2 thiếu `get_status`. Một số case có điểm thấp dù câu trả lời có nêu giới hạn dữ liệu, nên cần kiểm tra bằng chứng và trace thủ công.

## 5. Giới hạn

Điểm judge là ước lượng; H7 cho thấy judge có thể giải thích ngược với nội dung câu trả lời. Snapshot phản ánh dữ liệu đã lưu ngày 09/11/2026, không phải trạng thái live hiện tại. Mỗi phiên bản chỉ được chạy một lần, nên chênh lệch điểm có thể phản ánh độ biến động của model/judge thay vì tác động ổn định của thay đổi.
