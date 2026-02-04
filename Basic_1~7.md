# Hướng dẫn Terraform Chi Tiết - Tiếng Việt
## Phần 1-7: Từ Cơ bản đến Thực hành

---

## PHẦN 1: TERRAFORM LÀ CÁI GÌ?

### 1.1 Định nghĩa đơn giản

**Terraform** là một công cụ giúp bạn **tự động hóa việc tạo ra hạ tầng công nghệ thông qua mã code**.

Thay vì:
- ✗ Vào AWS Console, click chuột tạo EC2 (lâu, dễ sai)
- ✓ Viết file `.tf`, chạy lệnh `terraform apply` (nhanh, chính xác, có thể lặp lại)

### 1.2 Ví dụ thực tế

**Mà không dùng Terraform:**
```
Bước 1: Mở AWS Console → EC2
Bước 2: Click "Launch Instance"
Bước 3: Chọn AMI (hệ điều hành)
Bước 4: Chọn Instance Type (máy yếu hay mạnh)
Bước 5: Configure Network (VPC, Subnet)
Bước 6: Add Storage (ổ cứng)
Bước 7: Add Tags (tên, project, ...)
Bước 8: Configure Security Group (tường lửa)
Bước 9: Click "Launch"
Bước 10: Chờ 2-3 phút
```

**Dùng Terraform:**
```hcl
resource "aws_instance" "my_server" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  
  tags = {
    Name = "My Server"
  }
}
```
Sau đó chỉ cần chạy:
```bash
terraform apply
```
Xong! 30 giây là máy chủ chạy rồi.

### 1.3 Vì sao nên dùng Terraform?

| Lợi ích | Giải thích |
|---------|-----------|
| **Tự động hóa** | Không cần click chuột trong AWS Console |
| **Có thể lặp lại** | Tạo cùng hạ tầng lần thứ 2, 3, 4 dễ dàng |
| **Kiểm soát phiên bản** | Lưu code vào Git, biết ai thay đổi cái gì |
| **Kiểm tra trước** | `terraform plan` cho thấy sẽ thay đổi gì |
| **Xóa nhanh** | `terraform destroy` xóa tất cả chỉ 1 lệnh |
| **Làm việc nhóm** | Cả team dùng chung cấu hình |
| **Tiết kiệm chi phí** | Tự động tắt máy khi không cần |

### 1.4 Terraform hoạt động như thế nào?

```
┌─────────────┐
│ Bạn viết code│
│  (main.tf)  │
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│ Terraform đọc code  │
│ và hiểu bạn muốn gì │
└──────┬──────────────┘
       │
       ▼
┌──────────────────────────┐
│ Terraform nói với AWS    │
│ qua API: "Tạo máy chủ    │
│ với config này nhé!"     │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────┐
│ AWS tạo máy chủ      │
│ theo yêu cầu         │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│ Terraform lưu lại:   │
│ "Máy chủ đã tạo!"    │
│ (state file)         │
└──────────────────────┘
```

### 1.5 Terraform dùng ngôn ngữ gì?

**HCL (HashiCorp Configuration Language)** - Ngôn ngữ được thiết kế đặc biệt cho Terraform.

Ví dụ HCL:
```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  region        = "us-east-1"
  
  tags = {
    Name    = "My Web Server"
    Project = "My App"
  }
}
```

---

## PHẦN 2: CÀI ĐẶT TERRAFORM

### 2.1 Yêu cầu trước khi cài

- Máy tính có Internet
- Quyền admin (để cài phần mềm)

### 2.2 Cài trên Windows

**Cách 1: Dùng Chocolatey (Dễ nhất)**

```bash
# Bước 1: Mở PowerShell (chạy với quyền admin)
choco install terraform -y

# Bước 2: Kiểm tra đã cài đúng chưa
terraform -v
# Kết quả hiển thị: Terraform v1.5.0
```

**Cách 2: Tải tay từ website**

1. Vào https://www.terraform.io/downloads
2. Tải file Windows (64-bit)
3. Giải nén vào thư mục (ví dụ: `C:\terraform`)
4. Thêm vào PATH của hệ thống

### 2.3 Cài trên Mac

```bash
# Nếu có Homebrew
brew install terraform

# Hoặc tải từ website theo hướng dẫn
```

### 2.4 Cài trên Linux

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install terraform

# Hoặc CentOS/RHEL
sudo yum install terraform
```

### 2.5 Kiểm tra cài đặt

Mở Terminal/PowerShell, gõ:
```bash
terraform -v
```

Kết quả sẽ hiển thị phiên bản như:
```
Terraform v1.5.0
on darwin_arm64
```

✓ Nếu thấy được phiên bản = **Cài đặt thành công!**

---

## PHẦN 3: NHỮNG KHÁI NIỆM CƠ BẢN

### 3.1 Provider (Nhà cung cấp)

**Provider** = Cầu nối giữa Terraform và dịch vụ đám mây.

Terraform không biết cách tạo máy chủ trong AWS, nó cần một **plugin** gọi là **Provider**.

**Ví dụ:**

```hcl
# Provider AWS
provider "aws" {
  region = "us-east-1"
}

# Provider Azure
provider "azurerm" {
  version = "~> 3.0"
}

# Provider Google Cloud
provider "google" {
  project = "my-gcp-project"
  region  = "us-central1"
}
```

**Các provider phổ biến:**
- `aws` - Amazon Web Services
- `azurerm` - Microsoft Azure
- `google` - Google Cloud Platform
- `digitalocean` - DigitalOcean
- `kubernetes` - Kubernetes
- `docker` - Docker

### 3.2 Resource (Tài nguyên)

**Resource** = Những thứ bạn muốn tạo ra.

Nếu Provider là "công ty xây dựng", thì Resource là "những gì công ty sẽ xây": nhà, cửa, cửa sổ, v.v.

**Ví dụ tạo máy chủ AWS:**

```hcl
resource "aws_instance" "my_server" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  
  tags = {
    Name = "My First Server"
  }
}
```

**Giải thích:**
- `resource` = Từ khóa (bắt buộc)
- `aws_instance` = Loại tài nguyên (tạo máy chủ AWS)
- `my_server` = Tên bạn đặt cho nó (để tham chiếu sau)
- `ami` = Hệ điều hành (Windows, Linux, ...)
- `instance_type` = Cấu hình máy (yếu hay mạnh)

**Ví dụ tạo bucket S3:**

```hcl
resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-unique-bucket-name"
  
  tags = {
    Name = "My Storage"
  }
}
```

**Các resource phổ biến:**
- `aws_instance` - Máy chủ EC2
- `aws_s3_bucket` - Kho lưu trữ S3
- `aws_vpc` - Mạng riêng ảo
- `aws_security_group` - Tường lửa
- `aws_db_instance` - Cơ sở dữ liệu RDS

### 3.3 Variable (Biến)

**Variable** = Giống như "ô trống" mà bạn có thể điền giá trị vào.

Thay vì viết cứng giá trị, bạn khai báo biến và dùng lại nhiều lần.

**Ví dụ tạo biến:**

```hcl
variable "instance_type" {
  description = "Loại máy chủ (yếu hay mạnh)"
  type        = string
  default     = "t2.micro"
}

variable "environment" {
  description = "Môi trường (dev, prod)"
  type        = string
  default     = "dev"
}

variable "server_count" {
  description = "Số lượng máy chủ"
  type        = number
  default     = 1
}
```

**Dùng biến:**

```hcl
resource "aws_instance" "server" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = var.instance_type  # Dùng biến ở đây
  
  tags = {
    Name = "Server-${var.environment}"
  }
}
```

**Lợi ích:**
- ✓ Tái sử dụng code
- ✓ Dễ thay đổi giá trị
- ✓ Code gọn hơn

### 3.4 Data Source (Nguồn dữ liệu)

**Data Source** = Lấy thông tin từ AWS mà không tạo gì mới.

Ví dụ: Bạn muốn dùng **hình ảnh Windows 2022 mới nhất** từ AWS, nhưng bạn không biết ID của nó. Data Source sẽ giúp bạn tìm ra.

**Ví dụ:**

```hcl
# Tìm hình ảnh Amazon Linux 2023 mới nhất
data "aws_ami" "latest_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["al2023-ami-2023*"]
  }
}

# Dùng thông tin này để tạo máy chủ
resource "aws_instance" "server" {
  ami           = data.aws_ami.latest_linux.id  # ID lấy từ data source
  instance_type = "t2.micro"
}
```

**Khác biệt giữa Resource và Data Source:**

| Resource | Data Source |
|----------|------------|
| **Tạo mới** | **Chỉ đọc** thông tin |
| VD: tạo máy chủ | VD: lấy ID hình ảnh |
| Bạn quản lý nó | AWS quản lý sẵn |

### 3.5 Output (Kết quả)

**Output** = Hiển thị thông tin sau khi tạo xong.

Sau khi Terraform tạo máy chủ, bạn muốn biết địa chỉ IP, ID, v.v. Output sẽ giúp hiển thị.

**Ví dụ:**

```hcl
resource "aws_instance" "server" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
}

# Hiển thị thông tin
output "server_id" {
  description = "ID của máy chủ"
  value       = aws_instance.server.id
}

output "server_ip" {
  description = "Địa chỉ IP"
  value       = aws_instance.server.public_ip
}
```

**Sau khi `terraform apply`, kết quả hiển thị:**
```
Outputs:

server_id = "i-0a1b2c3d4e5f6g7h8"
server_ip = "54.175.102.97"
```

---

## PHẦN 4: CẤU TRÚC FILE TERRAFORM

### 4.1 File Terraform cơ bản

Một dự án Terraform thường có các file:

```
my-project/
├── provider.tf      ← Khai báo provider (AWS, Azure, ...)
├── variables.tf     ← Khai báo các biến
├── main.tf          ← Khai báo resources (máy chủ, storage, ...)
├── outputs.tf       ← Khai báo outputs (kết quả hiển thị)
├── terraform.tfvars ← Giá trị của biến
└── .gitignore       ← File không push lên Git
```

### 4.2 File `provider.tf` - Kết nối AWS

```hcl
terraform {
  required_version = "~> 1.0.0"  # Phiên bản Terraform
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Kết nối với AWS
provider "aws" {
  region = "us-east-1"  # Khu vực (Ireland, Singapore, ...)
  
  # Gắn tag cho tất cả resource
  default_tags {
    tags = {
      Terraform = "yes"          # Để biết tạo bởi Terraform
      Project   = "my-app"
      Owner     = "DevOps Team"
    }
  }
}
```

### 4.3 File `variables.tf` - Khai báo biến

```hcl
# Biến string (text)
variable "instance_type" {
  description = "Loại máy chủ"
  type        = string
  default     = "t2.micro"
}

# Biến number (số)
variable "server_count" {
  description = "Số lượng máy chủ"
  type        = number
  default     = 1
}

# Biến list (danh sách)
variable "allowed_ports" {
  description = "Cổng được phép"
  type        = list(number)
  default     = [22, 80, 443]
}

# Biến map (bộ từ điển)
variable "tags" {
  description = "Các thẻ"
  type        = map(string)
  default = {
    Environment = "dev"
    Team        = "DevOps"
  }
}

# Biến nhạy cảm (mật khẩu, token)
variable "db_password" {
  description = "Mật khẩu database"
  type        = string
  sensitive   = true  # Không hiển thị trong log
}
```

### 4.4 File `main.tf` - Tạo resources

```hcl
# Tạo máy chủ EC2
resource "aws_instance" "web_server" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = var.instance_type  # Dùng biến
  
  tags = {
    Name = "Web Server"
  }
}

# Tạo security group (tường lửa)
resource "aws_security_group" "web" {
  name = "web-sg"
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "Web Security Group"
  }
}
```

### 4.5 File `outputs.tf` - Hiển thị kết quả

```hcl
output "instance_id" {
  description = "ID của máy chủ"
  value       = aws_instance.web_server.id
}

output "instance_ip" {
  description = "Địa chỉ IP công cộng"
  value       = aws_instance.web_server.public_ip
}

output "security_group_id" {
  description = "ID của security group"
  value       = aws_security_group.web.id
}
```

### 4.6 File `terraform.tfvars` - Giá trị biến

```hcl
# Ghi các giá trị bạn muốn dùng
instance_type = "t2.small"
server_count  = 2
db_password   = "my-secure-password"

tags = {
  Environment = "production"
  Team        = "DevOps"
}
```

### 4.7 File `.gitignore` - Không push lên Git

```
# Không push những file này
.terraform/
*.tfstate
*.tfstate.*
.terraform.lock.hcl
terraform.tfvars  # Chứa mật khẩu
```

---

## PHẦN 5: CÁC LỆNH CƠ BẢN

### 5.1 Workflow từng bước

```
┌──────────────────────────┐
│ terraform init           │ Tải plugin, chuẩn bị sẵn sàng
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ terraform validate       │ Kiểm tra code có lỗi không
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ terraform fmt            │ Sắp xếp code gọn gàng
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ terraform plan           │ Xem sẽ tạo gì (không thực hiện)
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ terraform apply          │ Thực sự tạo resources
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ terraform destroy        │ (Tùy chọn) Xóa tất cả
└──────────────────────────┘
```

### 5.2 Lệnh `terraform init` - Khởi tạo

**Công dụng:** Tải plugin AWS, chuẩn bị Terraform.

```bash
terraform init
```

**Kết quả:**
```
Initializing the backend...
Initializing provider plugins...
Terraform has been successfully initialized!
```

**Cần chạy khi:**
- ✓ Lần đầu tiên
- ✓ Thêm provider mới
- ✓ Thêm backend mới

### 5.3 Lệnh `terraform validate` - Kiểm tra

**Công dụng:** Kiểm tra code có lỗi cú pháp không.

```bash
terraform validate
```

**Nếu không có lỗi:**
```
Success! The configuration is valid.
```

**Nếu có lỗi:**
```
Error: Missing required argument

  on main.tf line 5, in resource "aws_instance" "web":
   5:   instance_type = "t2.micro"

Missing required argument: "ami". Instance AMI required.
```

### 5.4 Lệnh `terraform fmt` - Sắp xếp code

**Công dụng:** Tự động sắp xếp code đẹp (căn lề, khoảng cách).

```bash
# Sắp xếp thư mục hiện tại
terraform fmt

# Sắp xếp toàn bộ folder (bao gồm subfolder)
terraform fmt -recursive
```

### 5.5 Lệnh `terraform plan` - Xem trước

**Công dụng:** Xem **sẽ tạo/sửa/xóa gì** mà không thực hiện.

```bash
terraform plan
```

**Kết quả hiển thị:**
```
Terraform will perform the following actions:

  # aws_instance.web_server will be created
  + resource "aws_instance" "web_server" {
      + ami                    = "ami-0df435f331839b2d6"
      + instance_type          = "t2.micro"
      + id                     = (known after apply)
      + public_ip              = (known after apply)
      + ... (nhiều thuộc tính khác)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

**Ký hiệu:**
- `+` = Tạo mới
- `~` = Sửa
- `-` = Xóa
- `±` = Tạo rồi xóa (thay thế)

### 5.6 Lệnh `terraform apply` - Tạo thật

**Công dụng:** Thực sự tạo resources trên AWS.

```bash
# Cách 1: Hỏi xác nhận
terraform apply

# Cách 2: Không hỏi (cẩn thận!)
terraform apply -auto-approve
```

**Quy trình:**
```
Đọc code → Kiểm tra → Tính toán → Hiển thị plan → Hỏi bạn "Bạn có chắc chắn không? (yes/no)"

Nếu bạn gõ "yes" → Thực hiện
```

**Kết quả sau khi thành công:**
```
aws_instance.web_server: Creation complete after 30s [id=i-0a1b2c3d4e5f6g7h8]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

instance_id = "i-0a1b2c3d4e5f6g7h8"
instance_ip = "54.175.102.97"
```

### 5.7 Lệnh `terraform destroy` - Xóa toàn bộ

**Công dụng:** Xóa **tất cả resources** đã tạo.

⚠️ **CẢNH BÁO:** Dùng cẩn thận! Xóa rồi khó khôi phục.

```bash
# Cách 1: Hỏi xác nhận
terraform destroy

# Cách 2: Không hỏi (rất rất cẩn thận!)
terraform destroy -auto-approve
```

**Kết quả:**
```
Destroy complete! Resources: 1 destroyed.
```

### 5.8 Lệnh `terraform refresh` - Cập nhật trạng thái

**Công dụng:** Cập nhật thông tin từ AWS (trong trường hợp ai đó sửa trực tiếp trên AWS Console).

```bash
terraform refresh
```

---

## PHẦN 6: VÍ DỤ THỰC TẾ - TẠO MÁY CHỦ AWS

### 6.1 Thiết lập dự án

**Bước 1:** Tạo thư mục dự án

```bash
mkdir my-terraform-project
cd my-terraform-project
```

**Bước 2:** Tạo các file

```bash
touch provider.tf variables.tf main.tf outputs.tf terraform.tfvars .gitignore
```

### 6.2 Viết file `provider.tf`

```hcl
terraform {
  required_version = "~> 1.0.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"  # Chọn khu vực gần bạn
  
  default_tags {
    tags = {
      Terraform = "yes"
      Project   = "My First Project"
    }
  }
}
```

### 6.3 Viết file `variables.tf`

```hcl
variable "instance_type" {
  description = "Loại máy chủ"
  type        = string
  default     = "t2.micro"  # Miễn phí 1 năm!
}

variable "server_name" {
  description = "Tên máy chủ"
  type        = string
  default     = "My Web Server"
}

variable "environment" {
  description = "Môi trường"
  type        = string
  default     = "dev"
}
```

### 6.4 Viết file `main.tf`

```hcl
# Lấy hình ảnh Linux mới nhất
data "aws_ami" "latest_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["al2023-ami-2023*"]
  }
  
  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
}

# Tạo máy chủ
resource "aws_instance" "web" {
  ami           = data.aws_ami.latest_linux.id  # Lấy từ data source
  instance_type = var.instance_type              # Lấy từ biến
  
  tags = {
    Name        = var.server_name
    Environment = var.environment
  }
}
```

### 6.5 Viết file `outputs.tf`

```hcl
output "instance_id" {
  description = "ID của máy chủ"
  value       = aws_instance.web.id
}

output "instance_ip" {
  description = "Địa chỉ IP công cộng"
  value       = aws_instance.web.public_ip
}

output "instance_type" {
  description = "Loại máy chủ"
  value       = aws_instance.web.instance_type
}
```

### 6.6 Viết file `terraform.tfvars`

```hcl
instance_type = "t2.micro"
server_name   = "My First Server"
environment   = "dev"
```

### 6.7 Viết file `.gitignore`

```
.terraform/
*.tfstate
*.tfstate.*
.terraform.lock.hcl
terraform.tfvars
```

### 6.8 Chạy lệnh

**Bước 1: Khởi tạo**
```bash
terraform init
```

**Bước 2: Kiểm tra**
```bash
terraform validate
```

**Bước 3: Xem trước**
```bash
terraform plan
```

Bạn sẽ thấy:
```
Terraform will perform the following actions:

  # aws_instance.web will be created
  + resource "aws_instance" "web" {
      + ami           = "ami-xxxxx"
      + instance_type = "t2.micro"
      + ...
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

**Bước 4: Tạo thật**
```bash
terraform apply
```

Terraform sẽ hỏi:
```
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: 
```

Gõ `yes` và Enter.

**Bước 5: Xem kết quả**
```
aws_instance.web: Creating...
aws_instance.web: Still creating... [10s elapsed]
aws_instance.web: Still creating... [20s elapsed]
aws_instance.web: Creation complete after 30s [id=i-0a1b2c3d4e5f6g7h8]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

instance_id = "i-0a1b2c3d4e5f6g7h8"
instance_ip = "54.175.102.97"
instance_type = "t2.micro"
```

✓ **Xong! Máy chủ đã tạo!**

Bạn có thể:
- SSH vào máy: `ssh -i your-key.pem ec2-user@54.175.102.97`
- Xem trên AWS Console: https://console.aws.amazon.com/ec2

**Bước 6: Xóa (khi không cần)**
```bash
terraform destroy
```

---

## PHẦN 7: HÀNH VI CỦA RESOURCE (TẠO, SỬA, XÓA)

### 7.1 Các trạng thái của Resource

Resource có 4 trạng thái:

```
1. Tạo mới (Create)
2. Sửa đổi (Update)
3. Xóa (Destroy)
4. Tạo lại (Recreate - xóa rồi tạo)
```

### 7.2 Khi nào Resource được **Tạo mới (Create)**?

Resource **chưa tồn tại** trong file config, nhưng bạn muốn tạo.

**Ví dụ:**

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
}
```

Khi chạy `terraform apply`:

```
+ resource "aws_instance" "web" will be created
```

→ Terraform tạo máy chủ mới trên AWS.

### 7.3 Khi nào Resource được **Sửa (Update)**?

Resource **đã tồn tại**, nhưng bạn thay đổi một số thuộc tính mà AWS có thể sửa được.

**Ví dụ thay đổi Tag:**

**File cũ:**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  
  tags = {
    Name = "Old Name"
  }
}
```

**File mới:**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  
  tags = {
    Name = "New Name"  # ← Thay đổi
  }
}
```

Chạy `terraform plan`:
```
~ resource "aws_instance" "web" will be updated in-place
  ~ tags = {
      ~ "Name" = "Old Name" -> "New Name"
    }
```

→ Sửa thẻ, máy chủ **không bị tắt**, dữ liệu **vẫn an toàn**.

### 7.4 Khi nào Resource được **Xóa (Destroy)**?

Resource **có trong state**, nhưng bạn **xóa nó khỏi code**.

**Ví dụ:**

**File cũ:**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
}

resource "aws_instance" "database" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
}
```

**File mới (xóa web):**
```hcl
resource "aws_instance" "database" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
}
```

Chạy `terraform plan`:
```
- resource "aws_instance" "web" will be destroyed
```

Chạy `terraform apply`:
```
aws_instance.web: Destroying... [id=i-0a1b2c3d4e5f6g7h8]
aws_instance.web: Destruction complete after 10s
```

→ Máy chủ **bị xóa trên AWS**, **không thể khôi phục**.

### 7.5 Khi nào Resource được **Tạo lại (Recreate)**?

Thay đổi **một số thuộc tính** mà AWS **không thể sửa được**. Terraform phải **xóa rồi tạo lại**.

**Ví dụ thay đổi Availability Zone (AZ):**

**File cũ:**
```hcl
resource "aws_instance" "web" {
  ami               = "ami-0df435f331839b2d6"
  instance_type     = "t2.micro"
  availability_zone = "us-east-1a"  # AZ cũ
}
```

**File mới:**
```hcl
resource "aws_instance" "web" {
  ami               = "ami-0df435f331839b2d6"
  instance_type     = "t2.micro"
  availability_zone = "us-east-1b"  # AZ mới
}
```

Chạy `terraform plan`:
```
±  resource "aws_instance" "web" must be replaced
  ~ availability_zone = "us-east-1a" -> "us-east-1b"

Plan: 1 to add, 0 to change, 1 to destroy.
```

Chạy `terraform apply`:
```
1. Xóa máy chủ cũ ở us-east-1a
2. Tạo máy chủ mới ở us-east-1b
```

⚠️ **Cảnh báo:** Máy chủ cũ **bị xóa**, dữ liệu **bị mất** (trừ khi bạn lưu vào EBS volume).

### 7.6 Bảng tóm tắt

| Hành động | Ký hiệu | Máy chủ | Dữ liệu |
|-----------|---------|---------|---------|
| **Tạo mới** | `+` | Bắt đầu từ đầu | Trống |
| **Sửa** | `~` | Vẫn chạy | An toàn |
| **Xóa** | `-` | Tắt + xóa | Mất |
| **Tạo lại** | `±` | Tắt rồi tạo mới | Mất |

### 7.7 Dependency (Phụ thuộc)

Terraform **tự động** hiểu thứ tự tạo resource.

**Ví dụ:**

```hcl
# 1. Tạo Security Group trước
resource "aws_security_group" "web" {
  name = "web-sg"
}

# 2. Tạo máy chủ, dùng Security Group ở bước 1
resource "aws_instance" "web" {
  ami                    = "ami-0df435f331839b2d6"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.web.id]  # Phụ thuộc vào SG
}
```

Terraform sẽ:
1. Tạo Security Group trước ✓
2. Lấy ID từ Security Group ✓
3. Tạo máy chủ dùng ID đó ✓

---

## TÓM LẠI PHẦN 1-7

| Phần | Nội dung | Học được gì |
|------|----------|------------|
| **1** | Terraform là gì? | Hiểu khái niệm cơ bản |
| **2** | Cài đặt | Có thể chạy Terraform |
| **3** | Khái niệm | Biết Provider, Resource, Variable,... |
| **4** | Cấu trúc file | Biết cách tổ chức code |
| **5** | Lệnh cơ bản | Có thể init, plan, apply, destroy |
| **6** | Ví dụ thực tế | Tạo máy chủ AWS đầu tiên |
| **7** | Hành vi resource | Hiểu thứ tự tạo, sửa, xóa |

---

## ⚡ CHEAT SHEET NHANH

```bash
# Khởi tạo
terraform init

# Kiểm tra lỗi
terraform validate

# Xem trước
terraform plan

# Tạo thực tế
terraform apply

# Xóa
terraform destroy

# Xem thông tin
terraform show
```

**File cấu trúc:**
```
provider.tf      → Kết nối AWS
variables.tf     → Khai báo biến
main.tf          → Tạo resources
outputs.tf       → Hiển thị kết quả
terraform.tfvars → Giá trị biến
.gitignore       → File không push Git
```

**Biến:**
```hcl
variable "name" {
  type    = string
  default = "value"
}

resource "..." "..." {
  property = var.name  # Dùng biến
}
```

---

Bạn đã học xong **Phần 1-7**! Tiếp theo là Meta-Arguments, Data Sources, State Management, v.v. Muốn tiếp tục không? 🚀
