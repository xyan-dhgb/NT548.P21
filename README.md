# Deploying a YouTube Clone App on AWS EKS with Jenkins and Terraform

- Đây là đề tài của đồ án môn học ***Công nghệ DevOps và ứng dụng - NT548.P21*** do **nhóm 11** thực hiện 
- Tên tiếng Việt: Triển khai ứng dụng YouTube trên AWS EKS với Jenkins và Terraform
- Thành viên thực hiện:
    - Nguyễn Duy Hùng (22520513 - MMTT2022.1)
    - Đinh Huỳnh Gia Bảo (22520101 - MMTT2022.1)
    - Mai Kim Trinh (22521537 - MMTT2022.3)
    - Lê Thị Tú Uyên (22521639 - MMTT2022.3)

## Tổng quát

- Đồ án này nhằm cung cấp cái nhìn toàn diện về quy trình làm việc của DevOps và tư duy sử dụng công nghệ để giải quyết bài toán được đưa ra. 

- Nói một cách chi tiết, đây là dự án nhằm thực hiện triển khai một ứng dụng YouTube Clone trên cụm Kubernetes được quản lý bởi AWS Elastic Kubernetes Service (EKS), sử dụng công nghệ Jenkins để tự động hóa CI/CD và Terraform để quản lý hạ tầng AWS.

## Phân tích giải pháp

### Giải pháp gốc: [Giải pháp](https://medium.com/@madithatisreedhar123/devsecops-deploying-a-youtube-clone-app-on-aws-eks-with-jenkins-and-terraform-4909a6f1b299)

![Giải pháp gốc](/asset/reference-img.webp)

- **Tổng quan:** Minh họa quy trình pipeline CI/CD truyền thống có tích hợp các công cụ bảo mật (DevSecOps).

- **Ưu điểm:** 
    - Có đầy đủ chuỗi CI/CD tích hợp bảo mật.
    - Hiển thị rõ các công cụ: SonarQube (Phân tích mã nguồn), OWASP (Quét gói phụ thuộc), Trivy (Quét image).
    - Sử dụng Terraform để triển khai Jenkins và EKS.

- **Nhược điểm:**
    - Giao diện phức tạp, thiếu sự phân tách rõ ràng giữa các tầng (Dev, CI, CD, Deploy).
    - Không phân nhóm logic các thành phần (Monitoring, Registry, Deployment Strategy...).

### Giải pháp cải tiến: Phân nhóm, áp dụng deployment strategies

- **Mô hình**:
![Giải pháp 1](/asset/first-solution.png)

- **Ưu điểm**: 
    - Phân vùng rõ ràng: Giữa Development, CI Pipeline, CD Pipeline, Deployment Strategies, AWS Cloud.
    - Có thêm các chiến lược triển khai: Blue/Green và Rollback → giúp tăng độ tin cậy cho production.
    - Monitoring tốt: Kết hợp Prometheus, Grafana, CloudWatch, Slack – thể hiện tính thực tế & observability cao.
    - Phân biệt rõ thành phần trong AWS EKS: Frontend, Backend, DB, Cache (nếu triển khai web động)
    - Bổ sung các công cụ bảo mật (Trivy, SonaQube) trong Testing & Security.

- **Nhược điểm**: 
    - Thiếu công cụ log như Splunk (ảnh gốc có) – bạn đang có Prometheus/Grafana nhưng log system thì chưa rõ.
    - Không có minh họa cụ thể luồng “người dùng” truy cập (frontend UI) 

- **Công nghệ sử dụng**: 

    - Môi trường làm việc: Vscode
    - Quản lý phiên bản và lưu trữ code: Git, Github
    - Tối ưu hóa quy trình CI/CD: Jenkins
    - Đóng gói ứng dụng và lưu trữ image: Docker, DockerHub
    - Xây dựng cơ sở hạ tầng AWS: Terraform
    - Quản lý cấu hình: Kubernete, Helm Charts
    - Quản lý cụm Kubernetes: AWS EKS
    - Bảo mật:
        - Phân tích chất lượng code: SonarQube
        - Quét lỗ hổng container: Trivy
    - Giám sát và cảnh báo: Prometheus, Grafana, AWS Cloudwatch