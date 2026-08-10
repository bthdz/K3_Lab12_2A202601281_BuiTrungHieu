# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Bùi Trung Hiếu |
| Mã học viên | 2A202601281 |
| Repo | https://github.com/buitrunghieu/DAY12-Agent |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | http://localhost:8000 |
| Platform | Railway / Render / Docker Compose (Local Fallback) |
| Ngày deploy | 2026-03-30 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | system port |
| `AGENT_API_KEY` | ✅ | đặt trong .env |
| `REDIS_URL` | ✅ | redis://redis:6379/0 |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

```bash
curl -i http://localhost:8000/health
curl -i http://localhost:8000/ready
```

## Kết Quả Chạy Thật

```json
{"status":"ok","service":"day12-agent","version":"1.0.0"}
{"status":"ready","redis":true}
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png`

---

## Phương Án Dự Phòng

Sử dụng Docker Compose chạy Local Fallback do chưa tạo được tài khoản Cloud.
