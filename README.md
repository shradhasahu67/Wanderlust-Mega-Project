# Wanderlust Mega Project End to End Implementation

### In this demo, we will see how to deploy an end to end three tier MERN stack application on EKS cluster.
#
### <mark>Project Deployment Flow:</mark>
<img src="https://github.com/DevMadhup/Wanderlust-Mega-Project/blob/main/Assets/DevSecOps%2BGitOps.gif" />

#

## Tech stack used in this project:
- GitHub (Code)
- Docker (Containerization)
- Jenkins (CI)
- OWASP (Dependency check)
- SonarQube (Quality)
- Trivy (Filesystem Scan)
- ArgoCD (CD)
- Redis (Caching)
- AWS EKS (Kubernetes)
- Helm (Monitoring using grafana and prometheus)

### How pipeline will look after deployment:
- <b>CI pipeline to build and push</b>
<img width="1902" height="967" alt="image" src="https://github.com/user-attachments/assets/a70d9253-726c-4ff9-a7bd-020d6a61565b" />

- <b>CD pipeline to update application version</b>
<img width="1918" height="933" alt="image" src="https://github.com/user-attachments/assets/39ffc070-2269-4542-9d94-8cabdef95802" />

- <b>ArgoCD application for deployment on EKS</b>
<img width="1918" height="841" alt="image" src="https://github.com/user-attachments/assets/ed490098-3d4f-4c18-9293-93dfc38d8e8a" />
<img width="834" height="543" alt="image" src="https://github.com/user-attachments/assets/177449a7-16a7-4f82-a88e-d4aa6e51790e" />


#
> [!Important]
> Below table helps you to navigate to the particular tool installation section fast.

| Tech stack    | Installation |
| -------- | ------- |
| Jenkins Master | <a href="#Jenkins">Install and configure Jenkins</a>     |
| eksctl | <a href="#EKS">Install eksctl</a>     |
| Argocd | <a href="#Argo">Install and configure ArgoCD</a>     |
| Jenkins-Worker Setup | <a href="#Jenkins-worker">Install and configure Jenkins Worker Node</a>     |
| OWASP setup | <a href="#Owasp">Install and configure OWASP</a>     |
| SonarQube | <a href="#Sonar">Install and configure SonarQube</a>     |
| Email Notification Setup | <a href="#Mail">Email notification setup</a>     |
| Monitoring | <a href="#Monitor">Prometheus and grafana setup using helm charts</a>
| Clean Up | <a href="#Clean">Clean up</a>     |
#

### Pre-requisites to implement this project:
#


- <b>Create 1 Master machine on AWS using terraform with 2CPU, 8GB of RAM (m7i-flex.large) and 30 GB of storage and install Docker on it.</b>
#
-create <mark>IAM user--> terraform-admin(add permissions, accesskey retrieved) </mark>
<img width="1918" height="334" alt="image" src="https://github.com/user-attachments/assets/559a067c-9e49-4a7d-ad51-d478bbbb3c99" />
<img width="1851" height="705" alt="image" src="https://github.com/user-attachments/assets/2cd3c397-0a2e-41d6-9eca-9069c86a4944" />

-Login as terraform-admin and generate key(terra-key)
<img width="1149" height="153" alt="image" src="https://github.com/user-attachments/assets/0944f17e-d6d0-4c1f-9ed6-0cc23a6931fb" />
<img width="1011" height="523" alt="image" src="https://github.com/user-attachments/assets/26c3cf49-6d78-40ac-8bf1-e61cb78cc96e" />
<img width="960" height="546" alt="image" src="https://github.com/user-attachments/assets/a7adb8b8-7201-477d-ac33-be9709c4b9de" />
<img width="997" height="826" alt="image" src="https://github.com/user-attachments/assets/12c2a10a-2315-4279-a81b-f5688dafa502" />


-terraform <mark>init->plan->apply </mark>
<img width="1224" height="487" alt="image" src="https://github.com/user-attachments/assets/dc09625f-0608-457e-8da6-cc51d3faf65e" />
<img width="1413" height="813" alt="image" src="https://github.com/user-attachments/assets/56f62bfa-9213-470a-a4d6-efc9f7088663" />
<img width="1423" height="793" alt="image" src="https://github.com/user-attachments/assets/63ef5ee0-f5cd-489b-8273-f3db1c207397" />
<img width="1429" height="1279" alt="image" src="https://github.com/user-attachments/assets/000fba73-3fcc-49a3-b436-cf24c7f07996" />
<img width="1429" height="1958" alt="image" src="https://github.com/user-attachments/assets/daab999b-8e10-4e2c-95f4-7189adf4e762" />


-ec2 instance is ready
<img width="1917" height="346" alt="image" src="https://github.com/user-attachments/assets/780d48b2-cd76-4ef0-88dc-8b5abde2b580" />


-Ssh into master machine
<img width="1291" height="740" alt="image" src="https://github.com/user-attachments/assets/ea165218-09db-4927-8586-6cb51fbf4f18" />
<img width="1425" height="1474" alt="image" src="https://github.com/user-attachments/assets/22903d30-c196-489a-8b87-de4777605eeb" />



- <b>Open the below ports in security group of master machine and also attach same security group to Jenkins worker node (We will create worker node shortly)</b>
![image](https://github.com/user-attachments/assets/4e5ecd37-fe2e-4e4b-a6ba-14c7b62715a3)
<img width="1892" height="742" alt="image" src="https://github.com/user-attachments/assets/bf110ec4-837d-4672-9c7c-2f3d0ba70de7" />


> [!Note]
> We are creating this master machine because we will configure Jenkins master, eksctl, EKS cluster creation from here.

update the master machine
<img width="1382" height="739" alt="image" src="https://github.com/user-attachments/assets/5f050f56-de87-4db3-a078-97d48a279470" />

Install & Configure Docker by using below command, "NewGrp docker" will refresh the group config hence no need to restart the EC2 machine.

```bash
sudo apt-get update
```
```bash
sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu && newgrp docker
```
<img width="1225" height="737" alt="image" src="https://github.com/user-attachments/assets/29ac20a8-2fba-44ff-b932-ba3ed129fc48" />
-To resolve the issue we get after running docker-ps---> run the ///var/run/docker.sock
To resolve the issue we get after running docker-ps---> run the ///var/run/docker.sock
<img width="955" height="128" alt="image" src="https://github.com/user-attachments/assets/141151d2-c680-4b67-bb73-ba983635c648" />
-alernative
<img width="1056" height="81" alt="image" src="https://github.com/user-attachments/assets/fc2a0dc0-3344-4b7e-9fbb-aca4bba8dce5" />



#
- <b id="Jenkins">Install and configure Jenkins (Master machine)</b>
```bash
sudo apt update -y
sudo apt install fontconfig openjdk-17-jre -y

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
  
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
  
sudo apt-get update -y
sudo apt-get install jenkins -y
```
- <b>Now, access Jenkins Master on the browser on port 8080 and configure it</b>.

<img width="1919" height="938" alt="image" src="https://github.com/user-attachments/assets/876fb482-45e7-447b-9a40-e8d4966c2900" />

-get password for initial login
<img width="974" height="95" alt="image" src="https://github.com/user-attachments/assets/0755b0cf-1241-4baa-9379-79aff8dbda1a" />
<img width="1545" height="789" alt="image" src="https://github.com/user-attachments/assets/136d82e9-0d19-4051-aaa0-1d535d8d1655" />
<img width="1673" height="785" alt="image" src="https://github.com/user-attachments/assets/c0bfaf29-2d4e-4e6b-bde1-9fa5890cb65e" />
<img width="1679" height="1563" alt="image" src="https://github.com/user-attachments/assets/e2a36ecc-3991-4c65-b6a5-84e4d9ba7222" />


#
- <b id="EKS">Create EKS Cluster on AWS (Master machine)</b>
  - Install **kubectl** (Master machine)(<a href="https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/Kubectl/Kubectl.sh">Setup kubectl </a>)
  ```bash
  curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
  chmod +x ./kubectl
  sudo mv ./kubectl /usr/local/bin
  kubectl version --short --client
  ```
  <img width="1562" height="221" alt="image" src="https://github.com/user-attachments/assets/f6857c51-bc9a-428b-b218-1df8d0b4cd22" />


  - Install **eksctl** (Master machine) (<a href="https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/eksctl%20/eksctl.sh">Setup eksctl</a>)
  ```bash
  curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
  sudo mv /tmp/eksctl /usr/local/bin
  eksctl version
  ```
  <img width="1584" height="350" alt="image" src="https://github.com/user-attachments/assets/13bea5ea-35f7-4542-86e7-b39128814f48" />

  - <b>Create EKS Cluster (Master machine)</b>
  ```bash
  eksctl create cluster --name=wanderlust \
                      --region=us-east-2 \
                      --version=1.30 \
                      --without-nodegroup
  ```
  - <b>Associate IAM OIDC Provider (Master machine)</b>
  ```bash
  eksctl utils associate-iam-oidc-provider \
    --region us-east-2 \
    --cluster wanderlust \
    --approve
  ```
  <img width="1584" height="970" alt="image" src="https://github.com/user-attachments/assets/88a358eb-4f60-4b2b-ad84-361785cf8941" />
<img width="1919" height="399" alt="image" src="https://github.com/user-attachments/assets/22aaf395-3777-46f6-bd2f-0333106cf170" />
<img width="1245" height="222" alt="image" src="https://github.com/user-attachments/assets/2d9f425e-8a7f-4d72-978b-c0d2e3d0afe1" />


  - <b>Create Nodegroup (Master machine)</b>
  <img width="1405" height="418" alt="image" src="https://github.com/user-attachments/assets/af792147-2e20-4894-ad0c-b4183e2bd45c" />
  <img width="1411" height="586" alt="image" src="https://github.com/user-attachments/assets/be0cc15d-1b2a-4d67-b0ef-6d69aca84149" />


> [!Note]
>  Make sure the ssh-public-key "eks-nodegroup-key is available in your aws account"
> <img width="1649" height="362" alt="image" src="https://github.com/user-attachments/assets/0a6da9ad-7c93-497e-86b3-6b98419b5948" />

#

#
- <b id="Sonar">Install and configure SonarQube (Master machine)</b>
```bash
docker run -itd --name SonarQube-Server -p 9000:9000 sonarqube:lts-community
```
<img width="1433" height="427" alt="image" src="https://github.com/user-attachments/assets/ab7b9454-9636-4404-bcb2-9e57c1fef58e" />

<img width="1918" height="771" alt="image" src="https://github.com/user-attachments/assets/a7c4f8e2-f738-4e34-94ba-ee232e5808a9" />


#
- <b id="Trivy">Install Trivy (Jenkins Worker)</b>
```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update -y
sudo apt-get install trivy -y
```
<img width="1556" height="613" alt="image" src="https://github.com/user-attachments/assets/827b945f-c95c-4c18-bee8-35188974072d" />



## Steps to add email notification
- <b id="Mail">Go to your Jenkins Master EC2 instance and allow 465 port number for SMTPS</b>
#
- <b>Now, we need to generate an application password from our gmail account to authenticate with jenkins</b>
  - <b>Open gmail and go to <mark>Manage your Google Account --> Security</mark></b>
> [!Important]
> Make sure 2 step verification must be on

<img width="1073" height="16101" alt="image" src="https://github.com/user-attachments/assets/dcc6b49a-d155-4ad5-a8b8-e4a449b8e9e7" />
<img width="1073" height="16101" alt="image" src="https://github.com/user-attachments/assets/fc3b8f29-d11b-489d-bb7c-f426436836a2" />
<img width="1073" height="16101" alt="image" src="https://github.com/user-attachments/assets/6e4ff6fe-95af-465c-b202-01895184299e" />

  - <b>Search for <mark>App password</mark> and create a app password for jenkins</b>
<img width="1073" height="16101" alt="image" src="https://github.com/user-attachments/assets/51eaa1ab-a152-47cd-a31a-0a67e01e4fb8" />

  
#
- <b> Once, app password is create and go back to jenkins <mark>Manage Jenkins --> Credentials</mark> to add username and password for email notification</b>
<img width="1073" height="16101" alt="image" src="https://github.com/user-attachments/assets/6573dcc6-c7b9-4c2c-b6e1-3b9ea2a8edbd" />
<img width="1073" height="16101" alt="image" src="https://github.com/user-attachments/assets/6c8081dd-9e30-4431-80d2-29d8a8a79f90" />

# 
- <b> Go back to <mark>Manage Jenkins --> System</mark> and search for <mark>Extended E-mail Notification</mark></b>
<img width="1079" height="16011" alt="image" src="https://github.com/user-attachments/assets/a9c681a5-5c69-4509-9446-5ac62f2481ea" />
#
- <b>Scroll down and search for <mark>E-mail Notification</mark> and setup email notification</b>
> [!Important]
> Enter your gmail password which we copied recently in password field <mark>E-mail Notification --> Advance</mark>



#
- <b id="Argo">Install and Configure ArgoCD (Master Machine)</b>
  - <b>Create argocd namespace</b>
  ```bash
  kubectl create namespace argocd
  ```
  <img width="1526" height="235" alt="image" src="https://github.com/user-attachments/assets/32757190-c99e-45bc-9dc4-e829b30adcd8" />

  - <b>Apply argocd manifest</b>
  ```bash
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```
  - <b>Make sure all pods are running in argocd namespace</b>
  ```bash
  watch kubectl get pods -n argocd
  ```
  - <b>Install argocd CLI</b>
  ```bash
  sudo curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64
  ```
  <img width="1441" height="347" alt="image" src="https://github.com/user-attachments/assets/c97c8c58-3b25-4933-98d7-4f8121cdce69" />

  - <b>Provide executable permission</b>
  ```bash
  sudo chmod +x /usr/local/bin/argocd
  ```
  - <b>Check argocd services</b>
  ```bash
  kubectl get svc -n argocd
  ```
  - <b>Change argocd server's service from ClusterIP to NodePort</b>
  ```bash
  kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
  ```
  <img width="1439" height="318" alt="image" src="https://github.com/user-attachments/assets/55c3006d-c19d-496a-9dd9-3c52aa00e814" />

  - <b>Confirm service is patched or not</b>
  ```bash
  kubectl get svc -n argocd
  ```
  <img width="1433" height="261" alt="image" src="https://github.com/user-attachments/assets/dbbc8350-065a-499e-a830-af56cbf4f431" />
<img width="1387" height="724" alt="image" src="https://github.com/user-attachments/assets/99a3f451-18ce-4e3f-b789-919a0f57717d" />

  
  - <b> Check the port where ArgoCD server is running and expose it on security groups of master node</b>
  -Use that port 31439 ( nodeport ) and add to edit inbound rules and to open argo cd.
<img width="1918" height="676" alt="image" src="https://github.com/user-attachments/assets/36380737-f9c5-46fd-8982-e4f77cbff3a9" />
<img width="1672" height="706" alt="image" src="https://github.com/user-attachments/assets/01190f9b-1cd1-440f-9fe1-ac4d1cca49a6" />

  - <b>Access it on browser, click on advance and proceed with</b>
  ```bash
  <public-ip-worker>:<port>
  ```
<img width="1917" height="919" alt="image" src="https://github.com/user-attachments/assets/2e9fe9de-5f21-405b-a519-56ccf40dc46b" />

  - <b>Fetch the initial password of argocd server</b>
  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
  ```
<img width="1923" height="986" alt="image" src="https://github.com/user-attachments/assets/0b5b2e8c-f5bc-4b3f-9ec3-0ba7908f8de6" />

  - <b>Username: admin</b>
  - <b> Now, go to <mark>User Info</mark> and update your argocd password
  - <img width="1099" height="694" alt="image" src="https://github.com/user-attachments/assets/d189a00f-b4db-49ac-96d0-3b9ca0e98a16" />

#
- 	Atttach github repo to argoCD
<img width="1925" height="481" alt="image" src="https://github.com/user-attachments/assets/9b0c950d-f9f1-4f7f-a106-6c59e42562f2" />
<img width="1407" height="807" alt="image" src="https://github.com/user-attachments/assets/883a37b2-cd59-4565-8e69-d00be98b984d" />
<img width="1598" height="357" alt="image" src="https://github.com/user-attachments/assets/abe26c1b-807c-4ba1-b40b-1a53935edfe3" />


#
## Steps to implement the project:
- <b>Go to Jenkins Master and click on <mark> Manage Jenkins --> Plugins --> Available plugins</mark> install the below plugins:</b>
  - OWASP
  - SonarQube Scanner
  - Docker
  - Pipeline: Stage View
    <img width="1912" height="773" alt="image" src="https://github.com/user-attachments/assets/cd32b9c6-e2e5-49ab-ae1a-a1f78a866d9a" />

#
- <b id="Owasp">Configure OWASP, move to <mark>Manage Jenkins --> Plugins --> Available plugins</mark> (Jenkins Worker)</b>
- <b id="Sonar">After OWASP plugin is installed, Now move to <mark>Manage jenkins --> Tools</mark> (Jenkins Worker)</b>
<img width="1918" height="823" alt="image" src="https://github.com/user-attachments/assets/37edd2f7-d8e5-4ac2-9a3b-357f93923834" />
#
- <b>Login to SonarQube server and create the credentials for jenkins to integrate with SonarQube</b>
  - Navigate to <mark>Administration --> Security --> Users --> Token</mark>
<img width="1917" height="784" alt="image" src="https://github.com/user-attachments/assets/e17fb018-4e51-41e0-8773-01fe564fc6ed" />
<img width="1923" height="1235" alt="image" src="https://github.com/user-attachments/assets/98953e6e-9d80-4d48-b155-05a945dfe57e" />

#
- <b>Now, go to <mark> Manage Jenkins --> credentials</mark> and add Sonarqube credentials:</b>
<img width="1923" height="1969" alt="image" src="https://github.com/user-attachments/assets/91e028f9-dc13-41eb-ad48-43c27aad35cb" />
<img width="1911" height="346" alt="image" src="https://github.com/user-attachments/assets/e5157736-aa73-4970-a972-65854a66c060" />

#
- <b>Go to <mark> Manage Jenkins --> Tools</mark> and search for SonarQube Scanner installations:</b>
<img width="1291" height="768" alt="image" src="https://github.com/user-attachments/assets/e9342966-dbd6-4628-af14-ee8e64b56e43" />

#
- <b> Similarly, Go to <mark> Manage Jenkins --> credentials</mark> and add Github credentials to push updated code from the pipeline:</b>
<img width="1910" height="720" alt="image" src="https://github.com/user-attachments/assets/39dc8167-3f97-48e0-9f44-203f67cddd79" />

> [!Note]
> While adding github credentials add Personal Access Token in the password field.
> <img width="1918" height="819" alt="image" src="https://github.com/user-attachments/assets/87d45fb6-28e4-4e32-903b-57cd56f02941" />
<img width="1797" height="391" alt="image" src="https://github.com/user-attachments/assets/68e554b8-a574-435b-8ef4-4f70adfe8695" />

#
- <b>Go to <mark> Manage Jenkins --> System</mark> and search for SonarQube installations:</b>
<img width="934" height="821" alt="image" src="https://github.com/user-attachments/assets/05c0e1b5-bd3b-48d7-b507-edf4da871862" />

#
- <b>Login to SonarQube server, go to <mark>Administration --> Webhook</mark> and click on create </b>
<img width="1509" height="1526" alt="image" src="https://github.com/user-attachments/assets/66f6ef8a-860b-40b8-a98a-271373937035" />
<img width="1509" height="1526" alt="image" src="https://github.com/user-attachments/assets/c047a9b1-3be3-4f22-90c0-bf3e665855c0" />

#
- <b>Navigate to <mark> Manage Jenkins --> System </mark> and search for Global Trusted Pipeline Libraries:</b>
  <img width="1034" height="830" alt="image" src="https://github.com/user-attachments/assets/96452c66-d4ff-4789-bb45-056a4d8a0111" />
-Integrate with shared
<img width="1148" height="461" alt="image" src="https://github.com/user-attachments/assets/fcf48eda-d0df-4246-a7f5-bdade6d0972f" />

#

#
- <b>Navigate to <mark> Manage Jenkins --> credentials</mark> and add credentials for docker login to push docker image:</b>
-get docker token
<img width="1396" height="786" alt="image" src="https://github.com/user-attachments/assets/3538c14d-15d2-4734-90d9-937663589f29" />
<img width="1899" height="828" alt="image" src="https://github.com/user-attachments/assets/f815a287-9588-4889-a343-dd3b7104094f" />

#

<mark>CI/CD PIPELINE<\mark>
- <b>Create a <mark>Wanderlust-CI</mark> pipeline</b>
<img width="1905" height="1562" alt="image" src="https://github.com/user-attachments/assets/eae92466-2906-433c-bcdf-4916ba3e64c0" />

#
- <b>Create one more pipeline <mark>Wanderlust-CD</mark></b>
<img width="390" height="530" alt="image" src="https://github.com/user-attachments/assets/7ad2580e-e2a2-442a-868b-a754ed2d08a9" />
<img width="837" height="1363" alt="image" src="https://github.com/user-attachments/assets/00da8ade-e78d-4931-8b82-ae840b33f114" />


-similarly CD pipeline
<img width="1914" height="549" alt="image" src="https://github.com/user-attachments/assets/1f6567e7-93ef-476e-a856-f60dab4a44cc" />
<img width="1917" height="468" alt="image" src="https://github.com/user-attachments/assets/96694871-5b7e-4d28-8c3d-e6ce45c1fcfe" />

#

#
- <b> Go to Master Machine and add our own eks cluster to argocd for application deployment using cli</b>
  - <b>Login to argoCD from CLI</b>
<img width="1549" height="162" alt="image" src="https://github.com/user-attachments/assets/e40e4935-9af3-49da-bf1a-8ed22d0a7ab5" />


  ![image](https://github.com/user-attachments/assets/7d05e5ca-1a16-4054-a321-b99270ca0bf9)

  - <b>Check how many clusters are available in argocd </b>
  -we only have default cluster initailly, add wanderlust cluster
<img width="1916" height="407" alt="image" src="https://github.com/user-attachments/assets/2623d17f-4c1a-4d59-b906-8b980190236f" />

  ```bash
  argocd cluster list
  ```
  - <b>Get your cluster name</b>
  ```bash
  kubectl config get-contexts
  ```
  <img width="1561" height="269" alt="image" src="https://github.com/user-attachments/assets/b1326c5b-1cc6-4b40-b928-23cebb954507" />

  - <b>Add your cluster to argocd</b>
  ```bash
  argocd cluster add Wanderlust@wanderlust.us-west-1.eksctl.io --name wanderlust-eks-cluster
  ```
  > [!Tip]
  > Wanderlust@wanderlust.us-west-1.eksctl.io --> This should be your EKS Cluster Name.
  > -	Through RBAC your argocd can access the k8s cluster
	Through RBAC your argocd can access the k8s cluster
<img width="1590" height="255" alt="image" src="https://github.com/user-attachments/assets/26bf9385-923f-4821-805b-342db561a2a7" />


  - <b> Once your cluster is added to argocd, go to argocd console <mark>Settings --> Clusters</mark> and verify it</b>
<img width="1918" height="528" alt="image" src="https://github.com/user-attachments/assets/d0cb4515-80d0-4590-a011-0bcea5b01827" />


- <b>Now, go to <mark>Applications</mark> and click on <mark>New App</mark></b>

<img width="1918" height="528" alt="image" src="https://github.com/user-attachments/assets/2a648c1f-bb26-4217-84bc-00a0c506057f" />
<img width="1347" height="850" alt="image" src="https://github.com/user-attachments/assets/deecd336-63ff-44de-bee4-ed1aa747ccff" />

> [!Important]
> Make sure to click on the <mark>Auto-Create Namespace</mark> option while creating argocd application

#
- <b>Open port 31000 and 31100 on worker node and Access it on browser</b>
```bash
<worker-public-ip>:31000
```
-	After appli deployed backend it will run on port nodeport:31100(outside pod)   target pod(inside pod)
	frontend  31000
	-Attach the 31100 nodeport in wanderlust sg
 	<img width="1918" height="744" alt="image" src="https://github.com/user-attachments/assets/e8594ebb-2798-43cc-b133-faa618d66b3e" />
<img width="1978" height="1069" alt="image" src="https://github.com/user-attachments/assets/14004535-0bd8-4fb1-9bda-1d95909c9fdb" />
<img width="1924" height="2288" alt="image" src="https://github.com/user-attachments/assets/96253bb1-ce8e-4bcd-bc38-97a6f98ecd7f" />
<img width="1914" height="949" alt="image" src="https://github.com/user-attachments/assets/6f0b70d4-6b18-42be-8755-58cdc1680933" />



- <b>Congratulations, your application is deployed on AWS EKS Cluster</b>
<img width="966" height="17873" alt="image" src="https://github.com/user-attachments/assets/b4def6be-b4ba-4d48-9a90-793ab1b72830" />
<img width="1918" height="841" alt="image" src="https://github.com/user-attachments/assets/48b1bfd8-64f1-41d9-b46d-ded74ce7199a" />
<img width="1924" height="1390" alt="image" src="https://github.com/user-attachments/assets/45736a4b-18c8-4c56-b186-4078896fb239" />



#
## How to monitor EKS cluster, kubernetes components and workloads using prometheus and grafana via HELM (On Master machine)
- <p id="Monitor">Install Helm Chart</p>
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```
```bash
chmod 700 get_helm.sh
```
```bash
./get_helm.sh
```
<img width="955" height="18076" alt="image" src="https://github.com/user-attachments/assets/900d016b-7ee6-4657-a9c2-7dba83dbe6c2" />


#
-  Add Helm Stable Charts for Your Local Client
```bash
helm repo add stable https://charts.helm.sh/stable
```
#
- Add Prometheus Helm Repository
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

#
- Create Prometheus Namespace
```bash
kubectl create namespace prometheus
```
```bash
kubectl get ns
```

#
- Install Prometheus using Helm
```bash
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
```

#
- Verify prometheus installation
```bash
kubectl get pods -n prometheus
```

#
- Check the services file (svc) of the Prometheus
```bash
kubectl get svc -n prometheus
```
<img width="1295" height="593" alt="image" src="https://github.com/user-attachments/assets/220ad1c1-4f08-4871-91d8-1f8d7742f775" />
<img width="1493" height="498" alt="image" src="https://github.com/user-attachments/assets/6ebb7916-8e6d-4d12-a2ab-22e5a5c1bb62" />
<img width="1283" height="759" alt="image" src="https://github.com/user-attachments/assets/57c529a3-1f43-434f-be17-faa82b162d9b" />




#
- Expose Prometheus and Grafana to the external world through Node Port
> [!Important]
> change it from Cluster IP to NodePort after changing make sure you save the file and open the assigned nodeport to the service.

```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```


#
- Verify service
```bash
kubectl get svc -n prometheus
```
Svc updated verify
<img width="1456" height="300" alt="image" src="https://github.com/user-attachments/assets/9443ef90-7021-46d1-8af0-155a064a96d8" />

#
- Now,let’s change the SVC file of the Grafana and expose it to the outer world
```bash
kubectl edit svc stable-grafana -n prometheus
```
#
- Check grafana service
```bash
kubectl get svc -n prometheus
```
<img width="1378" height="270" alt="image" src="https://github.com/user-attachments/assets/18139a20-c62f-4306-b1ab-e6f1f7de28ae" />

#
- Get a password for grafana
```bash
kubectl get secret --namespace prometheus stable-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
<img width="1432" height="88" alt="image" src="https://github.com/user-attachments/assets/bf4207a3-ac71-43d4-b28e-d54668403938" />

#
-Added inbound rules
<img width="1911" height="774" alt="image" src="https://github.com/user-attachments/assets/e4eda865-d1bd-4a24-8d71-b36182d0cf51" />
<img width="1636" height="394" alt="image" src="https://github.com/user-attachments/assets/1fbfb856-b650-4a24-a5f3-e108f1cb84d2" />
<img width="1924" height="1159" alt="image" src="https://github.com/user-attachments/assets/1299856b-7249-4894-ada1-9ecd4dfdebae" />

#  
- <b>add NVD API KEY</b>
<img width="1756" height="967" alt="image" src="https://github.com/user-attachments/assets/01de7926-a15d-47c1-af6e-f7afc6dc7c78" />
<img width="1807" height="427" alt="image" src="https://github.com/user-attachments/assets/603719bc-1bae-43fb-adbb-ff0138b3df50" />
<img width="1830" height="727" alt="image" src="https://github.com/user-attachments/assets/d684d5b2-9f98-4c0a-9633-8703907aad92" />


> [!Note]
> Username: admin

#
- Now, view the Dashboard in Grafana

<img width="1855" height="998" alt="image" src="https://github.com/user-attachments/assets/e19f75e4-d504-4022-bfa5-81923ff9cc58" />
<img width="1914" height="949" alt="image" src="https://github.com/user-attachments/assets/94aa05aa-c669-4235-bc7d-c104a60e4622" />
<img width="1919" height="846" alt="image" src="https://github.com/user-attachments/assets/676e51be-cc04-44ef-9d32-208ccaa0eb2d" />

#
- <b>Run CI/CD pipeline</b>
<img width="1902" height="967" alt="image" src="https://github.com/user-attachments/assets/00f63a87-d2ee-4bfe-bfed-620d483c83c3" />
<img width="1918" height="933" alt="image" src="https://github.com/user-attachments/assets/3cb69949-e850-465f-9d0c-042c9743c691" />

- <b>Email Notification</b>
<img width="1917" height="826" alt="image" src="https://github.com/user-attachments/assets/01919623-d225-47e1-9761-154a28e3ebe7" />

-prometheus
<img width="1918" height="948" alt="image" src="https://github.com/user-attachments/assets/3498574b-e3fa-4bf1-b74d-686a63b2f9af" />
<img width="1926" height="1594" alt="image" src="https://github.com/user-attachments/assets/ab32f819-df72-46d4-a96d-6aab3efb53fb" />

-grafana
<img width="1918" height="954" alt="image" src="https://github.com/user-attachments/assets/34b5e714-a206-408d-8b01-06f95ba2d955" />
<img width="1918" height="954" alt="image" src="https://github.com/user-attachments/assets/800dccc2-cb1c-4c1a-879b-44c13078c60d" />
<img width="1918" height="816" alt="image" src="https://github.com/user-attachments/assets/af863c41-e3f4-468c-b478-47bc024aaa81" />


#
## Clean Up
- <b id="Clean">Delete eks cluster</b>
```bash
eksctl delete cluster --name=wanderlust --region=us-east-1
```

#
terraform destroy


#
