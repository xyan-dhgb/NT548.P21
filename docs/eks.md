# EKS - Elastic Kubernetes Service 

> Nền tảng hàng đầu để chạy cụm Kubernetes, cả trên  Amazon Web Services (AWS) và trong trung tâm dữ liệu.

![AWS EKS](/asset/aws-eks-logo.png)

## Đặt vấn đề 

- Bối cảnh:
    - Ứng dụng có nhiều microservices (backend, frontend, worker, v.v.) cần triển khai.
    - Mỗi dịch vụ chạy trong container (Docker), có yêu cầu tự động scaling, update, rollback, load balancing, secret management, ...
    - Các dịch vụ cần phải đảm bảo chạy ổn định trong môi trường production, phân phối ở nhiều AZ (availability zones), có giám sát, có bảo mật.

- Vấn đề:
    - Chúng ta không thể quản lý thủ công các container bằng một câu lệnh `docker run.`
    - Nếu dùng ECS (Elastic Container Service) thì không đủ linh hoạt khi muốn custom logic (ví dụ: policy, advanced networking, tự define operator,...).

    ![Amazon Elastic Container Server](/asset/ecs-logo.webp)

    - Việc build 1 cụm Kubernetes tự quản lý từ đầu rất phức tạp, tốn công, khó bảo trì, nhất là security (certs, RBAC (Role-Based Access Control), HA,...).

- Đó là lúc EKS được "sinh ra" để khắc phục các nhược điểm đó. 

- EKS cung cấp Kubernetes as a Service, nhà phát triển chỉ lo phần app, AWS lo control plane. Đảm bảo HA (high availability), update, patching được tự động thực hiện.

- EKS có thể tích hợp với các service khác như IAM, CloudWatch, ALB (Application load balancer). NLB (Network Load Balancer), EBS (Elastic Block Storage), EFS (Elastic File System)

## Định nghĩa

### Kubernetes  - K8s (Cần nắm rõ)

#### Định nghĩa Kubernetes

- Kubernetes là một nền tảng mã nguồn mở, có thể mở rộng, di động để quản lý khối lượng công việc và dịch vụ được chứa trong container, hỗ trợ cả cấu hình khai báo và tự động hóa. Nó có một hệ sinh thái lớn, phát triển nhanh chóng. Các dịch vụ, hỗ trợ và công cụ Kubernetes có sẵn rộng rãi.

- Xem thêm ở [Slide](https://drive.google.com/file/d/1yhhD_puOqKYi5iWchg88-QbVZ_48Nj3s/view?usp=sharing)  và [Repo tự học Kubernetes](https://github.com/xyan-dhgb/Introduction-to-Kubernetes) này

#### Kiến trúc Kubernetes

![Kubernetes Cluster Architecture](/asset/kubernetes-cluster-architecture.svg)

- Một K8s cluster sẽ gồm tập các nodes chạy các ứng dung ảo hóa bang container. Trên mỗi node sẽ cần chạy một "kubelet", đây là chương trình để chạy k8s. Cần một node để làm "chủ" cluster, trên đó sẽ cài API server, scheduler ... Các máy còn lại sẽ chạy kubelet để sinh ra các container.

- Etcd: key-value stores chứa thông tin định danh, liên lạc giữa các nodes.

- API Server: server cung cấp Kubernetes API. Nó có nhiệm vụ đặt Pod vào Node, đồng bộ hoá thông tin của Pod bằng REST API tiếp nhận cài đặt của pod/service/replicationController.

- Controller Manager Service: “kube-controller manager”, quản lý tất cả các bộ điều khiển xử lý các tác vụ thông thường trong cluster. Chúng bao gồm Node Controller, Replication Controller, Endpoints Controller, and Service Account and Token Controllers.

- Scheduler Service: định thời, phân bố tài nguyên còn khả dung đến các ứng dung container

- Dashboard (optional)

### EKS 

- Amazon Elastic Kubernetes Service (EKS) cung cấp dịch vụ Kubernetes được quản lý hoàn toàn giúp loại bỏ sự phức tạp khi vận hành cụm Kubernetes. Với EKS, chúng ta có thể:

    - Triển khai ứng dụng nhanh hơn với chi phí vận hành thấp hơn
    - Mở rộng quy mô liền mạch để đáp ứng nhu cầu khối lượng công việc thay đổi
    - Cải thiện bảo mật thông qua tích hợp AWS và cập nhật tự động
    - Chọn giữa EKS tiêu chuẩn hoặc EKS hoàn toàn tự động

#### Các khái niệm riêng của EKS

- **EKS Control Plane**: Thành phần quản lý của Kubernetes được AWS vận hành, đảm bảo tính sẵn sàng cao và bảo mật.

- **Node Group**: Nhóm các EC2 instance hoặc tác vụ Fargate dùng để chạy workload (Pod) trong cụm.

- **Fargate for EKS**: Cho phép chạy Pod mà không cần quản lý EC2, theo mô hình serverless.

- **IAM Roles for Service Account (IRSA)**: Cho phép Pod trong EKS assume IAM role để truy cập tài nguyên AWS một cách an toàn.

- **EKS Add-ons**: Các plugin được quản lý như CoreDNS, VPC CNI, kube-proxy... giúp mở rộng chức năng cho cụm EKS.

#### VPC Networking với EKS

- Mỗi Pod dùng IP thật từ VPC CIDR (AWS VPC CNI).

- Phân biệt public/private subnet, NAT gateway, route table.

####  Cluster Security

- RBAC: Ai được làm gì trong cluster.

- IAM Integration: Mapping giữa IAM user/role và Kubernetes RBAC (aws-auth ConfigMap).

- NetworkPolicy: Hạn chế traffic giữa pods.

- Secrets encryption: Sử dụng KMS (Key Management System) để mã hóa Kubernetes secrets.

## Phân loại EKS

- Amazon EKS đơn giản hóa việc xây dựng, bảo mật và duy trì các cụm Kubernetes. Nó có thể tiết kiệm chi phí hơn khi cung cấp đủ tài nguyên để đáp ứng nhu cầu cao điểm so với việc duy trì các trung tâm dữ liệu. Hai trong số các cách tiếp cận chính để sử dụng Amazon EKS như sau:

![EKS standard và EKS Auto Mode](/asset/eks-standard-and-eks-auto-mode.png)

### EKS standard

- AWS quản lý control plane Kubernetes khi tạo cụm với EKS

### EKS Auto Mode

- EKS mở rộng khả năng kiểm soát của mình để quản lý cả các Node (data plane của Kubernetes).

## Đặc điểm của Amazon EKS

Các tính năng nổi bật của Amazon EKS

### Giao diện quản lý

- EKS cung cấp nhiều giao diện để tạo, quản lý và duy trì các cluster, bao gồm: AWS Management Console, Amazon EKS API/SDKs, AWS CDK, AWS CLI, eksctl CLI, AWS CloudFormation và Terraform.

### Công cụ kiểm soát truy cập

- EKS kết hợp các tính năng kiểm soát truy cập của cả Kubernetes và AWS IAM để quản lý quyền truy cập của người dùng và workload.

### Tài nguyên tính toán

- EKS hỗ trợ đầy đủ các loại EC2 instance, bao gồm cả các công nghệ mới như Nitro và Graviton, giúp tối ưu hóa hiệu suất và chi phí cho workload.

### Lưu trữ

- EKS Auto Mode tự động tạo các storage class sử dụng EBS volumes.
- Có thể tích hợp thêm các driver CSI để sử dụng Amazon S3, EFS, FSx, và File Cache cho nhu cầu lưu trữ ứng dụng.

### Bảo mật

- EKS áp dụng mô hình chia sẻ trách nhiệm về bảo mật giữa AWS và khách hàng.

### Công cụ giám sát

- Có thể sử dụng dashboard quan sát, Prometheus, CloudWatch, CloudTrail, và ADOT Operator để theo dõi cụm EKS.

### Tương thích và hỗ trợ Kubernetes

- Amazon EKS đạt chứng nhận Kubernetes-conformant, cho phép triển khai ứng dụng tương thích Kubernetes mà không cần chỉnh sửa lại.

- Hỗ trợ cả phiên bản tiêu chuẩn và mở rộng của Kubernetes.

## Dich vụ tương ứng

Chúng ta có thể sử dụng các dịch vụ AWS sau với các cụm được triển khai bằng Amazon EKS:

### Amazon EC2

![EC2](/asset/ec2.jpeg)

- Cung cấp tài nguyên tính toán theo nhu cầu, có khả năng mở rộng với Amazon EC2.

### Amazon EBS

![EBS](/asset/ebs.webp)

- Gắn thêm tài nguyên lưu trữ dạng block hiệu năng cao, có khả năng mở rộng với Amazon EBS.

### Amazon ECR

![ECR](/asset/ecr.png)

- Lưu trữ hình ảnh container một cách an toàn với Amazon ECR.

### Amazon CloudWatch

![Cloudwatch](/asset/cloudwatch.png)

- Giám sát tài nguyên AWS và ứng dụng theo thời gian thực với Amazon CloudWatch.

### Amazon Managed Service for Prometheus

![Managed Service for Prometheus](/asset/aws-prometheus.jpeg)

Theo dõi các chỉ số cho ứng dụng container hóa với Amazon Managed Service for Prometheus.

### Elastic Load Balancing

![ELB](/asset/elb.jpeg)

- Phân phối lưu lượng truy cập đến nhiều mục tiêu với Elastic Load Balancing.

### Amazon GuardDuty

![GuardDuty](/asset/guard.png)

- Phát hiện các mối đe dọa đối với cụm EKS với Amazon GuardDuty.

### AWS Resilience Hub

![Resilience Hub](/asset/res.jpeg)

- Đánh giá khả năng chịu lỗi của cụm EKS với AWS Resilience Hub.

## Các công nghệ quản lý tương đồng với EKS

| Công nghệ                          | Mô tả                        | Điểm mạnh                                     |
| ---------------------------------- | ---------------------------- | --------------------------------------------- |
| **GKE (Google Kubernetes Engine)** | Dịch vụ Kubernetes của GCP   | Native với GCP IAM, dễ tích hợp ML            |
| **AKS (Azure Kubernetes Service)** | Dịch vụ Kubernetes của Azure | Tích hợp với Azure AD, support tốt enterprise |
| **Rancher**                        | UI quản lý nhiều cluster K8s | Dễ dàng self-host và hybrid                   |
| **OpenShift (RedHat)**             | Enterprise K8s platform      | Cung cấp CI/CD, registry, RBAC đầy đủ         |
| **k3s / MicroK8s**                 | K8s nhẹ cho testing/local    | Tốt cho lab và thiết bị edge                  |


## Tham khảo

[1] [Kubernetes](https://kubernetes.io/docs/concepts/overview/)

[2] [Kubernetes Architecture](https://kubernetes.io/docs/concepts/architecture/)

[3] [EKS Overview](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html#eks-features)