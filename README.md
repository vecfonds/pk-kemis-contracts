# pk-kemis-contracts

Hợp đồng giữa các service của PkKemis và nguồn sự thật cho API. Kế hoạch: `../pk-kemis-web/docs/backend/PLAN.md` (mục 6) và `PHASE1.md`.

## Nội dung

| File | Service | Trạng thái |
|---|---|---|
| `openapi/identity.yaml` | identity | bản nháp 0.1.0, chờ chủ dự án duyệt (M1) |
| `openapi/learning.yaml` | learning | chưa viết (M3) |
| `openapi/classroom.yaml` | classroom | chưa viết (M4) |

AsyncAPI cho sự kiện sẽ thêm khi có Kafka.

## Quy tắc

- **Chỉ thêm, không xóa hay đổi nghĩa** trường đã phát hành. Thay đổi phá vỡ phải ra phiên bản mới (`/v2`).
- Đổi hợp đồng **trước**, rồi mới đổi mã service và web.
- Lỗi theo RFC 9457 (`application/problem+json`), có mã `code` ổn định và `detail` tiếng Việt cho người dùng.
- Thao tác gắn `x-phase: 2` có trong hợp đồng để định hình, nhưng giai đoạn 1 chưa cài.

## Kiểm tra

```bash
npx --yes @redocly/cli@1 lint openapi/identity.yaml
```

Hiện còn 3 cảnh báo chấp nhận được (thiếu `license`; `logout` và `jwks.json` không có đáp án 4xx). CI sẽ chạy lệnh này và so với `main` để chặn thay đổi phá vỡ (oasdiff).
