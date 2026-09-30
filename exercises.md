# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Khánh Linh  Mã học viên: 2A202602409

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu em quên đặt `AGENT_API_KEY` khi deploy, app dừng ngay lúc khởi động nên em phát hiện lỗi cấu hình trong deploy log. Nếu dùng mặc định `changeme`, service vẫn chạy và em có thể tưởng đã bảo vệ API, trong khi người ngoài đều biết hoặc đoán được key đó và gọi `/ask`.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Log em đã thấy: `{"event":"ask_completed","level":"info","timestamp":"2026-09-29T08:45:54.818376+00:00","user_id":"sv01","tokens_in":43,"tokens_out":47,"cost_usd":3.465e-05}`. Với các trường có cấu trúc này, em có thể lọc tất cả request của một user và cộng token/chi phí theo thời gian. Một dòng `print("đã trả lời xong")` không cho biết user nào gọi hoặc request đó tốn bao nhiêu.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f Dockerfile.single-stage -t agent:single .
docker build -f Dockerfile -t agent:multi .
docker images --format '{{.Repository}}:{{.Tag}} {{.Size}}' | grep -E '^agent:(single|multi) '
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 400 MB (`agent:single`) |
| Multi-stage | 382 MB (`agent:multi`) |

> Theo số đo trên máy em, bản multi-stage nhỏ hơn 18 MB. `docker history` cho thấy chênh lệch chủ yếu nằm ở layer cài/copy Python packages (79.2 MB ở bản 1-stage so với 65.9 MB ở bản multi-stage); đây không phải compiler bị loại bỏ trong phép so sánh này, vì cả hai dùng `python:3.11-slim` và builder hiện không cài thêm compiler. Multi-stage tách riêng bước cài dependency khỏi runtime, nhưng phần tiết kiệm cụ thể còn phụ thuộc cách cài và đóng gói. Cả hai image dùng cùng `requirements.txt`; số đo có thể đổi khi dependency hoặc base image thay đổi.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Theo quan sát của em, khi chỉ sửa `app/main.py`, Docker giữ cache cho stage builder vì `requirements.txt` không đổi; bước cài dependency được dùng lại. Ở runtime, `COPY . .` bị chạy lại vì source đổi, và lệnh `RUN useradd...` phía sau cũng chạy lại. Nếu `COPY . .` đặt trước `RUN pip install`, mỗi lần sửa code sẽ làm mất cache của bước cài package, khiến build lâu hơn.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Theo em hiểu, nếu process Python bị khai thác khi chạy root, kẻ tấn công có quyền root bên trong container. Nếu container còn quyền hoặc mount nhạy cảm (ví dụ Docker socket hay thư mục host), họ có thể dùng chúng để tác động tới host. `USER appuser` chạy process với quyền thường, cắt bớt quyền ngay từ đầu và giới hạn thiệt hại; nó không tự loại bỏ mọi lỗ hổng thoát container.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Theo em, có thể gửi tối đa 20 request trong khoảng 2 giây quanh ranh giới phút: gửi 10 request ở những giây cuối của phút hiện tại, rồi thêm 10 request ngay sau khi đồng hồ sang phút mới. Bộ đếm theo phút lịch vừa reset nên vẫn cho qua cả hai nhóm. Sliding window 60 giây sẽ thấy đủ 20 request trong cùng một cửa sổ và chặn nhóm vượt hạn mức.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Theo cách em hiểu, rate limit giới hạn số request trong một khoảng thời gian; cost guard giới hạn tổng chi phí theo user và tháng. Ví dụ, một user còn quota request nhưng đã tiêu vượt ngân sách tháng thì cost guard phải chặn request kế tiếp. Ngược lại, user gửi nhiều câu ngắn, rẻ trong vài giây có thể bị rate limit chặn dù ngân sách còn. Trong code hiện tại `guard.check(user_id)` chưa truyền chi phí ước tính; vì vậy guard chỉ phát hiện chi tiêu đã vượt ngân sách ở request sau, chứ chưa chặn trước một request đắt dựa trên ước tính.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Theo em, nếu Redis mất kết nối 30 giây và `/health` cũng kiểm tra Redis, liveness của cả ba container sẽ báo lỗi. Orchestrator có thể restart các container dù process vẫn chạy bình thường. Redis vẫn đang mất kết nối nên container vừa restart lại tiếp tục fail health check, tạo vòng restart và làm giảm số instance phục vụ. Tách riêng `/ready` giúp ngừng gửi traffic khi Redis chưa sẵn sàng, còn `/health` chỉ báo process có cần restart hay không.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Qua phần em chạy thử, với Redis mọi instance đọc cùng một history nên `history_length` tăng theo lịch sử dùng chung (request đầu thường là 0, các lượt sau tăng thêm hai message). Nếu dùng dict trong RAM, mỗi container giữ bản riêng: request chuyển sang instance khác có thể thấy history_length thấp hơn hoặc bằng 0, và restart container sẽ làm mất history của instance đó.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi em test `/ask` trên Render, một request có key chứa ký tự có dấu trả `500`. Em xem Render Logs và thấy `TypeError: comparing strings with non-ASCII characters is not supported` tại `secrets.compare_digest`. Nguyên nhân là hàm được gọi với chuỗi Unicode. Em đổi phép so sánh sang UTF-8 bytes, thêm test hồi quy, rồi deploy lại. Sau sửa, key sai có dấu trả `401`, còn request `/ask` dùng key Render hợp lệ trả `200`.
