# TỔNG QUAN QUÁ TRÌNH HOẠT ĐỘNG

Đây là sơ đồ mô phỏng quá trình thực hiện CI/CD Pipeline để triển khai  ứng dụng YouTube Clone trên AWS EKS với Jenkins và Terraform. Quá trình được chia thành các thành phần chính như sau:

## Development Environment

- Quá trình bắt đầu từ phía các lập trình viên (Developer), những người chịu trách nhiệm phát triển ứng dụng YouTube Clone trên môi trường cục bộ (máy tính cá nhân).

- Sau đó, mã nguồn sẽ được đẩy lên GitHub.

- Đồng thời, sử dụng Terraform giúp tự động hóa việc tạo hạ tầng trên AWS, bao gồm các tài nguyên như cụm EKS, VPC, subnet và các role cần thiết.

> [!NOTE]
> Bước nền quan trọng để đảm bảo hạ tầng cloud luôn sẵn sàng cho việc triển khai ứng dụng.

## CI Pipeline (Continuous Integration)

- Khi mã nguồn được đẩy lên GitHub, một **webhook** sẽ kích hoạt Jenkins, từ đó bắt đầu quá trình CI.

- Jenkins CI Job:

    - Tiến hành build Docker Image từ mã nguồn mới.

- Trước khi lưu trữ Image này, hệ thống sẽ thực hiện kiểm tra bảo mật **(Security check)** thông qua các công cụ như

    - SonarQube (kiểm tra chất lượng và bug trong code).
    
    - Trivy và Grype (phân tích các lỗ hổng bảo mật trong Docker image).

- Nếu mọi thứ ổn định và đạt yêu cầu, Docker image sẽ được đẩy lên Docker Hub, nơi lưu trữ trung tâm các image sẵn sàng cho việc triển khai.

## CD Pipeline (Continuous Deployment)

- Sau khi Image được lưu trữ thành công, một Jenkins CD Job khác sẽ được kích hoạt để thực hiện triển khai ứng dụng.

- Jenkins CD Job:

    - Jenkins sẽ lấy image từ Docker Hub và sử dụng công cụ kubectl kết hợp với Helm để triển khai ứng dụng lên cụm Amazon EKS (Elastic Kubernetes Service) – dịch vụ Kubernetes được quản lý bởi AWS.

- Các biểu đồ Helm (Helm charts) sẽ định nghĩa cấu hình, số lượng bản sao (replica), các biến môi trường… đảm bảo quá trình triển khai nhanh chóng và nhất quán..

## AWS EKS (Kubernetes Cluster trên AWS)

- Ứng dụng YouTube Clone được triển khai và chạy trong các Pod bên trong Amazon EKS.

- Cụm Kubernetes sẽ chịu trách nhiệm duy trì tính sẵn sàng và cân bằng tài nguyên giữa các Pod

- AWS Load Balancer sẽ định tuyến các yêu cầu từ người dùng tới các Pod phù hợp.

> [!NOTE]
> Sử dụng Load Balancer đảm bảo ứng dụng có thể chịu tải cao và phản hồi nhanh cho người dùng cuối.

## Monitoring & Alerting

- Hệ thống giám sát được triển khai song song, sử dụng Prometheus để thu thập các chỉ số như CPU, RAM, trạng thái container... từ cụm EKS.

- Những chỉ số này sẽ được hiển thị trực quan qua các dashboard của Grafana, giúp người quản trị dễ dàng theo dõi hiệu suất hệ thống.

- Nếu có sự cố như tài nguyên vượt ngưỡng, pod bị crash, Prometheus sẽ gửi cảnh báo đến để kịp thời xử lý.

## Thông báo (Notification)

- Để nâng cao khả năng phản ứng nhanh, hệ thống được cấu hình để gửi báo cáo hoặc thông báo lỗi đến các kênh giao tiếp như Slack và Telegram.

## End user

- Truy cập ứng dụng thông qua YouTube UI (giao diện clone).
