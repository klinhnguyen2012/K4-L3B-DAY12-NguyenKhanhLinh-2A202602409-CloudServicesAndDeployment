# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Khánh Linh |
| Mã học viên | 2A202602409 |
| Repo | https://github.com/klinhnguyen2012/K4-L3B-DAY12-NguyenKhanhLinh-2A202602409-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-ug19.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-09-30 |

## Biến Môi Trường Đã Set Trên Cloud

Chỉ ghi tên biến và nguồn giá trị, không ghi giá trị secret.

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Render tự gán |
| `AGENT_API_KEY` | ✅ | Được cấu hình trong Render Environment |
| `REDIS_URL` | ✅ | Internal connection string của Render Key Value `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | ✅ | Giá trị 10 |
| `MONTHLY_BUDGET_USD` | ✅ | Giá trị 10.0 |
| `LOG_LEVEL` | ✅ | Giá trị INFO |

## Lệnh Kiểm Tra

Đã chạy ngày 2026-09-30:

```bash
curl -i https://day12-agent-ug19.onrender.com/health
# HTTP/2 200
# {"status":"ok","service":"day12-agent","version":"1.0.0"}

curl -i https://day12-agent-ug19.onrender.com/ready
# HTTP/2 200
# {"status":"ready","redis":true}

curl -i -X POST https://day12-agent-ug19.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
# HTTP/2 401
# {"detail":"invalid or missing API key"}
```

Request `/ask` có API key giả chứa dấu đã trả `401` sau bản sửa CP3, thay vì `500` trước đó. Request với key Render hợp lệ đã trả `200`:

```bash
curl -i -X POST https://day12-agent-ug19.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $DEPLOY_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'
```

Không ghi giá trị API key vào tài liệu hoặc repo.

Kết quả kiểm tra có key hợp lệ ngày 2026-09-30:

```text
HTTP/2 200
{"answer":"Ngắn gọn: Docker là gì phụ thuộc vào ba yếu tố — cấu hình qua biến môi trường, health check để orchestrator biết trạng thái, và giới hạn tài nguyên.","user_id":"sv01","history_length":0,"cost_usd":2.265e-05,"tokens":{"in":3,"out":37}}
```

`pytest tests/test_cp5.py -q`: `9 passed, 4 skipped`.

## Ảnh Chụp Màn Hình

Thêm hai ảnh vào thư mục `screenshots/` trước khi nộp:

- `screenshots/dashboard.png` — Render dashboard, hiển thị service `day12-agent` ở trạng thái Live.
- `screenshots/health.png` — terminal hoặc trình duyệt hiển thị kết quả `/health` và `/ready`.

## Phương Án Dự Phòng

Không dùng phương án dự phòng; service đã được deploy công khai trên Render.
