
## 🎯 What we will build

We will use **2 EC2 instances**:

```text
                 AWS
                  |
          -------------------
          |                 |
     EC2 - Ansible      EC2 - Managed
       Controller          Node
       Ubuntu              Ubuntu
          |
       Ansible
          |
       SSH
          |
          v
     EC2 - Managed
```

### Roles

| EC2       | Purpose                                        |
| --------- | ---------------------------------------------- |
| **EC2-1** | Ansible Controller — install Ansible here      |
| **EC2-2** | Managed Node — Ansible will manage this server |

> Ansible is normally installed only on the **Controller**. The managed node generally does **not** need Ansible installed.

---

# Step 1 — Create the first EC2: Ansible Controller

Go to **AWS Console → EC2 → Instances → Launch instance**.

### Name

```text
ansible-controller
```

### AMI

Choose:

```text
Ubuntu Server 24.04 LTS
```

### Instance type

For this lab:

```text
t3.micro
```

or another small instance available in your account/region.

### Key pair

Create/select a key pair, for example:

```text
ansible-lab-key
```

Download the `.pem` file and keep it safe.

### Network

For a simple lab, use your default VPC.

### Security Group

Create:

```text
ansible-lab-sg
```

Inbound rules:

| Type | Port | Source                 |
| ---- | ---: | ---------------------- |
| SSH  |   22 | My IP                  |
| HTTP |   80 | 0.0.0.0/0 *(optional)* |

For this initial Ansible lab, **SSH 22** is the important one.

Click **Launch instance**.

---

# Step 2 — Create the second EC2: Managed Node

Create another EC2 with almost the same configuration.

### Name

```text
ansible-node1
```

### AMI

```text
Ubuntu Server 24.04 LTS
```

### Instance type

```text
t3.micro
```

### Key pair

Use the **same key pair**:

```text
ansible-lab-key
```

### Security Group

You can use the same:

```text
ansible-lab-sg
```

Launch it.

Now you should have:

```text
ansible-controller
       |
       |
ansible-node1
```

---

# Step 3 — Connect to the Controller

From your Windows machine, open **PowerShell**.

Go to the directory where your `.pem` file exists.

For example:

```powershell
cd Downloads
```

Connect to the controller.

Get the **Public IPv4 address** of `ansible-controller` from AWS.

Example:

```bash
ssh -i ansible-lab-key.pem ubuntu@<CONTROLLER-PUBLIC-IP>
```

For example:

```bash
ssh -i ansible-lab-key.pem ubuntu@13.234.XX.XX
```

If prompted:

```text
Are you sure you want to continue connecting?
```

Type:

```text
yes
```

You should now see something similar to:

```text
ubuntu@ip-172-31-10-20:~$
```

🎉 You are inside the Controller.

---

# Step 4 — Update Linux

Run:

```bash
sudo apt update
```

Then:

```bash
sudo apt upgrade -y
```

Check Ubuntu:

```bash
cat /etc/os-release
```

---

# Step 5 — Install Ansible

Run:

```bash
sudo apt install ansible -y
```

Wait for the installation to complete.

Check the version:

```bash
ansible --version
```

You should see something similar to:

```text
ansible [core ...]
  python version = ...
  jinja version = ...
```

🎉 **Ansible is installed.**

---

# Step 6 — Understand the important concept

At this point:

```text
Windows Laptop
       |
       | SSH
       v
Ansible Controller
       |
       | Ansible + SSH
       v
Managed Node
```

The Controller needs to be able to SSH into the Managed Node.

So now we need to configure SSH.

---

# Step 7 — Get the Managed Node's Private IP

Go to:

**AWS Console → EC2 → Instances → ansible-node1**

Copy its:

```text
Private IPv4 address
```

For example:

```text
172.31.20.50
```

Because both EC2 instances are in the same VPC, the Controller can communicate with the Managed Node using its private IP.

---

# Step 8 — Put the SSH Key on the Controller

Your Controller needs the private key that allows it to connect to `ansible-node1`.

On your **Windows machine**, you have:

```text
ansible-lab-key.pem
```

You need to copy it to the Controller.

One simple way is `scp` from PowerShell:

```powershell
scp -i ansible-lab-key.pem ansible-lab-key.pem ubuntu@<CONTROLLER-PUBLIC-IP>:/home/ubuntu/
```

Example:

```powershell
scp -i ansible-lab-key.pem ansible-lab-key.pem ubuntu@13.234.XX.XX:/home/ubuntu/
```

Now SSH into the Controller again:

```powershell
ssh -i ansible-lab-key.pem ubuntu@<CONTROLLER-PUBLIC-IP>
```

Check:

```bash
ls
```

You should see:

```text
ansible-lab-key.pem
```

---

# Step 9 — Protect the Private Key

Very important.

Run:

```bash
chmod 400 ansible-lab-key.pem
```

Check:

```bash
ls -l ansible-lab-key.pem
```

You should see permissions similar to:

```text
-r-------- ansible-lab-key.pem
```

---

# Step 10 — Test SSH from Controller → Managed Node

From the Controller:

```bash
ssh -i ansible-lab-key.pem ubuntu@<MANAGED-NODE-PRIVATE-IP>
```

Example:

```bash
ssh -i ansible-lab-key.pem ubuntu@172.31.20.50
```

If everything is correct, you should enter the second EC2:

```text
ubuntu@ip-172-31-20-50:~$
```

Check:

```bash
hostname
```

You should get something like:

```text
ip-172-31-20-50
```

Exit:

```bash
exit
```

You should return to:

```text
ubuntu@ansible-controller:~$
```

### ⭐ This is an important milestone

You have proven:

```text
Controller
    |
    | SSH works
    v
Managed Node
```

---

# Step 11 — Create an Ansible Inventory

On the Controller:

```bash
mkdir ansible-lab
cd ansible-lab
```

Create an inventory file:

```bash
nano inventory
```

Add:

```ini
[webservers]
node1 ansible_host=172.31.20.50 ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/ansible-lab-key.pem
```

Replace:

```text
172.31.20.50
```

with your Managed Node's actual **private IP**.

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

---

# Step 12 — Understand the Inventory

This:

```ini
[webservers]
```

means we created a group called:

```text
webservers
```

This:

```ini
node1
```

is the name we gave the server.

This:

```ini
ansible_host=172.31.20.50
```

tells Ansible where the server is.

This:

```ini
ansible_user=ubuntu
```

tells Ansible to use the Ubuntu user.

And:

```ini
ansible_ssh_private_key_file=/home/ubuntu/ansible-lab-key.pem
```

tells Ansible which SSH key to use.

So conceptually:

```text
Inventory
   |
   +--- webservers
           |
           +--- node1
                  |
                  +--- Private IP
                  +--- ubuntu user
                  +--- SSH key
```

---

# Step 13 — Check Inventory

Run:

```bash
ansible-inventory -i inventory --list
```

Ansible should display information about `node1`.

You can also run:

```bash
ansible-inventory -i inventory --graph
```

You should see:

```text
@all:
  |--@ungrouped:
  |--@webservers:
      |--node1
```

---

# Step 14 — The Most Important Ansible Test

Now run:

```bash
ansible all -i inventory -m ping
```

You should get:

```text
node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

🎉🎉🎉

This means:

```text
Ansible Controller
       |
       | SSH
       |
       v
Managed Node
       |
       v
Ansible Module
       |
       v
       PONG
```

---

# Step 15 — Understand `ansible all -i inventory -m ping`

This command looks complicated at first.

Break it down:

```bash
ansible
```

Run Ansible.

```bash
all
```

Target all hosts in the inventory.

```bash
-i inventory
```

Use the `inventory` file.

```bash
-m ping
```

Use the Ansible `ping` module.

So:

```bash
ansible all -i inventory -m ping
```

means:

> "Ansible, read my inventory, connect to all servers, and use the ping module to check whether you can communicate with them."

---

# Step 16 — Run Your First Real Command

Now let's execute a Linux command on the Managed Node.

```bash
ansible all -i inventory -m command -a "hostname"
```

Expected:

```text
node1 | CHANGED | rc=0 >>
ip-172-31-20-50
```

Try:

```bash
ansible all -i inventory -m command -a "uptime"
```

And:

```bash
ansible all -i inventory -m command -a "df -h"
```

And:

```bash
ansible all -i inventory -m command -a "free -h"
```

You are now remotely managing Linux using Ansible.

---

# Step 17 — Run an Ad-Hoc Command

For example:

```bash
ansible all -i inventory -m shell -a "whoami"
```

Output:

```text
node1 | CHANGED | rc=0 >>
ubuntu
```

Another example:

```bash
ansible all -i inventory -m shell -a "cat /etc/os-release"
```

---

# Step 18 — Install Nginx Using Ansible

Now we will do something more realistic.

Run:

```bash
ansible all -i inventory -m apt -a "name=nginx state=present" --become
```

Here:

```text
apt
```

is the Ansible module for Debian/Ubuntu package management.

```text
name=nginx
```

means install nginx.

```text
state=present
```

means nginx should be installed.

```text
--become
```

means use privilege escalation, similar to:

```bash
sudo
```

Check:

```bash
ansible all -i inventory -m command -a "systemctl status nginx" --become
```

---

# Step 19 — Verify Nginx

From the Controller:

```bash
ansible all -i inventory -m command -a "curl localhost"
```

You should receive the nginx HTML response.

You can also open the Managed Node's **Public IPv4 address** in your browser:

```text
http://<MANAGED-NODE-PUBLIC-IP>
```

You should see:

```text
Welcome to nginx!
```

🎉

---

# Step 20 — Your First Ansible Architecture

At this point your lab looks like:

```text
                    AWS
                     |
          +----------+----------+
          |                     |
          v                     v
   EC2 Controller          EC2 Managed Node
      Ubuntu                   Ubuntu
          |                     |
      Ansible                  Nginx
          |
          | SSH
          |
          +-------------------->
```

And your workflow is:

```text
Create EC2
    ↓
Install Ansible
    ↓
Configure SSH
    ↓
Create Inventory
    ↓
Test Ansible Ping
    ↓
Run Ad-Hoc Commands
    ↓
Install Nginx
    ↓
Manage Server
```

## 🎓 What I recommend you learn next

Don't jump directly into complex YAML. For a **first-time Ansible student**, continue in this order:

1. **Ansible architecture**
2. **Inventory**
3. **Ad-hoc commands**
4. **Modules**
5. **Variables**
6. **Facts**
7. **Playbook**
8. **Tasks**
9. **Handlers**
10. **Conditionals**
11. **Loops**
12. **Templates**
13. **Copy / File modules**
14. **Roles**
15. **Ansible Vault**
16. **Ansible with AWS**
17. **Ansible + Jenkins CI/CD**

The next natural hands-on is: **create your first `nginx.yml` Playbook and deploy/configure Nginx using YAML instead of ad-hoc commands.**


If you're using **Windows PowerShell** and your `.pem` file is in the **Downloads** folder, run:

```powershell
cd Downloads
```

Or:

```powershell
cd $HOME\Downloads
```

### Check that you're in the Downloads folder

```powershell
pwd
```

Then:

```powershell
dir
```

You should see your key file, for example:

```text
ansible-lab-key.pem
```

### If you're currently connected to EC2

If your prompt looks like:

```text
ubuntu@ip-172-31-10-20:~$
```

you're **inside the Linux EC2**, not Windows.

First exit:

```bash
exit
```

You should return to something like:

```text
PS C:\Users\YourName>
```

Then:

```powershell
cd Downloads
```

Now you can run the `scp` command to copy your `.pem` file to the Ansible controller.


# Step 12 — Understand Ansible Inventory

First, remember:

> **Inventory = list of servers that Ansible needs to manage.**

For example, suppose AWS has these EC2 instances:

```text
AWS
│
├── ansible-controller
│
├── web-server
│
└── database-server
```

Ansible needs to know:

* Which server?
* What name should Ansible use for it?
* What IP address?
* Which Linux user?
* Which SSH key?

We provide this information in the **inventory file**.

---

# 12.1 Create an inventory folder

On your **Ansible Controller**, run:

```bash
mkdir ~/ansible-lab
```

Then:

```bash
cd ~/ansible-lab
```

Check:

```bash
pwd
```

You should get something like:

```text
/home/ubuntu/ansible-lab
```

---

# 12.2 Create the inventory file

Run:

```bash
nano inventory
```

Put this inside:

```ini
[webservers]
web01 ansible_host=172.31.20.50 ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/ansible-lab-key.pem
```

Of course, replace:

```text
172.31.20.50
```

with your **Managed Node's private IP**.

---

# 12.3 What is `web01`?

This is important.

Look at:

```ini
web01 ansible_host=172.31.20.50
```

Here:

```text
web01
```

is the **name you give to the server inside Ansible**.

It does **not** have to be the actual AWS EC2 instance name.

For example, your AWS EC2 may be named:

```text
ansible-node1
```

but inside Ansible you can call it:

```text
web01
```

So:

```text
AWS name:
ansible-node1

Ansible name:
web01

Private IP:
172.31.20.50
```

Ansible uses `web01` as the friendly name.

---

# 12.4 Can I use any name?

Yes.

For example:

```ini
web01
```

or:

```ini
web-server
```

or:

```ini
production-web
```

or:

```ini
myserver
```

You can choose a meaningful name.

For example, if you have three web servers:

```ini
[webservers]
web01
web02
web03
```

This is much easier to understand than remembering IP addresses.

---

# 12.5 What does `[webservers]` mean?

This:

```ini
[webservers]
```

is a **group**.

Think of it like a folder containing servers.

For example:

```ini
[webservers]
web01
web02
web03
```

means:

```text
webservers
   |
   +-- web01
   +-- web02
   +-- web03
```

Then you can tell Ansible:

> Run this task on all webservers.

For example:

```bash
ansible webservers -i inventory -m ping
```

Ansible will contact:

```text
web01
web02
web03
```

---

# 12.6 Understanding the complete line

Let's break this:

```ini
web01 ansible_host=172.31.20.50 ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/ansible-lab-key.pem
```

### Part 1 — `web01`

```text
web01
```

This is the **Ansible hostname/alias**.

You choose this name.

---

### Part 2 — `ansible_host`

```text
ansible_host=172.31.20.50
```

This tells Ansible:

> When you want to contact `web01`, actually connect to this IP address.

So:

```text
web01
   ↓
172.31.20.50
```

---

### Part 3 — `ansible_user`

```text
ansible_user=ubuntu
```

This tells Ansible:

> Use the `ubuntu` Linux user when connecting.

So Ansible effectively connects like:

```bash
ssh ubuntu@172.31.20.50
```

---

### Part 4 — SSH private key

```ini
ansible_ssh_private_key_file=/home/ubuntu/ansible-lab-key.pem
```

This tells Ansible:

> Use this private key to authenticate to the server.

So conceptually Ansible is doing:

```bash
ssh -i /home/ubuntu/ansible-lab-key.pem ubuntu@172.31.20.50
```

---

# 12.7 Your inventory now looks like this

```ini
[webservers]

web01 ansible_host=172.31.20.50 ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/ansible-lab-key.pem
```

Think of it as:

```text
                 INVENTORY
                     |
             [webservers]
                     |
                   web01
                     |
          +----------+----------+
          |          |          |
         IP         User       Key
          |          |          |
     172.31.20.50  ubuntu   ansible-lab-key.pem
```

---

# Step 13 — Save the inventory

If you're using `nano`:

```text
Ctrl + O
```

Press:

```text
Enter
```

Then:

```text
Ctrl + X
```

---

# Step 14 — Verify the inventory

Run:

```bash
ansible-inventory -i inventory --list
```

You should see information about:

```text
web01
```

You can also use:

```bash
ansible-inventory -i inventory --graph
```

Expected:

```text
@all:
  |--@ungrouped:
  |--@webservers:
      |--web01
```

This tells you:

```text
all
 |
 +-- webservers
       |
       +-- web01
```

---

# Step 15 — Test `web01`

Now run:

```bash
ansible web01 -i inventory -m ping
```

Expected:

```text
web01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Notice something important.

We used:

```bash
ansible web01
```

not:

```bash
ansible 172.31.20.50
```

Because we gave the server the Ansible name:

```text
web01
```

---

# Step 16 — Test using the group name

Instead of:

```bash
ansible web01 -i inventory -m ping
```

you can use:

```bash
ansible webservers -i inventory -m ping
```

Because:

```text
webservers
   |
   +-- web01
```

Ansible will execute the command against `web01`.

---

# Step 17 — What if you have multiple servers?

Suppose you create:

```text
EC2
│
├── web01
├── web02
├── app01
└── db01
```

Your inventory could be:

```ini
[webservers]
web01 ansible_host=172.31.20.50 ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/ansible-lab-key.pem
web02 ansible_host=172.31.20.51 ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/ansible-lab-key.pem

[appservers]
app01 ansible_host=172.31.20.52 ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/ansible-lab-key.pem

[dbservers]
db01 ansible_host=172.31.20.53 ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/ansible-lab-key.pem
```

Now your inventory represents:

```text
                    ALL
                     |
       +-------------+-------------+
       |             |             |
   webservers    appservers    dbservers
       |             |             |
   +---+---+         |             |
   |       |         |             |
 web01   web02      app01         db01
```

You can then run:

### Only web servers

```bash
ansible webservers -i inventory -m ping
```

### Only application servers

```bash
ansible appservers -i inventory -m ping
```

### Only database servers

```bash
ansible dbservers -i inventory -m ping
```

### Everything

```bash
ansible all -i inventory -m ping
```

---

# ⭐ One important point for your current lab

Since you're just starting, **don't create multiple servers yet**.

Keep your inventory simple:

```ini
[webservers]
web01 ansible_host=<MANAGED-NODE-PRIVATE-IP> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/ansible-lab-key.pem
```

Then test:

```bash
ansible web01 -i inventory -m ping
```

If you get:

```text
web01 | SUCCESS
```

your **Controller → Managed Node Ansible connection is working**.

After that, the next step should be **your first `nginx.yml` Playbook**, where instead of manually running commands, you tell Ansible in YAML:

```text
Connect to web01
      ↓
Install Nginx
      ↓
Start Nginx
      ↓
Enable Nginx
      ↓
Verify Nginx
```

That is where Ansible starts becoming really useful.
