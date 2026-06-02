**"AWS configuration on VS Code"** setting up AWS CLI and connecting VS Code to your AWS account for DevOps work, follow this practical setup.

# 1. Install AWS CLI

### Verify installation

```bash
aws --version
```

If not installed:

### Linux

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install
```

### macOS

```bash
brew install awscli
```

### Windows

Download from:

[AWS CLI Installer](https://aws.amazon.com/cli/?utm_source=chatgpt.com)

---

# 2. Create AWS Access Keys

1. Login to AWS Console.
2. Open IAM.
3. Create a user (or use an existing user).
4. Attach required permissions.
5. Create Access Key.

You'll get:

```text
Access Key ID
Secret Access Key
```

---

# 3. Configure AWS CLI

Run:

```bash
aws configure
```

Enter:

```text
AWS Access Key ID: AKIA...
AWS Secret Access Key: xxxxx
Default region name: ap-south-1
Default output format: json
```

Example:

```bash
aws configure
```

```text
AWS Access Key ID [None]: AKIA********
AWS Secret Access Key [None]: ****************
Default region name [None]: ap-south-1
Default output format [None]: json
```

---

# 4. Verify Configuration

```bash
aws sts get-caller-identity
```

Expected output:

```json
{
  "UserId": "...",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/devops-user"
}
```

---

# 5. Check AWS Config Files

Credentials:

```bash
cat ~/.aws/credentials
```

```ini
[default]
aws_access_key_id=AKIA...
aws_secret_access_key=...
```

Config:

```bash
cat ~/.aws/config
```

```ini
[default]
region=ap-south-1
output=json
```

---

# 6. Install AWS Toolkit Extension in VS Code

Install:

[AWS Toolkit for VS Code](https://aws.amazon.com/visualstudiocode/?utm_source=chatgpt.com)

Features:

* Browse EC2
* Browse S3
* View Lambda functions
* Connect AWS profiles
* CloudFormation support
* ECS/EKS integration

---

# 7. Connect AWS Account in VS Code

1. Open VS Code.
2. Press:

```text
Ctrl + Shift + P
```

3. Search:

```text
AWS: Connect to AWS
```

4. Select profile:

```text
default
```

or

```text
dev
```

---

# 8. Multiple AWS Profiles (Recommended)

Configure:

```bash
aws configure --profile dev
```

```bash
aws configure --profile prod
```

Use profile:

```bash
aws s3 ls --profile dev
```

Example credentials:

```ini
[default]
aws_access_key_id=xxx

[dev]
aws_access_key_id=yyy

[prod]
aws_access_key_id=zzz
```

---

# 9. Useful Commands for DevOps Engineers

### List EC2

```bash
aws ec2 describe-instances
```

### List S3 Buckets

```bash
aws s3 ls
```

### List VPCs

```bash
aws ec2 describe-vpcs
```

### List EKS Clusters

```bash
aws eks list-clusters
```

### List IAM Users

```bash
aws iam list-users
```

---

# 10. Setup for Terraform

Verify AWS access:

```bash
aws sts get-caller-identity
```

Terraform automatically uses:

```bash
~/.aws/credentials
```

Test:

```bash
terraform init
terraform plan
```

---

For your DevOps learning path (AWS + Terraform + EKS), I recommend using **IAM Identity Center (SSO)** instead of long-lived access keys whenever possible, because most companies now use SSO-based authentication rather than storing permanent access keys on laptops.
