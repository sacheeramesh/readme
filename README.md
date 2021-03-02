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

## Compliance-Manager GUI setup

### Install Keycloak 

> Note: This is test deployment only. Production ready deployment to have high-availability in place. 
- Start keycloak
```bash
docker run -d --name cm_keycloak -p 8080:8080 -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin quay.io/keycloak/keycloak
```

- Update configs `/hms/apps/oss-compliance-manager/conf/compliance_manager.ini`
```bash
[auth]
secret_key = '\xe6\xcc\x1c&\x8f\xf9\xc0\x0b*\x0c\x0b\xc5\x82\xedk\x93\xbc$\xee\xa0y\x96\xc8\xb8'
client_secrets = /hms/apps/oss-compliance-manager/conf/client_secrets.json
auth_providers = http://oss.hsenidmobile.com/auth/realms/hsenidmobile
callback_uri = http://oss.hsenidmobile.com/oidc_callback
logout_uri = http://oss.hsenidmobile.com/auth/realms/hsenidmobile/protocol/openid-connect/logout
logout_redirect_uri = http://oss.hsenidmobile.com/
```

- Access Admin Console and Import the client data file in '/hms/apps/oss-compliance-manager/setup/keycloak_data/realm_export.json'
- Create user accounts and credentials. 
- Add the user's to relevant user groups. 


### Setup http conf and httpd service 

- Copy httpd configurations
```bash
cp /hms/apps/oss-compliance-manager/setup/system_config/oss_compliance_manager.conf /etc/httpd/conf.d/
```
<br/>

- Restart httpd
```bash
systemctl restart httpd
```

### Setup OSS Compliance Manager Service

- Configure logs for oss_compliance_manager service 
```bash
cp /hms/apps/oss-compliance-manager/setup/system_config/oss_compliance_manager_syslog.conf /etc/rsyslog.d/
sudo systemctl restart rsyslog
```
<br/>

- Copy systemd service unit file and enable service
```bash
cp /hms/apps/oss-compliance-manager/setup/system_config/oss_compliance_manager.service /etc/systemd/system/
systemctl enable oss_compliance_manager
systemctl start oss_compliance_manager
```
<br/>



## Specification

## Usage - CLI

- Main Help
```bash
Available Commands: Use [-h|--help] to view complete command reference.
  compliance_manager compliance list
  compliance_manager compliance show
  compliance_manager compliance attach
  compliance_manager compliance check
  compliance_manager compliance apply
  compliance_manager rule list
  compliance_manager rule show
  compliance_manager host list
  compliance_manager host show
  compliance_manager hostgroup list
  compliance_manager hostgroup show
  compliance_manager report list
  compliance_manager report view

```

> Follow the help message content for individual commands. 

<br/>




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




# Appendix I
## Keycloak Realm setup

### Install Keycloak
- Get docker images for keycloak and mariadb
```bash
sudo docker pull mariadb
sudo docker pull quay.io/keycloak/keycloak
```
<br/>

- Create a virtual network for keycloak
```bash
sudo docker network create keycloak-network
```
<br/>

- Create a mariadb data directory for keycloak

```bash
mkdir -p /hms/data/mariadb/keycloak_data
```
<br/>

- Create the mariadb container
```bash
sudo docker run -d \
    --name mariadb \
    --net keycloak-network \
    -p 3306:3306 \
    -v /hms/data/mariadb/keycloak_data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=beyondm@123 \
    -e MYSQL_DATABASE=keycloak \
    -e MYSQL_USER=keycloak \
    -e MYSQL_PASSWORD=beyondm@123 \
    mariadb
```
<br/>

- Create the keycloak container
```bash
sudo docker run -d \
  --name cm_keycloak \
  --net keycloak-network \
  -p 8080:8080 \
  -p 8443:8443 \
  -e KEYCLOAK_USER=admin \
  -e KEYCLOAK_PASSWORD=beyondm@123 \
  quay.io/keycloak/keycloak
```

### Configure realm
- Define realm hsenidmobile
-- Use `https://oss.hsenidmobile.com/auth/` as FrontEnd URL

- Define client oss_compliance_manager
- Define following client roles
```bash
oss_cm_admin		
oss_cm_auditor		
oss_cm_mgm			
oss_cm_super_admin	
oss_cm_user			
```

- Define following Realm Roles and assign client roles
```bash
oss_admin		
oss_auditor		
oss_mgm			
oss_super_admin	
oss_user	
```