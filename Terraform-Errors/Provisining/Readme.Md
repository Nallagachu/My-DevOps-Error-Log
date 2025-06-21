# 🚧 Terraform Provisioning Errors & Resolutions Guide

This document tracks the common provisioning errors encountered during a Terraform project and how they were resolved. It includes syntax issues, misconfigurations, and state locking problems while using AWS, `aws_instance` resources, remote backends (S3 & DynamoDB), and provisioners.

---

## ⚠️ Error 1: Unclosed Configuration Block

**Error Message:**
```bash
Error: Unclosed configuration block
on ec2.tf line 1, in resource "aws_instance" "roboshop":
1: resource "aws_instance" "roboshop" {
Root Cause:
The resource block was missing a closing brace (}).

Resolution:
Add the missing closing brace at the end of your resource:

hcl
Copy
Edit
provisioner "local-exec" {
  command = "echo ${self.private_ip} > iventory"
}
}  # ← This was missing
⚠️ Error 2: Attribute Redefined
Error Message:

bash
Copy
Edit
Error: Attribute redefined
on ec2.tf line 16, in resource "aws_instance" "roboshop":
16: command = "echo 'Destroy-time provisioner'"
Root Cause:
The command argument was defined more than once within the same provisioner block.

Resolution:
Separate the two local-exec provisioners properly:

hcl
Copy
Edit
provisioner "local-exec" {
  command     = "echo ${self.private_ip} > iventory"
  on_failure  = continue
}

provisioner "local-exec" {
  when        = destroy
  command     = "echo 'Destroy-time provisioner'"
  on_failure  = continue
}
⚠️ Error 3: Invalid Expression in Inline Block
Error Message:

bash
Copy
Edit
Error: Invalid expression
on ec2.tf line 37, in resource "aws_instance" "roboshop":
37:     inline =
38:     [
Root Cause:
A remote-exec block was not closed before another started, resulting in broken syntax.

Resolution:
Ensure proper brace closure between provisioner blocks:

hcl
Copy
Edit
provisioner "remote-exec" {
  inline = [
    "sudo dnf install nginx -y",
    "sudo systemctl start nginx"
  ]
}

provisioner "remote-exec" {
  when = destroy
  inline = [
    "systemctl stop nginx"
  ]
}
⚠️ Error 4: State Lock - PreconditionFailed (HTTP 412)
Error Message:

bash
Copy
Edit
Error: Error acquiring the state lock
Error message: operation error S3: PutObject, https response error StatusCode: 412
api error PreconditionFailed: At least one of the pre-conditions you specified did not hold
Root Cause:
Another process or user modified the Terraform state file in S3 between read and write operations. Terraform enforces optimistic locking to avoid corruption.

🔍 Analysis:
Lock Info helps identify who/what is holding the lock.

It’s usually triggered by:

Simultaneous Terraform operations.

CI/CD interference.

A stale lock after an interrupted process.

✅ Resolutions:
✅ Option 1: Wait & Retry
Let the current lock expire or wait until the other process finishes.

✅ Option 2: Force Unlock (Use with caution!)
bash
Copy
Edit
terraform force-unlock <LOCK_ID>
Only use if you're absolutely sure no active Terraform command is running.

✅ Option 3: Clean Up State
If you're trying to modify variables but old resources are still present:

bash
Copy
Edit
terraform state rm 'aws_instance.roboshop["mongodb"]'
terraform state rm 'aws_instance.roboshop["redis"]'
Then rerun:

bash
Copy
Edit
terraform plan
terraform apply
🧠 Key Learnings
Terraform syntax is strict — always match opening and closing braces.

Each block (e.g., provisioner, inline) must be complete before starting a new one.

Backend locking via S3/DynamoDB ensures consistency but can cause issues if not handled carefully.

terraform state commands are powerful tools to realign your state with real infrastructure.

📁 Structure Reminder
Double-check that:

Your backend block is in the correct file (main.tf or backend.tf)

The .terraform.lock.hcl is clean and valid

You're not duplicating attribute names inside any resource or block

✅ Suggested Improvements
Add syntax linting via tflint or terraform validate to catch issues early

Automate cleanup of stale locks in dev environments

Version-control changes to state-sensitive files carefully