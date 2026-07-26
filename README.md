# Hullu Security Lab

Hullu is a purposely vulnerable Alpine Linux virtual machine for cybersecurity education, ethical hacking practice, and classroom lab scenarios. It includes vulnerable web applications and common network services so students can practice enumeration, exploitation concepts, privilege escalation, DNS configuration, and basic service hardening in an isolated environment.

Do not expose this VM to the internet. Use it only in a private lab network.

## Download

The Hullu OVA image is available from SourceForge:

https://sourceforge.net/projects/hullu/files/

Use the Hullu 3 image if you want the newer DNS practice features. Compared with Hullu 2, Hullu 3 adds BIND 9, the DNS Lab web interface, and separate service accounts for running Flask-based services instead of running them directly as root.

## Main VM Pages

Replace `<Hullu-IP>` with the current VM address.

```text
Hullu home page: http://<Hullu-IP>/
DVWA: http://<Hullu-IP>/dvwa/
FlaskVA: http://<Hullu-IP>:5000/
DNS Lab: http://<Hullu-IP>:5053/
phpMyAdmin: http://<Hullu-IP>/phpmyadmin/
```

If Kali is configured to use Hullu as its DNS server, the Hullu home page can also be reached with:

```text
http://hullu.lab/
```

Use the full `http://` address because some browsers may treat `hullu.lab` as a search term or try HTTPS first.

## Included Labs and Services

- Apache web server for the Hullu home page, DVWA, and phpMyAdmin
- DVWA for PHP web application security practice
- FlaskVA for Python/Flask web application security practice
- DNS Lab for BIND 9 configuration practice
- BIND 9 DNS server for the `hullu.lab` lab domain
- MariaDB/MySQL for database-backed web labs
- phpMyAdmin for database administration practice
- SSH for VM administration
- SFTP over SSH for encrypted file transfer
- FTP practice, optional/manual service
- Samba/SMB practice, optional/manual service

## Common Services and Ports

| Service | Protocol | Port | Default status | Purpose |
| --- | --- | --- | --- | --- |
| Apache | HTTP | `80/tcp` | Started | Hullu home page, DVWA, phpMyAdmin |
| HTTPS | HTTPS | `443/tcp` | Optional | TLS/web service testing if configured |
| SSH | SSH | `22/tcp` | Started | VM administration |
| SFTP | SFTP over SSH | `22/tcp` | Started with SSH | Encrypted VM file transfer |
| BIND | DNS | `53/tcp`, `53/udp` | Started | `hullu.lab` DNS practice |
| MariaDB/MySQL | MySQL | `3306/tcp` | Started | Database-backed labs |
| FlaskVA | HTTP | `5000/tcp` | Started | Flask vulnerable app |
| DNS Lab | HTTP | `5053/tcp` | Started | BIND config editor/practice page |
| FTP | FTP | `21/tcp` | Not started by default | File-transfer practice |
| Samba | SMB/NetBIOS | `139/tcp`, `445/tcp` | Not started by default | SMB share practice |

Check open ports from Kali:

```sh
nmap -sV -p21,22,53,80,139,443,445,3306,5000,5053 <Hullu-IP>
```

## DVWA

DVWA is the PHP vulnerable web application included in Hullu.

Repository:

https://github.com/digininja/DVWA

Default access:

```text
http://<Hullu-IP>/dvwa/
```

Default login:

```text
admin / password
```
## FlaskVA

FlaskVA is the Python/Flask vulnerable application included in Hullu.

Repository:

https://github.com/kaledaljebur/FlaskVA

Default access:

```text
http://<Hullu-IP>:5000/
```

Default login:

```text
admin / mustang
```

## DNS Lab

DNS Lab is a small Flask interface for practicing BIND configuration. It lets students edit selected DNS files, check the configuration, apply changes, and reset records when the VM IP changes.

Default access:

```text
http://<Hullu-IP>:5053/
```

Main DNS files:

```text
/etc/bind/named.conf
/var/bind/pri/hullu.lab.zone
/var/bind/pri/reverse.zone
```

If Kali uses Hullu as its DNS server, test the lab domain:

```sh
dig @<Hullu-IP> hullu.lab A
dig @<Hullu-IP> www.hullu.lab A
dig @<Hullu-IP> -x <Hullu-IP>
```

If Hullu is moved to a new network, first renew or configure the VM IP, then use DNS Lab:

```text
Reset to Current IP
Apply DNS
```

## IP Configuration

Show the current IP address:

```sh
ip addr show eth0
ip route
```

Renew DHCP on Alpine Linux:

```sh
udhcpc -i eth0
```

Restart networking:

```sh
rc-service networking restart
```

To use DHCP persistently, `/etc/network/interfaces` should contain something like:

```text
auto eth0
iface eth0 inet dhcp
```

For a static IP, edit `/etc/network/interfaces`:

```text
auto eth0
iface eth0 inet static
    address 192.168.8.137
    netmask 255.255.255.0
    gateway 192.168.8.1
```

Then restart networking or reboot:

```sh
rc-service networking restart
```

After changing the VM IP, open DNS Lab and run:

```text
Reset to Current IP
Apply DNS
```

## SFTP Practice

SFTP is available through SSH on port `22`. It is different from FTP on port `21` and does not need a separate FTP service.

Use SFTP from Kali:

```sh
sftp root@<Hullu-IP>
```

Copy a file to Hullu with SCP:

```sh
scp file.txt root@<Hullu-IP>:/tmp/
```

Copy a file from Hullu to Kali:

```sh
scp root@<Hullu-IP>:/etc/passwd ./passwd.copy
```

Use SFTP to discuss encrypted file transfer, SSH credentials, and the difference between SFTP and clear-text FTP.

## FTP Practice

FTP is useful for teaching file-transfer risks and service misconfiguration. There is no web page to control FTP in Hullu. Start and stop it from the terminal.

If FTP is not installed:

```sh
apk add vsftpd
```

Start FTP manually:

```sh
rc-service vsftpd start
```

Stop FTP:

```sh
rc-service vsftpd stop
```

Enable FTP at boot only if you want it available by default:

```sh
rc-update add vsftpd default
```

Suggested student activities:

- Enumerate FTP with `nmap`
- Test anonymous login if enabled
- Upload and download files in a controlled lab share
- Demonstrate why clear-text FTP credentials are unsafe
- Practice weak password testing in an isolated environment
- Compare FTP permissions with local filesystem permissions

Example checks from Kali:

```sh
nmap -sV -p21 <Hullu-IP>
ftp <Hullu-IP>
```

## Samba Practice

Samba is useful for teaching SMB enumeration, share permissions, and common file exposure mistakes. There is no web page to control Samba in Hullu. Start and stop it from the terminal.

If Samba is not installed:

```sh
apk add samba samba-common-tools
```

Start Samba manually:

```sh
rc-service samba start
```

Stop Samba:

```sh
rc-service samba stop
```

Enable Samba at boot only if you want it available by default:

```sh
rc-update add samba default
```

Suggested student activities:

- Enumerate SMB shares
- Test guest access
- Identify readable and writable shares
- Mount SMB shares from Kali
- Review how Linux permissions affect Samba access
- Discuss why exposed backups and config files are dangerous

Example checks from Kali:

```sh
nmap -sV -p139,445 <Hullu-IP>
smbclient -L //<Hullu-IP> -N
```

Mount a share from Kali when credentials or guest access are available:

```sh
sudo mount -t cifs //<Hullu-IP>/<share-name> /mnt -o guest
```

## Default Credentials

```text
DVWA: admin / password
FlaskVA: admin / mustang
phpMyAdmin: root / blank or configured password
MySQL CLI: root / blank or configured password
```

## Notes for Instructors

Hullu is designed for learning. Keep the VM in a NAT, host-only, or otherwise isolated lab network. Start optional services like FTP and Samba only when needed for a scenario. Reset the VM state between classes or assessment runs when needed.

## Contact

Email: kaled.aljebur@gmail.com

Project: https://github.com/kaledaljebur/hullu







