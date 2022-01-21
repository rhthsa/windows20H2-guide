
# 5 Windows node setup <a name="5"></a>
## 5.1 Windows node Prerequisite <a name="5-1"></a>
Any Windows instances that are to be attached to the cluster as a node must fulfill the following requirements:

- The Docker container runtime must be installed on the instance.
- The instance must be on the same network as the Linux worker nodes in the cluster.
- Port 22 must be open and running an SSH server.
The default shell for the SSH server must be the Windows Command shell, or cmd.exe.
- Port 10250 must be open for log collection.

- An administrator user is present with the private key used in the secret set as an authorized SSH key.

- If you are creating a BYOH Windows instance on vSphere, communication with the internal API server must be enabled.

    - The hostname of the instance must follow the RFC 1123 DNS label requirements, which include the following standards:

    - Contains only lowercase alphanumeric characters or '-'.

    - Starts with an alphanumeric character.
    - Ends with an alphanumeric character.

## 5.2 Windows node vm installation <a name="5-2"></a>
Finding windows server 20H2 ISO image is  as hard as it gets, somehow we managed to have one in vSphere 

Log on to vsphere, right click on folder, select 'New Virtual Machine'

![](images/2022-01-20-02-13-10.png)

Click next
![](images/2022-01-20-02-13-51.png)

Input windows node name , Click next

![](images/2022-01-20-02-15-02.png)

Select RHCluster , click next

![](images/2022-01-20-02-15-30.png)

select datastore, click next 

![](images/2022-01-20-02-15-54.png)

click next

![](images/2022-01-20-02-17-51.png)

We use windows server 2019 but select most recent version if 2019 are not available

![](images/2022-01-20-02-18-50.png)

In customize hardware page , make sure you select thin provisioning for storage

![](images/2022-01-20-02-37-00.png)

In CD/ DVD select Datastore ISO

![](images/2022-01-20-02-28-08.png)

Select Datastore1 -> ISO -> en_windows_server_20h2_x64_dvd.iso

![](images/2022-01-20-02-33-48.png)

make sure connect at power on are selected, click next and click finish

![](images/2022-01-20-02-35-51.png)

Power On and complete windows installation process until log on step


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
