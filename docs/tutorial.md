# DevSecOps - Deploying a YouTube Clone App on AWS EKS with Jenkins and Terraform

## Mục đích:
- Tận dụng sức mạnh của `Jenkins` và `Terraform` để triển khai an toàn ứng dụng YouTube Clone trên Amazon Elastic Kubernetes Service `(EKS)`.
- Học cách tự động hóa quy trình triển khai phần mềm.
- Quản lý hạ tầng dưới dạng mã (IaC) với `Terraform`.
- Tích hợp liền mạch các thực hành bảo mật vào vòng đời phát triển phần mềm (DevSecOps).

## Tổng quan quy trình

- Tạo một kho Git để quản lý toàn bộ mã nguồn của ứng dụng YouTube Clone, bao gồm:
    - Mã nguồn ứng dụng frontend.
    - Dockerfile để đóng gói ứng dụng.
    - Jenkinsfile định nghĩa CI Pipeline.
    - Jenkinsfile-EKS-Terraform định nghĩa CD Pipeline.
    - Mã nguồn IaC viết bằng Terraform để triển khai cụm Amazon EKS và các thành phần hạ tầng liên quan.

- Khởi tạo một EC2 Instance để triển khai cả hai dịch vụ:
    - Jenkins: công cụ chính để điều phối các pipeline CI/CD.
    - SonarQube: công cụ phân tích mã nguồn tĩnh và kiểm tra bảo mật/chất lượng.

- Cấu hình Jenkins để tự động hóa toàn bộ quy trình CI/CD:
    - Kết nối Jenkins với GitHub qua webhook.
    - Cài đặt các plugin cho Docker, Kubernetes, SonarQube, Trivy và Slack để nhận thông báo.

- Xây dựng Jenkins CI Pipeline với các bước chính:
    - Kéo mã nguồn từ GitHub.
    - Phân tích mã nguồn với SonarQube để phát hiện lỗi, thực hành xấu, code smell và lỗ hổng bảo mật.
    - Build Docker Image từ Dockerfile.
    - Quét lỗ hổng bảo mật cho image bằng Trivy.
    - Đẩy image lên DockerHub nếu tất cả kiểm tra đều đạt.
    - Nếu kiểm tra thất bại, gửi thông báo qua Slack và dừng quy trình CI.

- Triển khai cụm Kubernetes trên AWS EKS bằng Terraform:
    - Sử dụng Terraform để định nghĩa, quản lý và tái sử dụng hạ tầng dưới dạng mã.
    - Tự động tạo cụm EKS, security group, role, node group và networking.

- Triển khai bằng Jenkins CD Pipeline:
    - Jenkins kéo image từ DockerHub và triển khai lên cụm EKS.
    - Áp dụng chiến lược Blue/Green Deployment để đảm bảo phát hành không ảnh hưởng đến người dùng cuối.

- Cảnh báo:
    - Gửi cảnh báo qua Slack khi có lỗi xảy ra.

## Các bước chi tiết

### Bước 1: Cài đặt AWS CLI
GitHub Repo cho Youtube-Clone: [Youtube-Clone](https://github.com/uniquesreedhar/Youtube-clone-app.git)

- Tạo một user với tên bất kỳ trên `AWS` console và lưu lại access keys.

- Cài đặt và cấu hình `Terraform` và `AWS CLI` trên máy local để tạo `Jenkins Server` trên AWS Cloud. Bao gồm chạy các script cài đặt cho Terraform và AWS CLI.

```bash
# Script cài đặt Terraform
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install terraform

# Script cài đặt AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

sudo apt install unzip -y

unzip awscliv2.zip

sudo ./aws/install
```

### Bước 2: Tạo Jenkins Server

- Cấu hình biến (variables.tf)

![Variable](/asset/variable-ec2.png)

- Định nghĩa tài nguyên EC2 (main.tf)

![Main](/asset/main-e2.png)

- Khởi tạo 2 server: Jenkins và SonarQube

- Lấy public IP từ output hoặc AWS Console để truy cập Jenkins và SonarQube qua trình duyệt.
    - Đảm bảo security group mở port 8080 (Jenkins), 9000 (SonarQube) và SSH (22).

### Bước 3: Cấu hình Jenkins Server

- Sao chép Public IP của `Jenkins Server` và dán vào trình duyệt với port 8080. Ví dụ: http://public-IP:8080

![Done Jenkins](/asset/doneJenkins.png)

- Chạy lệnh sau để lấy mật khẩu đăng nhập `Jenkins`:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Tuỳ chỉnh `Jenkins` bằng cách cài đặt các plugin được đề xuất hoặc chọn plugin để cài đặt.

- Chọn **Install suggested plugins**

- Tạo user nếu muốn đặt mật khẩu riêng mỗi lần khởi động lại

- Vào **Jenkins → Plugin → Available Plugins**. Cài đặt các plugin sau:
    - Eclipse Temurin Installer (Cài đặt không cần khởi động lại)
    - SonarQube Scanner (Cài đặt không cần khởi động lại)
    - NodeJs Plugin (Cài đặt không cần khởi động lại)
    - NodeJs Plugin (Cài đặt không cần khởi động lại)
    - Docker
    - Docker commons
    - Docker pipeline
    - Docker API
    - Docker Build step
    - Owasp Dependency Check
    - Terraform
    - Kubernetes
    - Kubernetes CLI
    - Kubernetes Client API
    - Kubernetes Pipeline DevOps steps
    - AWS Credentials
    - Pipeline: AWS Steps
    - Slack notifications

![Install Plugins](/asset/installPlugins.png)    

- Đi đến Jenkins Dashboard → Manage Jenkins → Credentials (AWS, GitHub, Docker, Sonarqube)

- Truy cập Manage Jenkins → Global Tool Configuration.
- Thiết lập các công cụ cần thiết:
        - **jdk17:**
![Config JDK](/asset/configJDK.png)
        - **NodeJS24.0.2**
![Config NodeJS](/asset/configNodeJS.png)
        - **SonarQube Scanner**
![Add Sonar Server](/asset/configSonar-scanner.png)
        - **DP Check**
![Add DP Check](/asset/configDP-Check.png)
        - **Docker**
![Config Docker](/asset/configDocker.png)

- Đi đến Jenkins Dashboard → Manage Jenkins → Credentials
    - Thêm các credentials:
        - AWS
![AWS credential](/asset/addAWSCredentials.png)
        - GitHub
![GitHub credential](/asset/addGithubCredentials.png)
        - Sonar-token
![Sonarqube credential](/asset/addSonarCredentials.png)
        - Docker
![Docker credential](/asset/addDockerCredentials.png)


### Bước 4: Cấu hình máy chủ Sonarqube

- Lấy địa chỉ IP Public của EC2 Instance, SonarQube chạy trên cổng 9000, vì vậy hãy truy cập SonarQube bằng trình duyệt tại địa chỉ <Public IP>:9000.

![Done Sonar](/asset/doneSonar.png)

- Nhập mật khẩu và tên người dùng trong sonarqube:
    - Username: admin
    - Password: admin

- Cập nhật mật khẩu

- Nhấn vào Administration → Security → Users → Chọn Tokens and Update Token → Đặt tên cho Token → và chọn Generate Token.

![Create Token For Jenkins On Sonar](/asset/createTokenForJenkinsOnSonar.png)

- Copy Token

- Trong SonarQube Dashboard, tiến hành thêm **quality gate**
    - Administration → Configuration → Webhooks
    - Chọn Create
    - Nhập thông tin dưới đây:
        Name: Jenkins
        #in url section of quality gate
        <http://jenkins-public-ip:8080>/sonarqube-webhook/

![Webhook for Jenkins on Sonar](/asset/createWebhookForJenkinsOnSonar.png)

### Bước 5: Tích hợp Slack trong Jenkins để thông báo
- **Tạo tài khoản và workspace Slack**
  - Đăng ký hoặc đăng nhập vào Slack tại [https://slack.com/](https://slack.com/).
  - Tạo workspace mới hoặc sử dụng workspace hiện có.

-  **Thêm ứng dụng Jenkins CI vào Slack**
    - Trong Slack, nhấn vào tên workspace → Tools and Settings → Manage apps (Quản lý ứng dụng).

![Mange Slack app](/asset/manageAppSlack.png)

- Tìm kiếm "Jenkins CI" và chọn kết quả phù hợp.

![Add Jenkins CI to Slack](/asset/addJenkinsIntoSlack.png)

- Nhấn Add to Slack (Thêm vào Slack).

- Chọn kênh Slack muốn nhận thông báo và nhấn Add Jenkins CI integration (Thêm tích hợp Jenkins CI).

- **Lấy thông tin xác thực từ Slack**
    - Sau khi thêm Jenkins CI, Slack sẽ cung cấp một **Integration Token** (hoặc Secret).
    - Ghi lại **team subdomain** và **integration token** để cấu hình trong Jenkins.

-  **Cài đặt và cấu hình plugin Slack trên Jenkins**
    - Di chuyển qua giao diện Jenkins, chọn **Manage Jenkins** → **Manage Credentials**.
        - Nhấn **Add Credentials**:
        - Scope: Global
         - Kind: Secret text
         - Secret: Dán Integration Token từ Slack
         - ID: Đặt tên dễ nhớ
         - Description: Nhập miêu tả

![Add Slack Credential](/asset/addSlackCredentials.png)

- Sau đó, di chuyển sang Manage Jenkins → System.
    - Workspace: Team subdomain được lấy từ Slack
    - Credential: Chọn Credential for slack app
    - Default channel: #tên_channel_của_nhóm

![Config Slack](/asset/configSlack.png)

-  **Kiểm tra kết nối**
  - Nhấn **Test Connection** để kiểm tra Jenkins đã gửi được thông báo đến Slack chưa.

![Test Slack](/asset/configSlackSuccessfully.png)  

### Bước 6: Tạo API key từ Rapid API

- Mở một tab mới trên trình duyệt và truy cập rapidapi.com.

- Đăng nhập hoặc đăng ký tài khoản bằng email tùy chọn.

- Sau khi tạo tài khoản thành công, tìm kiếm "YouTube" trên thanh tìm kiếm và chọn "YouTube v3".

- Sao chép API key được cung cấp.

![Get API](/asset/getAPIToken.png)

- Sử dụng API key này trong Jenkinsfile, phần build và push Docker image.

### Bước 7:  Tạo CI Pipeline

- Nhấn vào “New Item”. Đặt tên cho mục đó.

- Trong phần Pipeline, chọn “Pipeline script from SCM” Sau đó là URL kho lưu trữ GitHub của mã nguồn, nhập thông tin . Sau đó, cấu hình thông tin xác thực GitHub.

![Create Pipeline](/asset/createPipeline.png)

- Nhấp vào “Apply” rồi build.

![Done CI](/asset/doneCIpipeline.png)

- Sau khi hoàn thành các bước build và test, Docker Image sẽ được push lên Docker Hub

- Kiểm tra deploy bằng Docker

![Deploying Docker](/asset/deployusingdocker.png)

### Bước 8: Tạo Jenkins Pipeline cho EKS

- Trong phần Pipeline, chọn “Pipeline script from SCM”, sau đó chọn Git và cung cấp thông tin xác thực (credentials).

- Chỉ định nhánh là “main” và nhập đường dẫn đến file Jenkinsfile-EKS-Terraform.

![Create EKS Pipeline](/asset/createPipelineToCreateEKS.png)

- Nhấn Apply, sau đó chọn Build để bắt đầu.
 
- Quá trình này sẽ khởi tạo một cụm AWS EKS kèm theo các node group.

### Bước 9: Tạo CD Pipeline để deploy trên EKS

- Chạy lệnh sau trên máy Jenkins để cấu hình kết nối đến cụm EKS:
```bash
aws eks update-kubeconfig --region <region> --name <cluster_name>

kubectl get pods
```

- Lúc này sẽ xuất hiện lỗi yêu cầu cung cấp thông tin xác thực: “asked to provide credentials”.

- Để giải quyết, cần cài đặt công cụ aws-iam-authenticator.

- Tải binary từ Amazon S:
``` bash
curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.15.10/2020-02-22/bin/linux/amd64/aws-iam-authenticator
```

- Cấp quyền thực thi:
``` bash
chmod +x ./aws-iam-authenticator
```

- Di chuyển file đến thư mục hệ thống:
```bash
sudo mv ./aws-iam-authenticator /usr/local/bin
```

- Cấu hình AWS CLI:
```bash
aws configure
```
- Sao chép file cấu hình và lưu lại dưới dạng secret.txt.

- Trên Jenkins:
    - Vào Manage Jenkins → Credentials → Global.
    - Thêm mới credential dạng Secret file.
    - ID đặt là k8s (hoặc tên tùy ý, nhớ đồng bộ với pipeline).

- Thêm stage deploy vào Jenkins Pipeline
```bash
stage('Deploy to kubernets'){
            steps{
                withAWS(credentials: 'aws-key', region: 'us-east-1') {
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                        sh 'kubectl apply -f deployment.yml'
                    }
                }
            }   
        }
    }
```

- Sau khi deploy, kiểm tra tài nguyên trên EKS:
```bash
kubectl get all
```

- Sẽ thấy một Load Balancer được tạo trên AWS.

![Load Balancer](/asset/CI-CD-Total/load-balancer-security.png)

- Copy DNS name của Load Balancer và dán vào trình duyệt để truy cập ứng dụng.

### Bước 10: Dọn dẹp tài nguyên

- Trong Jenkins, tại pipeline EKS - Terraform, nhấn “Build with Parameters” và chọn action là “destroy” để bắt đầu quá trình huỷ cụm EKS và các tài nguyên liên quan.

- Nếu làm trên máy local, chạy lệnh sau để huỷ toàn bộ server và hạ tầng đã tạo:
```bash
terraform destroy
``` 