## PowerShell Autocompletion for Docker
* Open Powerhsell as Administrator
```
Set-ExecutionPolicy RemoteSigned
Install-Module -Scope CurrentUser posh-docker
Import-Module posh-docker
```

### Make module work after restarting powershell
- Open notepad from poweshell:
 ```
 notepad $PROFILE  
 ```
- Add line:
 ```
 Import-Module posh-docker
 ```
- Save File and restart powershell to test.

Now you should be able to  use [Tab]  for autocompletion of docker commands.