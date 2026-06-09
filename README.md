# 🚀 DevSecOps: Blue-Green Deployment of Swiggy Clone
### AWS ECS + AWS CodePipeline + Blue-Green Strategy

![AWS ECS](https://img.shields.io/badge/AWS_ECS-Container-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![AWS CodePipeline](https://img.shields.io/badge/CodePipeline-CI/CD-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Container-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![SonarQube](https://img.shields.io/badge/SonarQube-Quality-4E9BCD?style=for-the-badge&logo=sonarqube&logoColor=white)
![Trivy](https://img.shields.io/badge/Trivy-Security-1904DA?style=for-the-badge&logo=aquasecurity&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)

![](https://miro.medium.com/v2/resize:fit:802/1*sHlD2d3AfaxzYEDlegzHhg.png)

---

## 📌 Overview

A production-grade **DevSecOps pipeline** deploying a Swiggy-clone application
on **AWS ECS** using **Blue-Green deployment strategy** powered by
**AWS CodePipeline** and **AWS CodeDeploy** — with SonarQube code quality
scanning, Trivy security scanning, OWASP dependency checks, and
zero-downtime deployments.

---

## 🔵🟢 What is Blue-Green Deployment?

Blue-Green deployment minimizes downtime and risk during release of new
application versions. Two identical production environments — **Blue** and
**Green** — are maintained simultaneously.

- **Blue** = Live environment serving production traffic
- **Green** = Idle environment receiving new deployment

Once the new version is validated on Green, traffic switches instantly
from Blue to Green — enabling **zero-downtime deployment** and
**instant rollback** if issues arise.

---

## 🏗️ CI/CD Pipeline Flow

SOURCE STAGE
└── GitHub push → CodePipeline triggered automatically
BUILD STAGE (AWS CodeBuild)
├── SonarQube Static Code Analysis
├── OWASP Dependency Check
├── Trivy File System Scan
├── Docker Image Build
├── Trivy Image Scan
└── Push Image to Docker Hub
DEPLOY STAGE (AWS CodeDeploy - Blue/Green)
├── Deploy new version to GREEN environment
├── ALB health checks on GREEN
├── Traffic switch BLUE → GREEN
└── Terminate original task set


---

## 🛠️ Tech Stack

| Layer | Service | Purpose |
|-------|---------|---------|
| **Source** | GitHub | Source code repository |
| **CI/CD** | AWS CodePipeline | Pipeline orchestration |
| **Build** | AWS CodeBuild | Docker build + security scans |
| **Code Quality** | SonarQube | Static code analysis |
| **Security** | Trivy | File + image vulnerability scan |
| **Security** | OWASP | Dependency vulnerability check |
| **Registry** | Docker Hub | Docker image storage |
| **Deploy** | AWS CodeDeploy | Blue-Green deployment |
| **Compute** | AWS ECS (EC2) | Container orchestration |
| **Load Balancer** | AWS ALB | Traffic routing Blue↔Green |
| **Secrets** | AWS SSM Parameter Store | Encrypted credentials |

---

## 📁 Project Structure
DevSecOps-Blue-Green-ECS-Swiggy-Clone/
│
├── Swiggy_clone/                # React application source
│   ├── public/                  # Static assets
│   │   ├── Photos/              # App images
│   │   └── index.html           # HTML entry point
│   ├── src/                     # React components
│   │   ├── Components/          # UI components
│   │   ├── App.js               # Root component
│   │   └── index.js             # App entry point
│   ├── buildspec.yaml           # CodeBuild build spec
│   ├── Dockerfile               # Container definition
│   ├── appspec.yaml             # CodeDeploy spec
│   └── package.json             # Node dependencies
│
└── README.md

---

## 🚀 Step-by-Step Setup Guide

### Step 1 — Create SonarQube Server

Create EC2 instance (t2.medium, Ubuntu) and install Docker:

```bash
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
sudo chmod 777 /var/run/docker.sock
```

Run SonarQube as Docker container:
```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

![](https://miro.medium.com/v2/resize:fit:802/1*0wK1e8bOHEJlFiwztNGciA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*RETZJ22lP-5L26Un9MxysA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*mzG2HspPsn-sPLpFcwi52A.png)
![](https://miro.medium.com/v2/resize:fit:802/1*gptHLF_z4uBT6NEu26-Jtw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*tciW44uP3T7YW9dFVBQj-w.png)

Open port **9000** in security group and access SonarQube at `<public_ip>:9000`
Username & Password: **admin**

![](https://miro.medium.com/v2/resize:fit:802/1*aQkJ_5xLF8fs_x9yQJAusw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*ntn6wzIBgIXWGCP6tPaOFw.png)

---

### Step 2 — SonarQube Setup

Create a project, generate token and store in AWS SSM Parameter Store:

![](https://miro.medium.com/v2/resize:fit:802/1*LzS_Zd-OCV9MoisUPPu6og.png)
![](https://miro.medium.com/v2/resize:fit:802/1*FpW07qDsNtUYmuhosE_XSg.png)
![](https://miro.medium.com/v2/resize:fit:802/1*p5VtA-6SOBWCufqhC87d4Q.png)
![](https://miro.medium.com/v2/resize:fit:802/1*NgaUO2Uz7BR0W5acEayeQQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*92AU-98SH-A1JkIAF1d8yg.png)
![](https://miro.medium.com/v2/resize:fit:802/1*S_dAKo2-ufnAqjsbQ7M47A.png)
![](https://miro.medium.com/v2/resize:fit:802/1*tlLnmKodyEccaJJQ_ccinw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*EAiBMC6UuzD5eMJs5X4pRg.png)

Create SSM Parameters:
```bash
Parameter name: /cicd/sonar/sonar-token             Value: <sonar_token>
Parameter name: /cicd/docker-credentials/username   Value: <docker_username>
Parameter name: /cicd/docker-credentials/password   Value: <docker_password>
Parameter name: /cicd/docker-registry/url           Value: docker.io
```

---

### Step 3 — Create AWS CodeBuild Project

![](https://miro.medium.com/v2/resize:fit:802/1*TCikGTyQvlmoVRcQx3bRVA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*BnoI64cb7mq2r6tkz8DflQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*hNYAdmXEyoKRP4mNc4rdxA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*7I9PqgAKn8f9QaMAX_nWPg.png)
![](https://miro.medium.com/v2/resize:fit:802/1*g3H9AXupRVThK1trc1PXNw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*HZyWuCgqQhUxmSUK07Vqig.png)

Add IAM permissions to CodeBuild role:
- `AmazonSSMFullAccess`
- `AWSS3FullAccess`

![](https://miro.medium.com/v2/resize:fit:802/1*3iQjsyTslPp2gpnwuNqRSQ.png)

Successful build output:

![](https://miro.medium.com/v2/resize:fit:802/1*L5utCQecsFVB-4dJWH84eg.png)

**SonarQube Analysis:**
![](https://miro.medium.com/v2/resize:fit:802/1*7y-9yqT43IPgFaiZ0kXT9Q.png)

**Dependency-Check Reports:**
![](https://miro.medium.com/v2/resize:fit:802/1*ObhhizJMM-i9jBpFR1zUHA.png)

**Trivy File Scan:**
![](https://miro.medium.com/v2/resize:fit:802/1*IHT0tPfD1xfr940DrvrqxQ.png)

**Trivy Image Scan:**
![](https://miro.medium.com/v2/resize:fit:802/1*ARlLeQeWccslE0I_DRn_Lg.png)

---

### Step 4A — ECS Cluster Creation

![](https://miro.medium.com/v2/resize:fit:802/1*I3Jzh2DQpK2uHOXEh2JyBw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*1PTm6bPb1ZdptpuGzM06jg.png)
![](https://miro.medium.com/v2/resize:fit:802/1*kt_Ll2_mkGhbfX2NMNrQQQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*tMm3ad3-V0_l1GL_14jodA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*n2T-5UHOwF7bz8ikxNi2vw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*tGCgFpxO4dHHOjfhK_InWg.png)
![](https://miro.medium.com/v2/resize:fit:802/1*MNXhf0FuUWmOcxV8ya-KgQ.png)

---

### Step 4B — ECS Task Definition Creation

![](https://miro.medium.com/v2/resize:fit:802/1*_DAwaM6MGSw8nI4lWC6Mhw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*MYA3mtG46sUX5zqeBxZYlA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*BnsPIa1jtFShl2sjSLJo7Q.png)
![](https://miro.medium.com/v2/resize:fit:802/1*H8ncw2-QEIZoWclawPhqog.png)
![](https://miro.medium.com/v2/resize:fit:802/1*_keGf2xjEHFAKjT15Rxfyw.png)

---

### Step 4C — Load Balancer Creation

![](https://miro.medium.com/v2/resize:fit:802/1*jtrI2SFTInCkh_CGOj11sA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*P8TpRwloMZQWSA7SG9JAdA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*GgUYQKRRELtKn4atPL0ayw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*Q4tFWuEUAyvXWXyPhz8eYA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*gw-uf287FPgOoie18EXdfw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*Rlsws6HQ6atKoYilhziZ2A.png)
![](https://miro.medium.com/v2/resize:fit:802/1*WsPf5LVhKXQRfpg1M9LZYA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*-rreQ1ojPKb1LyGmyoR3YQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*TlMAxACt9kSvA393qbp9OQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*Bv4eAfRugCxiLcxiWfkNfQ.png)

Create CodeDeploy-ECS IAM Role:

![](https://miro.medium.com/v2/resize:fit:802/1*jReVRPYrZJMvqbT9R8wgTg.png)
![](https://miro.medium.com/v2/resize:fit:802/1*6IRX3oK8fmh3t4QTjnlqqw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*pCYLNvVgwKodaVuYM2c2hg.png)
![](https://miro.medium.com/v2/resize:fit:802/1*DIW5iJNEqCMmzA1qu7w_qA.png)

---

### Step 4D — ECS Service Creation

![](https://miro.medium.com/v2/resize:fit:802/1*McOKRq-zdebBRpSqGUciRQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*QQJ3ZFVr931988yro3SJLg.png)
![](https://miro.medium.com/v2/resize:fit:802/1*NJWlT_Lb_7r41GTDfoG93Q.png)
![](https://miro.medium.com/v2/resize:fit:802/1*5AganRfr1kiTNRwt60aAyg.png)
![](https://miro.medium.com/v2/resize:fit:802/1*8DGTlfVm2ZfUV8RyqLJ71A.png)
![](https://miro.medium.com/v2/resize:fit:802/1*KRqhpTMvmwtJU6ev3Rhp1A.png)
![](https://miro.medium.com/v2/resize:fit:802/1*_cAYotXrv29JDN_yohJ3qQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*QS6Dt0SI5ZFq_MGqk-Z-rA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*Wd8tFahwwYSITu0J0MuJ4A.png)
![](https://miro.medium.com/v2/resize:fit:802/1*MBEJkKJ8o_oIjy5qfF0fXw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*l8hui1XLdhEvkukOQVsavQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*9ddlozxr_dVezz6mlj3yHw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*sfMAF4hPa4ANcKTRXy7kIg.png)

Create `appspec.yaml`:
```yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "arn:aws:ecs:ap-south-1:<account_id>:task-definition/swiggy:1"
        LoadBalancerInfo:
          ContainerName: "swiggy"
          ContainerPort: 3000
```

![](https://miro.medium.com/v2/resize:fit:802/1*fAAmD928t3-AJXFjDrADgg.png)

---

### Step 5 — AWS CodePipeline Creation

![](https://miro.medium.com/v2/resize:fit:802/1*7tfko_PCqTiZodkViv082Q.png)
![](https://miro.medium.com/v2/resize:fit:802/1*-J9_-86kJ_GSjh2wW4G6HQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*LDbYe5qyC0_FVeRnfhKK9w.png)
![](https://miro.medium.com/v2/resize:fit:802/1*ggEZsskoeNNXe9aCVOMiKw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*oNsb4LrBhbHNB8rJ49Sn1A.png)
![](https://miro.medium.com/v2/resize:fit:802/1*4RWlRMDUAOzE33d38IMf4g.png)
![](https://miro.medium.com/v2/resize:fit:802/1*BnDgQI-FdOqfCHXvAcd48g.png)
![](https://miro.medium.com/v2/resize:fit:802/1*DTaV7VnsXlz8Y1gdzd8D6A.png)
![](https://miro.medium.com/v2/resize:fit:802/1*RNiF_0OhhQOkzOzkybx92g.png)
![](https://miro.medium.com/v2/resize:fit:802/1*G_dJd0zpjkf22Ja-x9TFfw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*Ex0hM9BMe-cDBvERFT7GNQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*OxYrxOIqSmq4vuabNObhlw.png)
![](https://miro.medium.com/v2/resize:fit:802/1*FQ0SFtJ_qWFyugtFs0yzoA.png)
![](https://miro.medium.com/v2/resize:fit:802/1*ThzVRYHjbRLTMCl7IAD8SQ.png)

---

### Step 6 — ECS Deployment

Make a code change and push to GitHub — pipeline triggers automatically:

![](https://miro.medium.com/v2/resize:fit:802/1*wsPehtBu4QT4EqnacQSABQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*ngpkjHB_CMx-MlodMU0GHg.png)
![](https://miro.medium.com/v2/resize:fit:802/1*UsOW6tdguiPF_IPmkVeZ2w.png)
![](https://miro.medium.com/v2/resize:fit:802/1*_2viBEoGEoyQ8LhrQSFoVQ.png)

Successful pipeline run:

![](https://miro.medium.com/v2/resize:fit:802/1*Iqj1Y6lE4h-vC6wEstrDIQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*M-suXmKdp1LoOpk2OeDVGA.png)

Traffic switched to GREEN (TG-2):

![](https://miro.medium.com/v2/resize:fit:802/1*9jwecsZr2gt_MkBVkuuE0w.png)
![](https://miro.medium.com/v2/resize:fit:802/1*ZBbtIUgMZ-wJVwtl4Ky4cQ.png)
![](https://miro.medium.com/v2/resize:fit:802/1*p6cH8ib_xAh5yX178-PWvg.png)

---

### Step 7 — Clean Up

```bash
# Delete in this order:
1. Delete CodePipeline
2. Delete ECS Cluster
3. Delete CodeBuild Project
4. Delete SonarQube EC2 Instance
5. Delete ALB and Target Groups
6. Delete SSM Parameters
```

---

## 🗺️ Future Roadmap

- [ ] Add Terraform IaC for full infrastructure provisioning
- [ ] Add automated rollback on health check failure
- [ ] Implement Canary deployment (10% → 50% → 100%)
- [ ] Add AWS WAF for application protection
- [ ] Multi-region active-active deployment
- [ ] Add Slack/email notifications on deployment

---

## 📜 License

MIT License — feel free to use, modify, and distribute.

---

## 🙌 Contact

**Shubham Haranale**

📩 [LinkedIn](https://www.linkedin.com/in/shubhamharanale7)
📧 shubhaminfosoft7@gmail.com
🐙 [GitHub](https://github.com/Shubhamharanale7)

> If you found this helpful, consider ⭐ starring the repo!
