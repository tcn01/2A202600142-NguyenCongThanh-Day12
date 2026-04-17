# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found
1. Hardcoded API key (`OPENAI_API_KEY`) và Database URL dính trực tiếp trên code bảo mật kém.
2. Không sử dụng thư viện quản lý môi trường Config (`DEBUG = True` được cố định).
3. Sử dụng lệnh `print()` để log các thông tin cực kì nhạy cảm (in thẳng Key cá nhân ra cửa sổ stdout).
4. Thiếu hụt trầm trọng Health Check endpoints (`/health`) cho Cloud theo dõi sập server.
5. Cài đặt cứng Cổng Port (`port=8000`) và địa chỉ host tĩnh (`localhost` thay vì `0.0.0.0`).

### Exercise 1.3: Comparison table
| Feature | Develop | Production | Why Important? |
|---------|---------|------------|----------------|
| Config  | Hardcoded thẳng vào biến | Đọc bằng Environment Variables | Giải quyết thay đổi tham số dễ dàng khi deploy; tránh việc gián điệp mạng có thể thấy key trên github. |
| Health check | None (Hoàn toàn không có) | `/health`, `/ready` | Để cho hệ thống cân bằng tải (LB / Cloud) định giá đường truyền, loại bỏ tự động nhánh rẽ hư hỏng ra khỏi traffic. |
| Logging | `print()` lệnh in sơ sài | Structured JSON Log | Tuân thủ định dạng chuẩn hóa, chủ động ẩn secrets, dễ dàng phân tích truy vấn đối soát theo kho dữ liệu Cloud Engine. |
| Shutdown | Forced Quit (Đứt điện đột ngột) | SIGTERM Handle/ Graceful | Server báo ngừng nhận kết nối, và kiên nhẫn đợi chạy dịch vụ cho các khách hàng còn lại ra về an toàn rồi mới tắt. |
| Port binding | Tĩnh `localhost:8000` | Động `0.0.0.0:$PORT` | Cho phép nhận các kết nối liên mạng internet và đồng hóa linh hoạt thông số theo chỉ định cấu hình hạ tầng Orchestration OS. |

## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. Base image là gì?: Sử dụng `python:3.11` (Bản đầy đủ, nặng ~1GB).
2. Working directory là gì?: Sử dụng `/app`. Mọi đường dẫn thư mục thi hành lệnh sẽ nằm trong đây.
3. Tại sao COPY requirements.txt trước?: Docker chạy bằng cơ chế xây tầng tệp Caching, việc đẩy Copy lệnh cài đặt môi trường lên sớm nhất giúp Docker dùng lại cache, không phải tốn thời gian tải qua internet mỗi lần ta gõ code sửa Python `app.py`.
4. CMD vs ENTRYPOINT khác nhau thế nào?: `CMD` cung cấp lệnh khởi chạy default và cho phép bạn override đè lệnh khác vào một cách rất dễ dàng lúc run. `ENTRYPOINT` luôn được thực thi một cách ép chế dưới định dạng tham số.

### Exercise 2.3: Image size comparison
- Develop: ~1 GB (Do base image mang đầy đủ OS và hàng loạt bộ trình compiler rườm rà).
- Production: Dưới 500 MB (Rất nhẹ, nhỏ do base `python:3.11-slim`).
- Difference: Giảm đến ~70-80% do Stage 2 đã gạt bỏ toàn bộ tệp compile không còn xài, chỉ mang sang mỗi các thư viện đã được biên dịch xong xuôi sang folder của Root (`/home/appuser/.local`).

## Part 4: API Security

### Exercise 4.1: API Key authentication
- API key được check ở đâu?: Check tĩnh ở cơ cấu Fast API Header, thông qua lệnh tiêm chặn Dependencies `verify_api_key(api_key: str = Security(api_key_header))` trỏ thẳng vào endpoint `ask_agent`.
- Điều gì xảy ra nếu sai key?: Trigger ngoại lệ `HTTPException` trả về Status 403 (Hoặc 401 Unauthorized nếu null).
- Làm sao rotate key?: Key linh động được import từ env nên muốn thay, chỉ cần thiết lập cấp biến shell `$AGENT_API_KEY` chạy thay cho key cũ, không chạm đến source code backend.

### Exercise 4.2-4.3: Rate limit testing
- Thuật toán: Window Counter Sliding.
- Limit: Quy định chặn IP theo user, cấu hình 10 request / 60 giây. Role "Teacher" bypass Admin ở mức 100 req. Quá tải trả về lỗi System 429 Too many requests.

### Exercise 4.4: Cost guard implementation
- Dựa vào giá token GPT OpenAI, có 2 tầng lưới an ninh: Guard riêng lẻ bằng chặn từng Client vượt quá ngưỡng nạp $1 / ngày (Raise Error 402) ; lưới vây bọc Total Global System Server khống chế tối đa mất 10 đô nếu bị BOT bắn API nát nhừ (bắn raise Error 503 để dập cả máy trạm tránh cháy túi mảng tài chính thẻ Visa).

## Part 5: Scaling & Reliability

### Exercise 5.1-5.5: Implementation notes
- Phân tích: Sự kết hợp của Nginx là tấm khiên tuyệt vời chia tải đều thay thế 3 replicas chạy agent. 
- Mọi History, State, Session cache được rút tiệt khỏi ổ cứng và ram Python, thay vào đó ném sang lưu kho tĩnh bằng Redis Cache ở mạng con ngầm ảo internal. Nếu Container của app 1 sập hoặc update nóng, Container số 2 chắp vá ngay Request tiếp theo đọc lại Redis và load history liền mạch trong nháy mắt. (Kiểm chứng Pass bài test Stateful Python test script của chuyên gia).
