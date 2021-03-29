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

## Host Profile Configuration

Host Profiles are located under */oss-compliance-manager/profiles/host_profiles* directory. The naming criteria for host profiles should be <host_group>.<host_name>.yaml. The following configuration changes are required* prior to executing the compliance-manager script. 
As prerequisite, create an entry mapping the IP address of the target host with an fqdn in the */etc/hosts* file on the controller node.

    host_id: < insert unique identifer >
    host_group: < insert host group >
    hostname: < insert host name > 
    fqdn: < insert fqdn configured in /etc/hosts >
    ssh_key: |
     -----BEGIN RSA PRIVATE KEY-----
     .... ssh private access key ...
     -----END RSA PRIVATE KEY-----
     user: <sudo_user_in_target_node>
     compliance_standard:
       - < compliance profile name | eg: cis_centos_7 >
    host_configs:
      os:
        updates:
          gpg_keys:
            - <gpg keys for configured repos on target>
            - <gpg keys for configured repos on target>
    
        boot:
          grub2_mypass: <grub_bootloader_password>
    
      app:
        time_sync:
          use_time_sync_ntp: < boolean || set 1 or 0 >
          use_time_sync_chrony: < boolean || set 1 or 0 >
          time_sync_package_name: < set as either ntp or chrony >
          time_synchronization: < set as either ntp or chrony >
          ntp_server_options: "iburst"
          ntp_time_synchronization_servers: 
            - 123.123.123.123
            - 456.456.456.456
    
      net:
        protocol:
          ipv6_disable: < True or False >
        firewall:
          system_firewall_firewalld: < True or False >
          system_firewall_iptables: < True or False >
          system_firewall_nftables: < True or False >
      # make sure to only add one entry as True depending on the firewall requirement. The rest should be configured as false
    
      log:
        logging:
          rsyslog_remoteloghost: < set a domain address to send remote logs >
          listen_rsyslog_messages: < True or False || set as True if the target is expected to listen to rsyslog messages >
    
      selinux:
        selinux_mode: 0
    # set selinux 0 for permissive, 1 for encforcing 

Notes:
1. The username added under the user: variable will be used as a parameter for the ssh AllowUsers configuration. SSH connections will be only limited that particular user until other users are added manually post remediation.

## Compliance Profile Configuration

Default configurations can be used without any modifications. If the requirement specifies to skip any rules, simply comment out the relevant entry from the associated compliance profile. All compliance profiles are located under */oss-compliance-manager/profiles/compliance_profiles* directory. See the Troubleshooting section for any possible errors and fixes.

        fs:
          id: cis_centos_7.sys.fs
          compliance_id: "1.1"
          name: Filesystem Configuration
          description: |
            Secure the FileSystem related configurations.
          rules: 
          #  - rule_id: hms_csp.os.fs.tmp.mount < This rule will be skipped >
          #    compliance_id: "1.1.2"
            - rule_id: hms_csp.os.fs.tmp.noexec
              compliance_id: "1.1.3"
            - rule_id: hms_csp.os.fs.tmp.nodev
              compliance_id: "1.1.4"
            - rule_id: hms_csp.os.fs.tmp.nosuid
              compliance_id: "1.1.5"

Run The Compliance Manager

To execute the compliance manager, move to the *oss-compliance-manager/bin/* directory and execute the compliance_manager.sh script as follows
For Auditing

    compliance_manager compliance check <host_id>

For Remediation

    compliance_manager compliance apply <host_id>

Once successful, all compliance reports will be stored under *oss-compliance-manager/reports* directory. 

## Troubleshooting

In this section, a collection of common run time issues and possible fixes are listed. For general information regarding run time errors, refer the log files created under oss-compliance-manager/work/ directory.

1.  rule_id in profile.yml does not match with rule_id in cis_rule.yml
Detection
If this issue occurs program will output the following error msg before creating a .log file.

      File "/hms/apps/oss-compliance-manager/src/classes/rule.py", line 76, in find_rule_by_id
        return rule[0][0]
    IndexError: list index out of range

  Resolution
  Change the value in rule_id in profile.yml or cis_rule.yml accordingly.

Note: It is recommended to change value in *profiles/compliance_profiles/profile_name.yml* because same rule_id could be used in other profiles as well.

