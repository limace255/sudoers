Role Name
=========

This role permits to deploy all that concerns 'sudo' :

It does :
- install 'sudo' if needed
- create folders to backup old sudo files
- remove useless files (that are not managed anymore)
- backup main + additional sudoers files
- deploy main sudoers file (very common, in order to avoid any risk)
- deploy global additional sudoers files
- deploy host specific additional sudoers files


Requirements
------------

Nothing is required

Role Variables
--------------

Variables for main sudoers file (/etc/sudoers) are in vars/main.yml.
After 5 trials user is kicked.

```yaml
sudo_pkg_mgr_opts: update_cache=yes

sudo_files:
  - filename: "main_sudo"
    defaults:
      - defaults: env_reset
      - defaults: mail_badpass
      - defaults: badpass_message="Wrong password, try again while russian roulette is clicking..."
      - defaults: passwd_tries=5
      - defaults: secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
      - defaults: logfile="/var/log/sudo.log"
    users:
      - name: "root"
      - name: "%admin"
      - name: "%sudo"
        nopasswd: yes
      - name: "%hiddens"
        nopasswd: yes
    blocks:
      - block: "#includedir /etc/sudoers.d"
```

In default.yml tha files related to package and paths; also subfiles and other files removal
```yaml
---
# defaults file for sudoers

# package name (version)
sudo_package: "sudo"

main_sudoers_file: "/etc/sudoers"
sudo_sudoers_d_path: "/etc/sudoers.d"
#sudoers_backup: True
sudoers_backup_path: "/etc/sudoers-backups"
sudoers_visudo_path: "/usr/sbin/visudo"

sudo_sudoers_user: "root"
sudo_sudoers_group: "root"
sudo_sudoers_mode: "0440"

# If we have to include subdirs
create_sudoers_subdir: true
# additionnal sudoers.d files
sudo_sub_files: []
# list of username or %groupname
sudo_users: []
# list of username or %groupname and their defaults
sudo_defaults: []
# delete other files in /etc/sudoers.d that we do not manage
purge_other_sudoers_files: true
```

All variables for additional sudoers files are in Inventory (group_vars and host_vars)

Dependencies
------------


Example Playbook
----------------

```yaml
- hosts: all
  become: yes
  gather_facts: true
  roles:
    - sudoers

# Main sudo file (/etc/sudoers) is already created
Variables (example for global) :
# We want a specific sudo file for admins
# We create host alias (LXC containers) + user alias (ops team) + cmnd alias (pagers + LXC commands)
# Then we grant privileges for user alias to run cmnd aliases on host alias
privileges:
  - filename: "sudo_admins"
    aliases:
      - host:
        - name: "LXC"
          hosts: "lxc-1, lxc-2", lxc-3"
      - user:
        - name: "OPS"
          users: "op-1, op-2, op-3, op-4"
      - cmnd:
        - name: "PAGERS"
          cmnds: "/bin/more, /usr/bin/pg, /usr/bin/less, /bin/cat, /usr/bin/tail"
        - name: "LXCOMMANDS"
          cmnds: "/usr/bin/lxc list, /usr/bin/lxc cluster list, /usr/bin/lxc launch*, /usr/bin/lxc info*, /usr/bin/lxc image*, /usr/bin/lxc shell*"
    users:
      - name: "OPS"
        nopasswd: yes
        commands: 'LXCOMMANDS'
      - name: "OPS"
        nopasswd: yes
        commands: 'PAGERS'
# Example of backup users management
  - filename: "backup_admin"
    aliases:
      - user:
        - name: "BACKUPERS"
          users: "backup-web, backup-DB, backup-file, replication"
      - cmnd:
        - name: "BACKUPCOMMANDS"
          cmnds: "/usr/bin/rsync*, /usr/bin/borgbackup*, /usr/bin/borg*, /usr/bin/ssh*, /usr/bin/chmod*, /usr/bin/chown*, /usr/bin/setfacl*"
    users:
      - name: backup-web
        nopasswd: yes
        commands: "/bin/bash /home/backup-user/scripts/*.sh"
      - name: BACKUPERS
        nopasswd: yes
        commands: BACKUPCOMMANDS
# Another example with monitoring tool privileges
  - filename: "sudo_zabbix"
    blocks:
      - block: "Defaults!SMARTCTL !logfile, !syslog, !pam_session"
      - block: "Defaults!SMARTCTL_DISCOVERY !logfile, !syslog, !pam_session"
    aliases:
      - cmnd:
        - name: "SMARTCTL"

# We might want a specific privileges on a particular host
more_privileges:
  - filename: "sudo_jenkinx"
    users:
      - name: "jenkins"
        nopasswd: yes
        commands:
        - /bin/bash /path/of/a/particular/script.sh
```

License
-------

BSD

Author Information
------------------


