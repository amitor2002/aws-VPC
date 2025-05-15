# 🚀 Full AWS Production CI/CD Network (Console-Only Deployment)

I built a fully private and production-ready CI/CD network on AWS **using only the AWS Console** — no Terraform, no CloudFormation. The goal was to practice designing a secure and scalable cloud architecture from scratch using core AWS services, spread across two Availability Zones for high availability.

### 🔧 What's Included?

- 🌐 **Public Weather App** served via an **Application Load Balancer (ALB)**
- 🦊 **Self-hosted GitLab** for source control (private subnet)
- 🔄 **Jenkins controller & agent** for CI/CD (private subnets)
- 🧱 Manually configured **VPC, subnets, NAT instance, route tables, security groups**
- 🧑‍💻 Optional **bastion host** with full SSH tunneling to private resources

---

## 🏗️ Infrastructure Breakdown

- **VPC CIDR:** `10.0.0.0/16`
- **Subnets:**
  - `10.0.1.0/24` – Public Subnet A (AZ a)
  - `10.0.2.0/24` – Public Subnet B (AZ b)
  - `10.0.10.0/24` – Private Subnet A (AZ a)
  - `10.0.11.0/24` – Private Subnet B (AZ b)
- **Internet Gateway (IGW):** Attached to the VPC for public access
- **Route Tables:**
  - Public: Routes `0.0.0.0/0` to IGW
  - Private: Routes `0.0.0.0/0` to NAT instance (ENI)
- **NAT Instance (not Gateway):**
  - Amazon Linux 2, t3.micro
  - Launched in Public Subnet B
  - Elastic IP + Source/Dest Check disabled
- **Security Groups:**
  - **ALB SG:** Inbound HTTP (80) from `0.0.0.0/0`
  - **Weather App SG:** Inbound from ALB only
  - **Jenkins/GitLab SG:** Inbound TCP 22, 80, 443 from VPC CIDR
  - **NAT SG:** Inbound SSH from admin IP, outbound to all
- **Application Load Balancer:**
  - Internet-facing
  - Listener on HTTP 80 → Target Group (Weather App)
  - Health checks on `/`
- **Instances:**
  - **Weather App:** EC2 in Public Subnet A (Target of ALB)
  - **GitLab & Jenkins Controller:** Private Subnet A
  - **Jenkins Agent:** Private Subnet B

---

## 🔐 Optional: Bastion Host & SSH Tunnels

To securely access **GitLab** and **Jenkins** running in private subnets, a **Bastion Host** is deployed in a public subnet. This setup allows tunneling from your local machine to internal services using SSH key forwarding and local port forwarding.

### ✅ Bastion Host Setup

- Launch a small EC2 instance in **Public Subnet A**
- Assign a **Security Group** that allows:
  - Inbound SSH (port 22) from your **local IP**
- From the Bastion, generate or upload SSH keys to access GitLab and Jenkins instances

### 🔐 SSH Tunnels (from your local machine)

Edit your `~/.ssh/config` file:

```ssh
Host bastion
  HostName <BASTION_PUBLIC_IP>
  User ubuntu
  IdentityFile ~/.ssh/id_rsa_bastion

Host gitlab
  HostName <GITLAB_PRIVATE_IP>
  User ubuntu
  IdentityFile ~/.ssh/id_rsa_bastion
  LocalForward 8443 <GITLAB_PRIVATE_IP>:80
  ProxyCommand ssh -W %h:%p bastion

Host jenkins
  HostName <JENKINS_PRIVATE_IP>
  User ubuntu
  IdentityFile ~/.ssh/id_rsa_bastion
  LocalForward 8880 <JENKINS_PRIVATE_IP>:8080
  ProxyCommand ssh -W %h:%p bastion
```

## ✅ Outcome

- 🌍 **Weather App** is publicly accessible through the **ALB**
- 🔐 **GitLab** and **Jenkins** are fully isolated in **private subnets**
- 🔄 The **Jenkins pipeline** runs internally, pulling from GitLab and building/deploying as expected
- 🧑‍💻 **SSH tunneling** enables secure admin/developer access to private services
- 🔌 Private subnets access the internet **only via the NAT instance**
- 🛠️ Every component was created **manually** in the AWS Console to reinforce deep understanding

---

## 🧠 Key Takeaways

- Hands-on AWS Console work builds real intuition for how cloud infrastructure operates under the hood
- Knowing how to configure VPCs, subnets, route tables, and security groups manually is essential for troubleshooting and architecture interviews
- This project mirrors a **real-world production setup** with security, availability, and modular design in mind
- Avoiding shortcuts (like public Jenkins or GitLab) pays off in learning how **secure CI/CD environments** should actually be structured
- Strong foundation for translating this architecture later into **Terraform or CloudFormation**

---

Thanks for checking this out!  
Feel free to fork, star ⭐️, or connect with me if you're building something similar — always happy to exchange ideas or give feedback.
