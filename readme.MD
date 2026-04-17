## Terraform AWS Security Group Module

This module creates a single **AWS Security Group** inside a given VPC.

It creates:

- 1 Security Group with name: `${project}-${environment}-${sg_name}`
- Default **egress allow-all** rule:
  - IPv4: `0.0.0.0/0`
  - IPv6: `::/0`

### What it does (flow)

- **1) Build tags**: merges `sg_tags` + common tags (`project`, `environment`, `terraform`)
- **2) Create security group**: `aws_security_group.main` in the provided `vpc_id`
- **3) Add egress rule**: allow all outbound traffic (IPv4 + IPv6)
- **4) Output**: exports the created Security Group ID

Diagram (simplified):

```text
VPC (vpc_id)
└─ Security Group: <project>-<environment>-<sg_name>
   └─ Egress: all protocols, all ports → 0.0.0.0/0 and ::/0
```

### Usage

```hcl
module "frontend_sg" {
  source = "../terraform-aws-securitygroups"

  project     = "roboshop"
  environment = "dev"

  sg_name        = "frontend"
  sg_description = "Frontend security group"
  vpc_id         = module.vpc.vpc_id

  sg_tags = {
    Owner = "team-a"
  }
}
```

Then run:

```bash
terraform init
terraform plan
terraform apply
```

### Inputs

- **`project` (required)**: Project name (used in name/tags)
- **`environment` (required)**: Environment name (used in name/tags)
- **`vpc_id` (required)**: VPC ID where the security group will be created
- **`sg_name` (required)**: Suffix name for the security group
- **`sg_description` (required)**: Description for the security group
- **`sg_tags` (optional)**: Additional tags to add (default `{}`)

### Outputs

- **`sg_id`**: ID of the created security group

### Notes / gotchas

- This module **does not create any ingress rules**. Add ingress rules in your root module (recommended) or extend this module if you want rules managed here.

### Cleanup

```bash
terraform destroy
```
