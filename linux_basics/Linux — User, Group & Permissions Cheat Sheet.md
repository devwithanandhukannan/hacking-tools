# Linux — User, Group & Permissions Cheat Sheet

> Compact reference in **.md** format covering user accounts, group accounts, passwords, permissions, ownership, and package installation commands.

---

## USER ACCOUNTS

* UID ranges (common convention):

  * `0` = `root` (superuser)
  * `1–999` = system accounts
  * `1000–65000` = standard (regular) user accounts

### Examples / Commands

```bash
# Create users
useradd hacker      # low-level user creation (may need additional flags)
adduser james       # friendlier interactive user creation

# View user accounts
cat /etc/passwd     # displays user account records (username:x:UID:GID:...)
```

> **Note:** `cat` prints file contents to the terminal.

---

## GROUP ACCOUNTS

### Commands

```bash
# Create a group
groupadd hr

# Show groups
cat /etc/group      # displays group records (groupname:x:GID:member1,member2,...)

# Add an existing user to a group (append; -a requires -G)
usermod -a -G <groupname> <username>
# Example:
usermod -a -G hr james
```

### Task examples from your notes

```bash
# Create a group named 'sales'
groupadd sales

# Create a user named 'rcb' and add to sales
adduser rcb         # or: useradd rcb && passwd rcb
usermod -a -G sales rcb
```

---

## PASSWORD & AUTH FILES

* Windows local account hashes stored (path shown): `C:\Windows\System32\config\SAM`
* Linux password hashes: `/etc/shadow` (only readable by root)

```bash
# Set or change password for a user
passwd rcb

# View shadow file (root only)
cat /etc/shadow
```

> If an account has no password configured, it may be represented as `!= NO PASSWORD IS CONFIGURED` in notes — actual representation varies (e.g., `*` or `!` in `/etc/shadow`).

---

## SWITCH USER

```bash
su kali              # switch to user 'kali' (you will be prompted for password)
```

---

## PERMISSIONS (r/w/x)

* `r` = read
* `w` = write
* `x` = execute

Sample long-style listing: `drwxr-xr-x` (d = directory)

### Numeric (octal) values

* `r` = 4
* `w` = 2
* `x` = 1

Examples:

* `rw-` = 4 + 2 = `6`
* `r-x` = 4 + 1 = `5`
* `rwx` = 4 + 2 + 1 = `7`
* `r--` = `4`
* `-w-` = `2`
* `--x` = `1`
* `-wx` = 2 + 1 = `3`

### Converting and examples

```bash
# Example mode calculations from notes
rw--wxr--   -> owner: rw- (6), group: -wx (3), other: r-- (4)  => 634
--xrwxr-x   -> owner: --x (1), group: rwx (7), other: r-x (5)  => 175

# Apply permission
chmod 634 test
```

---

## PRACTICAL DIRECTORY TASKS (from your notes)

```bash
# Create directories and set permissions
# 1) Create directory test99 and set permission rw-r----x (owner=6, group=4, other=1) -> 641
mkdir test99 && chmod 641 test99

# 2) Create directory test98 and set permission rwx----wx -> rwx (7), --- (0), -wx (3) => 703
mkdir test98 && chmod 703 test98

# 3) Create directory test92 and set permission rwx--x-wx
#    Breakdown: owner=rwx (7), group=--x (1), other=-wx (3) => 713
mkdir test92 && chmod 713 test92

# 4) Create directory test91 and set permission r--rwx--x
#    Breakdown: owner=r-- (4), group=rwx (7), other=--x (1) => 471
mkdir test91 && chmod 471 test91
```

> Double-check the numeric values before applying if these are critical — mistakes can lock out access.

---

## UGO (User/Group/Other)

* `u` = owner (user)
* `g` = group
* `o` = other (everyone else)

Symbols to modify: `+` (add), `-` (remove), `=` (set exactly)

```bash
# Examples
chown <username> <dir>       # change owner of <dir>
chgrp <groupname> <dir>      # change group of <dir>

# Create user 'raj', group 'dev', create dir test89 and set owner/group
adduser raj
groupadd dev
mkdir test89
chown raj test89
chgrp dev test89
# or in one command:
chown raj:dev test89
```

---

## PACKAGE / APPLICATION INSTALLATION (APT)

Common `apt` commands (Debian/Ubuntu-based systems):

```bash
sudo apt update            # refresh package lists
apt list --installed       # list installed packages (or: apt list)
apt search vlc             # search for package 'vlc'
sudo apt install vlc      # install vlc
sudo apt remove vlc       # remove package but keep config files
sudo apt purge vlc        # remove package and purge config files
```

---

## QUICK CHECKLIST (Tasks requested in your notes)

1. Create group `sales` and user `rcb`, add `rcb` to `sales`:

```bash
groupadd sales
adduser rcb
usermod -a -G sales rcb
```

2. Create user `raj` and group `dev`, create directory `test89` with owner `raj` and group `dev`:

```bash
adduser raj
groupadd dev
mkdir test89
chown raj:dev test89
```

3. Create directories and set permissions (examples provided above).

---

## NOTES & BEST PRACTICES

* `adduser` is interactive and usually preferred for creating human users (it creates home directory, sets shell, prompts for password).
* `useradd` is lower-level; use with flags like `-m` to create home directory, `-s` to set shell, etc.
* Always run `passwd <username>` to set a password for a new account.
* Use `sudo` when running administrative commands as non-root.
* Be careful when using `chmod` and `chown` on system directories — incorrect ownership/permissions can break services.

---

*Generated: 29/05*
