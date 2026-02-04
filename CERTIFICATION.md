# Hướng dẫn Terraform Chi Tiết - Tiếng Việt
## Phần 20: Certification Cheat Sheet - Chuẩn Bị Thi

---

## PHẦN 20: CERTIFICATION CHEAT SHEET

### 20.1 Chứng chỉ HashiCorp Certified Terraform Associate

**Tên:** HashiCorp Certified: Terraform Associate  
**Viết tắt:** HCA-TA  
**Nhà cấp:** HashiCorp  
**Phí thi:** $70 USD  
**Thời gian:** 60 phút  
**Câu hỏi:** 57 câu (multiple choice)  
**Điểm đạt:** 70% (40/57 đúng)  

### 20.2 Tại sao cần certification?

| Lợi ích | Mô tả |
|--------|-------|
| 📜 **Chứng minh kỹ năng** | Chứng tỏ bạn biết Terraform |
| 💼 **Tìm việc dễ hơn** | Nhà tuyển dụng tin tưởng |
| 💰 **Lương cao hơn** | Lương tăng 10-20% |
| 🎓 **Nâng cao dự án** | Tin tưởng để làm dự án lớn |
| 🌟 **Uy tín chuyên môn** | Công nhân kỹ thuật hạng A |

### 20.3 Nội dung thi

**5 Domains (5 chủ đề):**

| Domain | Tỷ lệ | Nội dung |
|--------|-------|---------|
| **1. IaC Concepts** | 15% | Khái niệm IaC |
| **2. State Management** | 20% | Quản lý state |
| **3. Terraform Basics** | 25% | Cơ bản Terraform |
| **4. Terraform Features** | 25% | Tính năng nâng cao |
| **5. Troubleshooting** | 15% | Xử lý lỗi |

---

## **DOMAIN 1: IAC CONCEPTS (15%)**

### 1.1 Infrastructure as Code là gì?

**IaC** = Quản lý hạ tầng bằng **code** (không phải thủ công).

#### Lợi ích:
```
✓ Lặp lại (Repeatable) - Tạo lại dễ
✓ Kiểm soát phiên bản (Version control) - Dùng Git
✓ Tự động (Automation) - Không bấm nút
✓ Kiểm tra trước (Preview) - Xem trước apply
✓ Nhất quán (Consistent) - Luôn như nhau
✓ Tài liệu sống (Living documentation) - Code là tài liệu
```

### 1.2 IaC vs Config Management vs Orchestration

| Loại | Tác dụng | Ví dụ |
|------|---------|-------|
| **IaC** | **Tạo** hạ tầng | Terraform, CloudFormation |
| **Config** | **Cấu hình** máy chủ | Ansible, Chef, Puppet |
| **Orchestration** | **Điều phối** containers | Kubernetes, Docker Swarm |

**Terraform = IaC** (tạo, không phải cấu hình)

### 1.3 Terraform vs CloudFormation vs Ansible

| Tiêu chí | Terraform | CloudFormation | Ansible |
|----------|-----------|----------------|---------|
| **Loại** | IaC | IaC | Config Mgmt |
| **Cloud** | Multi-cloud | AWS only | Multi-cloud |
| **Ngôn ngữ** | HCL | JSON/YAML | YAML |
| **Idempotent** | ✓ Có | ✓ Có | ✓ Có |
| **Agentless** | ✓ Có | ✓ Có | ✓ Có |

**Kết luận:** Terraform = Tốt nhất cho IaC

### 1.4 Các khái niệm cơ bản

**Provider:** Cầu nối tới AWS/Azure/GCP
```hcl
provider "aws" {
  region = "us-east-1"
}
```

**Resource:** Cái được tạo (máy chủ, bucket, ...)
```hcl
resource "aws_instance" "web" {
  ami = "ami-123"
}
```

**Data Source:** Lấy thông tin (không tạo)
```hcl
data "aws_ami" "latest" {
  most_recent = true
}
```

**Variable:** Biến tái sử dụng
```hcl
variable "instance_type" {
  type = string
}
```

**Output:** Hiển thị kết quả
```hcl
output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

**Module:** Gói code tái sử dụng
```hcl
module "web" {
  source = "./modules/web"
}
```

---

## **DOMAIN 2: STATE MANAGEMENT (20%)**

### 2.1 State file là gì?

**State** = File lưu **ánh xạ** giữa code và resource thực tế.

```
Code          State File         AWS
resource  ←→  id: i-abc123  ←→  EC2 Instance
"web"         type: t2.micro    (real)
```

### 2.2 State file chứa gì?

```json
{
  "version": 4,
  "resources": [
    {
      "type": "aws_instance",
      "name": "web",
      "instances": [
        {
          "attributes": {
            "id": "i-abc123",
            "instance_type": "t2.micro",
            "public_ip": "54.175.102.97"
          }
        }
      ]
    }
  ]
}
```

### 2.3 Backend là gì?

**Backend** = Nơi lưu trữ state file.

**Loại Backend:**

| Backend | Nơi lưu | Ưu điểm |
|---------|---------|--------|
| **local** | Máy tính | Đơn giản, không setup |
| **s3** | AWS S3 | An toàn, chia sẻ, locking |
| **terraform cloud** | Cloud | Quản lý tập trung, UI |
| **consul** | Consul | Động, chia sẻ |
| **azurerm** | Azure Storage | Cho Azure |
| **gcs** | Google Cloud | Cho GCP |

**Khuyến khích:** S3 + DynamoDB Lock

### 2.4 State Locking

**Vấn đề:** 2 người apply cùng lúc → Xung đột

**Giải pháp:** DynamoDB Lock tự động chặn

```
Person A: apply → Lock
Person B: apply → Chờ lock
Person A: xong → Unlock
Person B: apply → Được lock
```

**Lệnh unlock:**
```bash
terraform force-unlock <LOCK_ID>
```

### 2.5 State File - Best Practices

**✓ Nên làm:**
- ✓ Dùng remote state (S3)
- ✓ Enable encryption
- ✓ Enable versioning
- ✓ Enable locking
- ✓ Backup state file
- ✓ Restricted access (IAM)

**✗ Không nên:**
- ✗ Push state lên Git
- ✗ Share state file bằng email
- ✗ Chỉnh sửa state file thủ công
- ✗ Dùng local state cho team

---

## **DOMAIN 3: TERRAFORM BASICS (25%)**

### 3.1 Terraform Workflow (4 bước)

```
1. terraform init
   ↓ Download plugins, setup backend

2. terraform plan
   ↓ Xem sẽ thay đổi gì

3. terraform apply
   ↓ Tạo/sửa/xóa resource

4. terraform destroy
   ↓ Xóa tất cả resource
```

### 3.2 Các lệnh chính

| Lệnh | Tác dụng |
|------|---------|
| `init` | Khởi tạo (download plugins) |
| `validate` | Kiểm tra syntax |
| `fmt` | Sắp xếp code |
| `plan` | Xem preview |
| `apply` | Tạo thực tế |
| `destroy` | Xóa tất cả |
| `refresh` | Cập nhật state |
| `show` | Xem state |

### 3.3 Variable - Các loại dữ liệu

```hcl
# String
variable "instance_type" {
  type = string
}

# Number
variable "port" {
  type = number
}

# Bool
variable "enabled" {
  type = bool
}

# List
variable "subnets" {
  type = list(string)
}

# Map
variable "tags" {
  type = map(string)
}

# Set
variable "unique_ids" {
  type = set(string)
}

# Tuple
variable "mixed" {
  type = tuple([string, number])
}

# Object
variable "config" {
  type = object({
    name = string
    port = number
  })
}
```

### 3.4 Variable - Ưu tiên

**Cao nhất → Thấp nhất:**

1. `-var` (command line) ← **Cao nhất**
2. `-var-file`
3. `terraform.tfvars`
4. `TF_VAR_` environment variable
5. `default` value ← **Thấp nhất**

### 3.5 Output Values

```hcl
output "instance_id" {
  description = "ID của máy chủ"
  value       = aws_instance.web.id
  sensitive   = false
}
```

**Lấy output:**
```bash
terraform output
terraform output instance_id
terraform output -json
```

### 3.6 HCL Syntax (Cơ bản)

```hcl
# Comment
resource "aws_instance" "web" {
  # Block: resource
  # Type: aws_instance
  # Name: web
  
  ami           = "ami-123"  # Argument
  instance_type = "t2.micro"
  
  tags = {
    Name = "My Server"  # Map
  }
}
```

---

## **DOMAIN 4: TERRAFORM FEATURES (25%)**

### 4.1 Meta-Arguments

| Meta-Arg | Tác dụng |
|----------|---------|
| `count` | Tạo N cái giống |
| `for_each` | Tạo từ danh sách |
| `depends_on` | Chỉ định phụ thuộc |
| `provider` | Dùng provider nào |
| `lifecycle` | Kiểm soát vòng đời |

```hcl
# count
resource "aws_instance" "servers" {
  count = 3
  # Truy cập: aws_instance.servers[0]
}

# for_each
resource "aws_s3_bucket" "buckets" {
  for_each = {dev = "bucket-dev", prod = "bucket-prod"}
  # Truy cập: aws_s3_bucket.buckets["dev"]
}

# depends_on
resource "aws_instance" "web" {
  depends_on = [aws_security_group.web]
}

# lifecycle
resource "aws_instance" "web" {
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = true
    ignore_changes        = [tags]
  }
}
```

### 4.2 Data Sources

**Data source** = Lấy thông tin từ AWS (không tạo)

```hcl
# Lấy AMI mới nhất
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*"]
  }
}

# Dùng
resource "aws_instance" "web" {
  ami = data.aws_ami.latest.id
}
```

**Data sources phổ biến:**
- `aws_ami` - Hình ảnh
- `aws_vpc` - VPC
- `aws_subnets` - Subnets
- `aws_security_group` - Security groups
- `aws_availability_zones` - AZs
- `aws_caller_identity` - AWS account info

### 4.3 Modules

**Module** = Gói code tái sử dụng

```hcl
module "web_server" {
  source = "./modules/web-server"
  # Hoặc từ Registry:
  # source = "terraform-aws-modules/vpc/aws"
  
  vpc_cidr = "10.0.0.0/16"
  
  # Dùng output
  instance_id = module.web_server.instance_id
}
```

**Module structure:**
```
modules/web-server/
├── main.tf
├── variables.tf
├── outputs.tf
└── README.md
```

### 4.4 Workspaces

**Workspace** = Tách state cho nhiều môi trường

```bash
terraform workspace new dev
terraform workspace select dev
terraform workspace list
terraform workspace delete dev
```

**Dùng trong code:**
```hcl
terraform.workspace  # Giá trị: dev, prod, ...
```

### 4.5 Functions (Hàm)

**String functions:**
```hcl
length("hello")              # 5
upper("hello")               # HELLO
lower("HELLO")               # hello
substr("hello", 1, 2)        # el
format("%.2f", 3.14159)      # 3.14
```

**List functions:**
```hcl
concat([1,2], [3,4])         # [1,2,3,4]
flatten([[1,2], [3,4]])      # [1,2,3,4]
contains([1,2,3], 2)         # true
element([1,2,3], 1)          # 2
join(",", [1,2,3])           # "1,2,3"
```

**Map functions:**
```hcl
keys({a=1, b=2})             # [a, b]
values({a=1, b=2})           # [1, 2]
merge({a=1}, {b=2})          # {a=1, b=2}
lookup({a=1}, "a")           # 1
```

**Math functions:**
```hcl
min(1,2,3)                   # 1
max(1,2,3)                   # 3
ceil(3.4)                    # 4
floor(3.6)                   # 3
```

### 4.6 Expressions

**Conditional:**
```hcl
condition ? true_value : false_value

instance_type = var.environment == "prod" ? "t3.large" : "t2.micro"
```

**For expression:**
```hcl
[for item in list : item.name]

{for item in list : item.id => item.name}
```

**Splat:**
```hcl
aws_instance.servers[*].public_ip  # Lấy tất cả IP
```

---

## **DOMAIN 5: TROUBLESHOOTING (15%)**

### 5.1 Debugging

**Bật logging:**
```bash
export TF_LOG=DEBUG
export TF_LOG_PATH="terraform.log"
terraform plan
```

**Log levels:**
- TRACE - Rất chi tiết
- DEBUG - Chi tiết
- INFO - Cơ bản
- WARN - Cảnh báo
- ERROR - Lỗi

### 5.2 Lỗi phổ biến

**Lỗi 1: Syntax error**
```
Error: Argument or block definition required
```
**Giải pháp:** `terraform validate`

**Lỗi 2: Resource không tìm thấy**
```
Error: resource not found
```
**Giải pháp:** `terraform state list`

**Lỗi 3: Credential error**
```
Error: no valid credential sources found
```
**Giải pháp:** Set AWS credentials

**Lỗi 4: State lock error**
```
Error: Error acquiring the state lock
```
**Giải pháp:** `terraform force-unlock`

### 5.3 State File Issues

**Vấn đề 1: State lỗi thời**
```bash
terraform refresh  # Cập nhật state
```

**Vấn đề 2: Mất state file**
```bash
terraform state pull > backup.tfstate  # Backup
terraform state push backup.tfstate    # Restore
```

**Vấn đề 3: Resource sai trong state**
```bash
terraform state rm resource           # Xóa
terraform import resource_type id     # Import lại
```

### 5.4 Import - Khi cần

```bash
resource "aws_instance" "imported" {
  # Để trống
}

terraform import aws_instance.imported i-abc123
```

---

## ⚡ **QUICK REFERENCE - CÁC LỆNH QUAN TRỌNG**

```bash
# Workflow
terraform init      # Khởi tạo
terraform validate  # Kiểm tra
terraform fmt       # Sắp xếp
terraform plan      # Xem preview
terraform apply     # Tạo
terraform destroy   # Xóa

# State
terraform state list            # Liệt kê
terraform state show resource   # Chi tiết
terraform state mv old new      # Đổi tên
terraform state rm resource     # Xóa
terraform state pull            # Backup
terraform refresh               # Cập nhật

# Workspace
terraform workspace new env
terraform workspace select env
terraform workspace list
terraform workspace delete env

# Debug
export TF_LOG=DEBUG
terraform console  # Thử expression
terraform graph    # Xem dependency

# Import
terraform import resource_type id
```

---

## 📝 **CHEAT SHEET**

```hcl
# Block
resource "type" "name" {
  argument = "value"
  nested_block {
    argument = "value"
  }
}

# Reference
resource.type.name.attribute
module.name.output
data.type.name.attribute
var.variable_name
aws_instance.web.public_ip

# String interpolation
"${var.name}"
"Hello ${var.name}"

# Conditional
condition ? true_val : false_val

# For loop
[for x in list : x.name]
{for x in list : x.id => x.name}

# Splat
resource_type.name[*].attribute
```

---

## 🎯 **LỤC TÁCH - PHẦN KIẾN THỨC CẦN THI**

### **Domain 1: IaC Concepts (15%)**
- [ ] IaC là gì
- [ ] Lợi ích IaC
- [ ] Terraform vs CloudFormation vs Ansible
- [ ] Provider, Resource, Data Source
- [ ] Variable, Output, Module

### **Domain 2: State Management (20%)**
- [ ] State file là gì
- [ ] Local vs Remote state
- [ ] Backend types (s3, terraform cloud, ...)
- [ ] State locking
- [ ] State best practices

### **Domain 3: Terraform Basics (25%)**
- [ ] 4 bước workflow
- [ ] Các lệnh chính
- [ ] Variable types & priority
- [ ] Output values
- [ ] HCL syntax

### **Domain 4: Terraform Features (25%)**
- [ ] count, for_each, depends_on, provider, lifecycle
- [ ] Data sources
- [ ] Modules
- [ ] Workspaces
- [ ] Functions, expressions

### **Domain 5: Troubleshooting (15%)**
- [ ] Debugging (TF_LOG)
- [ ] Common errors
- [ ] State issues
- [ ] Import resources
- [ ] Lệnh validate, plan, graph

---

## 💡 **TIPS THI**

✅ **Chuẩn bị:**
1. Học 19 phần từ phần 1 đến 19
2. Làm bài tập thực hành
3. Ôn lại domain 1-5
4. Nhớ các lệnh chính
5. Hiểu khái niệm, không cần nhớ từng dòng code

✅ **Trong khi thi:**
1. Đọc kỹ câu hỏi
2. Loại bỏ đáp án sai
3. Không ngập ngừng quá lâu
4. Quay lại câu khó sau
5. Kiểm tra trước khi nộp

✅ **Điểm pass:**
- 70% = 40/57 câu
- Tỷ lệ thành công: ~80-90% (nếu chuẩn bị tốt)

---

## 🎓 **THỜI GIAN CHUẨN BỊ**

**Lộ trình 4 tuần:**

```
Tuần 1: Học Phần 1-7 (Basics)
Tuần 2: Học Phần 8-11 (Advanced)
Tuần 3: Học Phần 12-19 (State & Features)
Tuần 4: Ôn lại + Làm bài tập mock + Thi

Thời gian học: 10-15 giờ/tuần
Thời gian ôn: 5 giờ
Tổng cộng: 50 giờ
```
