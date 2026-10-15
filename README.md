# pk-kemis-contracts

Hợp đồng giữa các service của PkKemis và nguồn sự thật cho API. Kế hoạch: `../pk-kemis-web/docs/backend/PLAN.md` (mục 6) và `PHASE1.md`.

## Nội dung

| File | Service | Trạng thái |
|---|---|---|
| `openapi/identity.yaml` | identity | 0.6.0 đã cài (M1, M2, thao tác admin, nhật ký, M6 L7, M7: xác minh email, mã `last_admin`); **0.7.0** cho M8 (`UserSummary.version`), đã cài |
| `openapi/learning.yaml` | learning | 1.0.0 đã cài (M3, M6, M7: hạn chế đề với học sinh, gỡ snapshot nội bộ, `GET /v1/content`, `POST /v1/exam-overlap`), đối chiếu bằng `test/contract.spec.ts` ở learning |
| `openapi/classroom.yaml` | classroom | 1.0.0 đã cài (M4, M6, M7: gỡ `/paper` và nộp một lần, `GET /v1/classes/overview`), đối chiếu bằng `test/contract.spec.ts` ở classroom |
| `asyncapi/pkkemis.yaml` | mọi service (sự kiện trên Kafka) | **0.1.0** cho M8: phong bì CloudEvents, topic `identity.events` (`user.*`), `learning.events` (`xp.awarded`), DLQ; đã cài |

Kế hoạch M6 và thứ tự các lát: `../pk-kemis-web/docs/backend/M6-CONTENT.md`. M7 (dọn nợ, email): `../pk-kemis-web/docs/backend/M7-DEBT-EMAIL.md`. M8 (Kafka, outbox, sự kiện): `../pk-kemis-web/docs/backend/M8-EVENTS.md`.

## Quy tắc

- **Chỉ thêm, không xóa hay đổi nghĩa** trường đã phát hành. Thay đổi phá vỡ cần chủ dự án đồng ý và ra phiên bản lớn (0.x → 1.0.0 → 2.0.0). Khi chỉ web dùng đường đó (service chưa mở ra ngoài, như M7) thì giữ `/v1` và đổi theo thứ tự: web thôi dùng trước, service gỡ sau. Khi đã có bên dùng khác web (gateway mở `/v1/**`, ứng dụng khác) thì ra đường mới (`/v2/...`) và giữ đường cũ một thời gian.
- Đổi hợp đồng **trước**, rồi mới đổi mã service và web.
- Lỗi theo RFC 9457 (`application/problem+json`), có mã `code` ổn định và `detail` tiếng Việt cho người dùng.
- Thao tác gắn `x-phase: 2` có trong hợp đồng để định hình, nhưng giai đoạn 1 chưa cài.
- Thao tác gắn `x-slice: Ln` thuộc lát n của M6, `x-slice: 7a`…`7d` thuộc lát của M7, `x-slice: 8a`…`8f` thuộc lát của M8; trước khi lát đó xong, service chưa có thao tác hay sự kiện này. `x-transitional` là thao tác tạm, ghi rõ khi nào gỡ.
- Thao tác `deprecated: true` vẫn chạy nhưng không dùng cho mã mới; gỡ hẳn là thay đổi phá vỡ, cần chủ dự án đồng ý và ra phiên bản lớn.

### Sự kiện (AsyncAPI)

- AsyncAPI 3.0: mỗi topic là một kênh (`channels`), ai gửi ai nhận ghi ở `operations` (`x-service`); mỗi tin nhắn ghi bên phát (`x-producer`), bên nghe (`x-consumers`) và lát cài (`x-slice`).
- Mọi tin là một CloudEvent 1.0 dạng structured (JSON), khóa bản ghi Kafka là `subject` (mã người dùng). Mỗi service chỉ ghi topic `<service>.events` của mình; DLQ là `<topic>.dlq`.
- Cùng quy tắc chỉ thêm: thêm trường, thêm giá trị enum, thêm loại sự kiện, thêm topic. Thay đổi phá vỡ là loại mới `<tên>.v2` phát song song với loại cũ.
- Schema `data` đặt `additionalProperties: false`: test hợp đồng của bên phát bắt được trường ngoài hợp đồng (ví dụ email lỡ lọt vào). Bên nghe thì khoan dung: bỏ qua trường lạ và loại lạ.
- Sự kiện không bao giờ chứa email, ngày sinh, mật khẩu, token, IP, đáp án; không có chữ theo chế độ hiển thị.

## Kiểm tra

```bash
npx --yes @redocly/cli@1 lint openapi/identity.yaml
npx --yes @redocly/cli@1 lint asyncapi/pkkemis.yaml
```

Hiện còn các cảnh báo chấp nhận được: thiếu `license` ở ba file OpenAPI; với identity thêm `logout` và `jwks.json` không có đáp án 4xx. `asyncapi/pkkemis.yaml` không có cảnh báo. Redocly chỉ kiểm cấu trúc; service đối chiếu sự kiện thật với schema bằng test hợp đồng (như `test/contract.spec.ts` với OpenAPI), lấy ví dụ trong `examples` của từng tin nhắn làm dữ liệu cho test của bên nghe. CI sẽ chạy các lệnh này cho từng file và so với `main` để chặn thay đổi phá vỡ (oasdiff cho OpenAPI).
