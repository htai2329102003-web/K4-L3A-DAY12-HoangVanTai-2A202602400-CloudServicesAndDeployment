# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Hoàng Văn Tài |
| Mã học viên | 2A202602400 |
| Repo | https://github.com/htai2329102003-web/K4-L3A-DAY12-HoangVanTai-2A202602400-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-l3a-day12-hoangvantai-2a202602400-cloudservic-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-28 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis đã kết nối; `/ready` trả `redis: true` |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Các lệnh kiểm tra service đã deploy:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://k4-l3a-day12-hoangvantai-2a202602400-cloudservic-production.up.railway.app/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://k4-l3a-day12-hoangvantai-2a202602400-cloudservic-production.up.railway.app/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://k4-l3a-day12-hoangvantai-2a202602400-cloudservic-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://k4-l3a-day12-hoangvantai-2a202602400-cloudservic-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test-20260928" \
  -d '{"question":"Deploy is what?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://k4-l3a-day12-hoangvantai-2a202602400-cloudservic-production.up.railway.app/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test-20260928" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
GET /health
HTTP 200
{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready
HTTP 200
{"status":"ready","redis":true}

POST /ask without X-API-Key
HTTP 401
{"detail":"invalid or missing API key"}

POST /ask with X-API-Key from .env, X-User-Id: sv-test-20260928,
body {"question":"Deploy is what?"}
HTTP 200
{"answer":"Theo m\u00ecnh hi\u1ec3u, Deploy is what li\u00ean quan t\u1edbi c\u00e1ch h\u1ec7 th\u1ed1ng \u0111\u01b0\u1ee3c \u0111\u00f3ng g\u00f3i v\u00e0 v\u1eadn h\u00e0nh. \u0110i\u1ec3m m\u1ea5u ch\u1ed1t l\u00e0 t\u00e1ch c\u1ea5u h\u00ecnh ra kh\u1ecfi code v\u00e0 gi\u1eef service \u1edf tr\u1ea1ng th\u00e1i stateless.","user_id":"sv-test-20260928","history_length":0,"cost_usd":2.565e-05,"tokens":{"in":3,"out":42}}

15 rate-limit requests (HTTP status, in order)
200 200 200 200 200 200 200 200 200 429 429 429 429 429 429
429 response: {"detail":"rate limit exceeded"}
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- [screenshots/dashboard.png](screenshots/dashboard.png) — trang quản lý service trên platform
- [screenshots/health.png](screenshots/health.png) — kết quả gọi `/health` từ trình duyệt hoặc curl

---
