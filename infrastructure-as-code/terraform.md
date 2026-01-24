# Terraform Interview Questions and Answers

This document contains common interview questions related to Terraform and their answers for interview preparation.

### 1. What are Providers in Terraform?
**Answer:**
Providers are plugins that allow Terraform to interact with APIs of cloud providers, SaaS providers, and other services (e.g., AWS, Azure, Kubernetes, GitHub). They translate Terraform configuration into API calls.

### 2. What is Terraform State?
**Answer:**
State is a file (`terraform.tfstate`) that maps real-world resources to your Terraform configuration.
*   **Purpose:** It enables diff calculation, dependency tracking, and idempotency.
*   **Content:** It stores metadata about resources, including IDs and attributes.

### 3. What is the Terraform workflow?
**Answer:**
1.  `terraform init`: Initializes the working directory, downloads providers and modules.
2.  `terraform plan`: Creates an execution plan, showing what actions Terraform will take.
3.  `terraform apply`: Executes the plan to create, update, or delete infrastructure.
4.  `terraform destroy`: Destroys all resources managed by the configuration.

### 4. What is the difference between `resource` and `data`?
**Answer:**

| Feature | Resource | Data Source |
| :--- | :--- | :--- |
| **Purpose** | Creates/updates infrastructure. | Reads existing infrastructure. |
| **Management** | Managed by Terraform. | Read-only (managed externally). |
| **State** | Stored in state file. | Referenced for information. |

### 5. What is a Backend in Terraform?
**Answer:**
A backend defines where the Terraform state file is stored and how operations are locked to prevent corruption.
*   **Example:** Storing state in **AWS S3** and using **DynamoDB** for state locking.

### 6. What are Terraform modules?
**Answer:**
Modules are reusable, composable Terraform configurations that encapsulate infrastructure logic. They allow you to group resources together and reuse them across different environments or projects.

### 7. Difference between `count` and `for_each`?
**Answer:**

| Feature | count | for_each |
| :--- | :--- | :--- |
| **Basis** | Index-based (0, 1, 2...). | Key-based (Map or Set). |
| **Stability** | Fragile on reorder (modifying list order forces recreation). | Stable (adding/removing keys doesn't affect others). |
| **Input** | Integers. | Maps or Sets of strings. |

### 8. What are input variables and outputs?
**Answer:**
*   **Input Variables:** Parameterize configurations, allowing values to be passed into modules (like function arguments).
*   **Outputs:** Expose values from a module to the root configuration or other modules (like return values).

### 9. How does Terraform detect changes?
**Answer:**
Terraform compares three things:
1.  **Desired state:** Your `.tf` configuration files (HCL).
2.  **Current state:** The `terraform.tfstate` file.
3.  **Real infrastructure:** The actual resources in the cloud (via provider API).

It then creates an execution plan to align the real infrastructure with the desired state.

### 10. What is drift in Terraform?
**Answer:**
Drift occurs when the actual infrastructure is modified outside of Terraform (e.g., manually via AWS Console), causing a mismatch between the real world and the Terraform state.

### 11. How do you handle secrets in Terraform?
**Answer:**
*   **Never** store secrets in plain text in `.tf` files.
*   **Avoid** storing secrets in state if possible (state is often unencrypted by default).
*   **Best Practices:**
    *   Use **AWS Secrets Manager** or **HashiCorp Vault**.
    *   Let services fetch secrets at runtime (e.g., ECS task fetching from Parameter Store).
    *   Use IAM roles instead of long-lived credentials.

### 12. What is state locking?
**Answer:**
State locking prevents concurrent write operations to the state file, which could corrupt it.
*   **Mechanism:** When running `apply`, Terraform locks the state. If another user tries to run `apply`, they are blocked until the lock is released.
*   **Implementation:** DynamoDB table (for S3 backend).

### 13. What happens if the terraform state is deleted?
**Answer:**
Terraform loses track of the infrastructure it created. The resources still exist in the cloud, but Terraform doesn't know about them. The next `terraform apply` will attempt to create new resources, potentially causing errors (e.g., "Resource already exists").

### 14. How do you import existing infrastructure?
**Answer:**
Use the `terraform import` command to bring unmanaged resources into Terraform state.
```bash
terraform import aws_instance.example i-123456
```
(Note: You still need to write the corresponding `resource` block in your configuration).

### 15. How do you manage multiple environments?
**Answer:**
*   **Workspaces:** Good for small setups with identical infrastructure (uses same backend, different state keys).
*   **Separate State Files (Recommended):** Use completely separate state files and folder structures (e.g., `envs/dev`, `envs/prod`) or a wrapper tool like Terragrunt.

### 16. What is `terraform refresh`?
**Answer:**
It updates the state file by querying the real infrastructure provider to match the current settings.
*   **Note:** In modern Terraform, `refresh` is performed automatically during `plan` and `apply`.

### 17. How does Terraform handle partial failures?
**Answer:**
If an `apply` fails halfway through:
1.  Terraform records the resources that were successfully created in the state file.
2.  It marks the failed or tainted resources.
3.  The next `apply` will attempt to resume or fix the configuration from that point.

### 18. What are Provisioners in Terraform?
**Answer:**
Provisioners allow you to run scripts or commands on a resource (or locally) after it is created or destroyed. They are used for bootstrapping or configuration management that Terraform cannot model natively.

**Types of Provisioners:**

**1️⃣ local-exec**
Runs a command on the machine where Terraform is running.
```hcl
provisioner "local-exec" {
  command = "echo ${self.public_ip} >> ips.txt"
}
```

**2️⃣ remote-exec**
Runs commands on the created resource itself (requires SSH/WinRM).
```hcl
provisioner "remote-exec" {
  inline = [
    "sudo apt update",
    "sudo apt install nginx -y"
  ]
}
```

**3️⃣ file**
Copies files from the local machine to the resource.
```hcl
provisioner "file" {
  source      = "app.conf"
  destination = "/etc/app.conf"
}
```
