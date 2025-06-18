# 🛠️ Ansible-Driven Configuration Inside Kubernetes Pods

This project demonstrates how to use Ansible inside a Kubernetes pod (`ansible-master`) to remotely configure another pod (`app-managed`) over SSH. 
---

##  Components

- ansible-master: Pod with Ansible installed (controller node)
- app-managed : Pod running Ubuntu (target node), accessed via SSH

---

 Setup Instructions

Create the Pod

ansible-master.yaml

apiVersion: v1
kind: Pod
metadata:
  name: ansible-master
spec:
  containers:
  - name: ansible
    image: your-dockerhub-username/ansible-ubuntu:manual
    ports:
    - containerPort: 22


app-managed.yaml

apiVersion: v1
kind: Pod
metadata:
  name: app-managed
  labels:
    app: web
spec:
  containers:
  - name: app
    image: ubuntu:22.04
    command: ["sleep"]
    args: ["infinity"]
    ports:
    - containerPort: 22

Apply the Pods

kubectl apply -f ansible-master.yaml
kubectl apply -f app-managed.yaml
kubectl get pods -o wide

Enable SSH in app-managed Pod

kubectl exec -it app-managed -- bash

apt update
apt install -y openssh-server curl
mkdir /var/run/sshd
echo 'root:root' | chpasswd
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
/usr/sbin/sshd
sshd listens on port 22 and enables remote connections. The PermitRootLogin yes setting allows Ansible to connect as root.

Configure Ansible in ansible-master

kubectl exec -it ansible-master -- bash
ansible --version
Create an inventory file:

[web]
<app-managed-IP> ansible_user=root ansible_password=root ansible_ssh_common_args='-o StrictHostKeyChecking=no'
Create a playbook: setup.yml

- name: Install and start NGINX on web server
  hosts: web
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install NGINX
      apt:
        name: nginx
        state: present

    - name: Start and enable NGINX
      service:
        name: nginx
        state: started
        enabled: yes

Run the Playbook

ansible-playbook -i inventory setup.yml
Verify the NGINX Server

kubectl exec -it app-managed -- bash
curl localhost
You should see the default NGINX welcome page HTML output.

 Outcome
Ansible-based remote provisioning inside Kubernetes

Configured NGINX on a running container via SSH

Infrastructure-as-code demonstration with a hands-on Kubernetes example

Notes
Use a Docker image with Ansible preinstalled for ansible-master

Ensure both pods can communicate via internal IPs (check kubectl get pods -o wide)

SSH is configured manually here, but can be scripted or baked into custom images

 
