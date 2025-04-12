# TỔNG QUAN QUÁ TRÌNH HOẠT ĐỘNG

Đây là sơ đồ mô phỏng quá trình thực hiện CI/CD Pipeline để triển khai  ứng dụng YouTube Clone trên AWS EKS với Jenkins và Terraform. Quá trình được chia thành các thành phần chính như sau:

## Thành phần hệ thống

### Development Environment

- Quá trình bắt đầu từ phía các lập trình viên (Developer), những người chịu trách nhiệm phát triển ứng dụng YouTube Clone trên môi trường cục bộ (máy tính cá nhân).

- Sau đó, mã nguồn sẽ được đẩy lên GitHub.

- Đồng thời, sử dụng Terraform giúp tự động hóa việc tạo hạ tầng trên AWS, bao gồm các tài nguyên như cụm EKS, VPC, subnet và các role cần thiết.

> [!NOTE]
> Bước nền quan trọng để đảm bảo hạ tầng cloud luôn sẵn sàng cho việc triển khai ứng dụng.

### CI Pipeline (Continuous Integration)

- Khi mã nguồn được đẩy lên GitHub, một **webhook** sẽ kích hoạt Jenkins, từ đó bắt đầu quá trình CI.

- Jenkins CI Job:

    - Tiến hành build Docker Image từ mã nguồn mới.

- Trước khi lưu trữ Image này, hệ thống sẽ thực hiện kiểm tra bảo mật **(Security check)** thông qua các công cụ như

    - SonarQube (kiểm tra chất lượng và bug trong code).
    
    - Trivy và Grype (phân tích các lỗ hổng bảo mật trong Docker image).

- Nếu mọi thứ ổn định và đạt yêu cầu, Docker image sẽ được đẩy lên Docker Hub, nơi lưu trữ trung tâm các image sẵn sàng cho việc triển khai.

### CD Pipeline (Continuous Deployment)

- Sau khi Image được lưu trữ thành công, một Jenkins CD Job khác sẽ được kích hoạt để thực hiện triển khai ứng dụng.

- Jenkins CD Job:

    - Jenkins sẽ lấy image từ Docker Hub và sử dụng công cụ kubectl kết hợp với Helm để triển khai ứng dụng lên cụm Amazon EKS (Elastic Kubernetes Service) – dịch vụ Kubernetes được quản lý bởi AWS.

- Các biểu đồ Helm (Helm charts) sẽ định nghĩa cấu hình, số lượng bản sao (replica), các biến môi trường… đảm bảo quá trình triển khai nhanh chóng và nhất quán..

### AWS EKS (Kubernetes Cluster trên AWS)

- Ứng dụng YouTube Clone được triển khai và chạy trong các Pod bên trong Amazon EKS.

- Cụm Kubernetes sẽ chịu trách nhiệm duy trì tính sẵn sàng và cân bằng tài nguyên giữa các Pod

- AWS Load Balancer sẽ định tuyến các yêu cầu từ người dùng tới các Pod phù hợp.

> [!NOTE]
> Sử dụng Load Balancer đảm bảo ứng dụng có thể chịu tải cao và phản hồi nhanh cho người dùng cuối.

### Monitoring & Alerting

- Hệ thống giám sát được triển khai song song, sử dụng Prometheus để thu thập các chỉ số như CPU, RAM, trạng thái container... từ cụm EKS.

- Những chỉ số này sẽ được hiển thị trực quan qua các dashboard của Grafana, giúp người quản trị dễ dàng theo dõi hiệu suất hệ thống.

- Nếu có sự cố như tài nguyên vượt ngưỡng, pod bị crash, Prometheus sẽ gửi cảnh báo đến để kịp thời xử lý.

### Thông báo (Notification)

- Để nâng cao khả năng phản ứng nhanh, hệ thống được cấu hình để gửi báo cáo hoặc thông báo lỗi đến các kênh giao tiếp như Slack và Telegram.

### End user

- Truy cập ứng dụng thông qua YouTube UI (giao diện clone).

## Thắc mắc và giải đáp

> **Trong đồ án này có được phép sử dụng mã nguồn  để xây dựng web Youtube clone tĩnh được không?**

- Dùng YouTube Clone dạng tĩnh (static) trong đồ án DevSecOps là hoàn toàn được, và còn có một số lợi thế nhất định:

    - Mục tiêu đồ án là quy trình CI/CD + DevSecOps, không phải làm sản phẩm tính năng hoàn chỉnh.

    - Tập trung vào: pipeline, bảo mật, IaC, monitoring...

-  Khi không nên chỉ dùng app tĩnh:

    - Nếu giáo viên yêu cầu ứng dụng phải có `backend (API, database)` để đánh giá end-to-end deployment.

> **Vì sao Terraform lại chuyển hướng đến Jenkins trước tiên và sau đó tới EKS?**

- Terraform không chỉ dùng để cung cấp hạ tầng EKS của AWS, mà còn được gọi bên trong Jenkins Pipeline để tự động hóa toàn bộ quy trình khởi 
tạo và triển khai.

- Thành phần :

| Thành phần | Vai trò | Mối liên hệ |
|------------|---------|-------------|
| Terraform | Công cụ IaC | 	Tạo cluster EKS, node group, VPC, IAM |
Jenkins | CI/CD server | 	Chạy các bước tự động hóa gồm: test, scan, build, deploy |
| Amazon EKS | Kubernetes cluster | 	Nơi ứng dụng được chạy (host)

- Quy trình:

1. Developer commit lên GitHub
2. Jenkins được kích hoạt (trigger)
3. Jenkins chạy các job:
    - Kiểm thử + bảo mật (SonarQube, Owasp, Trivy)
    - Dùng Terraform để khởi tạo hạ tầng AWS (gọi từ trong Jenkinsfile)
4. Sau đó dùng kubectl / Helm để deploy ứng dụng lên EKS

- Tóm lại:

    - Terraform không tự động deploy lên EKS mà được Jenkins điều khiển trong pipeline.

    - Terraform chịu trách nhiệm tạo EKS (và các dịch vụ AWS liên quan).

    - Jenkins sẽ tiếp tục pipeline: build → scan → push image → deploy bằng Helm/kubectl.

> **Terraform nên được push trực tiếp lên GitHub, sau đó Jenkins CI lấy từ GitHub để kiểm tra sự cố hay gửi thẳng đến Jenkins CI luôn không cần qua GitHub, sau đó Jenkins CI gửi đến Jenkins CD?**

- Chúng ta nên push Terraform lên GitHub vì các lý do sau:
    - Có thể quản lý version (Git)
    - Theo dõi được ai đã chỉnh sửa
    - Làm việc nhóm

- Thêm một lý do nữa là chúng ta cần kiểm soát bảo mật trước khi apply

> [!IMPORTANT]
> Terraform không phải là phần của CI/CD ứng dụng, mà là CI/CD cho cơ sở hạ tầng.

- Kết luận:
    - Terraform nên nằm trong repo riêng &rarr; `infra-*`
    - Jenkins CI nên clone repo Terraform để kiểm tra trước khi apply
    - Jenkins CI clone repo chứa source code → build + scan + push image. Nếu Terraform an toàn thì sẽ được push đến Jenkins CD.
    - GitOps repo (Helm/Kustomize) được Jenkins cập nhật &rarr; ArgoCD lo phần CD (Nếu dùng ArgoCD)

> **Tại sao Terraform cần phải kiểm tra bảo mật?**

- Vì Terraform là Infrastructure as Code (IaC), nên nếu trong file `.tf` có lỗi như:

    - Gán public access cho S3 bucket

    - Tạo security group mở toàn bộ cổng (0.0.0.0/0) cho EC2 hoặc RDS

    - IAM Role quá rộng (gán AdministratorAccess)

    - Không bật encryption cho dữ liệu (EBS, RDS, S3…)

> [!WARNING]
> Hacker có thể khai thác lỗ hổng ngay từ cấu hình hạ tầng, dù code ứng dụng có an toàn.

- Các công cụ scan bảo mật Terraform phổ biến:

|Công cụ | Chức năng chính | Gợi ý tích hợp|
|--------|-----------------|---------------|
| tfsec	| can Terraform code để phát hiện lỗ hổng bảo mật | Dễ dùng, có thể chạy trong Jenkins
| Checkov | Kiểm tra best practices, compliance & security| Hỗ trợ nhiều IaC tools, mạnh mẽ| 
Terrascan | Static analysis cho Terraform | Hợp với DevSecOps|
| AWS Config Rules	| Scan cấu hình thực tế (runtime) trên AWS | Sau khi Terraform chạy xong|

> **Jenkins có thể tạo image khi GitHub trigger đến Jenkins CI không?**

- Jenkins có thể được sử dụng để tạo Docker Image. Cụ thể, chúng ta có thể sử dụng Jenkins Pipeline để xây dựng Docker Image từ Dockerfile và đẩy chúng lên kho lưu trữ như Docker Hub.

- Sau đây là cách thức hoạt động:
    
    - Jenkins Pipeline: Cần định nghĩa Jenkins Pipeline, thường sử dụng Jenkinsfile, phác thảo các bước để xây dựng và push Docker Image.
    
    - Dockerfile: Pipeline của chúng ta sử dụng Dockerfile, là tệp văn bản có hướng dẫn để xây dựng Docker Image, bao gồm hình ảnh cơ sở, sao chép tệp, chạy lệnh và thiết lập biến môi trường.

    - Xây dựng Docker Image: Pipeline sử dụng các lệnh Docker, như docker build, để tạo Image dựa trên các hướng dẫn Dockerfile.
        
    - Push lên kho lưu trữ: Sau khi Image được xây dựng, Pipeline có thể đẩy Image đó lên kho lưu trữ Docker, như Docker Hub, nơi Image có thể được lưu trữ và chia sẻ.

> **Webhook là gì?**

- Là một cơ chế thông báo tự động từ GitHub đến Jenkins.

- Khi có hành động như: push, merge, pull request, GitHub sẽ gửi HTTP POST đến một URL mà chúng ta đã cấu hình, đó chính là Jenkins.

- Jenkins server phải public hoặc dùng ngrok/nginx reverse proxy nếu chúng ta đang test local.

> 
