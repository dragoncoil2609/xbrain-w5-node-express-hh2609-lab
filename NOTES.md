# Báo cáo triển khai (Implementation Report) - HH2609

## Thông tin dự án
- **API Gateway URL**: https://vqk2n3tqml.execute-api.us-west-2.amazonaws.com
- **Kiến trúc**: AWS Lambda (ARM64) + API Gateway (HTTP API)
- **Framework**: Node.js Express

## Chiến lược lựa chọn (Strategy Choice)
Em đã lựa chọn **Chiến lược A: serverless-http** để thực hiện bài tập này.

### Lý do lựa chọn
- **Sự đơn giản**: Em chỉ cần thêm một file entry point duy nhất (`lambda.js`) với vài dòng code để bọc ứng dụng Express.
- **Phổ biến & Ổn định**: Đây là thư viện tiêu chuẩn và được cộng đồng tin dùng nhất khi chạy Express trên AWS Lambda.
- **Tính sạch sẽ (Cleanliness)**: Chiến lược này giúp em giữ file `app.js` hoàn toàn nguyên bản, không chứa bất kỳ logic nào liên quan đến Lambda, đáp ứng đúng yêu cầu "Lambda-unaware" của bài tập.
- **Tối ưu Chi phí & Hiệu năng**: 
    - **Chi phí**: Em sử dụng kiến trúc **ARM64** (Graviton2) giúp giảm chi phí vận hành khoảng 20% so với x86.
    - **Hiệu năng**: `serverless-http` là một wrapper cực nhẹ, giúp giảm thiểu thời gian khởi động lạnh (Cold Start) so với việc dùng các sidecar nặng nề hơn.

## Quy trình triển khai (Deployment Process)
Do một số hạn chế về quyền hạn trên tài khoản thực hành (không có quyền tạo ChangeSet cho SAM Transform), em đã chuyển đổi template sang dạng **CloudFormation thuần túy** và thực hiện triển khai thủ công qua AWS CLI:

1. **Build ứng dụng**:
   ```powershell
   uvx.exe --from aws-sam-cli sam build
   ```

2. **Đóng gói & Tải lên S3**:
   ```powershell
   uvx.exe --from aws-sam-cli sam package --template-file template.yaml --s3-bucket byol-express-hh038-unique-123 --output-template-file packaged.yaml --region us-west-2
   ```

3. **Triển khai Stack**:
   ```powershell
   aws cloudformation create-stack --stack-name byol-node-express-hh2609 --template-body file://packaged.yaml --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND --region us-west-2
   ```

## Kết quả đo lường (Metrics)
Dưới đây là kết quả đo lường thực tế từ hệ thống:
- **Init Duration (Cold Start)**: 273.56 ms
- **Warm Start Duration**: 3.16 ms
- **Nhận xét**: Thời gian khởi động dưới 300ms là một kết quả tốt, đảm bảo trải nghiệm người dùng mượt mà ngay cả khi ứng dụng khởi động lại.

### Minh chứng
![Kết quả truy cập và đo lường](./evidence/image.png)
![Minh chứng Lambda](./evidence/image%20copy.png)
