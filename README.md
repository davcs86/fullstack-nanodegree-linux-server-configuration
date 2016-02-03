# fullstack-nanodegree-linux-server-configuration

# Criteria # 1: User management

### - Create a new user named grader and grant this user sudo permissions.

```bash
$> adduser grader
$> usermod -a -G sudo grader
```

### - Give ssh acccess to grader

Create a local key `grader_udacity_key.rsa`

```bash
$> ssh-keygen -t rsa
```

Upload the `grader_udacity_key.rsa.pub` to the server with the root key

```bash
$> cat ~/.ssh/grader_udacity_key.rsa.pub | ssh -i ~/.ssh/udacity_key.rsa root@54.69.180.83 "mkdir /home/grader/.ssh && cat >> /home/grader/.ssh/authorized_keys"
```

**Logout from root account, then use the grader account**

```bash
$> exit
$> ssh -i ~/.ssh/grader_udacity_key.rsa grader@54.69.180.83
```

# Criteria # 2: Security

### - Disable remote login to root

With grader account

```bash
$> sudo nano /etc/ssh/sshd_config
   edit the options
   Port to 2200
   and 
   PermitRootLogin to no
```
  
restart ssh service

```bash
$> sudo service ssh restart
```

### - Install updates

```bash
$> sudo apt-get update
$> sudo apt-get upgrade
   For automatic package updates, install cron-apt
$> sudo apt-get install cron-apt
```

### - UTC timezone (and synchronization with NTP)

```bash
$> sudo tzselect
   Select option 12) TZ - I want to specify the time zone using the Posix TZ format.
   Write 'UTC-0', enter
   Confirm the change with 1, enter
```
NTP synchronization
```bash
$> sudo apt-get install ntp
```  

### - UFW Firewall & Fail2Ban to block multiple unsuccessful login attempts.

UFW

```bash
$> sudo apt-get install ufw
   deny all incoming connections by default
$> sudo ufw default deny incoming
   allow all outgoing connections by default (optional)
$> sudo ufw default allow outgoing
   allow ssh, http and ntp services
$> sudo ufw allow 2200/tcp
$> sudo ufw allow 80/tcp
$> sudo ufw allow 123/tcp
   enable the ufw firewall
$> sudo ufw enable
```

Fail2Ban

```bash
$> sudo apt-get install fail2ban
   copy the default settings
$> sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
$> sudo nano /etc/fail2ban/jail.local
   edit the options
   backend to polling
   and in the [ssh] section
   port to 2200
$> sudo service fail2ban restart
$> sudo iptables -L
   restart
$> sudo reboot
```
