# Terraform Interview Guide

> **Source:** [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `06-terraform/` folder
> **Purpose:** Master Terraform concepts — loops, modules, state, and best practices for interviews

---

## Table of Contents

| # | Topic |
|---|-------|
| [1](#1-for_each-vs-for-in-terraform) | `for_each` vs `for` in Terraform |
| [2](#2-terraform-modules) | Terraform Modules |

---

## 1. `for_each` vs `for` in Terraform

> **Q: What is the difference between `for_each` and `for` in Terraform?**

### Short Explanation

Both keywords relate to "looping," yet they operate at **different layers**:

- `for_each` is a **meta-argument** that drives how many resource instances Terraform creates.
- `for` is an **expression** you embed inside variables, locals, or arguments to build or transform collections.

---

### Answer

| Keyword | Role |
|---|---|
| **`for_each`** | Tells Terraform to create **one instance per element** in a map or set (resources, modules, or data blocks) |
| **`for`** | A collection **expression** used to **generate or reshape** lists/maps — it never creates resources on its own |

---

### Detailed Comparison

| Aspect | `for_each` (meta-argument) | `for` (expression) |
|---|---|---|
| **Primary role** | Creates or manages **multiple instances** | Builds or transforms **values** in place |
| **Where used** | Resource, module, data, `dynamic` block header | Any argument, variable default, local, output |
| **Input type** | Map or set of strings | List, map, or set (any iterable) |
| **Outcome** | New resource addresses e.g. `aws_instance.web["a"]` | A new collection value (no extra resources created) |

---

### Examples

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

Terraform plans **three distinct buckets**:

| Resource Address |
|---|
| `aws_s3_bucket.logs["us"]` |
| `aws_s3_bucket.logs["eu"]` |
| `aws_s3_bucket.logs["ap"]` |

---

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

This produces a map (no new resources created):

```json
{
  "10.0.0.0/24": { "protocol": "tcp", "port": 22 },
  "10.1.0.0/24": { "protocol": "tcp", "port": 22 }
}
```

---

### Key Takeaways

| Scenario | Use |
|---|---|
| Need Terraform to create N physical resources | `for_each` |
| Need a derived list or map for arguments or outputs | `for` |
| Build a map dynamically, then create resources from it | **Both** — `for` builds the map, `for_each` consumes it |

> **Pro tip:** They complement each other — build a map with `for`, then pass it to `for_each` for scalable resource creation.

---

## 2. Terraform Modules

> **Q: What are modules in Terraform and why should we use them?**

### Short Explanation

Modules are the **core unit of code organization** in Terraform. This question tests your understanding of **reusability**, **abstraction**, and **clean code practices** in Infrastructure as Code.

---

### Answer

Terraform modules are **self-contained packages of Terraform configuration** that can be reused across different projects or components.

We use them to:
- **Avoid repetition**
- **Enforce consistency**
- **Organize infrastructure** into logical components

---

### Detailed Explanation

A **Terraform module** is simply a folder containing `.tf` files. It can be sourced from:

| Location | Example |
|---|---|
| Local path | `source = "./modules/ec2-instance"` |
| GitHub | `source = "github.com/org/repo//modules/vpc"` |
| Terraform Registry | `source = "terraform-aws-modules/vpc/aws"` |

**Two types of modules:**

| Type | Description |
|---|---|
| **Root module** | The directory where `terraform apply` is run |
| **Child module** | A reusable set of configurations called from a root or another module |

---

### Why Use Modules?

| Benefit | Description |
|---|---|
| **Reusability** | Define once, use anywhere (e.g., a VPC module used by multiple environments) |
| **Abstraction** | Hide complex logic behind simple variables and outputs |
| **Consistency** | Standardize infrastructure setup across teams |
| **Scalability** | Easily manage large-scale infrastructure with modular breakdown |
| **Maintainability** | Isolate changes to individual components without affecting the whole setup |

---

### Example: Creating and Using an EC2 Module

**Folder structure:**

```
modules/
  ec2-instance/
    main.tf        <- resource definitions
    variables.tf   <- input variables
    outputs.tf     <- exported values
```

**`modules/ec2-instance/variables.tf`**

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
}

variable "ami_id" {
  description = "AMI ID for the instance"
  type        = string
}
```

**Calling the module from root (`main.tf`):**

```hcl
module "my_ec2" {
  source        = "./modules/ec2-instance"
  instance_type = "t2.micro"
  ami_id        = "ami-0abcd1234"
}
```

---

### Real-World Use Case

> *"In our company, we had to provision EC2 instances with the same tags, monitoring setup, and IAM roles across multiple environments. Instead of duplicating this logic, we created a reusable `ec2-instance` module. This allowed dev teams to spin up compliant EC2s with just a few input variables."*

---

### Key Takeaway

> **Terraform modules are like functions in programming** — they promote clean, DRY (Don't Repeat Yourself) code and help build scalable, maintainable infrastructure.

---

## Quick Reference Cheatsheet

```
=========================================================
         TERRAFORM INTERVIEW CHEATSHEET
=========================================================

  FOR_EACH vs FOR
  ---------------
  for_each = meta-argument   -> creates N resource instances
  for      = expression      -> builds/transforms collections
  Tip: for{} builds a map -> pass to for_each for resource creation

  MODULES
  -------
  A module = any folder with .tf files
  Root module   = where terraform apply runs
  Child module  = reusable config called from root or another module
  Source types  = local path | GitHub URL | Terraform Registry
  Benefits      = DRY, abstraction, consistency, scalability

=========================================================
```

---

## Interview Tips

| Do This | Not This |
|---|---|
| Explain `for_each` creates resources; `for` creates values | Confuse them as the same type of construct |
| Mention combining both — `for` builds a map, `for_each` consumes it | Say `for` can be used to create resources |
| Describe modules as "functions for infrastructure" | Forget to mention root vs child module distinction |
| Give a real-world example (VPC/EC2 module reused across envs) | Just give a textbook definition |
| Mention module sources — local, GitHub, Terraform Registry | Assume modules only work locally |

---

## Resources

- [Terraform `for_each` Meta-Argument Docs](https://developer.hashicorp.com/terraform/language/meta-arguments/for_each)
- [Terraform `for` Expressions Docs](https://developer.hashicorp.com/terraform/language/expressions/for)
- [Terraform Modules Documentation](https://developer.hashicorp.com/terraform/language/modules)
- [Terraform Registry — Public Modules](https://registry.terraform.io/)
- [Original Source: devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide)

---

> Star this repo if it helped you prepare for your DevOps interview!
> Drop the next topic's raw notes and they will be formatted and added here.
