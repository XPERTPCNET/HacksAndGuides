# OpenSSH server and client on Windows 10 (powershell)

## Check to see if it is already installed:
``` 
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*' 
```
### This should be the output:
```
Name  : OpenSSH.Client~~~~0.0.1.0
State : NotPresent

Name  : OpenSSH.Server~~~~0.0.1.0
State : NotPresent
```

## Install the OpenSSH Client:
```
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```
## Install the OpenSSH Server:
```
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```