# pk-kemis-contracts

Hợp đồng giữa các service của PkKemis và nguồn sự thật cho API. Kế hoạch: `../pk-kemis-web/docs/backend/PLAN.md` (mục 6) và `PHASE1.md`.

## Nội dung

| File | Service | Trạng thái |
|---|---|---|
| `openapi/identity.yaml` | identity | 0.5.0 đã cài (M1, M2, thao tác admin, nhật ký, M6 L7); 0.6.0 viết cho M7 (xác minh email, mã `last_admin`), chưa cài |
| `openapi/learning.yaml` | learning | 0.3.1 đã cài (M3, M6) và đối chiếu bằng `test/contract.spec.ts` ở learning; **1.0.0** viết cho M7 (phá vỡ có chủ đích: hạn chế đề với học sinh, gỡ snapshot nội bộ; thêm `GET /v1/content`, `POST /v1/exam-overlap`), chưa cài |
| `openapi/classroom.yaml` | classroom | 0.8.1 đã cài (M4, M6), đối chiếu bằng `test/contract.spec.ts` ở classroom; **1.0.0** viết cho M7 (phá vỡ có chủ đích: gỡ `/paper` và nộp một lần; thêm `GET /v1/classes/overview`), chưa cài |

Kế hoạch M6 và thứ tự các lát: `../pk-kemis-web/docs/backend/M6-CONTENT.md`. M7 (dọn nợ, email): `../pk-kemis-web/docs/backend/M7-DEBT-EMAIL.md`.

AsyncAPI cho sự kiện sẽ thêm khi có Kafka.

## Quy tắc

- **Chỉ thêm, không xóa hay đổi nghĩa** trường đã phát hành. Thay đổi phá vỡ cần chủ dự án đồng ý và ra phiên bản lớn (0.x → 1.0.0 → 2.0.0). Khi chỉ web dùng đường đó (service chưa mở ra ngoài, như M7) thì giữ `/v1` và đổi theo thứ tự: web thôi dùng trước, service gỡ sau. Khi đã có bên dùng khác web (gateway mở `/v1/**`, ứng dụng khác) thì ra đường mới (`/v2/...`) và giữ đường cũ một thời gian.
- Đổi hợp đồng **trước**, rồi mới đổi mã service và web.
- Lỗi theo RFC 9457 (`application/problem+json`), có mã `code` ổn định và `detail` tiếng Việt cho người dùng.
- Thao tác gắn `x-phase: 2` có trong hợp đồng để định hình, nhưng giai đoạn 1 chưa cài.
- Thao tác gắn `x-slice: Ln` thuộc lát n của M6, `x-slice: 7a`…`7d` thuộc lát của M7; trước khi lát đó xong, service chưa có thao tác này. `x-transitional` là thao tác tạm, ghi rõ khi nào gỡ.
- Thao tác `deprecated: true` vẫn chạy nhưng không dùng cho mã mới; gỡ hẳn là thay đổi phá vỡ, cần chủ dự án đồng ý và ra phiên bản lớn.

## Kiểm tra

```bash
npx --yes @redocly/cli@1 lint openapi/identity.yaml
```

Hiện còn các cảnh báo chấp nhận được: thiếu `license` ở cả ba file; với identity thêm `logout` và `jwks.json` không có đáp án 4xx. CI sẽ chạy lệnh này cho từng file và so với `main` để chặn thay đổi phá vỡ (oasdiff).
