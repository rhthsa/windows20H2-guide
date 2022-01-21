
# 5 Windows node setup <a name="5"></a>

## 5.3 Setting Up the Windows Server <a name="5-3"></a>

After log on to windows node (server core) run command
```cmd
PowerShell
```

then run elevated Power shell
```PowerShell
Start-Process PowerShell -verb RunAs
```
The first step in setting up your Windows Node is to install the Docker runtime. I am running the following PowerShell script to install Docker:

```PowerShell
# Powershell script to install docker: https://bit.ly/3DzUdsP

#

# Install NuGet provider to avoid the manual confirmation

Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force

# configure repository policy

Set-PSRepository PSGallery -InstallationPolicy Trusted

# install module with provider

Install-Module -Name DockerMsftProvider -Repository PSGallery -Force

# install docker package

Install-Package -Name docker -ProviderName DockerMsftProvider -Force
```

Once you run the above script (or the individual commands), you will need to reboot the Windows node for docker to start working:

```PowerShell
Restart-Computer -Force
```
Once the Windows node is up and running, you should be able to run docker from the CLI:

```
PS C:\Users\Administrator\Documents> docker version

20.10.6
```

copied public key from helper node to windows node as windows_node.pub



then use this file for install ssh [install-ssh.ps1](install-ssh.ps1)

```PowerShell
.\install-ssh.ps1 .\windows_node.pub

Path      :

Online    : True

RestartNeeded : False


LastWriteTime : 8/31/2021 2:01:50 PM

Length    : 0

Name      : administrators_authorized_keys

```


Add firewall for Docker logs
```PowerShell
New-NetFirewallRule -DisplayName "ContainerLogsPort" -LocalPort 10250 -Enabled True -Direction Inbound -Protocol TCP -Action Allow -EdgeTraversalPolicy Allow
```

Finally, we must make sure that the hostname of the Windows node follows the RFC 1123 DNS label standard. In short:

Contain only lowercase alphanumeric characters or '-'.
Start with an alphanumeric character.
End with an alphanumeric character.
Another thing to note is that the WMCO does not like capital letters in the hostname. Set the hostname to the short DNS name. You will also need to restart the Windows node:

```PowerShell
PS C:\Users\Administrator> Rename-Computer -NewName "windows-node" -Force -Restart
```
