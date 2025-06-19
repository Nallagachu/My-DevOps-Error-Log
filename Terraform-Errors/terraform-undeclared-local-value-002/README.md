# Terraform Error: Reference to Undeclared Local Value

## 🗓️ Date Encountered: June 20, 2025
## 🏷️ Category: Terraform

---

## 🧐 The Problem I Faced

While building out my AWS EC2 instance configuration in Terraform, I wanted to centralize the security group ID using a `local` value. I attempted to reference `local.sg_id` for the `vpc_security_group_ids` attribute of my `aws_instance` resource.

My intention was to define the `sg_id` in a `locals` block, but I had either forgotten to create the `locals` block entirely or made a syntax error, leading to Terraform not recognizing the `sg_id` I was trying to reference.

## 🤯 The Exact Error Message

When I ran `terraform plan`, I received the following error:

╷
│ Error: Reference to undeclared local value
│
│   on ec2.tf line 5, in resource "aws_instance" "roboshop":
│    5:   vpc_security_group_ids = local.sg_id
│
│ A local value with the name "sg_id" has not been declared.


*(Optionally: Embed your `screenshot-error.png` here for visual clarity. You can add it after saving this file.)*

## 📜 Relevant Code Snippets (Before Fix)

This is the snippet of my `ec2.tf` that was causing the error:

```terraform
resource "aws_instance" "roboshop" {
  ami           = "ami-09c813fb71547fc4f"
  instance_type = "t3.micro"
  vpc_security_group_ids = local.sg_id # <-- Error here, local.sg_id was not declared
  tags = {
    Name = "HelloWorld"
  }
}

resource "aws_security_group" "allow_all" {
    name        = "allow_all"
    description = "allow all traffic"

    ingress {
        from_port        = 0
        to_port          = 0
        protocol         = "-1"
        cidr_blocks      = ["0.0.0.0/0"]
        ipv6_cidr_blocks = ["::/0"]
    }

    egress {
        from_port        = 0
        to_port          = 0
        protocol         = "-1"
        cidr_blocks      = ["0.0.0.0/0"]
        ipv6_cidr_blocks = ["::/0"]
    }

    tags = {
        Name = "allow-all"
    }
}
🤔 My Initial Thoughts & Misconceptions
When I first saw the error, I double-checked the spelling of sg_id. I initially thought maybe local values behaved like global variables once defined anywhere, or perhaps Terraform would infer it from the aws_security_group.allow_all.id directly without explicit declaration. However, the error message clearly stated it was "undeclared," pointing me directly to the issue of definition.

🔬 Troubleshooting Steps I Took
Ran terraform plan to see the error output.
Read the error message carefully: "A local value with the name 'sg_id' has not been declared."
Inspected my .tf files to find where sg_id should have been declared and realized the locals block was missing or incorrect.
💡 The Root Cause
The root cause was simply that I attempted to use a local value (local.sg_id) without defining it within a locals { ... } block in any of my Terraform configuration files. Terraform requires local values to be explicitly declared before they can be referenced.

✅ The Step-by-Step Solution
The solution was to correctly declare the sg_id within a locals block, assigning it the ID of the aws_security_group.allow_all resource.

Create or add to an existing locals.tf file (or any .tf file in your configuration) the following locals block:

Terraform

locals {
  sg_id = [aws_security_group.allow_all.id]
}
Note: I put this in a separate locals folder as part of my project organization.

Re-run terraform plan:
After adding the locals block, terraform plan executed successfully, showing no errors and indicating the output values I had configured.

📜 Relevant Code Snippets (After Fix)
The added locals block:

Terraform

locals {
  sg_id = [aws_security_group.allow_all.id]
}
🎓 What I Learned (Key Takeaways)
locals Block is Essential: Terraform's local values must always be defined within a locals { ... } configuration block. They are not globally inferred from their usage.
Clear Error Messages: Terraform's error messages are often very precise and point directly to the problem, like "Reference to undeclared local value." Reading them carefully is key.
Variable Scope: Understanding the scope of variables (input variables, output values, local values, resource attributes) is crucial in Terraform.
🔗 Related Resources / Tools
Terraform Documentation: Local Values

#### Part B: Content for `commands-executed.txt`

Open `commands-executed.txt` using VS Code or `nano`:
`nano commands-executed.txt` (or open file in VS Code)

Copy the entire content below and paste it into your `commands-executed.txt` file.

--- Error Encountered ---
LENOVO@pandujyo MINGW64 /c/devops/repos/terraform/ec2 (main)
$ terraform plan
╷
│ Error: Reference to undeclared local value
│
│   on ec2.tf line 5, in resource "aws_instance" "roboshop":
│    5:   vpc_security_group_ids = local.sg_id
│
│ A local value with the name "sg_id" has not been declared.

This error also appeared if run from other subdirectories, e.g.:
LENOVO@pandujyo MINGW64 /c/devops/repos/terraform/for-loops (main)
$ terraform plan
╷
│ Error: Reference to undeclared local value
│
│   on ec2.tf line 5, in resource "aws_instance" "roboshop":
│    5:   vpc_security_group_ids = local.sg_id
│
│ A local value with the name "sg_id" has not been declared.

--- Solution Applied (Adding locals block) ---
Assuming the following content was added to a .tf file (e.g., locals.tf) in the configuration:
locals {
sg_id = [aws_security_group.allow_all.id]
}
--- Verification After Fix ---
LENOVO@pandujyo MINGW64 /c/devops/repos/terraform/locals (main)
$ terraform plan

Changes to Outputs:

ec2_tags = {
Project = "roboshop"
Terraform = "true"
environment = "dev"
version = "1.0" }
fina_name = "roboshop-dev-cart"
You can apply this plan to save these new output values to the Terraform state, without
changing any real infrastructure.