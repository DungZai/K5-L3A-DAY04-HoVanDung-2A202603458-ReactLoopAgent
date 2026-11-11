# Ghi chú phân tích · Day 04

- Họ và tên: Hồ Văn Dũng
- MSSV:2A202603458

## 1. Case được chọn

- ID case: A5
- Câu trả lời của agent: Liệt kê các incident Actions gần đây thuộc tháng 11/2026 để trả lời yêu cầu tìm incident trong năm 2025, rồi nói không tìm thấy sự cố năm 2025.
- Điểm ở lần chạy 1: 35/100 (eval_before.json hiện có, 22/22 case).

## 2. Bằng chứng trong trace

Trace cho thấy agent dùng snapshot ngày 09/11/2026 và đọc chi tiết 5 incident trong tháng 11/2026 để trả lời câu hỏi về năm 2025. Trích đoạn dưới đây giữ nguyên các trường liên quan từ trace; phần `result` đã lược bớt.

```text
{
  "event": "tool",
  "step": 1,
  "name": "get_status",
  "arguments": {"provider": "github", "component": "Actions"},
  "result": {
    "captured_at": "2026-11-09T10:50:47.06502+07:00",
    "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố."
  }
}
{
  "event": "tool",
  "step": 2,
  "name": "get_incident_details",
  "arguments": {"provider": "github", "incident_id": "t89g2q1800fr"},
  "result": {
    "ok": true,
    "created_at": "2026-11-05T16:50:11.086Z",
    "url": "https://stspg.io/lv4q3xm9r0rk"
  }
}
{
  "event": "validation_missing_source",
  "step": 3,
  "urls": ["https://stspg.io/lv4q3xm9r0rk"]
}
```

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Bổ sung hướng dẫn trong SYSTEM_PROMPT của agent.py để đối chiếu incident với khoảng thời gian được hỏi, nêu giới hạn 50 incident khi dữ liệu không bao phủ khoảng đó, và không dùng incident ngoài khoảng để trả lời thay.
- Vì sao chọn thay đổi này: Trace A5 cho thấy agent dùng dữ liệu tháng 11/2026 cho câu hỏi về năm 2025 và chưa làm nổi bật giới hạn tìm kiếm.

## 4. So sánh hai lần chạy

| Chỉ số                | `eval_before.json`                                | `eval_after.json`                                 |
| --------------------- | ------------------------------------------------- | ------------------------------------------------- |
| Điểm tổng             | 80.00/100                                         | 78.73/100                                         |
| Điểm case đã chọn     | A5: 35/100                                        | A5: 60/100                                        |
| Số model calls        | 54                                                | 49                                                |
| Token, chi phí nếu có | 101,783 token; $0.021177 theo usage trong báo cáo | 78,395 token; $0.0170445 theo usage trong báo cáo |

- Lỗi đã giải quyết: Điểm A5 tăng 25 điểm. Câu trả lời sau cải tiến thừa nhận không thể xác nhận danh sách đầy đủ cho năm 2025 và nêu giới hạn 50 incident gần nhất.
- Lỗi còn lại hoặc mới xuất hiện: A5 vẫn nói “không có bất kỳ sự cố nào được ghi nhận trong năm 2025”, dù dữ liệu tìm kiếm không đủ để khẳng định điều đó. Điểm tổng giảm 1.27 điểm, nên cần đọc từng case vì biến động chấm của judge và các lỗi ở case khác có thể làm giảm trung bình.

## 5. Giới hạn

Điểm LLM judge là ước lượng và có thể khác nhau giữa các lần chạy. Snapshot được chụp ngày 09/11/2026, còn tool chỉ tìm trong 50 incident gần nhất nên không thể xác nhận lịch sử năm 2025 đầy đủ. Mỗi phiên bản chỉ chạy một lần, vì vậy mức tăng của A5 chưa chứng minh cải tiến ổn định; điểm tổng còn giảm từ 80.00 xuống 78.73.
