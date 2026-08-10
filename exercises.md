# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng mẫu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Bùi Trung Hiếu  Mã học viên: 2A202601281

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Nếu để mặc định là "changeme", khi deploy ứng dụng lên môi trường production mà quên set biến môi trường AGENT_API_KEY, ứng dụng vẫn sẽ khởi động thành công. Kẻ tấn công có thể dễ dàng đoán ra khóa mặc định "changeme" này và tự do gọi vào hệ thống của bạn, dẫn tới rò rỉ dữ liệu hoặc làm cạn kiệt ngân sách LLM. Ngược lại, việc ứng dụng crash ngay khi thiếu biến giúp phát hiện lỗi lập tức ở khâu deploy/CI-CD.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

```json
{"timestamp": "2026-03-30T09:30:00Z", "level": "INFO", "event": "ask_completed", "user_id": "sv-123", "tokens_in": 15, "tokens_out": 45, "cost_usd": 0.00012}
```
1. Cho phép các công cụ Centralized Logging (Elasticsearch, Datadog, Grafana Loki) tự động parse và lọc dữ liệu theo trường như `user_id`, `cost_usd`.
2. Hỗ trợ tạo Alert và Dashboard đo lường chi phí, lượng token tiêu thụ chính xác theo thời gian thực thay vì chỉ hiển thị dòng chữ thuần không có cấu trúc.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1020 MB |
| Multi-stage | 270 MB |

> *Phần dung lượng chênh lệch (~750MB) chính là các công cụ build/compile trong Linux base image đầy đủ, pip build cache, wheel cache và các file tạm thời phục vụ cài đặt dependency không cần thiết ở môi trường runtime.*

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Khi sửa `app/main.py`, các layer cài đặt dependency (`RUN pip install`) được dùng lại hoàn toàn từ cache. Chỉ các layer từ `COPY app/ app/` trở đi mới phải thực hiện lại. Nếu đặt `COPY . .` lên trước `RUN pip install`, bất kỳ sự thay đổi code nhỏ nào cũng làm hỏng cache của layer `COPY . .`, buộc Docker phải chạy lại bước `RUN pip install` tốn rất nhiều thời gian.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Kẻ tấn công khai thác lỗ hổng Remote Code Execution (RCE) trong code Python để thực thi lệnh shell trong container. Vì container chạy quyền root, kẻ tấn công sẽ thoát khỏi container cách dùng kĩ thuật container escape. Khi ra tới máy host, họ lập tức có quyền root trên máy host. Lệnh `USER appuser` cắt đứt chuỗi này ngay trong container: dù khai thác được lỗ hổng Python, kẻ tấn công chỉ có quyền của user thường không có đặc quyền.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

Tối đa 20 request. Người dùng có thể gửi 10 request ở 1 giây cuối cùng của phút này (ví dụ 10:00:59), sau đó hệ thống reset đếm về 0 khi sang phút mới (10:01:00) và họ gửi tiếp 10 request ở 1 giây đầu tiên của phút tiếp theo (10:01:00). Tổng cộng 20 request trong 2 giây.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

Rate limit quản lý số lượng request theo thời gian, còn Cost guard quản lý tổng chi phí tài chính tiêu thụ.
- Rate limit cho qua nhưng Cost guard chặn: User gửi 1 request/phút (dưới rate limit) nhưng request đó xử lý prompt cực lớn làm chi phí vượt budget tháng.
- Cost guard cho qua nhưng Rate limit chặn: User mới dùng lần đầu trong tháng (ngân sách còn nguyên) nhưng gửi 100 request liên tục trong 1 giây (vượt rate limit).

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

1. Health check thấy Redis chết nên báo lỗi 500/503.
2. Orchestrator đánh giá cả 3 container đều không "healthy".
3. Orchestrator lập tức kill và khởi động lại cả 3 container.
4. Do Redis vẫn chưa sống lại trong 30s, các container mới khởi động lên lại bị báo un-healthy và tiếp tục bị restart liên tục (CrashLoopBackOff).

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Con số `history_length` sẽ nhảy thất thường (ví dụ: 0 -> 0 -> 2 -> 0 -> 2...) do mỗi request được Load Balancer điều phối ngẫu nhiên tới 1 trong 3 container. Mỗi container giữ 1 dict RAM riêng nên không thấy lịch sử hội thoại của nhau.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Lỗi thường gặp là Health check timeout do service khởi động chậm hơn khoảng thời gian chờ mặc định hoặc app cố định port 8000 thay vì đọc biến môi trường `$PORT` do cloud gán động. Cách sửa là đọc biến `$PORT` qua CMD shell form `sh -c "exec uvicorn app.main:app --host 0.0.0.0 --port ${PORT:-8000}"`.

