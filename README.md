# Cyber Security Notes

## Brute Forcing
### Tools
#### Z3
- Z3 is a theorem prover from Microsoft Research that makes it very easy to brute force questions by specifing rules.

## Vulnerability Scanning
### Ports 445 and 143
These ports are associated with file sharing (SMB) and SQL Server
- Sensitive info tend to be stored in these areas
- Use smbclient to show available shares and access them
- E.g. smbclient //127.0.0.1/SHARE$
