Ubuntu Server
=============
The steps taken to setup a Ubuntu web server.

By: <script src="https://gist.github.com/WillSquire/6ff67d0d94e2dfe20fca.js"></script>
Update
------
Update Aptitude:
```
sudo apt-get update
```

Users
-----
Add user/s:
```
adduser parent
```

Check user was created by listing out all users:
```
awk -F: '{ print $1 }' /etc/passwd
```

Add user to sudo group (**IF USER IS SUPERUSER/ROOT APPROPRIATE**):
```
gpasswd -a parent sudo
```

Check the groups the user belongs to with:
```
groups parent
```

Switch to a user with:
```
su - parent
```

SSH User
--------
If you have just switched user, you should be in the user home directory (i.e. `/home/parent`). Now make a .ssh directory in that location:
```
mkdir .ssh
```

Change permission of .ssh to something more secure:
```
chmod 700 .ssh
```

Create an authorized_keys file with the below command and then paste in each SSH RSA public key on a separate line in the file:
```
nano .ssh/authorized_keys
```

Change the permissions now on the authorized_keys file itself:
```
chmod 600 .ssh/authorized_keys
```

End the SSH session by putting in the following twice:
```
exit
```

Login with the new user to ensure it works:
```
ssh -i /Users/parent/.ssh/id_rsa_parent_will parent@178.62.84.199
```

SSH Configuration
-----------------
Open the SSH configuration:
```
sudo nano /etc/ssh/sshd_config
```

Change port to non-default port (i.e. `222` instead of `22`) to reduce attack attempts:
```
Port 222
```

If another sudo user was setup to use instead of root, disable root access to the server via SSH by changing PermitRootLogin to:
```
PermitRootLogin no
```

Restart the SSH service with:
```
sudo service ssh restart
```

Next check the config change was successful by opening another terminal window and logging in via the new SSH settings BEFORE logging out with:
```
ssh -i /Users/parent/.ssh/id_rsa_parent_will -p 222 parent@178.62.84.199
```

Firewall
--------
Using ufw (although iptables are better and nftables will be even better than that once released), allow the open ports before enabling the firewall. 

Open SSH port on a custom port (**DO NOT ENABLE IF THIS MIGHT BE INCORRECT**):
```
sudo ufw allow 222/tcp
```

Open HTTP port on default port:
```
sudo ufw allow 80/tcp
```

Open SSL/TLS port on default port:
```
sudo ufw allow 443/tcp
```

Open SMTP port on default port:
```
sudo ufw allow 25/tcp
```

Review port settings with:
```
sudo ufw show added
```

Enable firewall with:
```
sudo ufw enable
```

Timezone
--------
Configure the timezone by following the onscreen instructions with:
```
sudo dpkg-reconfigure tzdata
```

Swapfile
--------
“Adding "swap" to a Linux server allows the system to move the less frequently accessed information of a running program from RAM to a location on disk”

Generally, make the swap file size double the amount of server RAM (i.e. 1gb if 512mb RAM). Allocate this with:
```
sudo fallocate -l 1G /swapfile
```

Change the permissions of this area with:
```
sudo chmod 600 /swapfile
```

Format for swapfile with:
```
sudo mkswap /swapfile
```

Get the system to use the swapfile with:
```
sudo swapon /swapfile
```

And get the system to use the swap file automatically on boot with:
```
sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'
```

FTP
---
Install VSFTPD (very secure FTP):
```
sudo apt-get install vsftpd
```

Open VSFTPD configuration:
```
sudo nano /etc/vsftpd.conf
```

Uncommented/change the values of the following settings to the following value:
```
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
```

Restart VSFTPD:
```
sudo service vsftpd restart
```

Fail2Ban
--------
Install Fail2Ban:
```
sudo apt-get install fail2ban
```

Copy default Fail2Ban config:
```
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Configure Fail2Ban to ignore your IP. Change the following to your IP:
```
ignoreip = 62.232.90.202
```

Configure Fail2Ban to send email alerts by entering the destination email, sender email (i.e. site name) and change action to send emails with log lines on ban:
```
destemail = digital.infrastructure@madebyparent.com
sender = Fail2Ban@localhost
action = %(action_mwl)s
```

If using custom ports (i.e. custom SSH port), change port from ‘ssh’ to custom port number:
```
port    = 222
```

Restart Fail2Ban:
```
sudo service fail2ban restart
```

OR, stop and start with:
```
sudo service fail2ban stop
sudo service fail2ban start
```

Server image to be reused.

Useful links:  
https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04  
https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers  
https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server  
https://www.digitalocean.com/community/tutorials/how-to-set-up-vsftpd-on-ubuntu-12-04  
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04  
https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-12-04  
