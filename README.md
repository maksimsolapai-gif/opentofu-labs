
# 05. IasC: Terraform and Terragrunt Report

## 🔗 Repository Links
https://github.com/maksimsolapai-gif/opentofu-labs

📑 Final Report: Homeworks

📁 Final Repository Directory Structure

```
~/opentofu-labs/
├── lab1/
│   ├── .gitignore
│   ├── main.tf
│   ├── README.md
│   └── versions.tf
├── lab2/
│   ├── .gitignore
│   ├── main.tf
│   ├── outputs.tf
│   ├── README.md
│   ├── terraform.tfvars
│   ├── terraform.tfvars.example
│   └── variables.tf
└── lab3/
    ├── .gitignore
    ├── main.tf
    ├── outputs.tf
    ├── README.md
    └── modules/
        └── config-file/
            ├── main.tf
            ├── outputs.tf
            └── variables.tf
```
The local cache folders .terraform/ and dynamic state files terraform.tfstate are present on the disk but are hidden from the tracking tree because they have been successfully added to .gitignore.

---
🛠️ Configuration Files Reference📦 Laboratory Work №1: Basic Lifecycle

lab1/versions.tf

```
hcl
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.6.0"
    }
  }
}
```

lab1/main.tf
```
hcl
resource "random_pet" "my_pet" {
  length    = 3
  separator = "-"
}

resource "local_file" "pet_file" {
  filename = "${path.module}/pet_name.txt"
  content  = "My awesome pet name is: ${random_pet.my_pet.id}\n"
}
```

⚙️ Laboratory Work №2: Variables and Validation

lab2/variables.tf

```
hcl
variable "filename_prefix" {
  type        = string
  description = "Prefix for the created file name"
  default     = "secure-pet"

  validation {
    condition     = length(var.filename_prefix) > 3
    error_message = "The filename prefix must be longer than 3 characters."
  }
}

variable "secret_token" {
  type        = string
  description = "Secret token that should be masked in logs"
  sensitive   = true
}

variable "pet_length" {
  type        = number
  description = "Number of words in the generated pet name"
  default     = 2
}
```
lab2/main.tf
```
hcl
terraform {
  required_providers {
    local  = { source = "hashicorp/local" }
    random = { source = "hashicorp/random" }
  }
}

locals {
  computed_filename = "${upper(var.filename_prefix)}-CONFIG.txt"
}

resource "random_pet" "advanced_pet" {
  length = var.pet_length
}

resource "local_file" "advanced_file" {
  filename = "${path.module}/${local.computed_filename}"
  content  = "Token: ${var.secret_token}\nPet name: ${random_pet.advanced_pet.id}\n"
}
Используйте код с осторожностью.lab2/outputs.tfhcloutput "generated_pet_name" {
  value       = random_pet.advanced_pet.id
  description = "The generated pet name string"
}

output "file_path" {
  value       = local_file.advanced_file.filename
  description = "The path to the created file"
}

output "exposed_secret" {
  value       = var.secret_token
  sensitive   = true
}
```
lab2/terraform.tfvars.example

```
hcl
filename_prefix = "example-prefix"
secret_token    = "example-secret-token"
pet_length      = 3
Используйте код с осторожностью.🧩 Laboratory Work №3: Encapsulation with Moduleslab3/modules/config-file/variables.tfhclvariable "env_name" {
  type        = string
  description = "Environment name (dev, stage, prod)"
}

variable "prefix" {
  type    = string
  default = "app"
}
```
lab3/modules/config-file/main.tf

```
hcl
resource "random_pet" "mod_pet" {
  length = 2
}

resource "local_file" "mod_file" {
  filename = "${path.root}/dist/${var.env_name}-${var.prefix}-config.txt"
  content  = "Environment: ${var.env_name}\nGenerated Hostname: ${var.prefix}-${random_pet.mod_pet.id}\n"
}
```

lab3/modules/config-file/outputs.tf

```
hcloutput "full_hostname" {
  value = "${var.prefix}-${random_pet.mod_pet.id}"
}
Используйте код с осторожностью.lab3/main.tfhclterraform {
  required_providers {
    local  = { source = "hashicorp/local" }
    random = { source = "hashicorp/random" }
  }
}

locals {
  environments = {
    dev   = "development-backend"
    stage = "staging-backend"
    prod  = "production-backend"
  }
}

module "config_environments" {
  source   = "./modules/config-file"
  for_each = local.environments

  env_name = each.key
  prefix   = each.value
}
```

lab3/outputs.tf
```
hcl
output "deployed_hostnames" {
  value       = { for k, v in module.config_environments : k => v.full_hostname }
  description = "A map of the generated hostnames across all environments"
}
```

🛡️ Unified .gitignore Template (Used across directories)text
```
.terraform/
.terraform.lock.hcl
terraform.tfstate*
.terragrunt-cache/
dist/
*.tfvars
```
