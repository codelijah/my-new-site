# terraform-webapp

A small web server on AWS, built entirely with Terraform. One command creates the server, and one command removes it.

## What it builds

- An **EC2 instance** (Amazon Linux 2023, t3.micro) in Stockholm (`eu-north-1`)
- A **security group** allowing ports 80 (HTTP), 22 (SSH) and 8080, with all outbound traffic allowed
- An **Elastic IP**, so the server keeps the same public address
- A **start-up script** (user data) that installs Docker, clones the website from GitHub, builds the Docker image and runs the container on port 80

The website itself lives in a separate repo: [my-new-site](https://github.com/codelijah/my-new-site). Pushing to that repo triggers a GitHub Actions workflow that deploys the change to the server over SSH.

## How it works

1. `terraform apply` creates the firewall, the server and the IP.
2. On first boot, the server runs the start-up script and starts the site.
3. GitHub Actions in `my-new-site` logs in with an SSH key (stored as a GitHub Secret) and updates the running container on each push.

## Tools used

Terraform, AWS (EC2, IAM, Elastic IP), Docker, nginx, Git, GitHub Actions, Bash.

## How to run it

Requirements: Terraform installed, an AWS account, and AWS credentials configured in `~/.aws/credentials` for an IAM user with limited permissions.

```
terraform init
terraform plan
terraform apply
```

The public IP is printed as `server_ip`. Wait a few minutes for the start-up script to finish, then open `http://<server_ip>`.

To remove everything and stop charges:

```
terraform destroy
```

## Notes

- The Terraform state file and the `.terraform/` folder are not committed (see `.gitignore`).
- No credentials are stored in this repo.
- Each rebuild gives a new IP, so the `EC2_HOST` secret in the website repo must be updated.
