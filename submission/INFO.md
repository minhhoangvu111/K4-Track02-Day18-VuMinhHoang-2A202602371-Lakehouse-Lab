# Thông tin bài nộp

- **Họ tên:** Vũ Minh Hoàng
- **MSSV:** 2A202602371
- **Mã bài:** K4-Track02-Day18
- **Repo cá nhân:** `minhhoangvu111/K4-Track02-Day18-VuMinhHoang-2A202602371-Lakehouse-Lab`
- **Đường chạy:** Lightweight (`deltalake`, PyIceberg, DuckDB, Polars); NB1–NB8
- **Python:** 3.12.10
- **Hệ điều hành:** Windows 10 (10.0.19045), PowerShell
- **Ngày thực hiện:** 2026-10-05

## Ghi chú tái lập

- 8 notebook đã chạy và giữ output trong `notebooks/`.
- NB1–NB4 dùng root mặc định `_lakehouse/`.
- NB5–NB8 chạy với `LAKEHOUSE_ROOT=_lakehouse/submission_run` để tránh xóa/ghi đè catalog scratch cũ đang bị Windows từ chối quyền truy cập.
- Notebook được thực thi từng code cell bằng Python và lưu stdout vào `.ipynb`; Jupyter `nbconvert` không khởi tạo được kernel do lỗi quyền Windows khi tạo connection file.
- 8 ảnh trong `screenshots/` là ảnh render từ output notebook đã lưu, không phải ảnh chụp màn hình desktop.
