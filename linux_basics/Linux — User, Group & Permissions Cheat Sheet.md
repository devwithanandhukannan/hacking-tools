
# Linux Users & Groups Cheat Sheet

---

## 1. User Management

### Add a User
```bash
sudo useradd username                # Basic user
sudo useradd -m username             # With home directory
sudo useradd -m -s /bin/bash username # With shell
sudo useradd -u 1500 -g groupname username # With UID and primary group
````

### Set/Change Password

```bash
sudo passwd username                 # Set password for user
passwd                              # Change your own password
```

### Delete a User

```bash
sudo userdel username                # Delete user (keeps home)
sudo userdel -r username             # Delete user and home directory
```

### Modify a User

```bash
sudo usermod -l newname oldname      # Change username
sudo usermod -d /new/home username   # Change home directory
sudo usermod -s /bin/zsh username    # Change shell
sudo usermod -u 2001 username        # Change UID
sudo usermod -g groupname username   # Change primary group
sudo usermod -aG group1,group2 username # Add to secondary groups (append)
```

### Lock/Unlock User Account

```bash
sudo passwd -l username              # Lock account
sudo passwd -u username              # Unlock account
```

---

## 2. Group Management

### Add a Group

```bash
sudo groupadd groupname              # Create group
sudo groupadd -g 2000 groupname      # With specific GID
```

### Delete a Group

```bash
sudo groupdel groupname
```

### Add User to Group

```bash
sudo usermod -aG groupname username  # Add to secondary group(s)
sudo usermod -g groupname username   # Change primary group
```

### Remove User from Group

```bash
sudo gpasswd -d username groupname   # Remove user from group
```

### List Group Memberships

```bash
groups username                      # List groups for user
id username                          # Show UID, GID, and groups
```

---

## 3. Viewing and Editing User/Group Info

### View User Info

```bash
cat /etc/passwd                      # List all users
getent passwd username               # Info for specific user
```

### View Group Info

```bash
cat /etc/group                       # List all groups
getent group groupname               # Info for specific group
```

### Edit User/Group Files (Advanced)

```bash
sudo vipw                            # Safely edit /etc/passwd
sudo vigr                            # Safely edit /etc/group
```

---

## 4. File Permissions & Ownership

### Change File Owner/Group

```bash
sudo chown user file                 # Change owner
sudo chown user:group file           # Change owner and group
sudo chgrp group file                # Change group only
```

### Change File Permissions

```bash
chmod 755 file                       # rwxr-xr-x
chmod 644 file                       # rw-r--r--
chmod u+x file                       # Add execute for owner
chmod g-w file                       # Remove write for group
```

### Permission Numbers

| Number | Permission | rwx |
| ------ | ---------- | --- |
| 7      | rwx        | 111 |
| 6      | rw-        | 110 |
| 5      | r-x        | 101 |
| 4      | r--        | 100 |
| 3      | -wx        | 011 |
| 2      | -w-        | 010 |
| 1      | --x        | 001 |
| 0      | ---        | 000 |

---

## 5. Special Permissions

### Setuid, Setgid, Sticky Bit

```bash
chmod u+s file      # setuid (run as file owner)
chmod g+s file      # setgid (run as group)
chmod +t dir        # sticky bit (only owner/root can delete in dir)
```

* **Setuid**: `ls -l` shows `s` in owner: `-rwsr-xr-x`
* **Setgid**: `ls -l` shows `s` in group: `-rwxr-sr-x`
* **Sticky**: `ls -ld /tmp` shows `t`: `drwxrwxrwt`

### Sticky Bit Example

#### Add Sticky Bit (for this add letter 4) example normal user can change the password how? because it not access the /shadow file this is done by passwd command have a special permission to setup sudo to change the shadow file

```bash
sudo chmod +t /shared_folder
ls -ld /shared_folder
# drwxrwxrwt 2 root root 4096 Sep 30 12:00 /shared_folder
```

#### Remove Sticky Bit

```bash
sudo chmod -t /shared_folder
ls -ld /shared_folder
# drwxrwxr-x 2 root root 4096 Sep 30 12:00 /shared_folder
```

**Effect:** Only file/directory owner or root can delete files inside `/shared_folder`.

---

## 6. Private Groups

* **What?**: By default, each user gets a private group (same name as user).
* **Why?**: Prevents accidental sharing, isolates user files, simplifies permissions.

---

## 7. Permission Resolution Order

1. **Owner**: If you are the file owner, owner permissions apply.
2. **Group**: If you are in the file’s group, group permissions apply.
3. **Others**: If neither, "others" permissions apply.

---

## 8. Useful Commands

### Switch User

```bash
su - username                        # Switch to user (login shell)
sudo -i                              # Root shell
```

### List All Users/Groups

```bash
cut -d: -f1 /etc/passwd              # List usernames
cut -d: -f1 /etc/group               # List group names
```

### Find Files by Owner/Group

```bash
find / -user username                # Files owned by user
find / -group groupname              # Files owned by group
```

---

## 9. Example: Remove User from Group

```bash
sudo gpasswd -d bob developers
# Removes 'bob' from 'developers' group
```

---

## 10. Troubleshooting

* **User not in group after adding?**
  Log out and back in, or use `newgrp groupname` to refresh group membership.
* **Permission denied?**
  Check file owner/group with `ls -l`, and verify your user/group membership with `id`.

---

## 11. Summary Table

| Command                 | Description             |
| ----------------------- | ----------------------- |
| `useradd`               | Add user                |
| `userdel`               | Delete user             |
| `usermod`               | Modify user             |
| `groupadd`              | Add group               |
| `groupdel`              | Delete group            |
| `gpasswd -d user group` | Remove user from group  |
| `passwd`                | Set/change password     |
| `chown`                 | Change file owner/group |
| `chmod`                 | Change file permissions |
| `chgrp`                 | Change file group       |
| `id`                    | Show user/group IDs     |
| `groups`                | Show group memberships  |
| `chmod +t/-t dir`       | Add/remove sticky bit   |

---

**Tip:**
Always double-check group membership and file permissions with `id` and `ls -l`!

```

I can also make a **ready-to-download `.md` file** with this content if you want. Do you want me to do that?
```
