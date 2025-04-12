##### **EasyShop 🛍️** is a modern, full-stack e-commerce platform built with Next.js 14, TypeScript, and MongoDB. It features a beautiful UI with Tailwind CSS, secure authentication, real-time cart updates, and a seamless shopping experience.

## PreRequisites

> [!IMPORTANT]  
> Before you begin setting up this project, make sure the following tools are installed and configured properly on your system:

## Setup & Initialization <br/>

### 1. Install Terraform
* Install Terraform<br/>
#### Linux & macOS
```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform
```
### Verify Installation
```bash
terraform -v
```
### Initialize Terraform
```bash
terraform init
```
### 2. Install AWS CLI
AWS CLI (Command Line Interface) allows you to interact with AWS services directly from the command line.

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
```

 ```aws configure```

> #### This will prompt you to enter:<br/>
- **AWS Access Key ID:**<br/>
- **AWS Secret Access Key:**<br/>
- **Default region name:**<br/>
- **Default output format:**<br/>

> [!NOTE] 
> Make sure the IAM user you're using has the necessary permissions. You’ll need an AWS IAM Role with programmatic access enabled, along with the Access Key and Secret Key.

## Getting Started

> Follow the steps below to get your infrastructure up and running using Terraform:<br/>

1. **Clone the Repository:**
First, clone this repo to your local machine:<br/>
```bash
git clone https://github.com/LondheShubham153/tws-e-commerce-app.git
cd terraform
```
2. **Generate SSH Key Pair:**
Create a new SSH key to access your EC2 instance:
```bash
ssh-keygen -f terra-key
```
This will prompt you to create a new key file named terra-key.

3. **Private key permission:** Change your private key permission:
```bash
chmod 400 terra-key
```

4. **Initialize Terraform:**
Initialize the Terraform working directory to download required providers:
```bash
terraform init
```
5. **Review the Execution Plan:**
Before applying changes, always check the execution plan:
```bash
terraform plan
```
6. **Apply the Configuration:**
Now, apply the changes and create the infrastructure:
```bash
terraform apply
```
> Confirm with `yes` when prompted.

7. **Access Your EC2 Instance;** <br/>
After deployment, grab the public IP of your EC2 instance from the output or AWS Console, then connect using SSH:
```bash
ssh -i terra-key ubuntu@<public-ip>
```
8. **Update your kubeconfig:**
wherever you want to access your eks wheather it is yur local machine or bastion server this command will help you to interact with your eks.
> [!CAUTION]
> you need to configure aws cli first to execute this command:

```bash
aws configure
```

```bash
aws eks --region eu-west-1 update-kubeconfig --name tws-eks-cluster
```
9. **Check your cluster:**
```bash
kubectl get nodes
```

## Jenkins Setup Steps
> [!TIP]
> Check if jenkins service is running:

```bash
sudo systemctl status jenkins
```
## Steps to Access Jenkins & Install Plugins

1. **Open Jenkins in Browser:**
Use your public IP with port 8080:

**http://<public_IP>:8080**

2. **Initial Admin password:**
Start the service and get the Jenkins initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

3. **Start Jenkins (*If Not Running*):**
Get the Jenkins initial admin password:
```bash
sudo systemctl enable jenkins
sudo systemctl restart jenkins
```
4. **Install Essential Plugins:**
Navigate to:<br/>
**Manage Jenkins → Plugins → Available Plugins**<br/>
Search and install the following:<br/>
**Docker Pipeline**<br/>
**Pipeline View**

5. Set Up Docker & GitHub Credentials in Jenkins (Global Credentials)<br/>
<br/>
- GitHub Credentials:<br/>
Go to:<br/>
**Jenkins → Manage Jenkins → Credentials → (Global) → Add Credentials**
Use:<br/>
Kind: **Username with password or Personal Access Token**<br/>
ID: **github-credentials**<br/>

- DockerHub Credentials:<br/>
Go to the same Global Credentials section<br/>
Use:<br/>
Kind: **Username with password**<br/>
ID: **docker-hub-credentials**<br/>
Use these IDs in your Jenkins pipeline for secure access to GitHub and DockerHub.<br/>

6. Jenkins Shared Library Setup<br/>
Configure Trusted Pipeline Library:<br/>
Go to:<br/>
**Jenkins → Manage Jenkins → Configure System**<br/>

Scroll to Global Pipeline Libraries section<br/>

**Add a New Shared Library:** <br/>
**Name:** shared<br/>
**Default Version:** main<br/>
**Project Repository URL:** `https://github.com/<your user-name/jenkins-shared-libraries`.<br/>
**Note:** Make sure the repo contains a proper directory structure eq: vars/<br/>
	
7. Setup Pipeline<br/>
- Create New Pipeline Job<br/>
  - **Name:** EasyShop<br/>
  - **Type:** Pipeline<br/>
  > Press `Okey`<br/>

  > In **General**<br/>
  - **Description:** EasyShop<br/>
  - **Check the box:** `GitHub project`<br/>
  - **GitHub Repo URL:** `https://github.com/<your user-name/tws-e-commerce-app`<br/>
  > In **Trigger**<br/>
  - **Check the box:**`GitHub hook trigger for GITScm polling`<br/>
  > In **Pipeline**<br/>
  - **Definition:** `Pipeline script from SCM`<br/>
  - **SCM:** `Git`<br/>
  - **Repository URL:** `https://github.com/<your user-name/tws-e-commerce-app`<br/>
  - **Credentials:** `github-credentials`<br/>
  - **Branch:** master<br/>
  - **Script Path:** `Jenkinsfile`<br/>

**Fork Required Repos**<br/>
Fork App Repo:<br/>
* Open the Jenkinsfile<br/>
* Change the DockerHub username to yours<br/>

**Fork Shared Library Repo:**<br/>
* Edit vars/update_k8s_manifest.groovy<br/>
* Update with your DockerHub username<br/>

**Setup Webhook**<br/>
In GitHub:<br/>
* Go to **Settings → Webhooks**<br/>
* Add a new webhook pointing to your Jenkins URL<br/>
* Select: **"GitHub hook trigger for GITScm polling"** in Jenkins job<br/>

**Trigger the Pipeline**<br/>
Click **'Build Now'** in Jenkins

**7. CD – Continuous Deployment Setup**<br/>
**Prerequisites:**<br/>
Before configuring CD, make sure the following tools are installed:<br/>
* Installations Required:<br/>
**kubectl**<br/>
**AWS CLI**

**SSH into Bastion Server**<br/>
* Connect to your Bastion EC2 instance via SSH.

**Note:**<br/>
This is not the node where Jenkins is running. This is the intermediate EC2 (Bastion Host) used for accessing private resources like your EKS cluster.

**8. Configure AWS CLI on Bastion Server**
Run the AWS configure command:<br/>
```bash
aws configure
```
Add your Access Key and Secret Key when prompted.

**9. Update Kubeconfig for EKS**<br/>
Run the following important command:
```bash
aws eks update-kubeconfig --region eu-west-1 --name tws-eks-cluster
```
* This command maps your EKS cluster with your Bastion server.
* It helps to communicate with EKS components.

**10. Argo CD Setup**<br/>
Create a Namespace for Argo CD<br/>
```bash
kubectl create namespace argocd
```
1. Install Argo CD using Manifest
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
2. Watch Pod Creation
```bash
watch kubectl get pods -n argocd
```
3. This helps monitor when all Argo CD pods are up and running.<br/>

4. Check Argo CD Services
```bash
kubectl get svc -n argocd
```

5. Change Argo CD Server Service to NodePort
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

11. Access Argo CD GUI<br/>
Check Argo CD Server Port (again, post NodePort change)<br/>
```bash
kubectl get svc -n argocd
```
1. Port Forward to Access Argo CD in Browser<br/>
 Forward Argo CD service to access the GUI:
```bash
kubectl port-forward svc/argocd-server -n argocd <your-port>:443 --address=0.0.0.0 &
```
2. Replace <your-port> with a local port of your choice (e.g., 8080).<br/>
 Now, open https://<bastion-ip>:<your-port> in your browser.


Get the Argo CD Admin Password<br/>
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
1. Log in to the Argo CD GUI
* Username: admin
* Password: (Use the decoded password from the previous command)

2. Update Your Password
* On the left panel of Argo CD GUI, click on "User Info"
* Select Update Password and change it.

**Deploy Your Application in Argo CD GUI**<br/>
1. On the Argo CD homepage, click on the “New App” button.<br/>
2. Fill in the following details:<br/>
* Application Name:<br/>
 [Enter your desired app name]<br/>

* **Project Name:** <br/>
 Select 'default' from the dropdown.<br/>
* **Sync Policy:** <br/>
 Choose 'Automatic'.<br/>

3. In the “Source” section:<br/>
***Repo URL:** <br/>
 Add the Git repository URL that contains your Kubernetes manifests.<br/>
***Path:** <br/>
 Kubernetes (or the actual path inside the repo where your manifests reside)<br/>

4. In the “Destination” section:
* **Cluster URL:** <br/>
 https://kubernetes.default.svc (usually shown as "default")
* **Namespace:** <br/>
 tws-e-commerce-app (or your desired namespace)<br/>

5. Click on “Create”.

## **Congratulations!** <br/>
### Your project is now deployed via Argo CD 