# Terraform Interview Guide

> Source: [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `06-terraform/` folder

---

# Table of Contents

1. [for_each vs for in Terraform](#1-for_each-vs-for-in-terraform)
2. [Terraform Modules](#2-terraform-modules)

---

# 1. for_each vs for in Terraform

## Question
What is the difference between `for_each` and `for` in Terraform?

### Short Explanation
Both keywords relate to "looping," yet they operate at **different layers**: `for_each` is a **meta-argument** that drives how many resource instances Terraform creates, whereas `for` is an **expression** you embed inside variables, locals, or arguments to build or transform collections.

## ✅ Answer
- **`for_each`** tells Terraform to create **one instance per element** in a map or set (resources, modules, or data blocks).
- **`for`** is a collection **expression** used to **generate or reshape** lists/maps; it never creates resources on its own.

### Detailed Explanation

| Aspect | `for_each` (meta-argument) | `for` (expression) |
|---|---|---|
| **Primary role** | Creates or manages **multiple instances** | Builds or transforms **values** in place |
| **Where used** | Resource, module, data, `dynamic` block header | Any argument, variable default, local, output |
| **Input type** | Map or set of strings | List, map, or set (any iterable) |
| **Outcome** | New resource addresses (`aws_instance.web["a"]`) | A new collection value (no extra resources) |

---

#### Example 1 — `for_each` creating three S3 buckets

```hcl
resource "aws_s3_bucket" "logs" {
  for_each = {
    us = "us-east-1"
    eu = "eu-west-1"
    ap = "ap-southeast-1"
  }

  bucket = "app-logs-${each.key}"
  region = each.value
}
```

Terraform plans three distinct buckets:
`aws_s3_bucket.logs["us"]`, `aws_s3_bucket.logs["eu"]`, `aws_s3_bucket.logs["ap"]`.

#### Example 2 — `for` expression shaping data

```hcl
variable "allowed_cidrs" {
  type    = list(string)
  default = ["10.0.0.0/24", "10.1.0.0/24"]
}

locals {
  cidr_to_rule = {
    for cidr in var.allowed_cidrs :
    cidr => { protocol = "tcp", port = 22 }
  }
}
```

This produces a map:

```json
{
  "10.0.0.0/24": { "protocol": "tcp", "port": 22 },
  "10.1.0.0/24": { "protocol": "tcp", "port": 22 }
}
```

### Key Takeaways

- `for_each` → use when you need Terraform to create N physical resources.
- `for` → use when you need a derived list or map for arguments or outputs.
- They complement each other: you might build a map with `for`, then pass it to `for_each` for scalable resource creation.

---

# 2. Terraform Modules

## Question
What are modules in Terraform and why should we use them?

### Short Explanation
Modules are the core unit of code organization in Terraform. This question tests your understanding of **reusability**, **abstraction**, and **clean code practices** in Infrastructure as Code.

## ✅ Answer
Terraform modules are **self-contained packages of Terraform configuration** that can be reused across different projects or components.
We use them to **avoid repetition**, **enforce consistency**, and **organize infrastructure** into logical components.

### Detailed Explanation

A **Terraform module** is just a folder with `.tf` files. The folder can be local, on GitHub, or even on the Terraform Registry.

There are two types of modules:
- **Root module**: the directory where `terraform apply` is run.
- **Child module**: a reusable set of configurations called from a root or another module.

---

#### 🔁 Why use modules?

| Benefit | Description |
|---|---|
| **Reusability** | Define once, use anywhere (e.g., a VPC module used by multiple environments) |
| **Abstraction** | Hide complex logic behind simple variables and outputs |
| **Consistency** | Standardize infrastructure setup across teams |
| **Scalability** | Easily manage large-scale infrastructure with modular breakdown |
| **Maintainability** | Isolate changes to individual components without affecting the whole setup |

---

#### 📦 Example: Creating an EC2 module

**Folder structure:**
```
modules/
  ec2-instance/
    main.tf
    variables.tf
    outputs.tf
```

**Calling the module from root:**

```hcl
module "my_ec2" {
  source        = "./modules/ec2-instance"
  instance_type = "t2.micro"
  ami_id        = "ami-0abcd1234"
}
```

---

### 🧠 Real-world Use Case

> "In our company, we had to provision EC2 instances with the same tags, monitoring setup, and IAM roles across multiple environments. Instead of duplicating this logic, we created a reusable `ec2-instance` module. This allowed dev teams to spin up compliant EC2s with just a few input variables."

---

### Key Takeaway

> "Terraform modules are like functions in programming — they promote clean, DRY (Don't Repeat Yourself) code and help build scalable, maintainable infrastructure."

---
