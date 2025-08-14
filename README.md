# Deploy `arif0ahammed0reza/next-japan:latest` to AWS EC2 (Terraform + Ansible)

This project spins up a single Ubuntu EC2 instance, installs Docker and NGINX, runs your Docker image, and exposes it on port 80 with NGINX acting as an ingress (reverse proxy). **No SSL**.

## Prereqs (on your laptop)
- AWS credentials configured (e.g., with `aws configure`).
- Terraform >= 1.4
- Ansible >= 2.14
- Ansible Galaxy collection: `community.docker` (installer will prompt or run `ansible-galaxy collection install community.docker`).
- Your existing AWS key pair named **REZA-TA** in the chosen region. You have `REZA-TA.pem` locally.
- Open ports: 22 (SSH) and 80 (HTTP).

## 1) Configure Terraform
Edit `terraform/variables.tf` if needed:
- `region` (defaults to `ap-south-1`)
- `instance_type` (defaults to `t3.micro`)
- `key_name` (defaults to `REZA-TA` and **must already exist** in AWS)
- `ssh_ingress_cidr` (set to your IP in CIDR, e.g., `203.0.113.5/32` for better security)

## 2) Create the EC2 instance
```bash
cd terraform
terraform init
terraform apply
# confirm the plan; when finished, note the outputs:
terraform output public_ip
```

## 3) Point Ansible at the new server
Copy the public IP from Terraform and put it into `ansible/inventory.ini` replacing `X.X.X.X`.

## 4) Run Ansible
```bash
cd ../ansible
ansible-galaxy collection install community.docker
ansible-playbook -i inventory.ini -u ubuntu --key-file /path/to/REZA-TA.pem site.yml
```

When the play finishes, browse to:  
`http://<EC2_PUBLIC_IP>/`

## Variables you can tweak
By default we assume the container exposes port **3000** internally (typical for Next.js). If your image uses a different internal port, change `container_internal_port` and/or `app_port` in `ansible/site.yml` accordingly.

## Common fixes
- If SSH fails, ensure the key pair **name** in Terraform matches the one that created your `REZA-TA.pem` and that you're in the correct **region**.
- Ubuntu AMI uses user `ubuntu` for SSH.
- If NGINX shows a 502, the app might not be listening on the expected port; adjust `app_port` or `container_internal_port` and rerun the playbook.
