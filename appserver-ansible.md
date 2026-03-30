# PPD JCP AppServer Upgrade & Rollback Automation 🚀

This repository contains Ansible playbooks and roles to automate **backup, upgrade, service restart, and rollback** of the JCP AppServer environment.

---

## 📌 Overview

This automation performs:

* ✅ Backup of existing application files
* ✅ Deployment of new build artifacts
* ✅ Restart of Tomcat services
* ✅ Rollback capability in case of failure

---

## 📂 Inventory Configuration

### Inventory File Example

```ini
[ppd-jcp-appserver1]

10.140.100.84 ansible_ssh_private_key_file=/home/userapp/.ssh/id_rsa ansible_user=userapp

[all-ppd-jcp-appserver:children]

ppd-jcp-appserver1
```

### Hosts File (`/etc/ansible/hosts`)

```ini
[ppd-jcp-appserver1]

10.140.100.84 ansible_ssh_pass=MARCH19 ansible_ssh_user=userapp

[all-ppd-jcp-appserver:children]

ppd-jcp-appserver1
```

---

## ⚙️ Upgrade Playbook

### File: `ppd-jcp-appserver-upgrade.yml`

```yaml
- hosts: all-ppd-jcp-appserver
  remote_user: userapp
  vars_files:
    - vars/vars.yml
  roles:
    - ppd-jcp-appserver-backup
    - ppd-jcp-appserver-upgrade
    - ppd-jcp-appserver-tomcat-services
```

---

## 🧩 Roles Description

### 🔹 1. Backup Role

**Path:** `roles/ppd-jcp-appserver-backup/tasks/main.yml`

* Creates dated backup directory
* Backs up:

  * `com` directory
  * `application.xml`
  * `task-executors.xml`
  * `persistence.xml`
* Stores with timestamp

---

### 🔹 2. Upgrade Role

**Path:** `roles/ppd-jcp-appserver-upgrade/tasks/main.yml`

* Fetches build artifacts from remote server
* Copies:

  * `com.zip`
  * `application.xml`
  * `persistence.xml`
  * `task-executors.xml`
* Extracts and deploys application files
* Updates permissions
* Validates deployment

---

### 🔹 3. Tomcat Service Role

**Path:** `roles/ppd-jcp-appserver-tomcat-services/tasks/main.yml`

* Stops running Tomcat processes
* Starts Tomcat service
* Waits for successful startup
* Checks logs for exceptions
* Verifies running PID

---

## ▶️ Upgrade Execution

```bash
ansible-playbook ppd-jcp-appserver-upgrade.yml -i inventory
```

---

## 🔄 Rollback Playbook

### File: `ppd-jcp-app-server-rollback.yml`

```yaml
- hosts: all-jcp-app
  remote_user: userapp
  vars_files:
    - vars/vars.yml
  roles:
    - ppd-jcp-app-server-rollback
    - ppd-jcp-appserver-tomcat-services
```

---

## 🔁 Rollback Role Details

### 🔹 Backup Restore Role

**Path:** `roles/ppd-jcp-app-server-rollback/tasks/main.yml`

* Finds latest backup directory
* Identifies oldest backup set
* Deletes current deployment
* Restores from backup
* Verifies restored files

---

### 🔹 Tomcat Restart Role

* Stops existing Tomcat processes
* Starts service
* Waits for startup confirmation
* Validates logs
* Confirms process is running

---

## ▶️ Rollback Execution

```bash
ansible-playbook ppd-jcp-app-server-rollback.yml -i inventory
```

---

## 📋 Prerequisites

* Ansible installed
* SSH access to target servers
* Proper inventory configuration
* Required permissions for `userapp`
* Network access to build server (10.140.100.66)

---

## ⚠️ Important Notes

* Always verify backup before upgrade
* Validate variables in `vars.yml`
* Ensure sufficient disk space
* Test in staging before production
* Monitor logs after deployment

---

## 📁 Directory Structure (Example)

```
jcp-ansible/
│── roles/
│   ├── ppd-jcp-appserver-backup/
│   ├── ppd-jcp-appserver-upgrade/
│   ├── ppd-jcp-appserver-tomcat-services/
│   ├── ppd-jcp-app-server-rollback/
│── vars/
│   └── vars.yml
│── inventory
│── ppd-jcp-appserver-upgrade.yml
│── ppd-jcp-app-server-rollback.yml
```

---

## 🤝 Contribution

Feel free to raise issues or submit pull requests for improvements.

---

## 📜 License

Internal project – not for public distribution.

---
