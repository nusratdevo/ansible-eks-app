
## Tools Used:

- AWS Account
- Jenkins
- Github
- SonarQube
- Trivy
- Docker & Dockerhub
- teraform (for ec2 & Eks Setup)
- Aws Cli

### Step 1: Clone repository
```shell 
    git clone https://github.com/nusratdevo/Tetris-argo-app.git
    cd Tetris-argo-app/Jenkins-terraform/
```


### Step 2: EC2 Setup using terraform
- Run the following commands. 
``` shell 
terraform init
terraform validate 
terraform plan
terraform apply --auto-approve
terraform destroy --auto-approve
```
- after all work done infrastructure should be destroy: `terraform destroy`
- SSH into the instance from your local machine.

### Step 3: check every tools install successfully

```shell
jenkins --version
docker --version
terraform --version
which terraform
path: /usr/bin/terraform
aws version
docker ps
trivy --version
kubectl version --client --short
```

### Step 4: Open jenkins in browser, login and install tools

```shell
Open jenkins on port <EC2 Public_IP>:8080
administrative password : sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
### Step 5: Sonar server configuration

- Sonarqube works on Port 9000, so <Public_IP>:9000. username: admin, password: admin
- Create SonarQube token : Click on Administration ,Give a name for token → and click on Generate Token

- In the Sonarqube Dashboard, Create Webhook ``` url <http://public_ip:8080/sonarqube-webhook/>```
- In jenkins dashbord add sonar server: name(soner-server), ``` url <http://public_ip:9000>```

### step 6: Set up Jenkins Plugins:

- Goto Jenkins Dashboard → Manage Jenkins → Plugins → Available Plugins

* Docker
* Docker Commons
* Docker Pipeline
* Docker API
* NodeJs Plugin
* SonarQube Scanner
* Owasp Dependency Check
* Terraform
* AWS Credentials
* Pipeline: AWS Steps
* Prometheus metrics plugin
* Eclipse Temurin Installer (Install without restart)
* Prometheus Plugin (to monitor pipeline)
* Kubernetes
* Kubernetes CLI
* Kubernetes Client API
* Kubernetes Pipeline DevOps steps

### step 5: Set up Jenkins Tools Configuration:

- Goto Jenkins Dashboard → Manage Jenkins → Tools

* JDK Installations (Install from adomptium.net), Name (jdk17)
* SonarQube Scanner Installations(Install from Maven Central),Name (sonar-scanner)
* NodeJs   Installations (Install from nodejs.org) Name(nodejs)
* Dependency-Check Installations (Install from github.com) Name(DP-Check)
* Docker Installation (Download from docker.com) Name(docker)
* Terraform Installations (install directory /usr/bin) Name(terraform)

### step 6: Set up credentials:

- setup credentials: dockerhub, sonar-token, AWS Credentials, Github, Ansible
- Goto Dashboard → Manage Jenkins → Credentials → system → Global credentials (unrestricted)→ Add Credentials
- Dockerhub: kind(username with password)->username(dockerhub username)->password(dockerhub pass)->Id(dockerHub)
- AWS Credentials: kind(AWS Credentials), ID(aws-key)
- Github Credentials: Create personal access token (classic), kind(username with password), ID(github)
- Ansible Credentials: kind(SSH username with private key)->Id(SSH)->username(ubuntu)->private key(Enter directly)->Key(Pem_file)

### Step: write an Ansible playbook to create a docker image

- Dockerhub Secret:- Go to docker hub and click on your profile –> Account settings –> security –> New access token
Provide a name and click on generate token, Copy the token save it in a safe place
- Run Ansible playbook to generate Docker Image ,Tag And push to dockerhub
```shell
```

### Step:- EKS Cluster Setup

- Create a pipeline providing a name.
- Under pipeline section:
```shell
i. Choose Pipeline script from SCM
ii. Then git.
iii. Select the credentials of your github.
iii. repository Branch :main
iv. Give path of Jenkinsfile:Jenkinsfile-EKS-Terraform
vi. Discard old build(check), builds keep 2
vii.  Click save and apply.
```
-Then build the job.
After 10–15 minutes pipeline will get successful.

### step 7: Create EKS Cluster with terraform In parameterize pipline
- create new item with pipline name
- Navigate to Pipeline section. Check This build is parameterize
- name (action), parameter (apply, destroy)->save
- Click on build with parameter Apply.
Image: 


### Step : Verify Deployed Application
``` kubectl get all ```
- On the same Jenkins server instance update the config:
```shell
aws configure
aws eks update-kubeconfig --region us-east-1 --name Tetris-EKS-Cluster
sudo cat /home/ubuntu/.kube/config
```
- copy the file that content and save in a local file with any name as EKS-SECRET file.
- goto manage jenkins->credentials->system->global credentails: kind (Secret file) and ID (k8s)
- Add Stage in Jenkins pipeline to deploy Kubernetes cluster.
```shell
kubectl get pods
```
- This will give error as authentication required.

### Step: To authenticate install aws-iam-authenticator
- AWS-IAM-Authenticator
```shell
curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.15.10/2020-02-22/bin/linux/amd64/aws-iam-authenticator
sudo curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.15.10/2020-02-22/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
sudo chmod +x ./aws-iam-authenticator
sudo mv ./aws-iam-authenticator /usr/local/bin
```
- Then run
```sudo kubectl get pods```
- This will give "No resources found in the default namespace"

### Step:- Create Jenkins Pipeline for running simple ansible playbook for Kubernetes deployment.
- Create a pipeline providing a name: ``2048-game``
- Under pipeline section:
```shell
i. Choose Pipeline script from SCM
ii. Then git.
iii. Select the credentials of your github.
iii. repository Branch :main
iv. Give path of Jenkinsfile:Jenkinsfile
vi. Discard old build(check), builds keep 2
vii.  Click save and apply.
```
-Then build the job.
- Then run
```sudo kubectl get pods```
```kubectl get all ```
- This will create a loadbalancer on AWS console
- Copy and paste the DNS name on your browser
- You can access the same application on Jenkins server too ``` <ec2_public_ip>:80 ```

Image:
