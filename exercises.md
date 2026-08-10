# Phiếu Phản Ánh — K3 Ngày 12

Họ và tên: Vũ Hải Nam 

Mã học viên: 2A202601173

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Tình huống đó là lúc 1h sáng sau khi fix bug xong thì em tiến hành deploy lên cloud nhưng quên cấu hình biến môi trường agent_api_key đó, tuy nhiên do việc vẫn để mặc định changeme nên app không hiện lỗi gì mà vẫn chạy được. Nhưng khi người dùng tiến hành xài app thì lại lỗi liên tục do việc gọi API bị lỗi, vì API được cấu hình giả. Việc "chết sớm" này giúp em nhanh chóng phát hiện và sửa lỗi trước khi đến tay người dùng.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Dòng log thu được là "{"event": "ask_completed", "level": "info", "timestamp": "2026-08-10T06:16:16.452297+00:00", "user_id": "sv01", "tokens_in": 436, "tokens_out": 43, "cost_usd": 9.12e-05}"
Hai việc với dòng log này là: 
1. Có thể đẩy dòng log này vào các công cụ phân tích như kibana để hệ thống tự động nhận diện các trường "cost_usd" "tokens_in" và vẽ biểu đồ chi phí tổng theo thời gian.
2. Dễ dàng tìm kiếm chính xác các request của riêng user_id = "sv01" hoặc lọc các request có cost_úd > 0.0001 mà không cần bóc tách thủ công

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.7 GB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

Phần dung lượng chênh lệch là do:
1. Loại bỏ hệ điều hành linux chứa nhiều công cụ không cần thiết
2. Loại bỏ các công cụ dùng để biên dịch code
3. Loại bỏ các file tạm và bộ nhớ đệm

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Khi sửa 1 ký tự trong app/main.py rồi build, các layer ở phía dưới (các layer sau) từ lớp layer bị sửa sẽ phải chạy lại từ đầu, các layer từ đầu đến trước layer bị sửa sẽ dùng lại từ cache.
Nếu đặt COPY .. lên trước RUN pip install thì cache bị phá vỡ do đó bắt buộc phải chạy lại từ đầu.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Nếu không dùng lệnh user thì kẻ tấn công có thể lợi dụng 1 lỗ hổng trong code để tiêm mã độc, từ đó chạy mã ngầm từ xa. Do container mặc định chạy bằng root nên mã độc sẽ có quyền root trong container và thoải mái cài đặt, sửa file hệ thống. Lệnh USER giúp tạo một user cấp thấp không có đủ quyền admin nên dù hacker có khai thác được lỗ hổng hệ thống thì cũng không chỉnh sửa được file.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

Nếu dùng cách đếm theo phút đồng hồ, thì khi ở 0:59 người dùng gửi 10 request, tại phút 1:00 thì đồng hồ reset lại 1 lần nữa và lúc này người dùng có thể gửi 10 request, do đó chỉ trong vòng vài giây người dùng đã có thể gửi 20 request, vi phạm hạn mức.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.


Rate limit quan tâm tới số lượng request trong khoảng thời gian ngắn nhằm chống DDoS, còn cost guard thì quan tâm tới tổng số USD mà user đã dùng trong 1 khoảng thời gian nhằm chống việc tiêu thụ quá nhiều tiền do AI xài quá nhiều token. 
Tình huống rate limit cho qua nhưng cost guard chặn: Khi user hỏi từng request một cách chậm rãi như là yêu cầu tóm tắt cả bộ truyện harry porter, và đính kèm file truyện dài cỡ 1k trang thì khi đó rate limit cho qua còn cost guard phải chặn lại do sẽ phải mất rất nhiều tokens cho input và do đó sẽ vượt hạn mức 1 tháng.
Trong trường hợp ngược lại, người dùng xài tools tự động hỏi các câu hỏi ví dụ "Hi" liên tục 34 lần 1 phút thì rate limit chặn nhưng cost guard không chặn do chi phí tốn rất ít

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.


1. Redis mất kết nối
2. Nền tảng điều phối gọi vào endpoint /health của 3 container để kiểm tra định kỳ
3. Vì /health có gọi redis nên nó bị lỗi và trả về mã lỗi
4. Hệ thống thấy /health báo lỗi nên hiểu nhầm 3 container bị treo
5. Hệ thống kill 3 container và khởi động lại cả 3
6. Khi redis khởi động lại thì toàn bộ web service cần khởi động lại từ đầu, làm rớt toàn bộ kết nối đang dùng.
---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Nếu lịch sử được lưu trong một dict python thì con số history_length có thể thay đổi thất thường, lý do là vì RAM chia các container làm việc hoàn toàn độc lập với nhau, mỗi lần gửi request thì dựa trên việc request rơi vào container nào thì sẽ trả history_length dựa trên bộ nhớ của container đó. Còn nếu xài trên redis thì do cơ chế stateless, các container đều đọc chung vùng nhớ nên giá trị history_length sẽ tăng dần đều.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Lỗi em gặp phải là sai REDIS_URL, thông báo lỗi là 504 Service Unavaiable, em tìm ra nguyên nhân khi đọc log sau khi chạy pytest, phát hiện render vẫn trả về 200 nhưng không kết nối được tới redis. Em sửa bằng cách vào trang quản trị upstash, chuyển sang tab TCP để lấy chuỗi url rồi dán lại giá trị vào biến REDIS_URL trong tab Environment của Render rồi restart lại render.
