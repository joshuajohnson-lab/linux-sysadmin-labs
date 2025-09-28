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

useradd hannah  
groupadd devs  
usermod -aG devs hannah  
chmod 770 /srv/devproject  
