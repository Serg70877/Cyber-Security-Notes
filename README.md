# Cyber Security Notes

## Vulnerability Scanning
### Ports 445 and 143
These ports are associated with file sharing (SMB) and SQL Server
- Sensitive info tend to be stored in these areas
- Use smbclient to show available shares and access them
- E.g. smbclient //127.0.0.1/SHARE$
