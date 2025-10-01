# Linux SysAdmin Labs Portfolio

This repository showcases my hands-on Linux system administration projects.  
Each lab demonstrates real troubleshooting and configuration tasks, and is documented with **objectives, steps, commands, and screenshots** to demonstrate practical skills in user management, permissions, SSH security, and troubleshooting.

---

## Labs
- [Lab 1: User & Permissions Management](#lab-1-linux-user--permissions-management)
- [Lab 2: Troubleshooting Shared Directory Permissions](#lab-2-troubleshooting-shared-directory-permissions)
- [Lab 3: SSH Hardening](#lab-3-secure-ssh-configuration)
- [Lab 4: Web Server Deployment & Permissions](#lab-4-web-server-deployment-and-permissions)
- [Lab 5: Automated Backups with Cron + Rsync](#Lab-5-Automated-Backups-with-Cron-and-Rsync)

---

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

-	`/srv/project`
-	Intended ownership: `root:devs`

**Permissions: 2770 (drwxrws---)**  
- `rwx` for owner (root)  
- `rws` for group (devs), including the setgid bit (‘s’)  
- `---` for others (permission denied)  

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
•	Owner: `root`  
•	Group: `root` ❌ (should be `devs`)  
•	Permissions: `rwx` for owner, `rws` for group  

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

The `s` in `drwxrws---` means the setgid bit is active. This ensures new files inherit the directory’s group `devs` rather than the user’s primary group.  
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
Both files belong to `devs`, allowing collaboration.
<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/4bafe203-02c8-4e05-bc3d-c57150803cdf" />


**Key Takeaways:**
- Always check both ownership and permissions when troubleshooting.  
- In shared directories, the setgid bit `chmod g+s` is crucial for group collaboration.  
- A single wrong group assignment can break access for all users.

---  

## Lab 3: Secure SSH Configuration  
**Objective:** Configure SSH for **key-based login only** disabling password and root login. Verify the setup works and practice safe recovery using the VirtualBox console.  

**Step 1 - Generate SSH Keypair (on Host Machine)**

On Windows (my host PC), opened Git Bash and ran:  
``` bash  
ssh-keygen -t ed25519
```  
- Pressed Enter to accept the default save location (C:\Users\<username>\.ssh\id_ed25519).
- Passphrase: Left blank for now.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9106b172-7877-4f99-ab34-e53b0502a66a" />

**Notes:**    
- `id_ed25519` → private key (keep secret).
- `id_ed25519.pub` → public key (shareable, to install on VM).

**Step 2 - Install Public Key on VM**  

*Note: NAT port forwarding set (2222 → 22) on the Hypervisor*  

Command (from host):  
``` bash  
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 joshuajohnson@127.0.0.1
```
- Prompted for password once.  
- Public key copied to `~/.ssh/authorized_keys`.  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/aee46651-312d-4ba2-b76e-373a50318d00" />

**Step 3 - Test Key-Based Login**  

On host machine, ran:  
``` bash
ssh -i ~/.ssh/id_ed25519 -p 2222 joshuajohnson@127.0.0.1
```
*Command Explained:*  
- `-i` → use the private key.
- `-p 2222` → connect through the forwarded port.  
- `joshuajohnson@127.0.0.1` → VM’s user + loopback.  

*Result:*  
It logged me in without asking for a password.  
In other words, ✅ key-based auth is working.  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e6e54cee-d6de-4501-b470-caa29c038ed7" />

**Step 4 - Harden SSH Config**  

*1. Backup the current config*  
ALways save a copy first:  
``` bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

*2. Edit the SSH Config*  
Open the file:  
``` bash
sudo nano /etc/ssh/sshd_config
```  

Find and set (or add if missing) these lines:  
```  
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no
PermitRootLogin no
```

*Notes:*  
- `PubkeyAuthentication yes` → allow key-based login.
- `PasswordAuthentication no` → disable password login.
- `ChallengeResponseAuthentication no` → disable alternate login prompts.
- `PermitRootLogin no` → root can’t log in directly over SSH.

Save and exit (`Ctrl+0`, `Enter`, `Ctrl+X`)  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cb8f2901-0f83-4deb-a865-e1d60bf7cbcf" />


*3. Check Syntax*  
``` bash
sudo sshd -t
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fa4704f5-4c93-4c51-9a26-ec0a7e628077" />


(no output = no syntax errors).

**Step 5 - Restart & Verify**
1. Restart SSH
``` bash
sudo systemctl restart ssh
```

2. Test Key Login (from host)
Opened a new terminal (current working session still open).
From host, tried:
``` bash
ssh -i ~/.ssh/id_ed25519 -p 2222 joshuajohnson@127.0.0.1
```  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9e9fb024-c093-4144-a8a6-01cf865cd56f" />  

**Success:**  
- Key login works.
- No password prompt

3. Test Password Login (should fail)  
Force password-only attempt:  
```  bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no -p 2222 joshuajohnson@127.0.0.1
```
- Should fail now (since password authentication has been disabled).  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d05070a3-5aa1-4e1a-997e-30e669ffa7fa" />  

**Step 6 - Recovery (if locked out)**  
From VirtualBoxConsole:  
``` bash
sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config
sudo systemctl restart ssh
```  

### Lessons Learned

- Key-based login is stronger and safer than passwords.
- Always back up configs before editing.
- SSHD applies the last occurrence of a setting, so clean duplicates to avoid confusion.
- Always test with a second terminal before restarting services.

---

## Lab 4: Web Server Deployment and Permissions
**Objectives:**   
- Install a web server (Nginx)
- Host a simple “Hello World” webpage.
- Break file permissions deliberately.
- Troubleshoot and fix so the web server works properly.

**Step 1 - Install Nginx**  
On the VM:  
``` bash
sudo apt update
sudo apt install -y nginx
```

Check if it's running  
``` bash
systemctl status nginx
```  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/07d296a4-c784-4238-94b0-26075d816803" />  

Expected: Active (running).  

Test in Browser:  
- Open your host browser and go to http://127.0.0.1:8080 (if you set up port forwarding host 8080 → guest 80).  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f09a698b-6013-44b1-bd61-c7c03d25fe15" />  

- Or just run inside VM:
``` bash
curl http://localhost
```  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3562585b-9890-4986-9067-1e39d5bbafcc" />  

**Step 2 - Deploy Custom Web Page**  

Replace the default index.html:
``` bash
echo "Hello from Lab 4 Web Server!" | sudo tee /var/www/html/index.html
```

Test:  
``` bash
curl http://localhost
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b7c4e22f-f5cc-45bd-ae1d-6e71ce99118d" />

**Step 3 - Break Permissions**  
A classic sysadmin error:  
```
sudo chown root:root /var/www/html/index.html
sudo chmod 600 /var/www/html/index.html
```
**Expected:** "403 Forbidden or something similar.  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5b2c4231-5ee6-424a-aa99-941e623a83cb" />  

**Step 4 - Fix Permissions**  
To fix:  
```bash
sudo chown www-data:www-data /var/www/html/index.html
sudo chmod 644 /var/www/html/index.html
```
**Why?**  
- Web server runs as user www-data (on Ubuntu/Debian).
- Needs read access to the file.  

Test Again:  
``` bash
curl http://localhost
```
Page should work again.  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/79ccaf19-059e-4b6d-acf4-6f7e2b23303b" />  

### Lessons Learned

- Nginx installation is straightforward (apt install nginx), but you must know how to verify service status (systemctl status nginx) and test using curl or a browser.
- The default “Welcome to Nginx” page can cause confusion — always check which index.html file is being served.
- Browsers and proxies can cache old content, so sometimes updates don’t appear until you force-refresh (Ctrl+F5) or adjust cache headers.
- File ownership and permissions matter:
- Wrong owner/permissions (e.g., root:root with 600) → Nginx can’t read → 403 Forbidden.
- Correct owner/permissions (www-data:www-data, 644) → page works again.
- In real-world jobs, updating web content means also managing caching, deployment consistency, and troubleshooting at multiple layers (browser, server, CDN).

---  

# Lab 5: Automated Backups with Cron and Rsync

## Objective
Create an automated backup system that:
- Copies files from a user’s home directory to a backup location.
- Uses `rsync` for efficiency.
- Runs automatically via `cron`.
- Demonstrates restoring files from the backup.

---

## Step 1: Prepare Backup Directory
**Commands:**
```bash
sudo mkdir -p /backups
sudo chown $USER:$USER /backups
```
**Notes:**  
- `/backups` will store user home backups.
- Ownership changed to current user so `rsync` can write.  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/92ca69fc-cfa3-44e0-b0dd-1d174b9d6895" />  

## Step 2: Manual RSync Backup  
**Command:**  
``` bash
rsync -avh --delete /home/joshuajohnson/ /backups/home_backup/
```
**Flags explained:**  
- `-a` → archive mode (preserves permissions, symlinks, timestamps).  
- `-v` → verbose output.  
- `-h` → human-readable.  
- `--delete` → keeps backup in sync by removing deleted files.  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ff8d1dbd-e229-453f-a532-da72c875af3f" />  

## Step 3: Automate with Cron
**Command:**
``` bash  
crontab -e
```
**Cron entry for daily backup at 2 AM:**  
``` bash
0 2 * * * rsync -avh --delete /home/joshuajohnson/ /backups/home_backup/
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bf2c1ad8-21f3-4d76-a4cf-114230e17d85" />  

Command to confirm:  
``` bash
crontab -l
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/11351097-7715-4a26-b888-cbd4f3e79f85" />  

## Step 4: Test Cron Job  

Run rsync manually to simulate the cron job:
``` bash
rsync -avh --delete /home/joshuajohnson/ /backups/home_backup/
```
Check Backup Folder:  
``` bash
ls -l /backups/home_backup/
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6ccc4d94-e66e-49e4-8b82-61793cf28f42" />  

## Step 5: Restore From Backup  

Simulate Data Loss:  
``` bash
rm ~/hello.txt
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e77915ae-ea91-41d2-ad54-4b0253708aac" />  


Restore from Backup:  
``` bash  
cp /backups/home_backup/hello.txt ~/
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f656ae9d-24b8-49bc-96b6-8b69fd34ebbc" />  

### Lessons Learned
- `rsync` is powerful for backups — efficient and preserves file properties.
- `cron` ensures automation: jobs run on schedule without manual intervention.
- Always verify both backup and restore to confirm data safety.
- In real-world use, combine with off-site backups (cloud or remote servers) for disaster recovery.




---  
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
