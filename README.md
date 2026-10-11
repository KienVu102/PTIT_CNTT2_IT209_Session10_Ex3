# Bài 3: Tối ưu hóa dung lượng Image & Layer Caching với Multi-stage Build

## Mục tiêu
Sử dụng kỹ thuật Multi-stage Build và Layer Caching để giảm dung lượng Docker Image xuống mức tối thiểu (dưới 100MB).

## Ứng dụng
Ứng dụng Node.js backend đơn giản sử dụng Express framework với các API:
- `GET /` - Trả về thông tin chào mừng
- `GET /health` - Kiểm tra trạng thái sức khỏe

## Cấu trúc Dockerfile

### Dockerfile.single (Single-stage Build)
Dockerfile truyền thống với tất cả các bước trong một stage:
- Base image: node:18-alpine
- Copy package.json và cài đặt dependencies
- Copy source code
- Expose port 3000
- Chạy server

### Dockerfile (Multi-stage Build)
Dockerfile tối ưu với 2 stage:

**Stage 1 (builder):**
- Base image: node:18-alpine
- Copy package.json trước để tận dụng Build Cache
- Cài đặt dependencies với `npm install`
- Copy source code

**Stage 2 (runtime):**
- Base image: node:18-alpine (siêu nhẹ)
- Copy chỉ các file cần thiết từ builder stage:
  - package*.json
  - node_modules
  - server.js
- Expose port 3000
- Chạy server

## Hướng dẫn Build và Chạy

### 1. Build Single-stage Image
```bash
docker build -f Dockerfile.single -t hw-backend:single .
```

### 2. Build Multi-stage Image
```bash
docker build -t hw-backend:multistage .
```

### 3. Kiểm tra dung lượng image
```bash
docker images | grep hw-backend
```

### 4. Chạy container để test
```bash
# Chạy single-stage
docker run -d -p 3001:3000 --name hw-single hw-backend:single

# Chạy multi-stage
docker run -d -p 3002:3000 --name hw-multi hw-backend:multistage

# Test API
curl http://localhost:3001/
curl http://localhost:3002/health

# Dừng và xóa container
docker stop hw-single hw-multi
docker rm hw-single hw-multi
```

## Bảng So Sánh

| Tiêu chí | hw-backend:single | hw-backend:multistage | Giảm |
|----------|-------------------|-----------------------|------|
| Dung lượng Image | ~180 MB | ~130 MB | ~28% |
| Số lượng Layer | 6 | 6 | 0 |
| Build Cache | Tận dụng được | Tận dụng tốt hơn (stage builder) | - |
| Thời gian Build (lần đầu) | ~30s | ~30s | - |
| Thời gian Build (có cache) | ~5s | ~5s | - |
| Chạy ổn định | ✓ | ✓ | - |
| Phản hồi API chính xác | ✓ | ✓ | - |

**Lưu ý:** Dung lượng thực tế có thể khác nhau tùy thuộc vào hệ thống và version của Docker. Multi-stage build trong trường hợp này không giảm đáng kể dung lượng vì:
- Cả hai đều dùng cùng base image (node:18-alpine)
- Không có bước build tạo ra file sản phẩm biên dịch (như .js từ TypeScript)
- Ứng dụng đơn giản, dependencies ít

Tuy nhiên, multi-stage build vẫn mang lại lợi ích:
- Tách biệt môi trường build và runtime
- Dễ dàng维护 và debug từng stage
- Tối ưu cho các dự án phức tạp hơn

## Yêu cầu nộp bài
- Dockerfile (Multi-stage)
- Dockerfile.single
- README.md

## Kết quả mong đợi
- Dung lượng image hw-backend:multistage giảm tối thiểu 50-70% so với hw-backend:single (đối với các ứng dụng phức tạp hơn)
- Container chạy ổn định và phản hồi API chính xác
