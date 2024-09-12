# Commands for CTA Waiver Test

Use docker to create a mock ssh-server by building `Dockerfile.ssh-server` for practice.

## Users & Group Management

- Creating a user with
    - Default Home Directory
    - Bash login shell
    - Secondary Groups
    - Password
```bash
useradd -m -s /bin/bash -G <groups> <user>
passwd <user>
```

- Creating a group
```bash
groupadd <group>
```

- Deleting a user along with home directory
```bash
userdel -rf <user>
```

- Deleting a group even if its primary 
```bash
groupdel -f <group>
```

- Managing users in groups
    - Modify the `/etc/group` file directly OR
```bash
usermod -aG <group> <user>
gpasswd -<a|d> <user> <group>
``` 

## File & Directory Permissions

```bash
chmod 400 <file|dir>
chown -R <user>:<group> <file|dir>
```

## Zip & Tar Operations

```bash
zip -r <zipfile.zip> <path/to/dir>
unzip <zipfile.zip>

tar -czf <archive.tar.gz> <path/to/dir>
tar -xvf <archive.tar.gz> -C <path/to/dir> 
```

## System Management

```bash
# To list all running processes
## UNIX Syntax
ps -ef
## BSD Syntax
ps aux

# Filter by user
ps -u <user>

# Filter by process name
ps -C <process-name>

# To Kill processes
kill -9 <PID>
```

## SCP

```bash
scp -r <user>@<host>:<remote-path> local-path
scp -r local-path <user>@<host>:<remote-path>
```

## Su & Sudo

- Running commands as other users
```bash
su <user> -c "<command>"
sudo -u <user> <command>
```

- Sudoers File
```bash
# On ALL hosts -> As ALL users:groups -> ALL commands 
root ALL=(ALL:ALL) ALL
%sudo ALL=(ALL:ALL) ALL

# Same as above but without password
root ALL=NOPASSWD: ALL

# Deny certain commands
%sudo ALL=(ALL:ALL) ALL, !<path/to/command>
```

## IP Addresses & Opened Ports

```bash
<netstat|ss> -tulpn
```

## Install/remove software packages

```bash
apt <update|install|remove|> <package>
apt list --installed
apt list -qq <package>
```

# Start/Stop Services

```bash
service <service> <start|restart|status|stop>

systemctl <start|restart|status|stop> <service>
```

## Docker

```bash
docker <pull|run|push> <image:tag>

docker commit <ctr-id> <image:tag>
```

## Cron

- First check if cron is running
```bash
service cron/d status
service cron/d start
```

- Configuration
```bash
## Using crontab command

# To list cronjobs of user
crontab -l
# To edit cronjobs
crontab -e
* * * * * </path/to/script>

## Using /etc/crontab (System-Wide)

* * * * * <user> <path/to/script>
```

# Iptables

- If running in Docker containers we need to add 2 flags in the `docker run` command to have iptables working properly inside the container.

```bash
docker run --cap-add=NET_ADMIN --cap-add=NET_RAW .....
```

- Iptables are a collection of tables -> chains -> rules -> policies. There are 3 chains - INPUT (for incoming connections), OUTPUT (for outgoing connections) and FORWARD (for routing packets). 
    - Rules can be added to chains and are applied one by one. If any rule matches the appropriate policy is applied and the remaining rules are skipped. For example, the chain shown below will block ssh requests coming from source IP since the first rule in the chain has DROP policy.
```bash
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
DROP       tcp  --  172.17.0.3           anywhere             tcp dpt:ssh
ACCEPT     tcp  --  172.17.0.3           anywhere             tcp dpt:ssh
```

- Configuration:
```bash
# Block all packets for this port on all interfaces.
iptables -A INPUT -p tcp --dport <port> -j DROP
 
# Interface specific
iptables -A INPUT -i <interface> -p tcp --dport <port> -j DROP
 
### only drop port for given IP or Subnet ##
iptables -A INPUT -i <interface> -p tcp --dport <port> -s <ip/cidr> -j DROP
```

# Openssl

- To request a new certificate signed by newly generated RSA key in a single command
```bash
# can use -nodes to avoid passphrase and leave the RSA key unencrypted, but this will not create the CSR which is fine for a self-signed certificate
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365
```

- In steps - Create an RSA key -> Create a CSR -> Create the certificate
```bash
openssl genrsa -aes256 -out domain.key 2048

openssl req -key domain.key -new -out domain.csr

openssl x509 -signkey domain.key -in domain.csr -req -days 365 -out domain.crt
```
- Host the certificate using openssl
```bash
openssl s_server -key key.pem -cert cert.pem -accept <port> -www
```

- Verify TLS using openssl
```bash
openssl s_client -connect localhost:<port>
```
- Openssl Rand Module

```bash
openssl rand <format> <number>

openssl rand -<base64|hex> 32
```

- Openssl Enc Module

```bash
openssl enc -<cipher> -in plaintext.txt -out encrypted.enc

openssl enc -<cipher> -d -in encrypted.enc -out decrypted.txt
```

- Openssl sha256 module, its same as sha256sum command

```bash
openssl sha256 <file> -out <outfile>

# This signs the hash of myfile.txt using private_key.pem and saves the signature to signature.bin
openssl sha256 -sign private_key.pem -out signature.bin myfile.txt

# If the signature is valid, OpenSSL will confirm it.
openssl sha256 -verify public_key.pem -signature signature.bin myfile.txt
```












