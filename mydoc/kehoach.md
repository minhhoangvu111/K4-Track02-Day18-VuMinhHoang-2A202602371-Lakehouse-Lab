# Kế hoạch hoàn thành K4-Track02-Day18 Lakehouse Lab

## 1. Mục tiêu và nguyên tắc

Hoàn thành đủ 8 notebook bắt buộc theo đường lightweight, tạo output thực tế có thể kiểm tra, giải thích các số đo chính, chạy bộ kiểm thử và chuẩn bị đúng cấu trúc submission. Bài làm là cá nhân; không tạo số liệu giả hoặc bỏ/hạ assertion. Bonus architecture brief là tùy chọn và chỉ bắt đầu sau khi hoàn tất phần bắt buộc.

**Đường chạy đề xuất:** Lightweight (đủ NB1–NB8, chạy offline sau cài dependency; phù hợp Windows/PowerShell). NB1–NB4 có thể thay bằng Spark, nhưng không cần thiết để đạt phạm vi bài lab.

**Thời lượng tham khảo:** khoảng 3–4 giờ cho bắt buộc theo đề, không tính thời gian tải/cài đặt và xử lý sự cố; bonus thêm 4–8 giờ.

## 2. Lộ trình tổng quan

| Phase | Nội dung | Checkpoint hoàn thành |
|---|---|---|
| 0 | Xác nhận fork, môi trường và dữ liệu | Smoke pass, origin đúng, biết khu vực dữ liệu sinh |
| 1 | Delta nền tảng: NB1–NB3 | Schema, tối ưu, time travel/MERGE/RESTORE đạt ngưỡng |
| 2 | Medallion: NB4 | Bronze/Silver/Gold và chất lượng tổng hợp được xác nhận |
| 3 | Lakehouse nâng cao: NB5–NB6 | Catalog/Iceberg và maintenance đạt ngưỡng |
| 4 | AI lakehouse: NB7–NB8 | Vector lifecycle, agent replay và provenance có bằng chứng |
| 5 | Kiểm định, đóng gói, nộp | 24 tests, 8 notebook có output, ảnh, reflection, INFO, push/PR |
| Tùy chọn | Bonus architecture brief | Brief 3–6 trang, có đủ bằng chứng rubric bonus |

---

## Phase 0 — Chuẩn bị repo và môi trường

### Việc cần làm

1. Kiểm tra repository đã là fork cá nhân và tên theo mẫu `K4-Track02-Day18-HoVaTen-MSSV-Lakehouse-Lab`; xác nhận `origin` trỏ tới fork cá nhân. Đổi tên repository trên GitHub nếu chưa đúng; đổi tên thư mục local không thay thế bước này.
2. Xác nhận Python 3.10–3.14 (README kiểm tra trên Python 3.11), Git và dependencies. Trên Windows, dùng PowerShell; môi trường `.venv` có sẵn trong workspace nhưng cần xác minh.
3. Chạy smoke test: `\.venv\Scripts\python.exe scripts/verify_lite.py`. Nếu môi trường chưa sẵn sàng, tạo venv, cài `requirements.txt`, rồi chạy lại.
4. Đọc `docs/SUBMISSION.md`, `docs/RUBRIC.md`, `docs/RULES.md`; xác định nơi ghi nhận ảnh/output. Không commit `.venv/`, `_lakehouse/`, cache hoặc blobs sinh.

### Cổng kiểm tra (Gate 0)

- [ ] Python/dependencies hoạt động và smoke test báo 9/9 checks PASS.
- [ ] `git remote -v` cho thấy origin là fork cá nhân đúng tên.
- [ ] Đã xác định `_lakehouse/` là dữ liệu sinh cục bộ, không đưa toàn bộ vào submission.
- [ ] Đã tạo checklist bằng chứng theo tám notebook.

**Bằng chứng nên lưu:** phiên bản Python/hệ điều hành cho `submission/INFO.md`; output smoke để tham chiếu (không nhất thiết chụp ảnh riêng).

## Phase 1 — Delta nền tảng (NB1–NB3)

### Checkpoint 1 — Delta transaction và schema enforcement (NB1)

**Thực hiện:** chạy `notebooks/01_delta_basics` từ đầu; tạo bảng, thử ghi `age='thirty'`, sau đó thêm cột `tier` với `schema_mode="merge"`.

**Tiêu chí đạt:**
- Có ít nhất 2 JSON commit trong `_delta_log/`.
- Ghi sai kiểu bị chặn; giữ output lỗi thực tế làm bằng chứng.
- Schema có `tier`, query cho thấy 2 nhóm tier.

**Giải thích cần ghi:** transaction log ghi lại commit; schema enforcement bảo vệ kiểu dữ liệu; schema evolution chỉ xảy ra khi chủ động bật.

**Lưu:** notebook output và ảnh thể hiện `_delta_log`/commit JSON, lỗi write, schema/query sau evolution (`nb01_delta_log.png`).

### Checkpoint 2 — Small files, OPTIMIZE và Z-ORDER (NB2)

**Thực hiện:** chạy `notebooks/02_optimize_zorder`; ghi số file, truy vấn và phép đo trước/sau OPTIMIZE/Z-ORDER.

**Tiêu chí đạt:**
- Trước tối ưu có ít nhất 100 file; số file giảm rõ sau tối ưu.
- Đạt speedup ≥ 3× **hoặc** files-pruned ratio ≥ 10×.
- Ghi min/max `user_id` từ file stats sau Z-order.

**Giải thích cần ghi:** compaction giảm overhead small files; Z-order cải thiện file skipping qua clustering/stats. Nêu thời gian có thể dao động theo cache, CPU, I/O và máy.

**Lưu:** số trước/sau, speedup, pruning, stats và ảnh (`nb02_optimize.png`).

### Checkpoint 3 — MERGE, time travel và RESTORE (NB3)

**Thực hiện:** chạy `notebooks/03_time_travel`; tạo các version, MERGE upsert 100K dòng, truy vấn version cũ, tạo trạng thái dữ liệu lỗi rồi RESTORE.

**Tiêu chí đạt:**
- MERGE 100K thành công.
- `history()` sau RESTORE có ít nhất 5 version, bao gồm hàng RESTORE và MERGE.
- Version hiện tại có `score < 0` bằng 0; RESTORE tạo commit mới chứ không xóa lịch sử.

**Lưu:** output MERGE, truy vấn version cũ, history sau restore, count lỗi và ảnh (`nb03_time_travel.png`).

### Gate 1

- [ ] NB1–NB3 chạy không lỗi, đạt ngưỡng rubric.
- [ ] Mỗi notebook có giải thích ngắn, bám số liệu thực tế; không chỉ dựa vào dòng PASS.
- [ ] Output và ảnh được giữ ở vị trí dự kiến trong `submission/`.

## Phase 2 — Medallion pipeline (NB4)

### Checkpoint 4 — Bronze → Silver → Gold

**Thực hiện:** chạy `notebooks/04_medallion`; nếu thiếu dữ liệu, sinh Bronze bằng `scripts/generate_data_lite.py`. Parse/chuẩn hóa và deduplicate sang Silver, rồi tổng hợp Gold.

**Tiêu chí đạt:**
- Ba lớp/bảng Bronze, Silver, Gold hiện diện trên storage.
- Silver ít dòng hơn Bronze do dedup.
- Gold có ít nhất 7 ngày × 3 model; kiểm tra p50 ≤ p95, `cost_usd` dương và `error_rate` trong [0,1].

**Giải thích cần ghi:** Bronze giữ raw, Silver làm sạch/khử trùng, Gold phục vụ phân tích; giá cost là giá minh họa của lab.

**Lưu:** row counts, bảng Gold và ảnh (`nb04_medallion.png`).

### Gate 2

- [ ] Xác nhận trực tiếp nội dung Gold vì notebook không assert mọi tiêu chí chất lượng.
- [ ] Kết quả ba bảng và giải thích có trong notebook đã chạy.

## Phase 3 — Catalog, Iceberg và maintenance (NB5–NB6)

### Checkpoint 5 — Iceberg catalog, hidden partitioning (NB5)

**Thực hiện:** chạy `notebooks/05_iceberg_catalog`; tạo bảng qua catalog với `day(ts)`, đo `plan_files()` khi lọc theo `ts`, đổi tên field và tiến hóa partition spec.

**Tiêu chí đạt:**
- Tạo bảng thông qua catalog; partition transform `day(ts)`.
- Hidden-partition pruning ≥ 5× với predicate trên `ts` (không thay bằng `ts_day`).
- Báo cáo ba tầng metadata và metadata:data byte ratio.
- `latency_millis` vẫn giữ `field_id` 4 sau rename; tối thiểu 2 `spec_id` cùng tồn tại và dữ liệu vẫn đọc được.

**Lưu:** pruning plan, metadata tree/ratio, field ID và spec IDs; ảnh (`nb05_iceberg.png`).

### Checkpoint 6 — Maintenance jobs (NB6)

**Thực hiện:** chạy `notebooks/06_maintenance` tuần tự, ghi số đo trước/sau cho 5 jobs. Dữ liệu scratch của lab mới là phạm vi an toàn cho retention 0.

**Tiêu chí đạt:**
1. Compaction giảm số file ít nhất 10×.
2. Clustering khiến ít nhất 50% file có thể skip cho point query, chứng minh bằng min/max stats.
3. Delta vacuum thu hồi bytes; Iceberg expiry đưa snapshot còn 3.
4. Tìm và xóa đủ 3 Delta orphan; sweep manifest list Iceberg không còn được tham chiếu.
5. Ghi checkpoint `*.checkpoint.parquet` cùng `_last_checkpoint`.
6. Dữ liệu hiện hành còn nguyên sau maintenance.

**Giải thích cần ghi:** xóa tham chiếu/tombstone khác với xóa vật lý; trong đường PyIceberg của lab, expiry snapshot có thể không xóa manifest-list vật lý, cần sweep riêng. Delta vacuum không tìm được orphan chưa từng commit; nêu đúng phạm vi/version thư viện đã đo.

**Lưu:** bảng trước/sau và bằng chứng file/snapshot/orphan/checkpoint (`nb06_maintenance.png`).

### Gate 3

- [ ] NB5/NB6 đạt từng ngưỡng riêng; lưu ý checkpoint không thay thế phần giải thích.
- [ ] Không xóa dữ liệu ngoài scratch lab khi thao tác retention/cleanup.

## Phase 4 — AI Lakehouse (NB7–NB8)

### Checkpoint 7 — Multimodal, vector và lifecycle (NB7)

**Thực hiện:** nếu cần, sinh corpus bằng `scripts/generate_ai_data.py`; chạy `notebooks/07_vectors_multimodal`. So sánh inline blob/pointer, float32/int8, SQL semantic search và tình huống xóa trong bảng nhưng external index cũ chưa đồng bộ.

**Tiêu chí đạt:**
- Random-read amplification ≥ 5×, giải thích theo row-group granularity.
- int8 nhỏ hơn ít nhất 3×; báo cáo recall@10 ≥ 0.80 và topic fidelity ≥ 0.95.
- SQL semantic search trả neighbors đúng chủ đề.
- Lifecycle bug tái hiện: 0 hit trong bảng đã cập nhật, nhưng >0 hit ở external index cũ; ghi nhận CDF delete events.

**Lưu:** các metric, kết quả search và delete/index evidence (`nb07_vectors.png`).

### Checkpoint 8 — Agent trajectory, replay và provenance (NB8)

**Thực hiện:** chạy `notebooks/08_agents_provenance`; kiểm tra Bronze/Silver/Gold trajectory, training version pin, replay, lớp MCP mô phỏng offline, phân loại provenance và subject delete.

**Tiêu chí đạt:**
- Silver partition theo `agent_version`; Gold có cả hai policy.
- Training run pin table version; replay tại version pin khớp số bước đã ghi.
- 5 lượt `list_tables` chỉ đọc catalog một lần; destructive call chưa xác nhận trả `input_required`; simulated task hoàn tất.
- Có đủ 4 provenance buckets; `UNCLASSIFIED` bị loại khỏi trainable set; subject không còn ở version hiện tại.

**Giải thích giới hạn:** đây là mô phỏng offline, không phải MCP server hay kiểm định tuân thủ. Replay chỉ đối chiếu số bước, không so nội dung; cờ confirmed do caller truyền. Bucket là minh họa, không kết luận quyền dữ liệu. Xóa ở version hiện tại không xóa lịch sử phiên bản cũ.

**Lưu:** output các tiêu chí và ảnh (`nb08_agents.png`).

### Gate 4

- [ ] NB7/NB8 đạt ngưỡng và lưu bằng chứng.
- [ ] Khi giải thích provenance, nêu rõ đây là quy tắc minh họa; không dùng nó để kết luận quyền sử dụng dữ liệu thật.

## Phase 5 — Kiểm định và đóng gói submission

### Checkpoint 9 — Kiểm thử tái lập

Chạy từ gốc repo:

```powershell
.\.venv\Scripts\python.exe -m pytest
.\.venv\Scripts\python.exe scripts/run_all.py
```

Kỳ vọng theo đề: 24 pytest pass và cả 8 lightweight notebooks chạy headless thành công. `run_all.py` kiểm tra notebook/script nhưng **không** tạo output lưu trong `.ipynb`.

### Checkpoint 10 — Tạo bộ nộp

1. Mở từng notebook trong Jupyter và chạy từ đầu đến cuối, lưu notebook với output. Không dùng riêng kết quả `run_all.py` thay output notebook.
2. Chép đủ 8 `.ipynb` đã chạy vào `submission/notebooks/`, bỏ helper `_setup.ipynb`.
3. Tạo `submission/INFO.md`: họ tên, MSSV, mã lab, đường chạy, Python và OS; ghi NB1–NB4 dùng Spark nếu có.
4. Lưu tối thiểu một screenshot cho từng NB trong `submission/screenshots/`, đặt tên rõ ràng; đảm bảo ảnh thể hiện metric/đầu ra chấm được.
5. Viết `submission/REFLECTION.md` tối đa 200 từ: chọn một anti-pattern trong “Top 5 Lakehouse Anti-Patterns”, giải thích vì sao hệ thống/dữ liệu quan tâm dễ gặp và cách phòng tránh. Khai phạm vi dùng AI tại reflection hoặc `submission/AI_USAGE.md` nếu cần.
6. Chỉ commit source/config cần thiết; không đưa `.venv/`, `_lakehouse/`, cache/blob sinh hoặc secrets/dữ liệu nhạy cảm.

### Gate cuối — Checklist chốt bài

- [ ] Đủ 8 notebook đã thực thi, output còn nguyên, không có lỗi chưa xử lý/output giả.
- [ ] Đủ ảnh cho 8 notebook; đối chiếu tất cả con số với rubric.
- [ ] `INFO.md`, reflection ≤ 200 từ và khai báo AI (nếu áp dụng) đầy đủ.
- [ ] Smoke, 24 tests và run-all pass; nếu có lỗi, ghi rõ và xử lý trước khi nộp.
- [ ] `git status` chỉ có file dự kiến; không có secrets, venv hay dữ liệu lakehouse sinh.
- [ ] Commit và push lên fork cá nhân; xác minh file đã hiện trên GitHub.
- [ ] Mở PR về upstream với tiêu đề `[K4-Track02-Day18] HoVaTen - MSSV - Lakehouse Lab` (thêm ` [+bonus]` nếu có bonus).
- [ ] Gửi liên kết repo, PR và commit SHA theo kênh lớp. Deadline mặc định 23:59 ngày tổ chức lab, Asia/Ho_Chi_Minh; xác nhận ngày cụ thể theo lịch/thông báo key coach.

## Bonus tùy chọn — Architecture brief

Chỉ thực hiện khi phần bắt buộc đã qua Gate cuối. Chọn một tình huống dữ liệu khó và viết `submission/bonus/ARCHITECTURE.md` dài 3–6 trang khi render. Bám form ở `docs/bonus/BONUS-CHALLENGE.md`: có ít nhất 5 quyết định, mỗi quyết định so sánh tối thiểu 2 phương án và tradeoff; giả định quy mô/latency/budget nhất quán với phép tính storage/compute; một sơ đồ áp dụng ít nhất 4 khái niệm Day18; ít nhất 3 failure modes có phát hiện và rollback; MVP 1 tuần có acceptance criteria cùng cách xác minh cơ chế khó nhất. Code PoC tùy chọn. Rubric bonus tối đa 10 điểm, không làm không mất điểm bắt buộc.

## Quy tắc quản lý tiến độ

- Không chuyển phase khi gate trước chưa đạt hoặc chưa ghi nhận rõ điểm nghẽn.
- Mỗi notebook: chạy → đọc output → kiểm tra ngưỡng → giải thích → lưu notebook và screenshot.
- Khi notebook tự sinh dữ liệu thiếu, vẫn ghi lại lệnh/cách sinh để lần chạy sau tái lập.
- Ưu tiên lightweight trên Windows; Spark là tùy chọn chỉ cho NB1–NB4 và đòi hỏi thêm Docker/WSL cùng tài nguyên lớn hơn.

---

## Nhật ký thực hiện — Phase 1 (2026-10-05)

**Trạng thái:** NB1–NB3 đã chạy thành công và đạt các ngưỡng kỹ thuật của Phase 1. Bản notebook đã thực thi, có output console được lưu, nằm trong `submission/notebooks/`.

| Notebook | Kết quả thực chạy | Đánh giá |
|---|---|---|
| NB1 | Có 2 JSON commit; `age='thirty'` bị chặn với lỗi cast string sang Int64; `schema_mode="merge"` thêm `tier`; DuckDB trả 2 nhóm (`premium`, `null`) | Đạt |
| NB2 | 200 → 55 files; median 655.4 ms → 53.0 ms (**12.4× speedup**); 1/55 file cover user 4242 (**55× pruning**) | Đạt |
| NB3 | MERGE 100K trong 0.57 s (50K update + 50K insert); RESTORE về v2 trong 0.25 s; history 5 versions gồm MERGE và RESTORE; `score < 0` còn 0 dòng | Đạt |

**Notebook đã lưu:** `submission/notebooks/01_delta_basics.ipynb`, `02_optimize_zorder.ipynb`, `03_time_travel.ipynb`. Output lỗi schema của NB1 đã được xem trực tiếp; cờ check cuối notebook trong mã nguồn là placeholder hardcoded, không dùng riêng nó làm bằng chứng.

**Ảnh bằng chứng đã xuất:** `submission/screenshots/nb01_delta_log.png`, `nb02_optimize.png`, `nb03_time_travel.png`. Đây là ảnh render từ output notebook đã lưu, không phải ảnh chụp desktop. Việc hoàn thiện các phần còn lại của submission vẫn thuộc Phase 5. Jupyter nbconvert không khởi tạo kernel được trong môi trường Windows hiện tại do lỗi quyền `WinError 5`; các cell đã được chạy trực tiếp bằng Python và stdout được ghi vào notebook. Đây là cách lưu output của lần chạy này; trước khi nộp nên mở notebook trong Jupyter, kiểm tra hiển thị và lưu lại theo quy trình thông thường.


## Nhật ký thực hiện — Phase 2 (2026-10-05)

**Trạng thái:** Checkpoint 4 / NB4 hoàn thành trên đường lightweight. Notebook đã chạy lại và lưu output tại `submission/notebooks/04_medallion.ipynb`; nguồn `notebooks/04_medallion.py` được bổ sung assertions cho toàn bộ điều kiện chất lượng Gold vốn rubric yêu cầu nhưng notebook trước đó chưa kiểm tra đầy đủ.

- Bronze: **200.000** dòng; Silver: **190.052** dòng, dedup bỏ **9.948** dòng.
- Gold: **24** dòng, phủ **8 ngày × 3 model**.
- Kiểm tra trực tiếp và assert thành công: ba bảng tồn tại trên storage; Silver < Bronze; p50 ≤ p95; `cost_usd` dương; `error_rate` trong [0,1].
- Ảnh bằng chứng render từ output đã lưu: `submission/screenshots/nb04_medallion.png`.

Ảnh là bản render output, không phải ảnh chụp desktop. Jupyter nbconvert vẫn bị chặn bởi lỗi quyền Windows khi khởi tạo kernel; notebook output được chạy và ghi trực tiếp bằng Python, cùng đường chạy đã dùng cho Phase 1.

## Nhật ký thực hiện — Hoàn thiện hồ sơ NB5–NB8 (2026-10-05)

Các notebook còn lại đã được chạy và đạt assertion trong mã:

- **NB5:** pruning 10×; field ID 4 còn nguyên sau rename; hai partition spec cùng đọc được.
- **NB6:** compaction 200 → 11 file (18×); clustering bỏ qua 90% file; vacuum thu hồi 16.1 MB; tìm/xóa 3 orphan; checkpoint được tạo; Iceberg expiry 20 → 3 snapshot và sweep manifest orphan.
- **NB7:** amplification 200×; int8 nhỏ hơn 5.8×; recall@10 0.904; topic fidelity 1.000; stale index còn trả 8 tài liệu đã xóa (lifecycle bug được tái hiện).
- **NB8:** replay pinned version khớp 1,578 bước; 5 lượt list chỉ đọc catalog một lần; destructive call yêu cầu input; task hoàn tất; bốn bucket được phân vùng và 334 UNCLASSIFIED bị loại; subject hiện tại giảm 8 → 0 dòng.

NB5–NB8 lưu output dưới `submission/notebooks/` và ảnh render tương ứng dưới `submission/screenshots/`. Do các catalog/file scratch cũ bị Windows từ chối quyền xóa/ghi đè, lần chạy được cô lập dưới `_lakehouse/submission_run` qua `LAKEHOUSE_ROOT`; dữ liệu cũ được giữ nguyên.
