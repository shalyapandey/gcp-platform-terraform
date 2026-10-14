# Practice Methodology, Git Workflow & Daily Cheat Sheet

This guide contains your day-to-day operational instructions: how to back up your work to GitHub, how to study effectively without copy-pasting, how to keep your GCP bill near $0, and your daily Terraform/Git command reference.

---

## 1. Connecting & Pushing to Your Personal GitHub Account

### First-Time GitHub Setup (Pick Option A or B)

#### Option A: One-Click in VS Code (Fastest)
1. Open VS Code in this repository (`gcp-platform-terraform`).
2. Click the **Source Control** icon on the left sidebar (or press `Ctrl + Shift + G`).
3. Click the blue **"Publish Branch"** button.
4. Select **"Publish to GitHub public repository"** (great for showing employers!) or **"Publish to GitHub private repository"**.

#### Option B: Via GitHub.com + Terminal
1. Go to [https://github.com/new](https://github.com/new) and create a repository named `gcp-platform-terraform` (**leave "Add a README file" unchecked**).
2. Run these commands in your terminal:
   ```bash
   git remote add origin https://github.com/<YOUR_GITHUB_USERNAME>/gcp-platform-terraform.git
   git branch -M main
   git push -u origin main
   ```

---

## 2. Your Daily "Never Lose Work" Git Workflow

### Months 1: Direct Commits at the End of Each Study Session
At the end of every study session (after destroying any expensive GCP resources):
```bash
# 1. Check which files changed (ensure no .tfstate or .json keys appear!)
git status

# 2. Stage your changes
git add .

# 3. Commit with a clear message
git commit -m "feat(week-1): bootstrap gcs state bucket and automation sa"

# 4. Push to GitHub
git push
```

### Months 2–4: Real Platform Engineering PR Workflow (Even When Working Solo!)
Starting in Month 2, never push directly to `main`. Practice like a real Platform Engineering team:
```bash
# 1. Create a feature branch for the week's task
git checkout -b feat/week-5-golden-path-module

# 2. Format and validate your Terraform before committing
terraform fmt -recursive
terraform validate

# 3. Commit and push the feature branch
git add .
git commit -m "feat(module): add golden-path cloud run v2 module with validations"
git push -u origin feat/week-5-golden-path-module
```
Then open a **Pull Request** on GitHub, review your own diff (and automated GitHub Actions `terraform plan` comment in Month 3+), and click **Merge Pull Request**. By Month 4, you will have 30+ closed PRs in your portfolio!

### What `.gitignore` Protects You From Committing
Your [`.gitignore`](./.gitignore) is pre-configured to block:
* `*.tfstate` and `*.tfstate.*` — **NEVER** commit state files to Git; they contain plaintext secrets.
* `*.tfvars` — Local variable values (if you want to commit a template, name it `terraform.tfvars.example`).
* `**/.terraform/*` — Large provider plugin binaries downloaded by `terraform init`.
* `*credentials*.json` and `*-key.json` — GCP credentials.

---

## 3. How to Practice & Grow (From Beginner to Expert)

### Rule #1: Set Up a Zero-Cost GCP Sandbox ("Destroy Before Bed")
* **Use the $300 GCP Free Trial** on a personal Google account, and set a $5 Billing Budget Alert right away.
* **Always run `terraform destroy` on billable resources before ending your session:**
  * *Free/Pennies to leave running:* GCS buckets (empty), Service Accounts, IAM bindings, VPC networks, Subnets, Firewall rules, Secret Manager (with <6 secret versions).
  * *Costs money hourly if left running 24/7:* **Cloud NAT**, **GKE Clusters**, **Cloud SQL instances**, **Compute Engine VMs**, **Load Balancers**.
  * Running `terraform apply` at the start of your study session and `terraform destroy` at the end proves your code is 100% reproducible!

### Rule #2: The "Blank File + Official Docs" Method (No Copy-Pasting)
To build real muscle memory, never copy-paste full blocks from tutorials:
1. Put **VS Code** on the left half of your screen and the **[Terraform Google Provider Docs](https://registry.terraform.io/providers/hashicorp/google/latest/docs)** on the right half.
2. On every documentation page, train yourself to read these 4 sections:
   * **Example Usage:** Shows the basic HCL block structure.
   * **Argument Reference:** Shows all inputs you can configure (*Required* vs. *Optional*).
   * **Attributes Reference:** Shows what outputs the resource generates *after* creation (e.g., `.id`, `.self_link`, `.uri`, `.email`) so you can wire it to other resources.
   * **Import (at the bottom of the page):** Shows the exact ID format needed to `import` an existing GCP resource into Terraform.

### Rule #3: Run the Weekly "Chaos Drills"
Beginners only practice the happy path (`terraform apply`). Senior Platform Engineers are paid for what they do when things go wrong:
1. **ClickOps Drift Drill:** Change a resource manually in the GCP Console -> run `terraform plan` to see how Terraform catches the drift -> run `terraform apply` to overwrite the manual change.
2. **Rename Without Deleting (`moved` block):** Rename a resource in `.tf` code -> see Terraform plan a destructive replacement (`-/+`) -> add a `moved {}` block so `terraform plan` shows `0 to destroy`.
3. **Brownfield Import (`import` block):** Create a resource manually in the GCP Console -> bring it under Terraform management using an `import {}` block and `terraform plan -generate-config-out=imported.tf`.
4. **The `count` vs. `for_each` Trap:** Create a list of resources with `count`, remove the first item, and watch every subsequent resource get destroyed/recreated due to index shifting (`[1] -> [0]`). Convert to `for_each` to fix it permanently.

---

## 4. Essential Daily CLI Cheat Sheet

### Google Cloud Auth (Zero JSON Keys)
```bash
# Log in to gcloud CLI
gcloud auth login

# Set your active GCP project
gcloud config set project <YOUR_GCP_PROJECT_ID>

# Authenticate Terraform locally via Application Default Credentials (ADC)
gcloud auth application-default login
```

### Daily Terraform Commands
```bash
# Initialize provider plugins and backend (run first in any new folder)
terraform init

# Migrate local state to GCS remote backend after adding `backend "gcs" {}`
terraform init -migrate-state

# Auto-format all .tf files cleanly
terraform fmt -recursive

# Check syntax and internal consistency without calling GCP APIs
terraform validate

# Run native unit/integration tests (.tftest.hcl)
terraform test

# Preview what Terraform will create (+), change (~), or replace (-/+)
terraform plan

# Apply changes to GCP
terraform apply

# Inspect resources currently tracked in your state file
terraform state list
terraform state show <resource_address>

# Tear down billable resources at the end of a study session
terraform destroy
```

---

## 5. Recommended Reference Bookmarks
* **Provider Docs:** [Terraform Google Provider Registry](https://registry.terraform.io/providers/hashicorp/google/latest/docs)
* **GCP Official Best Practices:** [Best Practices for Using Terraform on Google Cloud](https://cloud.google.com/docs/terraform/best-practices/general-style-structure)
* **Google's Reference Architecture:** [GoogleCloudPlatform/cloud-foundation-fabric (FAST)](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric/tree/master/fast)
* **Book:** *Terraform: Up & Running (3rd Edition)* by Yevgeniy Brikman
