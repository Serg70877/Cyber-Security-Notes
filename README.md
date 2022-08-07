# Cyber Security Notes

## Installed Tools
- OpenVPN
- nmap
- Sublime Text

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
- Use smbclient to show available shares and access them
- E.g. smbclient //127.0.0.1/SHARE$

## Database
### Accessing Microsoft SQL Server
- pymssql can be used to connect to Microsoft SQL Servers.
- Goal is to use xp_cmdshell to gain RCE.

## Useful URLS
- https://github.com/swisskyrepo/PayloadsAllTheThings
- https://github.com/undergroundwires/CEH-in-bullet-points
