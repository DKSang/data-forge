# Data Forge — bộ skill data engineering

Bộ skill giúp agent cùng người dùng thiết kế, xây dựng và vận hành hệ thống dữ liệu **theo bài toán**, không theo một danh sách công nghệ cố định. Dùng được cho dự án local, on-prem, cloud hoặc hybrid — từ một script nhập dữ liệu đến nền tảng phục vụ nhiều nhóm.

> **Trạng thái:** source package đã sẵn sàng để publish lên GitHub, nhưng repository chưa được publish. Các lệnh GitHub bên dưới dùng `OWNER` làm placeholder; thay bằng GitHub owner thực tế sau khi repository được tạo công khai. Vì vậy, các lệnh này hiện chưa chạy được.

## Bắt đầu nhanh

### Cài một skill bằng `npx skills`

Sau khi repository được publish, cài riêng skill cần dùng bằng cách thay `OWNER` và tên skill:

```bash
npx skills@latest add OWNER/data-forge --skill forge-de-design
```

Lệnh trên chỉ cài `forge-de-design`. Đổi tên sau `--skill` để cài skill khác; dùng lựa chọn của installer để chọn agent/runtime. Có thể chạy lại cho từng peer skill khi cần handoff. Không cần cài cả bộ.

### Cài thủ công

Nếu đã có source checkout, bỏ qua bước clone. Sau khi repository được publish, có thể lấy source như sau:

```bash
git clone https://github.com/OWNER/data-forge.git
```

Sao chép **nguyên thư mục** `skills/<group>/<skill>/` vào thư mục skills trực tiếp mà runtime hỗ trợ. Giữ nguyên `SKILL.md` và toàn bộ `references/` bên trong.

Ví dụ với Claude Code:

```text
skills/design/forge-de-design/  →  .claude/skills/forge-de-design/
```

Mỗi thư mục skill là một đơn vị cài đặt độc lập. Các runtime khác có thể dùng thư mục skills riêng; runtime cần nhận diện định dạng `SKILL.md`. Chỉ clone hoặc tải source repo không tự kích hoạt skill trong repo dự án.

## Cập nhật và gỡ bỏ

| Cách cài | Cập nhật | Gỡ bỏ |
| --- | --- | --- |
| `npx skills` | Chạy `npx skills update` và chọn skill Data Forge cần cập nhật nếu installer hỏi. | Chạy `npx skills remove` và chọn skill Data Forge cần gỡ nếu installer hỏi. |
| Thủ công | Sao chép thư mục skill mới nhất đè lên thư mục đã cài; giữ nguyên `references/`. | Xóa đúng thư mục skill đã cài. |

Các lệnh CLI chỉ áp dụng cho skill đã cài và được quản lý bằng `npx skills`. Gỡ skill không tự xóa tài liệu thiết kế hoặc project knowledge trong repo đích; chúng thuộc repo dự án và tuân theo quy ước của repo đó.

## Bộ skill

| Skill | Nhóm | Dùng khi |
| --- | --- | --- |
| [forge-de](skills/setup/forge-de/SKILL.md) | setup | Bắt đầu/tiếp tục sáng kiến dữ liệu và định tuyến đến bước phù hợp. |
| [forge-de-brain](skills/brain/forge-de-brain/SKILL.md) | brain | Truy xuất/lưu định nghĩa, hợp đồng và quyết định dự án đã được xác nhận. |
| [forge-de-design](skills/design/forge-de-design/SKILL.md) | design | Làm rõ nhu cầu, đánh giá nguồn, mô hình hóa và trade-off trước tài liệu stage. |
| [forge-de-deliver](skills/build/forge-de-deliver/SKILL.md) | build | Xây, sửa hoặc kiểm thử thay đổi đã được yêu cầu và có phạm vi rõ. |
| [forge-de-operate](skills/operate/forge-de-operate/SKILL.md) | operate | Duy trì chất lượng/trust, security/privacy, observability, ứng phó và cải tiến. |

`forge-de` định hướng và định tuyến; `forge-de-brain` giữ ngữ cảnh dự án đã xác nhận; `forge-de-design` thống nhất trade-off theo từng chặng; `forge-de-deliver` triển khai yêu cầu build đã rõ; `forge-de-operate` hướng dẫn vận hành. Mỗi skill dùng được độc lập trong phạm vi của nó. Handoff sang skill khác là tùy chọn: cài thêm peer skill khi cần workflow liên vai trò.

Sau khi cài, gọi skill theo tên trong runtime hỗ trợ skills, hoặc mô tả nhu cầu để agent tự chọn skill phù hợp.

## Nguyên tắc làm việc và an toàn

- Bắt đầu từ người dùng dữ liệu, quyết định kinh doanh, chất lượng, freshness và năng lực đội ngũ. Không mặc định cloud, Spark, streaming hay medallion architecture.
- Trao đổi các trade-off có ảnh hưởng trước khi ghi tài liệu thiết kế; cần xác nhận rõ cho từng stage liên quan. Chưa thống nhất thì không tạo file thiết kế nháp hay tracking file.
- **Duyệt thiết kế, yêu cầu triển khai và cho phép tác động remote/production là ba quyền riêng biệt.** Duyệt một stage không tự cho phép sửa cloud resources, chạy job ghi dữ liệu, phát sinh chi phí hay thao tác phá huỷ.
- Ưu tiên dữ liệu giả và kiểm thử local; nêu rõ điều gì đã kiểm chứng và điều gì chưa.

Ví dụ giả lập nằm trong [examples/scenarios.md](examples/scenarios.md). Có thể kết hợp skill chuyên nền tảng sau khi nền tảng đã được chọn; chúng không thay thế việc làm rõ mục tiêu và trade-off.

## Phát triển và đánh giá

Mỗi skill có frontmatter `name`/`description`, hướng dẫn chính trong `SKILL.md` và tài liệu chuyên sâu trong `references/`. [BOOK-COVERAGE.md](BOOK-COVERAGE.md) theo dõi khái niệm theo chương, trạng thái phê duyệt nội dung Data Forge và giới hạn đối chiếu trực tiếp với sách; đây là coverage plan, không phải tuyên bố đã bao phủ toàn bộ sách.

Chạy kiểm tra cấu trúc tại source checkout:

```bash
python -m unittest discover -s tests -v
```

GitHub Actions tại `.github/workflows/check.yml` cấu hình chạy cùng bộ test trên push và pull request sau khi source được đưa lên GitHub. Các prompt eval dùng dữ liệu giả nằm trong `evals/`; workspace, transcript và benchmark sinh cục bộ dưới `.claude/skills/*-workspace/` không thuộc gói phân phối.

## Giấy phép và nguồn

Nội dung Data Forge mà người đóng góp có quyền cấp phép được phát hành theo MIT; xem [LICENSE](LICENSE). Giấy phép không cấp quyền đối với sách hoặc tài liệu bên thứ ba. Repo diễn giải khái niệm bằng ngôn ngữ riêng, không phân phối văn bản, hình hoặc bảng từ *Fundamentals of Data Engineering*.
