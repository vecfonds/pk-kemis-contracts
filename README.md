# pk-kemis-contracts

Hợp đồng giữa các service của PkKemis và nguồn sự thật cho API. Kế hoạch: `../pk-kemis-web/docs/backend/PLAN.md` (mục 6) và `PHASE1.md`.

## Nội dung

| File | Service | Trạng thái |
|---|---|---|
| `openapi/identity.yaml` | identity | 0.4.0 đã cài (M1, M2, thao tác admin, nhật ký); 0.5.0 viết cho M6 lát L7 (ẩn khỏi bảng xếp hạng), chưa cài |
| `openapi/learning.yaml` | learning | 0.2.0 đã cài và đối chiếu bằng `test/contract.spec.ts` ở learning (M3); 0.3.0 viết cho M6 (nội dung, giấu đáp án, tiến độ, sổ tay lỗi sai, xếp hạng, API nội bộ), cài dần theo lát |
| `openapi/classroom.yaml` | classroom | 0.7.0 đã cài (M4 lát A–D), đối chiếu bằng `test/contract.spec.ts` ở classroom; 0.8.0 viết cho M6 (danh mục từ learning, tiến độ học sinh, lượt làm bài của lớp ở máy chủ), cài dần theo lát |

Kế hoạch M6 và thứ tự các lát: `../pk-kemis-web/docs/backend/M6-CONTENT.md`.

AsyncAPI cho sự kiện sẽ thêm khi có Kafka.

## Quy tắc

- **Chỉ thêm, không xóa hay đổi nghĩa** trường đã phát hành. Thay đổi phá vỡ phải ra phiên bản mới (`/v2`).
- Đổi hợp đồng **trước**, rồi mới đổi mã service và web.
- Lỗi theo RFC 9457 (`application/problem+json`), có mã `code` ổn định và `detail` tiếng Việt cho người dùng.
- Thao tác gắn `x-phase: 2` có trong hợp đồng để định hình, nhưng giai đoạn 1 chưa cài.
- Thao tác gắn `x-slice: Ln` thuộc lát n của M6; trước khi lát đó xong, service chưa có thao tác này. `x-transitional` là thao tác tạm, ghi rõ khi nào gỡ.
- Thao tác `deprecated: true` vẫn chạy nhưng không dùng cho mã mới; gỡ hẳn là thay đổi phá vỡ, cần chủ dự án đồng ý và ra phiên bản lớn.

## Kiểm tra

```bash
npx --yes @redocly/cli@1 lint openapi/identity.yaml
```

Hiện còn các cảnh báo chấp nhận được: thiếu `license` ở cả ba file; với identity thêm `logout` và `jwks.json` không có đáp án 4xx. CI sẽ chạy lệnh này cho từng file và so với `main` để chặn thay đổi phá vỡ (oasdiff).
