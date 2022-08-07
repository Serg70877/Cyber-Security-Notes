# Cyber Security Notes

## Installed Tools
- OpenVPN
- nmap
- smbclient
- Sublime Text
- redis-tools
- gobuster

## Brute Forcing
### Tools
#### Z3
- Z3 is a theorem prover from Microsoft Research that makes it very easy to brute force questions by specifing rules.
- Can be accessed through Python bindings.

## Vulnerability Scanning
### Port 21
Usually associated with ftp
- anonymous can occasionally be logged into without a password
- Use telnet command

### Port 21
Usually associated with telnet
- root can occasionally be logged into without a password
- Use ftp command

### Ports 445 and 143
These ports are associated with file sharing (SMB) and SQL Server
- Sensitive info tend to be stored in these areas
- Use smbclient to show available shares with -L and access them
- WorkShares can occasionally be logged into without a password
- E.g. smbclient //127.0.0.1/SHARE$

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
