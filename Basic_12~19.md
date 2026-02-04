# Hướng dẫn Terraform Chi Tiết - Tiếng Việt
## Phần 12-19: State Management, Workspaces, Modules, Import, Debug

---

## PHẦN 12: STATE FILE - FILE LƯU TRỮ TRẠNG THÁI

### 12.1 State file là gì?

**State file** = File lưu trữ **thông tin về resource đã tạo**.

#### Ví dụ đơn giản:

```
Code:
resource "aws_instance" "web" {
  ami = "ami-123"
}

Sau khi apply:
1. Terraform tạo máy chủ trên AWS
2. AWS trả về: ID = "i-abc123"
3. Terraform lưu vào file: terraform.tfstate
   {
     "resource_id": "i-abc123",
     "instance_type": "t2.micro"
   }
```

**Lần sau:**
```
Khi chạy plan:
1. Terraform đọc state file → "Biết đã tạo máy chủ i-abc123"
2. So sánh code hiện tại với state
3. Nếu code không thay đổi → "No changes"
4. Nếu code thay đổi → "Sẽ sửa"
```

### 12.2 Tại sao cần state file?

**Nếu không có state file:**
- ❌ Terraform không biết đã tạo gì
- ❌ Lần sau chạy apply sẽ tạo lại tất cả
- ❌ `terraform destroy` không biết xóa gì

### 12.3 State file lưu ở đâu?

#### Mặc định: Local (cùng thư mục)

```
my-project/
├── main.tf
├── terraform.tfstate        ← State file ở đây
└── terraform.tfstate.backup
```

⚠️ **Vấn đề:** State file chứa **mật khẩu, IP, SSH key** → Không nên đẩy lên Git!

#### Tốt hơn: Remote State (S3)

```
S3 Bucket (AWS)
    ↓
terraform.tfstate (lưu ở S3)
    ↓
Team có thể chia sẻ
```

**Ưu điểm:**
- ✓ Lưu an toàn trên AWS
- ✓ Cả team có thể dùng
- ✓ Backup tự động
- ✓ Mã hóa tự động

### 12.4 Thiết lập Remote State (S3)

**Bước 1: Tạo S3 bucket** (trên AWS Console hoặc Terraform)

**Bước 2: Tạo DynamoDB table** (cho state locking)

**Bước 3: Khai báo backend**

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

**Bước 4: Chạy init**

```bash
terraform init
# Kết quả: State được upload lên S3
```

### 12.5 State Locking - Chặn xung đột

**Vấn đề:**
```
Người A: terraform apply
  ↓
Người B: terraform apply (cùng lúc)
  ↓
Lộn xộn! Ai apply sau thắng
```

**Giải pháp: DynamoDB Lock**

```
Người A: apply
  ↓
Lock: "A đang làm"
  ↓
Người B: apply
  ↓
"Chờ... A đang lock"
  ↓
Người A: xong
  ↓
Unlock → Người B có thể làm
```

✓ **Chỉ 1 người apply cùng lúc!**

---

## PHẦN 13: TERRAFORM REFRESH - CẬP NHẬT STATE

### 13.1 Refresh là gì?

**Refresh** = Cập nhật state file để khớp với **AWS thực tế**.

### 13.2 Ví dụ:

```
1. Bạn tạo máy chủ bằng Terraform
   State file: tag = "Production"

2. Ai đó vào AWS Console, sửa tag → "Test"
   AWS: tag = "Test"
   State file: tag = "Production" (chưa cập nhật)

3. Bạn chạy terraform refresh
   Terraform lấy lại info từ AWS
   State file giờ = "Test" (cập nhật rồi)
```

### 13.3 Khi nào refresh?

- ✓ Ai đó sửa resource trực tiếp trên AWS Console
- ✓ Trước khi apply (chắc chắn state đúng)

**Lưu ý:** `terraform plan` tự động refresh, không cần chạy riêng.

```bash
terraform refresh
# Hoặc
terraform apply -refresh-only
```

---

## PHẦN 14: TERRAFORM SHOW - XEM STATE

### 14.1 Show là gì?

**Show** = Hiển thị nội dung state file (xem resource có gì).

### 14.2 Cách dùng:

```bash
# Xem tất cả
terraform show

# Xem 1 resource
terraform show aws_instance.web

# Xem dạng JSON
terraform show -json
```

**Ví dụ output:**
```
resource "aws_instance" "web":
  id            = "i-abc123"
  instance_type = "t2.micro"
  public_ip     = "54.175.102.97"
```

---

## PHẦN 15: STATE COMMANDS - QUẢN LÝ STATE

### 15.1 Các lệnh chính

```bash
terraform state list          # Liệt kê resource
terraform state show resource # Xem chi tiết
terraform state mv old new    # Đổi tên resource
terraform state rm resource   # Xóa khỏi state
terraform state pull          # Tải state về
terraform state push          # Upload state
terraform state taint res     # Đánh dấu xóa-tạo lại
```

### 15.2 `terraform state list` - Liệt kê

```bash
terraform state list
```

**Kết quả:**
```
aws_instance.web
aws_security_group.web
aws_s3_bucket.app
```

### 15.3 `terraform state show` - Chi tiết

```bash
terraform state show aws_instance.web
```

**Kết quả:**
```
id           = "i-abc123"
instance_type = "t2.micro"
public_ip    = "54.175.102.97"
```

### 15.4 `terraform state mv` - Đổi tên

**Vấn đề:**
Bạn đổi tên resource trong code từ `web` → `main`.
Nếu apply trực tiếp → xóa `web` tạo `main` → Mất máy!

**Giải pháp:**
```bash
terraform state mv aws_instance.web aws_instance.main
```

✓ **Máy vẫn an toàn, chỉ đổi tên trong state!**

### 15.5 `terraform state rm` - Xóa khỏi state

```bash
terraform state rm aws_instance.web
```

**Kết quả:**
- ✓ Máy chủ vẫn chạy trên AWS
- ✗ Terraform không quản lý nó nữa

⚠️ **Cảnh báo:** Không có cách khôi phục!

### 15.6 `terraform state taint` - Đánh dấu tạo lại

```bash
terraform state taint aws_instance.web
terraform apply
```

**Kết quả:**
- ✓ Xóa máy chủ cũ
- ✓ Tạo máy chủ mới (cùng config)

---

## PHẦN 16: WORKSPACES - QUẢN LÝ MÔI TRƯỜNG

### 16.1 Workspace là gì?

**Workspace** = Cách để quản lý **nhiều state file** từ **1 code**.

#### Ví dụ:

```
Cách cũ:
dev/
  main.tf
  terraform.tfstate (dev)

prod/
  main.tf
  terraform.tfstate (prod)
```

```
Cách dùng Workspace:
project/
  main.tf
  terraform.tfstate (default)
  terraform.tfstate.d/
    dev/terraform.tfstate
    prod/terraform.tfstate
```

### 16.2 Lệnh Workspace

```bash
# Tạo workspace
terraform workspace new dev
terraform workspace new prod

# Liệt kê
terraform workspace list
# Kết quả:
#   default
# * dev
#   prod

# Xem workspace hiện tại
terraform workspace show

# Chuyển workspace
terraform workspace select prod

# Xóa workspace
terraform workspace delete dev
```

### 16.3 Dùng workspace

```bash
# Tạo ở dev
terraform workspace select dev
terraform apply -var-file="dev.tfvars"

# Tạo ở prod
terraform workspace select prod
terraform apply -var-file="prod.tfvars"
```

✓ **Code giống nhau, state riêng!**

---

## PHẦN 17: MODULES - TÁI SỬ DỤNG CODE

### 17.1 Module là gì?

**Module** = Gói code có thể **dùng lại nhiều lần**.

#### Ví dụ:

**Cách cũ (dài):**
```hcl
# Tạo VPC
resource "aws_vpc" "my_vpc" { ... }

# Tạo Subnet
resource "aws_subnet" "my_subnet" { ... }

# Tạo Security Group
resource "aws_security_group" "my_sg" { ... }

# Tạo máy chủ
resource "aws_instance" "my_server" { ... }
```

❌ 40 dòng!

**Cách dùng Module (ngắn):**
```hcl
module "web_server" {
  source = "./modules/web-server"
  
  vpc_cidr  = "10.0.0.0/16"
  instance_type = "t2.micro"
}
```

✓ 5 dòng!

### 17.2 Tạo Module

**Bước 1: Tạo thư mục**
```bash
mkdir -p modules/web-server
```

**Bước 2: `modules/web-server/variables.tf`**
```hcl
variable "vpc_cidr" {
  type = string
}

variable "instance_type" {
  type = string
}
```

**Bước 3: `modules/web-server/main.tf`**
```hcl
resource "aws_vpc" "vpc" {
  cidr_block = var.vpc_cidr
}

resource "aws_instance" "server" {
  ami           = "ami-123"
  instance_type = var.instance_type
  # ...
}
```

**Bước 4: `modules/web-server/outputs.tf`**
```hcl
output "instance_id" {
  value = aws_instance.server.id
}

output "instance_ip" {
  value = aws_instance.server.public_ip
}
```

### 17.3 Dùng Module

```hcl
# main.tf
module "web_server" {
  source = "./modules/web-server"
  
  vpc_cidr      = "10.0.0.0/16"
  instance_type = "t2.micro"
}

# Dùng output từ module
output "web_ip" {
  value = module.web_server.instance_ip
}
```

### 17.4 Tái sử dụng

```hcl
# Module cho dev
module "dev_web" {
  source = "./modules/web-server"
  vpc_cidr      = "10.0.0.0/16"
  instance_type = "t2.micro"
}

# Module cho prod
module "prod_web" {
  source = "./modules/web-server"
  vpc_cidr      = "10.1.0.0/16"
  instance_type = "t3.large"
}
```

✓ **2 environment từ 1 module!**

---

## PHẦN 18: IMPORT - NHẬP RESOURCE CÓ SẴN

### 18.1 Import là gì?

**Import** = Đưa resource **tạo bên ngoài Terraform** vào quản lý.

### 18.2 Ví dụ:

```
Tình huống:
1. Mấy tháng trước, ai đó tạo máy chủ trên AWS Console
2. Bây giờ muốn dùng Terraform quản lý
3. Giải pháp: Import nó vào Terraform
```

### 18.3 Cách import (3 bước)

**Bước 1: Tìm resource ID**
```
AWS Console → EC2 → Instance ID: i-abc123
```

**Bước 2: Khai báo resource rỗng**
```hcl
resource "aws_instance" "imported" {
  # Để trống
}
```

**Bước 3: Import**
```bash
terraform import aws_instance.imported i-abc123
```

**Bước 4: Cập nhật code**
```bash
terraform state show aws_instance.imported
# Xem thông tin, rồi viết code
```

```hcl
resource "aws_instance" "imported" {
  ami           = "ami-123"
  instance_type = "t2.micro"
}
```

**Bước 5: Kiểm tra**
```bash
terraform plan
# No changes ✓
```

---

## PHẦN 19: DEBUG - TÌM LỖI

### 19.1 Bật Debug Logging

```bash
export TF_LOG=DEBUG
terraform plan
```

### 19.2 Log Levels

| Level | Chi tiết | Dùng khi |
|-------|----------|---------|
| TRACE | Rất chi tiết | Bug khó |
| DEBUG | Chi tiết | Debug thường |
| INFO | Cơ bản | Theo dõi |
| WARN | Cảnh báo | - |
| ERROR | Lỗi | Lỗi quan trọng |

### 19.3 Lưu vào file

```bash
export TF_LOG=DEBUG
export TF_LOG_PATH="terraform.log"
terraform apply
cat terraform.log
```

### 19.4 Lỗi phổ biến

**Lỗi 1: Credential**
```
Error: no valid credential sources found
```
**Giải pháp:** Set AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY

**Lỗi 2: State lock**
```
Error: Error acquiring the state lock
```
**Giải pháp:** `terraform force-unlock <ID>`

**Lỗi 3: Validation**
```
Error: Invalid value for variable
```
**Giải pháp:** Kiểm tra giá trị biến

### 19.5 Tips Debug

```bash
terraform validate     # Kiểm tra syntax
terraform fmt         # Sắp xếp code
terraform plan -json  # Xem JSON
terraform show        # Xem state
terraform state list  # Liệt kê resource
```

---

## 📋 TÓM TẮT PHẦN 12-19

| Phần | Nội dung | Lệnh |
|------|---------|------|
| 12 | State file, remote state | `backend "s3"` |
| 13 | Refresh state | `terraform refresh` |
| 14 | Xem state | `terraform show` |
| 15 | Quản lý state | `state list/show/mv/rm` |
| 16 | Workspaces | `workspace new/select` |
| 17 | Modules | `module "name" { source }` |
| 18 | Import | `terraform import` |
| 19 | Debug | `TF_LOG=DEBUG` |

---

## ⚡ CHEAT SHEET

```hcl
# Remote State
terraform {
  backend "s3" {
    bucket = "my-state"
    key    = "prod/terraform.tfstate"
  }
}

# Workspace
terraform workspace new dev
terraform workspace select dev

# Module
module "web" {
  source = "./modules/web"
  var1   = "value1"
}
```

```bash
# State commands
terraform state list
terraform state show aws_instance.web
terraform state mv old new
terraform state taint resource
terraform refresh

# Import
terraform import aws_instance.web i-abc123

# Debug
export TF_LOG=DEBUG
terraform plan
```

---

## ✅ **HOÀN THÀNH PHẦN 12-19!**

Bạn giờ biết:
- ✅ Quản lý state file
- ✅ Dùng workspaces
- ✅ Tạo modules
- ✅ Import resource cũ
- ✅ Debug lỗi

**Bước tiếp theo:** Làm dự án thực tế! 🚀
