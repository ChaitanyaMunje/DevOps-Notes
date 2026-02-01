# 📘 Ansible Notes

---

## 📌 Introduction

Ansible is an IT automation tool for creating infrastructure.

---

## ✅ Advantages of Using Ansible

1. **Simple and easy to use**  
   Sensible code is written in YAML which makes it easy to read.

2. **Agentless architecture**  
   Ansible does not require to be installed on remote machines which makes it easy to set up.

3. **Configuration Management**  
   Ansible is used to automate configuration management tasks such as application deployment and infrastructure management.

4. **Scalability**  
   Ansible is used to manage large number of systems which makes it easy for large scale deployments.

5. **Open Source**  
   It is open source and free to use.

6. **Integrations**  
   It can be integrated with Azure, Docker, and many more.

---

## 🧩 Three Main Components of Ansible

1. **Inventory**  
   Inventory contains details about the servers on which we will be performing the tasks.

2. **Playbook**  
   The task which we have to perform is mentioned in this file such as installing a service.

3. **Module**  
   It is the smallest program to do a task.  
   Example: Installing nginx, starting a service, etc.

---

## 🖥 Scenario Details

### Main Server
- Virtual machine name: `3in1-VM`
- IP Address: `20.244.83.151`

### Remote Server
- Virtual machine name: `azure-pipeline-vm`
- IP Address: `4.213.120.58`

### Video Scenario
- Main server: `centos01`
- Remote server: `centosQA`

---

## 🔑 Passwordless SSH Setup

On the **main server**, execute the following commands:

1. Generate SSH key:
   ```
   ssh-keygen
   ```

2. Press **Enter** when prompted to generate the key.

3. Copy SSH key to remote server:
   ```
   ssh-copy-id 4.213.120.58
   ```

4. Connect to remote server without password:
   ```
   ssh rootAgent@4.213.120.58
   ```

---

## ⚙️ Installation of Ansible on Main Server

### Documentation Link
```
https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu
```

### Installation Steps

```
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

### Verify Installation

```
ansible --version
ansible localhost -m ping
```

---

## 🧪 Creating the First Playbook

### Directory
```
/etc/ansible/playbooks
```

### Create Playbook File
```
vi first_pb.yml
```

### Playbook Code

```
- name: First Basic Playbook
  hosts: localhost

  tasks:
  - name: Test connectivity
    ping:
```

### Run Playbook

```
ansible-playbook first_pb.yml
```

### Syntax Check

```
ansible-playbook --syntax-check first_pb.yml
```

---

## 🌐 Installing Nginx on Main Server

### Create Playbook

```
vi app-install.yml
```

### Playbook Code

```
- name: Install and start a service
  hosts: localhost
  become: true

  tasks:
  - name: Installing nginx
    package:
      name: nginx
      state: present

  - name: Starting nginx service
    service:
      name: nginx
      state: started
      enabled: true
```

### Run Playbook

```
ansible-playbook app-install.yml
```

---

## 📝 Line-by-Line Explanation

```
hosts: localhost        # Target machine
become: true            # Escalate privileges
package                 # Package module (apt/yum)
service                 # Service management
```

---

## 🧾 Managing Remote Machines Using Inventory

### Edit Hosts File

```
vi hosts
```

Add remote IP and save.

### Update Playbook

```
hosts: all
```

This will run playbook on all machines in inventory.

---

## 📦 Installing Application on Remote Machine

Change `hosts: localhost` to:

```
hosts: all
```

Run:

```
ansible-playbook app-install.yml
```

---

## 📁 Copying Files Using Ansible Copy Module

### Playbook Code

```
- name: Copying files from main to remote
  hosts: all

  tasks:
  - name: Copy files
    copy:
      src: /etc/ansible/playbooks/copy_files.yml
      dest: /tmp/
      owner: rootAgent
      group: rootAgent
      mode: ugo=rw
      backup: true
```

Run using:

```
ansible-playbook copy_files.yml
```

---

## 📂 Ansible File Module

### Create & Delete Files/Folders

```
- name: Creating and deleting files
  hosts: all

  tasks:
  - name: Creating a file
    file:
      path: /tmp/hello.txt
      state: touch
      owner: rootAgent
      group: rootAgent
      mode: u=rwx,g=rw,o=r

  - name: Creating a folder
    file:
      path: /tmp/myfolder
      state: directory

  - name: Deleting a file
    file:
      path: /tmp/hello.txt
      state: absent

  - name: Deleting a folder
    file:
      path: /tmp/myfolder
      state: absent
```

---

## ▶️ Running Script on Remote Machine

```
chmod +x script.sh
```

### Playbook

```
- name: Run a shell script
  hosts: all

  tasks:
  - name: Run a shell script
    shell: ./script.sh >>script.log
    args:
      chdir: /tmp/
      creates: script.log
```

---

## ⏰ Running CRON Job Using Ansible

### Create CRON Job

```
- name: Cron Job
  hosts: all

  tasks:
  - name: Run CRON job
    cron:
      name: Run Test Script
      minute: 30
      hour: 18
      day: 15
      month: "*"
      weekday: "*"
      user: rootAgent
      job: /tmp/script.sh
```

### Remove CRON Job

```
- name: Remove cron job
  hosts: all

  tasks:
  - name: Remove cron job
    cron:
      name: Run Test Script
      state: absent
      user: rootAgent
```

---

## 👤 User Management Using Ansible

```
- name: User Management
  hosts: all
  become: true

  tasks:
  - name: User Creation
    user:
      name: chaitanya
      comment: user adding to QA team
      shell: /bin/bash
      group: QA
```

---

## ⚡ AdHoc Tasks with Ansible

```
ansible 4.213.120.58 -m ping
```

---

## 🏷 Tags in Ansible

### Example

```
tags: install-nginx
tags: start-nginx
```

### Commands

```
ansible-playbook app-install.yml --list-tags
ansible-playbook app-install.yml -t install-nginx
ansible-playbook app-install.yml --skip-tags start-nginx
```

---

## 📦 Variables in Ansible

### Example

```
vars:
  - app: nginx
```

Usage:
```
name: "{{app}}"
```

---

## 🗂 Variables in Inventory

```
azure-pipeline-vm ansible_host=4.213.120.58
```

---

## 🔀 Conditions in Ansible

```
when: ansible_os_family == "RedHat"
when: ansible_os_family == "Debian"
```

Check OS details:

```
ansible azure-pipeline-vm -m setup
```

---

## 🔁 Loops in Ansible

Loops are used to perform repeated tasks like installing multiple applications.

---

## 🎭 Roles in Ansible

### Create Role

```
cd /etc/ansible/roles
sudo ansible-galaxy init httpd_setup
```

Navigate into the role directory to continue configuration.

---

### ✅ End of Notes
