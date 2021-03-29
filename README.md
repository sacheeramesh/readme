# Compliance Manager

## Introduction 

Compliance manager project provides the ability to an administrator to 
define Compliance Rules, and reuse them in Compliance standards. 

## Tech Stack
### Core Server
- Python 3
    - Benedict
    - PyYaml
    - Yamlpath
    - jinja2
- Bash
- Ansible

### API Server 
- Python 3
    - Python-Flask
    - SQL-Alchemy
    - Json
    - PyYaml
    - Benedict
    - jinja2

### Reports 
- HTML
- Bootstrap 4 
- JQuery

## Architecture

### Architecture diagram

![image info](https://drive.google.com/uc?id=1zA8ms0NLGuvhpJzr9eaGzxh7Wma1QiRj)
<br/>

### Component Relationship
![image info](https://drive.google.com/uc?id=1Z31JYaqfIIP25wPzUs2k3d8zH8T4ygPA)

### Compliance Audit / Remediation flow
![image info](https://drive.google.com/uc?id=1B4If3jQWBRSXFK2TwBjE3ZjQLISmuYxw)


## Installation

- Create cmuser and set password
```bash
ADMIN_USER=cmuser
sudo useradd cmuser
passwd ${ADMIN_USER}
```
<br/>

- Create passwordless sudo access to the cmuser
```bash
ADMIN_USER=cmuser
sudo touch /etc/sudoers.d/${ADMIN_USER}
sudo chmod 0660 /etc/sudoers.d/${ADMIN_USER}
echo "${ADMIN_USER} ALL = (root) NOPASSWD:ALL" > /etc/sudoers.d/${ADMIN_USER}
echo 'Defaults:${ADMIN_USER} !requiretty' >> /etc/sudoers.d/${ADMIN_USER}
sudo chmod 0440 /etc/sudoers.d/${ADMIN_USER}
```
<br/>

- Install Ansible and Python3 packages
```bash
sudo yum install python3 -y

sudo yum install epel-release -y
sudo yum install ansible -y
```
<br/>

- Copy oss-compliance-manager to /hms/apps directory
```bash
cp -r oss-compliance-manager /hms/apps
```
<br/>

- Create python3 virtual environment
```bash
cd /hms/apps/
python3 -m venv cm_venv
```
<br/>

- Activate the virtual environment and install dependencies
```bash
source /hms/apps/cm_venv/bin/activate
pip install -r /hms/apps/oss-compliance-manager/requirements.txt
```
<br/>

- Create following alias in .bashrc file of the cmuser user.  
```bash
alias compliance_manager='sh /hms/apps/oss-compliance-manager/bin/compliance_manager.sh'
```

## Specification

## Usage

### Preparing remote hosts
- Create cmuser and set password
```bash
ADMIN_USER=cmuser
sudo useradd cmuser
passwd ${ADMIN_USER}
```
<br/>

- Create passwordless sudo access to the cmuser
    
```bash
ADMIN_USER=cmuser
sudo touch /etc/sudoers.d/${ADMIN_USER}
sudo chmod 0660 /etc/sudoers.d/${ADMIN_USER}
echo "${ADMIN_USER} ALL = (root) NOPASSWD:ALL" > /etc/sudoers.d/${ADMIN_USER}
echo 'Defaults:${ADMIN_USER} !requiretty' >> /etc/sudoers.d/${ADMIN_USER}
sudo chmod 0440 /etc/sudoers.d/${ADMIN_USER}
```
<br/>

- Setup keybased authentication to target host ansible user for login from oss-compliance-manager host.
```bash
# From Compliance Manager Host
sudo su - cmuser
ssh-copy-id cmuser@<hostname>
``` 
- Create a host profile from the template
- Update host profile in the oss-compliance-manager


