# Automate EC2 provisioning in AWS using Jenkins and Ansible Playbook

# Project Blog link :-

# Project Overview :-
- We will learn how to create new EC2 instances using the Ansible playbook and automate using Jenkins Pipeline. in the end, we will play the game of 2048.

# Project Steps :-

# 1. Launch an Ubuntu(22.04) T2 medium Instance:-
- Launch an AWS T2 medium Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group.
<img width="960" alt="image" src="https://github.com/rutikdevops/DevOps-Project-9/assets/109506158/c69ab020-9ac7-4be6-a705-92a0fa970585">

# 2. Install Jenkins and Trivy:-
- To Install Jenkins :- Connect to your console, and enter these commands to Install Jenkins
```bash
vi jenkins.sh
```

```bash
#!/bin/bash
sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```

```bash
sudo chmod 777 jenkins.sh
./jenkins.sh    # this will installl jenkins
```

- Once Jenkins is installed, you will need to go to your AWS EC2 Security Group and open Inbound Port 8080, since Jenkins works on Port 8080.
- Now, grab your Public IP Address

```bash
<EC2 Public IP Address:8080>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
<img width="555" alt="image" src="https://github.com/rutikdevops/DevOps-Project-9/assets/109506158/4223bac5-8721-4d1c-b89d-92e1a476045d">

- Unlock Jenkins using an administrative password and install the suggested plugins.
- Jenkins will now get installed and install all the libraries.
- Create a user click on save and continue.
- Jenkins Getting Started Screen.



# 3. Install Trivy:-
```bash
vi trivy.sh
```

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```

# 4. Install Ansible :-

- connect to your Jenkins machine using Putty or Mobaxtreme
- Now we are going to run the below commands on the Jenkins machine

```bash
sudo apt-get update                                                  # Update your system packages
sudo apt install software-properties-common                          # First Install Required packages to install Ansible
sudo add-apt-repository --yes --update ppa:ansible/ansible           # Add the ansible repository via PPA
sudo apt install python3                                             # Install Python3 on Jenkins for Ansible
sudo apt install ansible -y                                          # Install Ansible on Ubuntu 22.04 LTS
sudo apt install ansible-core                                        # Copy
ansible --version                                                    # To check version
sudo apt install python3-pip -y                                      # Install Python-pip3
sudo pip3 install boto boto3                                         # Install Boto Framework - AWS SDK
sudo apt-get install python3-boto -y                                 # Ansible will access AWS resources using Boto SDK.
pip list boto | grep boto
```

# 5. let's create and attach an IAM role to Jenkins machines for the provision of a new ec2 instance
- Navigate to AWS CONSOLE --> Click the "Search" field --> Type "IAM enter" --> Click "Roles" --> Click "Create role" --> Click "AWS service" --> Click "Choose a service or use case" --> Click "EC2" --> Click "Next" --> Click the "Search" field. --> Add permissions policies --> AmazonEC2FullAccess --> click Next --> Click the "Role name" field. --> Type "Jenkins-cicd" --> Click "Create role" --> Click "EC2"

- go to the Jenkins instance and add this role to the Ec2 instance:-
  select Jenkins instance --> Actions --> Security --> Modify IAM role --> Add a newly created Role and click on Update IAM role.

- Let's go to the Jenkins machine and add the Ansible Plugin :-
  Manage Jenkins --> Plugins --> Available Plugins --> search for Ansible and install


















