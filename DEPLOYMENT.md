# Deployment Information

## Public URL
https://ai-agent-hohk.onrender.com
*(Chú ý: Hãy thay đường link trên bằng tên URL thực tế của bạn trên Render/Railway ở Bước 3.1)*

## Platform
Render (Sử dụng Blueprint IaC file deployment tự sinh qua GitHub /render.yaml)

## Test Commands

### Health Check
```bash
curl https://ai-agent-hohk.onrender.com/health
# Expected: {"status": "ok", "uptime_seconds": ..., "version": "1.0.0", "environment": "production"}
```

### API Test (with authentication)
```bash
curl -X POST "https://ai-agent-hohk.onrender.com/ask" \
  -H "X-API-Key: key-cho-sinh-vien-12345" \
  -H "Content-Type: application/json" \
  -d "{\"user_id\": \"test\", \"question\": \"Hello Agent, what is deployment?\"}"
# Expected: {"question": "...", "answer": "...", "model": "..."} với Status Code 200
```

## Environment Variables Set
Cấu hình biến môi trường đã được cung cấp ngầm định hoặc tuỳ biến tự nhập Dashboard trên nền tảng Cloud:
- `ENVIRONMENT` = production
- `PYTHON_VERSION` = 3.11.0
- `OPENAI_API_KEY` = <Bản quyền key OpenAI> 
- `AGENT_API_KEY` = <Tự động Generate nhờ thuộc tính generateValue: true ở tập lệnh render hoặc chỉ định tay trên Railway>
- `PORT` = (Render/Railway tự cấp phát biến môi trường nội bộ)

## Screenshots
*(Sinh viên nhớ chủ động tạo một thư mục `/screenshots/` vào cùng chỗ này chứa 3 bức hình của mình và ném nó lên GitHub nhé!)*
- [Deployment dashboard](screenshots/dashboard.png)
- [Service running (health ok)](screenshots/running.png)
- [Test results (POST Request thành công)](screenshots/test.png)
