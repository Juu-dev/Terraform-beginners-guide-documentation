# Hướng dẫn Terraform Chi Tiết - Tiếng Việt
## Phần 8-11: Meta-Arguments, Biến Nâng Cao, Output, Data Sources

---

## PHẦN 8: META-ARGUMENTS (TỪ KHÓA ĐẶC BIỆT CỦA TERRAFORM)

**Meta-Arguments** = Những chỉ dẫn đặc biệt cho Terraform về cách quản lý resource.

Khác với thuộc tính bình thường (như `ami`, `instance_type`), meta-arguments điều khiển **hành vi của Terraform**, không phải **cấu hình AWS**.

### 8.1 `count` - Tạo nhiều resource giống nhau

#### Vấn đề:
Bạn muốn tạo **3 máy chủ giống nhau**, nhưng code cũ là:

```hcl
resource "aws_instance" "server1" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
}

resource "aws_instance" "server2" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
}

resource "aws_instance" "server3" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
}
```

❌ Dài dòng, dễ sai, khó bảo trì.

#### Giải pháp: Dùng `count`

```hcl
resource "aws_instance" "servers" {
  count         = 3  # Tạo 3 cái
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  
  tags = {
    Name = "Server-${count.index + 1}"  # Server-1, Server-2, Server-3
  }
}
```

#### Giải thích:
- `count = 3` → Tạo 3 resource
- `count.index` → Số thứ tự (0, 1, 2)
- `count.index + 1` → Số thứ tự bắt đầu từ 1

#### Ví dụ chi tiết:

**Code:**
```hcl
variable "server_count" {
  type    = number
  default = 3
}

resource "aws_instance" "servers" {
  count         = var.server_count
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer-${count.index + 1}"
  }
}

# Hiển thị thông tin
output "server_ids" {
  value = aws_instance.servers[*].id  # Lấy ID của tất cả
}

output "server_ips" {
  value = aws_instance.servers[*].public_ip  # Lấy IP của tất cả
}
```

**Chạy lệnh:**
```bash
terraform plan
```

**Kết quả:**
```
+ aws_instance.servers[0] will be created
+ aws_instance.servers[1] will be created
+ aws_instance.servers[2] will be created

Plan: 3 to add, 0 to change, 0 to destroy.
```

**Output:**
```
server_ids = [
  "i-0a1b2c3d4e5f6g7h8",
  "i-1a2b3c4d5e6f7g8h9",
  "i-2a3b4c5d6e7f8g9h0"
]

server_ips = [
  "54.175.102.1",
  "54.175.102.2",
  "54.175.102.3"
]
```

#### Thay đổi số lượng:

**Tăng từ 3 lên 5 máy:**

```hcl
variable "server_count" {
  default = 5  # Thay đổi từ 3
}
```

Chạy `terraform apply`:
```
Plan: 2 to add, 0 to change, 0 to destroy.  # Thêm 2 máy
```

**Giảm từ 5 xuống 2 máy:**

```hcl
variable "server_count" {
  default = 2  # Giảm xuống
}
```

Chạy `terraform apply`:
```
Plan: 0 to add, 0 to change, 3 to destroy.  # Xóa 3 máy
```

#### Khi nào dùng `count`?
- ✓ Tạo N cái giống nhau
- ✓ Số lượng biết trước
- ✓ Không cần xác định tên riêng

---

### 8.2 `for_each` - Tạo resource từ danh sách

#### Vấn đề:
Bạn muốn tạo **3 bucket S3 cho các môi trường khác nhau** (dev, uat, prod):

```hcl
resource "aws_s3_bucket" "dev_bucket" {
  bucket = "my-dev-bucket"
}

resource "aws_s3_bucket" "uat_bucket" {
  bucket = "my-uat-bucket"
}

resource "aws_s3_bucket" "prod_bucket" {
  bucket = "my-prod-bucket"
}
```

❌ Dài dòng, khó quản lý.

#### Giải pháp: Dùng `for_each` với map (từ điển)

```hcl
resource "aws_s3_bucket" "buckets" {
  for_each = {
    dev  = "my-dev-bucket"
    uat  = "my-uat-bucket"
    prod = "my-prod-bucket"
  }
  
  bucket = each.value  # Lấy tên bucket
  
  tags = {
    Name = each.key    # dev, uat, prod
    Env  = each.value
  }
}
```

#### Giải thích:
- `for_each = {...}` → Danh sách các cặp key-value
- `each.key` → Khóa (dev, uat, prod)
- `each.value` → Giá trị (my-dev-bucket, ...)

#### Ví dụ chi tiết:

**Code:**
```hcl
variable "environments" {
  type = map(string)
  default = {
    dev  = "my-dev-app-bucket"
    uat  = "my-uat-app-bucket"
    prod = "my-prod-app-bucket"
  }
}

resource "aws_s3_bucket" "app_buckets" {
  for_each = var.environments
  bucket   = each.value
  
  tags = {
    Name        = each.value
    Environment = each.key
  }
}

output "bucket_names" {
  value = {
    for env, bucket in aws_s3_bucket.app_buckets : env => bucket.id
  }
}
```

**Chạy lệnh:**
```bash
terraform plan
```

**Kết quả:**
```
+ aws_s3_bucket.app_buckets["dev"] will be created
+ aws_s3_bucket.app_buckets["uat"] will be created
+ aws_s3_bucket.app_buckets["prod"] will be created

Plan: 3 to add, 0 to change, 0 to destroy.
```

**Output:**
```
bucket_names = {
  "dev"  = "my-dev-app-bucket"
  "uat"  = "my-uat-app-bucket"
  "prod" = "my-prod-app-bucket"
}
```

#### `for_each` với `set` (danh sách không trùng):

Nếu chỉ có danh sách đơn (không cần key-value):

```hcl
resource "aws_iam_user" "users" {
  for_each = toset(["alice", "bob", "charlie"])
  name     = each.value
  
  tags = {
    Name = each.key  # each.key == each.value khi dùng set
  }
}
```

#### So sánh `count` vs `for_each`:

| `count` | `for_each` |
|---------|-----------|
| Tạo N cái giống nhau | Tạo từ danh sách |
| Dùng `count.index` | Dùng `each.key`, `each.value` |
| Xóa 1 cái ở giữa → tất cả ID thay đổi | Xóa 1 cái ở giữa → ID khác không đổi |
| Dùng khi số lượng | Dùng khi có tên riêng |

#### Khi nào dùng `for_each`?
- ✓ Tạo resource từ danh sách
- ✓ Mỗi resource có tên/key riêng
- ✓ Muốn xóa 1 cái mà không ảnh hưởng tới cái khác

---

### 8.3 `depends_on` - Chỉ định phụ thuộc

#### Vấn đề:
Terraform **tự động** hiểu phụ thuộc (ví dụ: máy chủ phụ thuộc vào security group). Nhưng có lúc Terraform **không hiểu được** phụ thuộc ẩn.

Ví dụ: Bạn tạo máy chủ, nhưng **đôi khi Terraform tạo security group sau máy chủ** → Lỗi.

#### Giải pháp: Dùng `depends_on`

```hcl
# Tạo security group
resource "aws_security_group" "web" {
  name = "web-sg"
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Tạo máy chủ, phụ thuộc vào security group
resource "aws_instance" "web" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  # Chỉ định rõ: phụ thuộc vào security group
  depends_on = [aws_security_group.web]
}

# Tạo Elastic IP, phụ thuộc vào máy chủ
resource "aws_eip" "web_eip" {
  instance = aws_instance.web.id
  
  depends_on = [aws_instance.web]
}
```

#### Giải thích:
- `depends_on = [...]` → Danh sách resource phải tạo trước
- Terraform sẽ tạo `aws_security_group.web` trước
- Sau đó tạo `aws_instance.web`
- Cuối cùng tạo `aws_eip.web_eip`

#### Thứ tự tạo:

```
1. aws_security_group.web
   ↓
2. aws_instance.web (phụ thuộc vào security group)
   ↓
3. aws_eip.web_eip (phụ thuộc vào máy chủ)
```

#### Khi nào cần `depends_on`?
- ✓ Phụ thuộc ẩn (Terraform không tự nhận ra)
- ✓ Cần chắc chắn thứ tự tạo
- ✓ Một resource cần khởi động service trước (ví dụ: RDS phải sẵn sàng trước khi app kết nối)

**Lưu ý:** Thông thường Terraform tự hiểu phụ thuộc, không cần `depends_on`.

---

### 8.4 `provider` - Dùng nhiều provider

#### Vấn đề:
Bạn muốn tạo **resource ở 2 khu vực AWS khác nhau** (us-east-1 và ap-southeast-1):

```hcl
provider "aws" {
  region = "us-east-1"
}

# Không biết cách tạo ở ap-southeast-1
resource "aws_s3_bucket" "us_bucket" {
  bucket = "my-us-bucket"
}

resource "aws_s3_bucket" "ap_bucket" {
  bucket = "my-ap-bucket"  # Tạo ở ap-southeast-1?
}
```

❌ Provider chỉ kết nối 1 region.

#### Giải pháp: Dùng `provider` meta-argument

**Bước 1: Khai báo 2 provider**

```hcl
# Provider khu vực 1 (us-east-1)
provider "aws" {
  region = "us-east-1"
  alias  = "us"  # Tên gọi
}

# Provider khu vực 2 (ap-southeast-1)
provider "aws" {
  region = "ap-southeast-1"
  alias  = "ap"  # Tên gọi
}
```

**Bước 2: Chỉ định provider cho resource**

```hcl
# Tạo bucket ở us-east-1
resource "aws_s3_bucket" "us_bucket" {
  provider = aws.us
  bucket   = "my-us-bucket"
  
  tags = {
    Region = "US"
  }
}

# Tạo bucket ở ap-southeast-1
resource "aws_s3_bucket" "ap_bucket" {
  provider = aws.ap
  bucket   = "my-ap-bucket"
  
  tags = {
    Region = "AP"
  }
}
```

**Chạy lệnh:**
```bash
terraform plan
```

**Kết quả:**
```
+ aws_s3_bucket.us_bucket (on provider "aws.us") will be created
+ aws_s3_bucket.ap_bucket (on provider "aws.ap") will be created
```

#### Ví dụ với VPC ở 2 region:

```hcl
# VPC ở us-east-1
resource "aws_vpc" "us_vpc" {
  provider   = aws.us
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "US VPC"
  }
}

# VPC ở ap-southeast-1
resource "aws_vpc" "ap_vpc" {
  provider   = aws.ap
  cidr_block = "10.1.0.0/16"
  
  tags = {
    Name = "AP VPC"
  }
}
```

#### Khi nào dùng `provider`?
- ✓ Tạo resource ở nhiều khu vực
- ✓ Dùng nhiều AWS account
- ✓ Kết hợp AWS + Azure + GCP

---

### 8.5 `lifecycle` - Kiểm soát vòng đời

#### Vấn đề:
Khi thay đổi cấu hình, Terraform **mặc định xóa rồi tạo lại**. Điều này có thể gây ngắt dịch vụ.

Ví dụ: Thay đổi Availability Zone (AZ):
```
1. Xóa máy chủ cũ → Dừng dịch vụ
2. Tạo máy chủ mới → Dịch vụ offline một lúc
```

❌ Người dùng không thể truy cập.

#### Giải pháp 1: `create_before_destroy`

```hcl
resource "aws_instance" "web" {
  ami               = "ami-0df435f331839b2d6"
  instance_type     = "t2.micro"
  availability_zone = "us-east-1a"
  
  lifecycle {
    create_before_destroy = true  # Tạo mới trước, xóa cũ sau
  }
}
```

**Quy trình:**
```
1. Tạo máy chủ mới ở us-east-1b ← Dịch vụ vẫn chạy
2. Xóa máy chủ cũ ở us-east-1a ← Chuyển traffic qua cái mới
3. Xong ✓
```

✓ **Không ngắt dịch vụ!**

#### Giải pháp 2: `prevent_destroy`

```hcl
resource "aws_db_instance" "database" {
  allocated_storage = 20
  engine           = "mysql"
  instance_class   = "db.t3.micro"
  username         = "admin"
  password         = "password123"
  
  lifecycle {
    prevent_destroy = true  # Ngăn chặn xóa
  }
}
```

**Nếu bạn chạy `terraform destroy`:**
```
Error: Resource has lifecycle.prevent_destroy set,
but the plan calls for this resource to be destroyed.
To avoid accidental destruction, remove the
prevent_destroy flag and re-run the operation.
```

✓ **Bảo vệ dữ liệu quan trọng!**

#### Giải pháp 3: `ignore_changes`

**Vấn đề:** Ai đó sửa tag trực tiếp trên AWS Console, Terraform nhìn thấy sự thay đổi và muốn "sửa lại".

```hcl
# Trên AWS Console, tag thay đổi từ "dev" → "prod"

resource "aws_instance" "web" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  
  tags = {
    Environment = "dev"  # Code nói "dev", nhưng AWS là "prod"
  }
}

# Chạy terraform plan → Terraform muốn sửa lại thành "dev"
```

**Giải pháp: Bỏ qua thay đổi tag**

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  
  tags = {
    Environment = "dev"
  }
  
  lifecycle {
    ignore_changes = [tags]  # Bỏ qua thay đổi tag
  }
}
```

**Chạy `terraform plan`:**
```
No changes. Your infrastructure matches the configuration.
```

✓ **Terraform không phàn nàn!**

#### Bỏ qua nhiều thuộc tính:

```hcl
lifecycle {
  ignore_changes = [tags, instance_type, ami]
}
```

#### Bỏ qua tất cả:

```hcl
lifecycle {
  ignore_changes = all
}
```

⚠️ **Cảnh báo:** Bỏ qua quá nhiều có thể gây lộn xộn.

#### Khi nào dùng `lifecycle`?
- ✓ `create_before_destroy` → Không muốn ngắt dịch vụ
- ✓ `prevent_destroy` → Bảo vệ dữ liệu quan trọng
- ✓ `ignore_changes` → Cho phép thay đổi thủ công

---

## PHẦN 9: BIẾN NÂNG CAO (ADVANCED VARIABLES)

### 9.1 Các loại biến

Ở phần 4, bạn học các loại biến cơ bản: `string`, `number`, `list`, `map`.

Bây giờ học thêm các tính năng nâng cao.

### 9.2 Validation (Kiểm tra giá trị)

#### Vấn đề:
Bạn khai báo biến `instance_type`, nhưng người dùng có thể nhập sai:

```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

# Ai đó nhập: instance_type = "invalid-type"
# Terraform không phàn nàn, AWS mới phàn nàn → Muộn quá!
```

❌ Kiểm tra muộn.

#### Giải pháp: Validation rule

```hcl
variable "instance_type" {
  type        = string
  default     = "t2.micro"
  description = "Loại máy chủ (t2.micro, t2.small, t3.medium)"
  
  validation {
    condition     = contains(["t2.micro", "t2.small", "t3.medium"], var.instance_type)
    error_message = "Instance type phải là: t2.micro, t2.small hoặc t3.medium."
  }
}
```

**Nếu nhập sai:**
```bash
terraform plan
```

**Kết quả:**
```
Error: Invalid value for variable

  on variables.tf line 1:
   1: variable "instance_type" {

Unsupported value: "invalid-type" does not match any of the provided options.
Instance type phải là: t2.micro, t2.small hoặc t3.medium.
```

✓ **Kiểm tra sớm!**

#### Ví dụ validation khác:

**Kiểm tra AMI ID:**
```hcl
variable "ami_id" {
  type    = string
  default = "ami-0df435f331839b2d6"
  
  validation {
    condition     = length(var.ami_id) > 4 && substr(var.ami_id, 0, 4) == "ami-"
    error_message = "AMI ID phải bắt đầu bằng 'ami-'"
  }
}
```

**Kiểm tra số port:**
```hcl
variable "port" {
  type    = number
  default = 8080
  
  validation {
    condition     = var.port >= 1 && var.port <= 65535
    error_message = "Port phải từ 1 đến 65535"
  }
}
```

**Kiểm tra environment:**
```hcl
variable "environment" {
  type    = string
  default = "dev"
  
  validation {
    condition     = contains(["dev", "uat", "prod"], var.environment)
    error_message = "Environment phải là: dev, uat hoặc prod"
  }
}
```

### 9.3 Sensitive (Biến nhạy cảm)

#### Vấn đề:
Bạn khai báo mật khẩu, token, key:

```hcl
variable "db_password" {
  type    = string
  default = "my-password-123"
}

# Terraform hiển thị password trong log, plan, output
# Cả cộng sự, Git, CI/CD đều thấy → Nguy hiểm!
```

❌ Lộ thông tin nhạy cảm.

#### Giải pháp: Mark as sensitive

```hcl
variable "db_password" {
  type        = string
  description = "Mật khẩu database"
  sensitive   = true  # Đánh dấu là nhạy cảm
}

variable "api_key" {
  type        = string
  description = "API key"
  sensitive   = true
}

variable "private_key" {
  type        = string
  description = "SSH private key"
  sensitive   = true
}
```

**Chạy `terraform plan`:**
```
+ resource "aws_db_instance" "database" {
    ...
    password = (sensitive value)  ← Không hiển thị!
    ...
  }
```

✓ **Mật khẩu bị ẩn!**

**Output nhạy cảm:**

```hcl
output "db_password" {
  value     = random_password.db.result
  sensitive = true  # Output cũng bị ẩn
}
```

**Chạy `terraform apply`:**
```
Outputs:

db_password = <sensitive>
```

#### Khi nào đánh dấu sensitive?
- ✓ Mật khẩu, token, key
- ✓ Private key SSH
- ✓ API secret
- ✓ Database credentials

---

### 9.4 Tệp tfvars - Cách dùng

Ở phần 4, bạn học cơ bản. Bây giờ học cách dùng trong thực tế.

#### Cách 1: `terraform.tfvars` (tự động load)

**File structure:**
```
project/
├── main.tf
├── variables.tf
├── terraform.tfvars   ← Tự động load
└── outputs.tf
```

**variables.tf:**
```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "environment" {
  type    = string
  default = "dev"
}

variable "server_count" {
  type    = number
  default = 1
}
```

**terraform.tfvars:**
```hcl
instance_type = "t2.small"
environment   = "production"
server_count  = 3
```

**Chạy:**
```bash
terraform plan
# Tự động dùng giá trị từ terraform.tfvars
```

#### Cách 2: `-var-file` (chọn file)

**File structure:**
```
project/
├── main.tf
├── variables.tf
├── dev.tfvars   ← Cho môi trường dev
├── prod.tfvars  ← Cho môi trường prod
└── outputs.tf
```

**dev.tfvars:**
```hcl
instance_type = "t2.micro"
environment   = "dev"
server_count  = 1
db_password   = "dev-password-123"
```

**prod.tfvars:**
```hcl
instance_type = "t3.large"
environment   = "prod"
server_count  = 5
db_password   = "prod-secure-password"
```

**Chạy cho dev:**
```bash
terraform plan -var-file="dev.tfvars"
terraform apply -var-file="dev.tfvars"
```

**Chạy cho prod:**
```bash
terraform plan -var-file="prod.tfvars"
terraform apply -var-file="prod.tfvars"
```

#### Cách 3: `-var` (command line)

```bash
# Ghi đè 1 biến
terraform plan -var="instance_type=t3.medium"

# Ghi đè nhiều biến
terraform apply \
  -var="instance_type=t3.large" \
  -var="server_count=5" \
  -var="environment=prod"
```

#### Cách 4: Environment variable (`TF_VAR_`)

```bash
# Set environment variable
export TF_VAR_instance_type=t3.medium
export TF_VAR_server_count=5

# Terraform tự động đọc
terraform plan
```

**Kiểm tra:**
```bash
echo $TF_VAR_instance_type
# Kết quả: t3.medium
```

#### Ưu tiên (nếu nhiều cách):

**Cao nhất → Thấp nhất:**
1. `-var` (command line) ← **Ưu tiên cao nhất**
2. `-var-file`
3. `terraform.tfvars`
4. `TF_VAR_` environment variable
5. `default` value ← **Ưu tiên thấp nhất**

**Ví dụ:**
```bash
# terraform.tfvars: instance_type = "t2.micro"
# default: instance_type = "t2.small"
# command line: -var="instance_type=t3.large"

terraform plan -var="instance_type=t3.large"
# Kết quả: dùng t3.large (command line thắng)
```

---

## PHẦN 10: OUTPUT VALUES (HIỂN THỊ KỸT QUẢ)

Ở phần 4, bạn học cơ bản output. Bây giờ học các tính năng nâng cao.

### 10.1 Output cơ bản

**Nhớ lại:**
```hcl
output "instance_id" {
  description = "ID của máy chủ"
  value       = aws_instance.web.id
}
```

**Chạy `terraform apply`:**
```
Outputs:

instance_id = "i-0a1b2c3d4e5f6g7h8"
```

### 10.2 Output từ multiple resources (nhiều cái)

#### Vấn đề:
Bạn tạo 3 máy chủ với `count`, muốn output **tất cả ID**:

```hcl
resource "aws_instance" "servers" {
  count         = 3
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
}

# Làm sao lấy tất cả ID?
```

#### Giải pháp: Dùng `[*]` (splat)

```hcl
output "all_server_ids" {
  description = "ID của tất cả máy chủ"
  value       = aws_instance.servers[*].id
}

output "all_server_ips" {
  description = "IP của tất cả máy chủ"
  value       = aws_instance.servers[*].public_ip
}
```

**Chạy `terraform apply`:**
```
Outputs:

all_server_ids = [
  "i-0a1b2c3d4e5f6g7h8",
  "i-1a2b3c4d5e6f7g8h9",
  "i-2a3b4c5d6e7f8g9h0"
]

all_server_ips = [
  "54.175.102.1",
  "54.175.102.2",
  "54.175.102.3"
]
```

#### Với `for_each`:

```hcl
resource "aws_s3_bucket" "buckets" {
  for_each = {
    dev  = "my-dev-bucket"
    uat  = "my-uat-bucket"
    prod = "my-prod-bucket"
  }
  bucket = each.value
}

output "bucket_names" {
  value = {
    for env, bucket in aws_s3_bucket.buckets : env => bucket.id
  }
}
```

**Output:**
```
bucket_names = {
  "dev"  = "my-dev-bucket"
  "prod" = "my-prod-bucket"
  "uat"  = "my-uat-bucket"
}
```

### 10.3 Output nhạy cảm (Sensitive output)

#### Vấn đề:
Output chứa mật khẩu, key:

```hcl
resource "random_password" "db" {
  length = 16
}

output "db_password" {
  value = random_password.db.result  # Hiển thị password!
}

# Chạy apply → Mọi người thấy password
```

❌ Lộ mật khẩu.

#### Giải pháp: Mark as sensitive

```hcl
output "db_password" {
  value     = random_password.db.result
  sensitive = true  # Ẩn output
}
```

**Chạy `terraform apply`:**
```
Outputs:

db_password = <sensitive>
```

✓ **Mật khẩu bị ẩn!**

Nhưng nếu bạn muốn xem:
```bash
terraform output db_password
# Kết quả: xxxxxxxxxx (còn ẩn)

# Hoặc xem state file (không khuyến khích)
terraform state show random_password.db
```

### 10.4 Output dạng map/object

#### Tạo output phức tạp:

```hcl
output "server_info" {
  description = "Thông tin máy chủ"
  value = {
    ids           = aws_instance.web[*].id
    ips           = aws_instance.web[*].public_ip
    instance_type = "t2.micro"
    region        = "us-east-1"
  }
}
```

**Output:**
```
server_info = {
  "ids"            = [
    "i-0a1b2c3d4e5f6g7h8",
    "i-1a2b3c4d5e6f7g8h9",
  ]
  "ips"            = [
    "54.175.102.1",
    "54.175.102.2",
  ]
  "instance_type"  = "t2.micro"
  "region"         = "us-east-1"
}
```

#### Output dạng bảng:

```hcl
output "servers_table" {
  value = [
    for i, server in aws_instance.servers : {
      index = i
      id    = server.id
      ip    = server.public_ip
      az    = server.availability_zone
    }
  ]
}
```

**Output:**
```
servers_table = [
  {
    "az"    = "us-east-1a"
    "id"    = "i-0a1b2c3d4e5f6g7h8"
    "index" = 0
    "ip"    = "54.175.102.1"
  },
  {
    "az"    = "us-east-1b"
    "id"    = "i-1a2b3c4d5e6f7g8h9"
    "index" = 1
    "ip"    = "54.175.102.2"
  },
]
```

### 10.5 Lấy output từ dòng lệnh

```bash
# Lấy 1 output cụ thể
terraform output instance_id
# Kết quả: i-0a1b2c3d4e5f6g7h8

# Lấy output dạng JSON
terraform output -json
# Kết quả:
# {
#   "instance_id": {
#     "value": "i-0a1b2c3d4e5f6g7h8"
#   },
#   "instance_ip": {
#     "value": "54.175.102.97"
#   }
# }

# Lấy 1 phần tử từ list
terraform output -json | jq '.all_server_ips.value[0]'
# Kết quả: "54.175.102.1"
```

---

## PHẦN 11: DATA SOURCES (NGUỒN DỮ LIỆU)

### 11.1 Data source là gì?

**Data source** = Lấy thông tin từ AWS (hoặc hệ thống khác) **mà không tạo gì mới**.

Khác biệt:

| Resource | Data Source |
|----------|-------------|
| **Tạo mới** | **Chỉ đọc** |
| VD: Tạo máy chủ | VD: Lấy ID hình ảnh |
| Terraform quản lý | AWS quản lý sẵn |

### 11.2 Data source phổ biến

#### Data source: `aws_ami` (Lấy hình ảnh)

**Vấn đề:**
Bạn muốn tạo máy chủ từ **Amazon Linux 2023 mới nhất**, nhưng không biết ID.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-???????????"  # ID nào?
  instance_type = "t2.micro"
}
```

❌ Cần tìm ID thủ công trên AWS Console.

#### Giải pháp: Dùng data source

```hcl
# Tìm hình ảnh Amazon Linux 2023 mới nhất
data "aws_ami" "latest_linux" {
  most_recent = true  # Hình ảnh mới nhất
  owners      = ["amazon"]  # Từ chính AWS
  
  filter {
    name   = "name"
    values = ["al2023-ami-2023*"]  # Tên hình ảnh
  }
  
  filter {
    name   = "architecture"
    values = ["x86_64"]  # 64-bit
  }
}

# Dùng ID từ data source
resource "aws_instance" "web" {
  ami           = data.aws_ami.latest_linux.id  # Lấy từ data source
  instance_type = "t2.micro"
}

output "ami_id" {
  value = data.aws_ami.latest_linux.id
}
```

**Chạy `terraform apply`:**
```
data.aws_ami.latest_linux: Reading...
data.aws_ami.latest_linux: Read complete after 2s

aws_instance.web: Creating...
aws_instance.web: Creation complete after 30s [id=i-0a1b2c3d4e5f6g7h8]

Outputs:

ami_id = "ami-0a3c3a20c09d6f377"
```

✓ **Tự động tìm hình ảnh mới nhất!**

#### Lợi ích:
- ✓ Không cần nhập ID thủ công
- ✓ Luôn dùng hình ảnh mới nhất
- ✓ Code dễ bảo trì

### 11.3 Data source: VPC, Subnet, Security Group

#### Tìm VPC mặc định:

```hcl
data "aws_vpc" "default" {
  default = true  # VPC mặc định của account
}

output "vpc_id" {
  value = data.aws_vpc.default.id
}
```

#### Tìm Subnet trong VPC:

```hcl
data "aws_subnets" "available" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]  # Trong VPC này
  }
  
  filter {
    name   = "availability-zone"
    values = ["us-east-1a"]  # Khu vực này
  }
}

resource "aws_instance" "web" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  subnet_id     = data.aws_subnets.available.ids[0]  # Subnet đầu tiên
}
```

#### Tìm Security Group:

```hcl
data "aws_security_group" "default" {
  name   = "default"
  vpc_id = data.aws_vpc.default.id
}

resource "aws_instance" "web" {
  ami                    = "ami-0df435f331839b2d6"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [data.aws_security_group.default.id]
}
```

### 11.4 Data source: Key Pair

#### Tìm key pair sẵn có:

```hcl
data "aws_key_pair" "my_key" {
  key_name           = "my-key-pair"  # Tên key
  include_public_key = true
}

resource "aws_instance" "web" {
  ami           = "ami-0df435f331839b2d6"
  instance_type = "t2.micro"
  key_name      = data.aws_key_pair.my_key.key_name
}

output "public_key" {
  value = data.aws_key_pair.my_key.public_key
}
```

### 11.5 Data source: Availability Zones

#### Lấy danh sách AZ:

```hcl
data "aws_availability_zones" "available" {
  state = "available"  # Chỉ AZ có sẵn
}

output "azs" {
  value = data.aws_availability_zones.available.names
}
```

**Output:**
```
azs = [
  "us-east-1a",
  "us-east-1b",
  "us-east-1c",
  "us-east-1d",
]
```

#### Dùng để tạo máy chủ ở các AZ khác nhau:

```hcl
resource "aws_instance" "web" {
  count               = length(data.aws_availability_zones.available.names)
  ami                 = "ami-0df435f331839b2d6"
  instance_type       = "t2.micro"
  availability_zone   = data.aws_availability_zones.available.names[count.index]
  
  tags = {
    Name = "Web-${count.index + 1}"
    AZ   = data.aws_availability_zones.available.names[count.index]
  }
}
```

### 11.6 Data source vs Resource - Tóm tắt

**Resource:**
```hcl
resource "aws_instance" "web" {  # Tạo máy chủ
  ami = "ami-0df435f331839b2d6"
}
```

**Data source:**
```hcl
data "aws_ami" "latest" {  # Tìm hình ảnh sẵn có
  most_recent = true
}
```

**Cách dùng:**
- Resource: `resource.name`
- Data source: `data.type.name`

**Ví dụ:**
```hcl
resource "aws_instance" "web" { ... }
data "aws_ami" "latest" { ... }

output "resource_id" {
  value = aws_instance.web.id  # Từ resource
}

output "data_id" {
  value = data.aws_ami.latest.id  # Từ data source
}
```

### 11.7 Khi nào dùng data source?

- ✓ Lấy ID hình ảnh mới nhất
- ✓ Tìm VPC/Subnet/Security Group sẵn có
- ✓ Lấy thông tin từ resource cũ (không do Terraform tạo)
- ✓ Lấy dữ liệu động (số AZ, key pair, ...)

---

## 📋 TÓM TẮT PHẦN 8-11

### Phần 8: Meta-Arguments
- `count` → Tạo N cái giống nhau
- `for_each` → Tạo từ danh sách
- `depends_on` → Chỉ định phụ thuộc
- `provider` → Dùng nhiều provider
- `lifecycle` → Kiểm soát vòng đời

### Phần 9: Biến Nâng Cao
- `validation` → Kiểm tra giá trị
- `sensitive` → Ẩn thông tin nhạy cảm
- Cách dùng tfvars trong thực tế

### Phần 10: Output
- Output từ multiple resources
- Output nhạy cảm
- Output phức tạp (map, object)
- Lấy output từ dòng lệnh

### Phần 11: Data Sources
- Lấy thông tin từ AWS
- `aws_ami`, `aws_vpc`, `aws_subnets`, ...
- Khi nào dùng data source

---

## ⚡ CHEAT SHEET PHẦN 8-11

**Meta-Arguments:**
```hcl
# count
count = 3
aws_instance.servers[*].id

# for_each
for_each = {dev = "bucket1", prod = "bucket2"}
each.key, each.value

# depends_on
depends_on = [aws_security_group.web]

# provider
provider = aws.us

# lifecycle
lifecycle { create_before_destroy = true }
```

**Validation:**
```hcl
validation {
  condition     = contains(["a", "b"], var.value)
  error_message = "Error!"
}
```

**Sensitive:**
```hcl
variable "password" {
  sensitive = true
}

output "password" {
  sensitive = true
}
```

**Data Source:**
```hcl
data "aws_ami" "latest" {
  most_recent = true
  filter { name = "name", values = ["..."] }
}

aws_instance.web.ami = data.aws_ami.latest.id
```

---

Bạn đã học xong **Phần 8-11**! Muốn tiếp tục **Phần 12+** không? 🚀
