# ACIT2420 A3P1 - Tutorial: Setup nginx server on a Debian 12 server

# Starting from a Fresh Debian 12 server on DigitalOcean

First, create a new droplet in your DigitalOcean project if you don’t have one already. If you don’t know what settings to use, follow with these configurations:

- Debian
- Version 12 x64
- Basic plan
- Premium AMD
- $7 per month

Assuming you have an SSH key set and all the SSH configurations completed, we can connect to the server as the root user using this `ssh` command in your terminal:

where `<path-to-ssh-key>` and `<ip-address>` are your SSH key and droplet IP address.

```bash
ssh -i <path-to-ssh-key> root@<ip-address>
```

> The `-i` option specifies your ****************identity**************** which uses your SSH key to login to the droplet.
> 

You should now be connected to your new droplet server as the root user via ssh.

---

# Creating a new regular user

Before we create our nginx server, we want to create a new regular user with administrator privileges to perform all of our tasks. By not using the root user to complete our tutorial, we can see which user has been using the server to do certain tasks. If anything goes wrong in the system, we can see who is responsible for those issues.

## Creating a new user with bash as the login shell

To create a new user with bash as their default command line shell, default configurations, and a home directory, use the following command:

where <user-name> is the name you want the new user to have.

```bash
useradd -ms /bin/bash <user-name>
```

> The `-m` option creates a home directory for the new user with the default configuration files in `/etc`, if one does not exist.
> 

> The `-s` option sets the user’s login shell to `/bin/bash` as their default.
> 

To set/change the password for the user:

```bash
passwd <user-name>
```

You should now have a new user with bash as their default login shell and home directory with default configurations.

## Performing administrative tasks

To give the user admin privileges, add the new user to the **sudo** group to let them use '`sudo`' privileges:

```bash
usermod -aG sudo <user-name>
```

> The `-a` option appends the specified user into the `sudo` group. If this option is not used, the pre-existing list of users in the group will be replaced with the specified user(s) in the command.
> 

> The `-G` option allows you to specify multiple groups to add the user in. In this case, it is not strictly necessary.
> 

You should now have sudo/admin privileges when using your new user.

## Accessing the server via SSH

To access the droplet server as a regular user via SSH, you need to recursively copy the SSH files from the root user to the home directory of the new user:

```bash
sudo cp -r /root/.ssh /home/<user-name>
```

> The `-r` option means recursive or copying everything within the source to the destination.
> 

You will also need to recursively change the ownership of copied SSH files to the new user:

```bash
sudo chown -R <user-name>:<user-name> /home/<user-name>/.ssh/
```

> The `-R` option means recursive for the `chown` command. This is capitalized because its `-r` option means something else..
> 

Now the new user can SSH into the server from the host terminal with:

```bash
ssh -i .\\.ssh\\do-key <user-name>@164.92.68.144
```

You should now be able to access the server via SSH with your newly created user.

## Revoking the root user’s SSH access to the server

To prevent anyone from using the root user via SSH, we need to disable root login (via SSH) by modifying `/etc/ssh/sshd_config` :

```bash
sudo vim /etc/ssh/sshd_config
```

Navigate to line 33 and change `PermitRootLogin` from ********“yes”******** to ************“no”************.
The updated file should look like this:

```bash
# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/bin:/usr/bin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin yes
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem       sftp    /usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
ClientAliveInterval 120
```

Restart the SSH service to apply the new changes by doing the following:

```bash
sudo systemctl restart ssh.service
```

Now this message will show if the root login is used for SSH:

```bash
root@<ip-address>: Permission denied (publickey).
```

You should now not be able to SSH into the server using the root user.

---

# Installing nginx

To get a web server running with nginx, we need to check for updates before installing new packages with:

```bash
sudo apt update
```

Then install nginx with:

```bash
sudo apt install nginx
```

The service should now be running on your server automatically. You can check if the server is running by using this command: `systemctl status nginx.service` .

---

# Configuring nginx to serve a sample website

To configure nginx to host a sample website, create a new directory in `/var/www` with an `index.html` file inside with your HTML contents:

```bash
sudo mkdir /var/www/<new-site>
sudo vim /var/www/<new-site>/index.html
```

<aside>
💡 **For example:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420 A3P1</title>
</head>
<body>
    <p>Hello, World</p>
</body>
</html>
```

</aside>

Delete the existing default config file from the `/etc/nginx/sites-available` directory to prevent errors when running a new site on the same port (as the default would):

```bash
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
```

Create a configuration file to get nginx to run the new site’s directory files:

```bash
sudo vim /etc/nginx/sites-available/<new-site>.conf
```

and paste the following:

```bash
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	root /var/www/<new-site>;

	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}
}
```

Now the new site is available but has yet to be enabled. To enable it, create a symbolic link of the conf file from `/etc/nginx/sites-available` in `/etc/nginx/sites-enabled` by:

```bash
sudo ln -s /etc/nginx/sites-available/<new-site>.conf /etc/nginx/sites-enabled/<new-site>.conf
```

> The `-s` option means “**symbolic**”. A symbolic link acts as a shortcut to the parent file and will change if the parent file changes.
> 

Test the configuration by using:

```bash
sudo nginx -t
```

> The `-t` option means “**test**” and is used to troubleshoot the configuration file.
> 

If there are no errors, these messages should show:

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

and restart the nginx service to apply the newly enabled website:

```bash
sudo systemctl restart nginx.service
```

You can use `curl` to make a request from the host to the new site that has been made on the server with:

where the `<ip-address>` is the same IP address as your droplet.

```bash
curl <ip-address>
```

You should now be able to send requests to your web server, hosted by nginx, and get your sample HTML file as a response.