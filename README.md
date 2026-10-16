# pk-kemis-contracts

Hợp đồng giữa các service của PkKemis và nguồn sự thật cho API. Kế hoạch: `../pk-kemis-web/docs/backend/PLAN.md` (mục 6) và `PHASE1.md`.

## Nội dung

| File | Service | Trạng thái |
|---|---|---|
| `openapi/identity.yaml` | identity | 0.7.0 đã cài (M1, M2, thao tác admin, nhật ký, M6 L7, M7: xác minh email, mã `last_admin`; M8: `UserSummary.version`); **0.8.0** cho M10 (`UserSummary.createdAt`, `GET /internal/users` theo trang cho social), chưa cài |
| `openapi/learning.yaml` | learning | 1.1.0 đã cài (M3, M6, M7: hạn chế đề với học sinh, gỡ snapshot nội bộ, `GET /v1/content`, `POST /v1/exam-overlap`; M9: `LeaderboardEntry.frameId`), đối chiếu bằng `test/contract.spec.ts` ở learning; **1.2.0** cho M10 (`GET /v1/me/progress`, thành tích `GET /v1/achievements`, `/v1/me/achievements`, bốn đường nội bộ chỉ cho social với khóa riêng), chưa cài |
| `openapi/classroom.yaml` | classroom | 1.1.0 đã cài (M4, M6, M7: gỡ `/paper` và nộp một lần, `GET /v1/classes/overview`; M9: mua thêm lượt làm bài), đối chiếu bằng `test/contract.spec.ts` ở classroom; **1.2.0** cho M10 (phát `classroom.events`, `GET /internal/class-members` cho social), chưa cài |
| `openapi/commerce.yaml` | commerce (Spring Boot) | 0.1.0 đã cài (M9: ví và sổ cái, cửa hàng, nạp qua cổng mô phỏng, thu Xu cho classroom, admin điều chỉnh ví, đối soát); **0.2.0** cho M10 (social đọc `/internal/frames/equipped` bằng khóa riêng), chưa cài |
| `openapi/social.yaml` | social (mới, NestJS) | **0.1.0** cho M10: bạn bè, lời mời, cổ vũ, chặn, quyền riêng tư của trang cá nhân, trang cá nhân người khác đã lọc quyền, bảng xếp hạng bạn bè, "Bạn bè gần đây", thông báo (hỏi lại 60 giây), thông báo chung của admin; người được chỉ bằng mã hồ sơ (`handle`), không bằng mã tài khoản; chưa cài |
| `asyncapi/pkkemis.yaml` | mọi service (sự kiện trên Kafka) | 0.2.0 đã cài (M8: phong bì CloudEvents, `identity.events`, `learning.events`, DLQ; M9: `commerce.events`, SASL/SCRAM và ACL `x-acl`); **0.3.0** cho M10 (social nghe `identity.events`, `learning.events`, `commerce.events`, `classroom.events`; learning thêm `level.changed`, `achievement.unlocked`, `streak.milestone`, `chapter.completed`; topic mới `classroom.events` khóa theo lớp và `social.events`; `user.*` thêm `createdAt`), chưa cài |

Kế hoạch M6 và thứ tự các lát: `../pk-kemis-web/docs/backend/M6-CONTENT.md`. M7 (dọn nợ, email): `../pk-kemis-web/docs/backend/M7-DEBT-EMAIL.md`. M8 (Kafka, outbox, sự kiện): `../pk-kemis-web/docs/backend/M8-EVENTS.md`. M9 (commerce, ví, mua thêm lượt): `../pk-kemis-web/docs/backend/M9-COMMERCE.md`. M10 (social: bạn bè, quyền riêng tư, thông báo, thành tích): `../pk-kemis-web/docs/backend/M10-SOCIAL.md`.

## Quy tắc

- **Chỉ thêm, không xóa hay đổi nghĩa** trường đã phát hành. Thay đổi phá vỡ cần chủ dự án đồng ý và ra phiên bản lớn (0.x → 1.0.0 → 2.0.0). Khi chỉ web dùng đường đó (service chưa mở ra ngoài, như M7) thì giữ `/v1` và đổi theo thứ tự: web thôi dùng trước, service gỡ sau. Khi đã có bên dùng khác web (gateway mở `/v1/**`, ứng dụng khác) thì ra đường mới (`/v2/...`) và giữ đường cũ một thời gian.
- Đổi hợp đồng **trước**, rồi mới đổi mã service và web.
- Lỗi theo RFC 9457 (`application/problem+json`), có mã `code` ổn định và `detail` tiếng Việt cho người dùng.
- Thao tác gắn `x-phase: 2` có trong hợp đồng để định hình, nhưng giai đoạn 1 chưa cài.
- Thao tác gắn `x-slice: Ln` thuộc lát n của M6, `x-slice: 7a`…`7d` thuộc lát của M7, `x-slice: 8a`…`8f` thuộc lát của M8, `x-slice: 9a`…`9h` thuộc lát của M9, `x-slice: 10a`…`10i` thuộc lát của M10; trước khi lát đó xong, service chưa có thao tác hay sự kiện này. `x-transitional` là thao tác tạm, ghi rõ khi nào gỡ.
- `x-callers` (đường `/internal/**`): service nào được gọi, mỗi bên một khóa riêng. `x-local-only`: chỉ có khi bật bằng biến ở máy local (cổng thanh toán mô phỏng), không bao giờ bật trên máy chủ thật.
- **Đường có tiền** (commerce, mua thêm lượt ở classroom): header `Idempotency-Key` bắt buộc; giá do máy chủ giữ, máy khách chỉ gửi ý định và có thể gửi kèm giá đã thấy (`expectedPrice`) để bị từ chối thay vì bị trừ giá khác; service gọi nhau chống lặp bằng `reference` duy nhất mãi mãi.
- Thao tác `deprecated: true` vẫn chạy nhưng không dùng cho mã mới; gỡ hẳn là thay đổi phá vỡ, cần chủ dự án đồng ý và ra phiên bản lớn.

### Sự kiện (AsyncAPI)

- AsyncAPI 3.0: mỗi topic là một kênh (`channels`), ai gửi ai nhận ghi ở `operations` (`x-service`); mỗi tin nhắn ghi bên phát (`x-producer`), bên nghe (`x-consumers`) và lát cài (`x-slice`).
- Mọi tin là một CloudEvent 1.0 dạng structured (JSON), khóa bản ghi Kafka là `subject` (mã người dùng). Mỗi service chỉ ghi topic `<service>.events` của mình; DLQ là `<topic>.dlq`.
- Cùng quy tắc chỉ thêm: thêm trường, thêm giá trị enum, thêm loại sự kiện, thêm topic. Thay đổi phá vỡ là loại mới `<tên>.v2` phát song song với loại cũ.
- Schema `data` đặt `additionalProperties: false`: test hợp đồng của bên phát bắt được trường ngoài hợp đồng (ví dụ email lỡ lọt vào). Bên nghe thì khoan dung: bỏ qua trường lạ và loại lạ.
- Sự kiện không bao giờ chứa email, ngày sinh, mật khẩu, token, IP, đáp án; không có chữ theo chế độ hiển thị.
- Từ 0.2.0 mỗi kênh có `x-acl` (`write`, `read`: tài khoản Kafka nào được ghi, đọc). Platform dựng ACL đúng theo đó (M9 lát 9.0); thêm bên nghe là thêm vào `read` trước, và vào `write` của DLQ của topic đó.
- Khóa bản ghi là mã người dùng, trừ kênh ghi `x-subject: class` (từ 0.3.0, `classroom.events`): khóa và `subject` là mã lớp, để mọi sự kiện của một lớp đến đúng thứ tự.
- Người dùng trong API công khai của social chỉ được chỉ bằng mã hồ sơ (`handle`); mã tài khoản thật của người khác không xuống trình duyệt.

## Kiểm tra

```bash
npx --yes @redocly/cli@1 lint openapi/identity.yaml
npx --yes @redocly/cli@1 lint openapi/commerce.yaml
npx --yes @redocly/cli@1 lint openapi/social.yaml
npx --yes @redocly/cli@1 lint asyncapi/pkkemis.yaml
```

Hiện còn các cảnh báo chấp nhận được: thiếu `license` ở năm file OpenAPI; với identity thêm `logout` và `jwks.json` không có đáp án 4xx. `asyncapi/pkkemis.yaml` không có cảnh báo. Redocly chỉ kiểm cấu trúc; service đối chiếu sự kiện thật với schema bằng test hợp đồng (như `test/contract.spec.ts` với OpenAPI), lấy ví dụ trong `examples` của từng tin nhắn làm dữ liệu cho test của bên nghe. CI sẽ chạy các lệnh này cho từng file và so với `main` để chặn thay đổi phá vỡ (oasdiff cho OpenAPI).
