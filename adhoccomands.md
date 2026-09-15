
## 1. What are Ad Hoc Commands?

In Ansible, an **ad hoc command** is used when you want to perform a quick task on managed servers.

### Simple example

Suppose you have 3 servers:

```text
             Ansible Control Node
                    |
          ansible command
                    |
       +------------+------------+
       |            |            |
    Server-1     Server-2     Server-3
```

You want to check whether all servers are reachable.

Instead of creating a playbook, simply run:

```bash
ansible all -m ping
```

That's an **ad hoc command**.

---

# 2. Basic Syntax

```bash
ansible <host-pattern> -m <module> -a "<arguments>"
```

For example:

```bash
ansible all -m command -a "uptime"
```

### Understand the command

| Part       | Meaning                  |
| ---------- | ------------------------ |
| `ansible`  | Ansible command          |
| `all`      | All servers in inventory |
| `-m`       | Module                   |
| `command`  | Module to execute        |
| `-a`       | Arguments                |
| `"uptime"` | Command to execute       |

---

# 3. Hands-on Lab Setup

Let's assume your inventory is:

```ini
[webservers]
web1 ansible_host=192.168.1.101
web2 ansible_host=192.168.1.102

[dbservers]
db1 ansible_host=192.168.1.103
```

Test connectivity first:

```bash
ansible all -m ping
```

Expected:

```text
web1 | SUCCESS => {
    "ping": "pong"
}

web2 | SUCCESS => {
    "ping": "pong"
}

db1 | SUCCESS => {
    "ping": "pong"
}
```

---

# 4. Most Important Ad Hoc Commands

For your students, I recommend focusing on these modules:

```text
ping
command
shell
user
setup
apt
yum
file
copy
service
```

The first 8 are especially important for beginners.

---

## 5. `ping` Module

### Use case

Check whether Ansible can connect to managed servers.

```bash
ansible all -m ping
```

### Real-time use case

Before deploying an application to 50 servers, you want to make sure Ansible can communicate with them.

```bash
ansible all -m ping
```

### Hands-on

**Task:** Check connectivity to only web servers.

```bash
ansible webservers -m ping
```

Check only DB server:

```bash
ansible dbservers -m ping
```

---

# 6. `command` Module

Used to execute Linux commands.

### Example

```bash
ansible all -m command -a "uptime"
```

Another:

```bash
ansible all -m command -a "hostname"
```

Check disk:

```bash
ansible all -m command -a "df -h"
```

Check memory:

```bash
ansible all -m command -a "free -m"
```

### Real-time use case

Operations team receives an alert:

> "Server disk usage is high."

Instead of SSHing into every server:

```bash
ansible all -m command -a "df -h"
```

You can check all servers at once.

### Hands-on

Ask students to perform:

```bash
ansible all -m command -a "hostname"
ansible all -m command -a "uptime"
ansible all -m command -a "df -h"
ansible all -m command -a "free -m"
```

---

# 7. `shell` Module

Similar to `command`, but supports shell features such as:

* pipes `|`
* redirects `>`
* variables
* shell operators

### Example

```bash
ansible all -m shell -a "ps -ef | grep nginx"
```

Because `|` is a shell feature, use `shell`.

### Another example

```bash
ansible all -m shell -a "cat /etc/passwd | grep ubuntu"
```

### Real-time use case

You want to find whether a particular application process is running:

```bash
ansible webservers -m shell -a "ps -ef | grep java"
```

### Hands-on

Run:

```bash
ansible all -m shell -a "hostname | tr 'a-z' 'A-Z'"
```

Then:

```bash
ansible all -m shell -a "df -h | grep /"
```

### Important teaching point

Don't teach students to use `shell` for everything.

Prefer:

```bash
command
```

when a normal command is sufficient.

Use:

```bash
shell
```

when you actually need shell functionality.

---

# 8. `user` Module

Used to create, modify or remove Linux users.

### Create user

```bash
ansible all -m user -a "name=devops"
```

Check:

```bash
ansible all -m command -a "id devops"
```

### Create user with shell

```bash
ansible all -m user -a "name=devops shell=/bin/bash"
```

### Remove user

```bash
ansible all -m user -a "name=devops state=absent"
```

### Real-time use case

Your company has 20 Linux servers.

A new employee joins.

Instead of manually creating the account on every server:

```bash
ansible all -m user -a "name=developer state=present"
```

One command creates the user across all servers.

### Hands-on

Students should:

```bash
ansible webservers -m user -a "name=devops state=present"
```

Verify:

```bash
ansible webservers -m command -a "id devops"
```

Remove:

```bash
ansible webservers -m user -a "name=devops state=absent"
```

---

# 9. `setup` Module

The `setup` module collects information about the server.

These are called **Ansible Facts**.

```bash
ansible all -m setup
```

This produces a lot of information.

Instead, use filters.

### OS information

```bash
ansible all -m setup -a "filter=ansible_distribution"
```

### IP address

```bash
ansible all -m setup -a "filter=ansible_default_ipv4"
```

### Memory

```bash
ansible all -m setup -a "filter=ansible_memory_mb"
```

### Hostname

```bash
ansible all -m setup -a "filter=ansible_hostname"
```

### Real-time use case

Suppose you have:

```text
100 servers
```

and you need to identify which servers are running Ubuntu.

Instead of logging into every server:

```bash
ansible all -m setup -a "filter=ansible_distribution"
```

---

# 10. `apt` Module

Used mainly for Debian/Ubuntu systems.

### Install Nginx

```bash
ansible webservers -m apt -a "name=nginx state=present" --become
```

### Remove Nginx

```bash
ansible webservers -m apt -a "name=nginx state=absent" --become
```

### Update package cache

```bash
ansible all -m apt -a "update_cache=yes" --become
```

### Install Git

```bash
ansible all -m apt -a "name=git state=present" --become
```

### Real-time use case

You have 30 Ubuntu servers and need Git installed on all of them.

```bash
ansible all -m apt -a "name=git state=present" --become
```

No need to SSH into 30 servers.

---

# 11. `yum` Module

Used mainly with RHEL/CentOS/Amazon Linux versions that use Yum.

### Install Git

```bash
ansible all -m yum -a "name=git state=present" --become
```

### Install Nginx

```bash
ansible webservers -m yum -a "name=nginx state=present" --become
```

### Remove Git

```bash
ansible all -m yum -a "name=git state=absent" --become
```

### Real-time use case

You have 50 RHEL servers and need to install a security package.

```bash
ansible all -m yum -a "name=<package-name> state=present" --become
```

---

# 12. `file` Module

Used to manage:

* files
* directories
* permissions
* ownership
* symbolic links

### Create directory

```bash
ansible all -m file -a "path=/opt/app state=directory"
```

### Create empty file

```bash
ansible all -m file -a "path=/opt/app/test.txt state=touch"
```

### Delete file

```bash
ansible all -m file -a "path=/opt/app/test.txt state=absent"
```

### Change permissions

```bash
ansible all -m file -a "path=/opt/app/test.txt mode=0644"
```

### Change ownership

```bash
ansible all -m file -a "path=/opt/app/test.txt owner=devops group=devops"
```

### Real-time use case

Before deploying an application:

```text
/opt/myapp
/opt/myapp/logs
/opt/myapp/config
```

You can create these directories remotely.

---

# 13. `copy` Module

Used to copy files from the **Ansible Control Node → Managed Node**.

Suppose your control node has:

```text
/home/ansible/index.html
```

Copy it to the web server:

```bash
ansible webservers -m copy \
-a "src=/home/ansible/index.html dest=/var/www/html/index.html" \
--become
```

### Copy with permissions

```bash
ansible webservers -m copy \
-a "src=index.html dest=/var/www/html/index.html mode=0644" \
--become
```

### Real-time use case

You have a configuration file:

```text
nginx.conf
```

and need to distribute it to 50 servers.

Instead of manually copying:

```bash
ansible webservers -m copy \
-a "src=nginx.conf dest=/etc/nginx/nginx.conf" \
--become
```

---

# 14. `service` Module

Very useful in real DevOps operations.

### Start Nginx

```bash
ansible webservers -m service -a "name=nginx state=started" --become
```

### Stop

```bash
ansible webservers -m service -a "name=nginx state=stopped" --become
```

### Restart

```bash
ansible webservers -m service -a "name=nginx state=restarted" --become
```

### Enable at boot

```bash
ansible webservers -m service \
-a "name=nginx enabled=yes" \
--become
```

---

# 15. Complete Beginner Hands-on

I would give your students this exercise.

### Step 1 — Test servers

```bash
ansible all -m ping
```

### Step 2 — Find hostname

```bash
ansible all -m command -a "hostname"
```

### Step 3 — Check uptime

```bash
ansible all -m command -a "uptime"
```

### Step 4 — Check OS

```bash
ansible all -m setup -a "filter=ansible_distribution"
```

### Step 5 — Create user

```bash
ansible webservers -m user -a "name=devops state=present"
```

### Step 6 — Create directory

```bash
ansible webservers -m file \
-a "path=/opt/myapp state=directory"
```

### Step 7 — Create file

```bash
ansible webservers -m file \
-a "path=/opt/myapp/app.txt state=touch"
```

### Step 8 — Copy file

```bash
ansible webservers -m copy \
-a "src=app.txt dest=/opt/myapp/app.txt"
```

### Step 9 — Install Nginx

For Ubuntu:

```bash
ansible webservers -m apt \
-a "name=nginx state=present" --become
```

### Step 10 — Start Nginx

```bash
ansible webservers -m service \
-a "name=nginx state=started" --become
```

### Step 11 — Verify

```bash
ansible webservers -m shell -a "systemctl status nginx --no-pager"
```

---

# 16. One Real-Time Scenario for Students

Tell students this story:

> **"Imagine you are a DevOps engineer and your company has 20 Linux web servers. The manager asks you to install Nginx, create an application directory, copy the configuration file and start Nginx on all servers."**

Without Ansible:

```text
SSH Server 1
SSH Server 2
SSH Server 3
...
SSH Server 20
```

With Ansible:

```text
                 Ansible
                    |
        +-----------+-----------+
        |           |           |
      Web-1       Web-2       Web-20
        |           |           |
      Nginx       Nginx       Nginx
```

Commands:

```bash
ansible webservers -m apt -a "name=nginx state=present" --become

ansible webservers -m file \
-a "path=/opt/myapp state=directory"

ansible webservers -m copy \
-a "src=app.conf dest=/opt/myapp/app.conf"

ansible webservers -m service \
-a "name=nginx state=started" --become
```

**Then explain the important limitation:**

> Ad hoc commands are excellent for **quick, one-time operational tasks**.
> When the task becomes repeatable, complex, or needs version control, use an **Ansible Playbook**.

### What I would emphasize in your class

| Module    | Student should understand         | Practical importance |
| --------- | --------------------------------- | -------------------: |
| `ping`    | Connectivity                      |                ⭐⭐⭐⭐⭐ |
| `command` | Execute simple Linux command      |                ⭐⭐⭐⭐⭐ |
| `shell`   | Execute shell commands/pipes      |                 ⭐⭐⭐⭐ |
| `user`    | Manage Linux users                |                 ⭐⭐⭐⭐ |
| `setup`   | Collect server facts              |                 ⭐⭐⭐⭐ |
| `apt`     | Ubuntu/Debian packages            |                ⭐⭐⭐⭐⭐ |
| `yum`     | RHEL/CentOS/Amazon Linux packages |                ⭐⭐⭐⭐⭐ |
| `file`    | Files/directories/permissions     |                ⭐⭐⭐⭐⭐ |
| `copy`    | Copy configuration/files          |                ⭐⭐⭐⭐⭐ |
| `service` | Start/stop/restart services       |                ⭐⭐⭐⭐⭐ |

This gives students a **hands-on foundation before you move into Inventory → Variables → Playbooks → Handlers → Roles**.
