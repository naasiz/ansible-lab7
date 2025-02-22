## **Ansible Web Server Deployment Lab**
### **Overview**
This lab sets up an **Nginx web server** on two AWS EC2 instances using **Ansible** and **Terraform**.  
The playbook installs Nginx, configures the web root directory, deploys a template-based index page, and ensures the service is running.

---

### **1 Generating SSH Keys**
Before running Terraform, generate an SSH key pair to access the AWS instances.

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/aws -N ""
```
- `-t rsa -b 4096` → Creates an RSA key with **4096-bit encryption**.  
- `-f ~/.ssh/aws` → Saves the key pair as **aws** and **aws.pub** in `~/.ssh/`.  
- `-N ""` → No passphrase for automated use.

After generating the key, import it into AWS:
```bash
./scripts/import_lab_key ~/.ssh/aws.pub
```

---

### **2 Running Terraform to Provision Infrastructure**
Navigate to the Terraform directory and initialize:
```bash
cd ~/4640-w7-lab-start-w25/terraform
terraform init
```

Apply Terraform configuration to create two EC2 instances:
```bash
terraform apply -auto-approve
```
After completion, Terraform will output the **public IP addresses** of both instances.

---

### **3 Setting Up Ansible Inventory**
Edit the inventory file with the IP addresses of the EC2 instances:
```bash
nano ~/4640-w7-lab-start-w25/ansible/inventory/hosts.yml
```
Example inventory file:
```yaml
all:
  children:
    web:
      hosts:
        server-one:
          ansible_host: <PUBLIC_IP_1>
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/aws
        server-two:
          ansible_host: <PUBLIC_IP_2>
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/aws
```

---

### **4 Running the Ansible Playbook**
Navigate to the Ansible directory:
```bash
cd ~/4640-w7-lab-start-w25/ansible
```

Verify syntax before execution:
```bash
ansible-playbook --syntax-check playbook.yml
```

Run the playbook:
```bash
ansible-playbook -i inventory/hosts.yml playbook.yml
```

---

### **5 Verify Deployment**
1. SSH into an instance and check Nginx status:
   ```bash
   ssh -i ~/.ssh/aws ubuntu@<PUBLIC_IP_1>
   systemctl status nginx
   ```
   Expected output: **Active (running)** 

2. Open a browser and visit:
   ```
   http://<PUBLIC_IP_1>
   http://<PUBLIC_IP_2>
   ```
   We should see the **system information webpage** like so:
   ![Screenshot 2025-02-21 155620](https://github.com/user-attachments/assets/22c9f84e-1ec9-43a7-867e-8e0e0ba836bf)


---

### **7 Destroying the Infrastructure**
To remove all AWS resources:
```bash
cd ~/4640-w7-lab-start-w25/terraform
terraform destroy -auto-approve
```
