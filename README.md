# Linux SysAdmin Labs Portfolio

This repository showcases my hands-on Linux system administration projects.  
Each lab demonstrates real troubleshooting and configuration tasks, and is documented with **objectives, steps, commands, and screenshots** to demonstrate practical skills in user management, permissions, SSH security, and troubleshooting.

---

# Table of Contents:  
**1. Lab 1: Linux User & Permissions Management**  
**2. Lab 2: Troubleshooting Shared Directory Permissions**  
**3. Lab 3: Secure SSH Configuration**  

## Lab 1: Linux User & Permissions Management  

**Objective:** Practice creating and managing Linux users, groups, and permissions.  

**Setup**
- Ubuntu 22.04 VM on VirtualBox  
- Commands run as root user  

**Steps Performed:**  
1. Created users:  
``` bash
sudo adduser hannah  
sudo adduser johnny  
sudo adduser Charlie  
```
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/6d5c6a7e-4c55-4bbe-a29f-60845ecb46b1" />
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/0e0580e8-3f6b-400f-a165-e37f1fe84e33" />

2.	Created group:
``` bash 
sudo groupadd devs
```
3.	Added users to the group devs (leaving charlie out):
``` bash
sudo usermod -aG devs hannah
sudo usermod -aG devs johnny
```
4.	Created a shared Project directory for the group devs:
``` bash
sudo mkdir /srv/project
sudo chown :devs /srv/project
sudo chmod 2770 /srv/project
```
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/2f932970-6520-4b33-b826-044ee701ff72" />

5.	Test Access:  

a) As hannah (allowed)
``` bash
su - hannah
cd /srv/project
touch file_from_hannah.txt
ls -l
exit
```  
b) As Charlie (denied)
``` bash
su - charlie
cd /srv/project
```
c) As johnny (allowed)
``` bash
su - johnny
cd /srv/project
touch file_from_johnny.txt
ls -l
```
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/c3dea09d-ff80-4f6a-be6a-6aefc70b8d00" />
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/eb2bffbd-f5f5-43a6-a650-2dadcbf8bb6b" />
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/dcfbc9e6-28df-42ff-bcc5-d19add219118" />  

**Results**  
- hannah and johnny can read/write inside project.  
- Other users (charlie) cannot access (Permission denied).  

**Issues & Fixes**
- Issue: Forgot to add user to group → couldn’t access folder.  
- Fix: Ran usermod -aG developers hannah.  

**Learning**
- Understood how Linux permissions (rwx) work with groups.  
- Practiced chmod, chown, usermod.  

---

## Lab 2: Troubleshooting Shared Directory Permissions
**Objective:** Practice diagnosing and fixing permission issues in a shared group directory used by multiple users.

### Setup ###  
**Users:**
- hannah (UID 1001, member of devs)  
- johnny (UID 1002, member of devs)  

**Shared directory:**

-	/srv/project
-	Intended ownership: root:devs

**Permissions: 2770 (drwxrws---)**  
- rwx for owner (root)  
- rws for group (devs), including the setgid bit (‘s’)  
- --- for others (permission denied)  

**Step 1: Break the Setup**  

We intentionally introduced a problem:  
``` bash
sudo chown root:root /srv/project  
```
Now the directory ownership is:  
``` bash
drwxrws--- 2 root root 4096 Sep 25 12:31 /srv/project  
```
**Effect:** only root owns the directory, and the devs group cannot access it.  
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/281abec0-1db2-458e-b496-fc20bec8bb55" />  


**Step 2: Observe the Problem**  

When hannah tries to create a file:  
``` bash
sudo -u hannah touch /srv/project/test.txt  
```
**Result:**  
``` bash
touch: cannot touch '/srv/project/test.txt': Permission denied  
```
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/9fcf28a5-35f1-498a-8afd-a9fa490e7a8b" />  


**Step 3: Diagnose**  
**Check directory details:**
``` bash
ls -ld /srv/project
```  

**Output:**
``` bash
drwxrws--- 2 root root 4096 Sep 25 12:31 /srv/project
```
•	Owner: root  
•	Group: root ❌ (should be devs)  
•	Permissions: rwx for owner, rws for group  

**Conclusion:** The wrong group ownership is blocking access.
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/01a77348-3268-407b-8044-10a17ddcd54b" />  


**Step 4: Fix**
	**Change the group back to devs:**
``` bash
sudo chown :devs /srv/project
```
**Confirm:**
``` bash
ls -ld /srv/project
```
**Now shows:**
```
drwxrws--- 2 root devs 4096 Sep 25 12:31 /srv/project
```
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/e0a5f3ae-68fd-4bfe-ada5-0265fad4016c" />  


**Step 5: Test**
	**Try again as hannah:**
``` bash
sudo -u hannah touch /srv/project/test.txt  
ls -l /srv/project  
```  
**Output:**
``` bash
-rw-rw-r-- 1 hannah devs 0 Sep 25 14:00 test.txt
```
**Success:** file created inside are owned by hannah:devs.
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/36bcb5f3-1a61-466d-a829-14e2c14d1f23" />  


**Step 6: Verify Setgid Behaviour**  

The ‘s’ in drwxrws--- means the setgid bit is active. This ensures new files inherit the directory’s group (devs) rather than the user’s primary group.  
Example: if johnny creates a file:  
``` bash
sudo -u johnny touch /srv/project/notes.md  
ls -l /srv/project  
```
**Output:**  
``` bash
-rw-rw-r-- 1 hannah devs  0 Sep 25 14:00 test.txt
-rw-rw-r-- 1 johnny devs  0 Sep 25 14:05 notes.md
```
Both files belong to devs, allowing collaboration.
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/4bafe203-02c8-4e05-bc3d-c57150803cdf" />


**Key Takeaways:**
- Always check both ownership and permissions when troubleshooting.  
- In shared directories, the setgid bit (chmod g+s) is crucial for group collaboration.  
- A single wrong group assignment can break access for all users.

---  

## Lab 3: Secure SSH Configuration (In Progress)

**Planned Tasks:**
- Generate and use SSH keypair (`ed25519`) for authentication  
- Install public key on VM with `ssh-copy-id`  
- Test key-only login via port forwarding  
- Harden `sshd_config` (disable passwords and root login)  
- Safe recovery via VirtualBox console if locked out  

**Status:** Steps 1–2 completed. Hardening config and recovery tests in progress.  

---

## Skills Demonstrated
- Linux user and group management  
- File and directory permissions (chmod, chown, setgid)  
- Shared directory troubleshooting  
- SSH key-based authentication  
- Secure server configuration (sshd_config)  
- System troubleshooting and recovery  

---

**Author:** Joshua Johnson  
**LinkedIn:** https://www.linkedin.com/in/s-joshua-johnson-7b71a715a/  
**Portfolio Repo:** You are here!
