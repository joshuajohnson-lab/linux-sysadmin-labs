# Linux SysAdmin Labs Portfolio

This repository contains my hands-on Linux system administration projects.  
Each lab is documented with **objectives, steps, commands, and screenshots** to demonstrate practical skills in user management, permissions, SSH security, and troubleshooting.

---

## Lab 1: User & Group Permissions
**Objective:**  
Practice Linux user and group management with directories and file permissions.

**Tasks Completed:**
- Created users and groups.  
- Configured directory permissions for shared and private access.  
- Verified access and fixed "Permission Denied" issues.

**Key Commands Used:**

```bash  
useradd hannah  
groupadd devs  
usermod -aG devs hannah  
chmod 770 /srv/devproject  
```

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
    -	--- for others (permission denied)

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

**Step 2: Observe the Problem**  

When hannah tries to create a file:  
``` bash
sudo -u hannah touch /srv/project/test.txt  
```
**Result:**  
``` bash
touch: cannot touch '/srv/project/test.txt': Permission denied  
```
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
**Success:** Both files belong to devs, allowing collaboration.
