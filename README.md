# Scalable Deployments with Splunk Deployment Server
## Introduction
When I first learned how to setup a Splunk Server I used to connect my endpoints one at a time using the Splunk Universal Forwarder(SUF). This was no problem for my little lab setup with 2 endpoints but this was practice. In our actual production environment I would have to deploy config changes to over 900 vms.
This daunting task gave me stressful nights of sleep as I pondered how I would take on this massive task ahead of me. Until I stumbled upon the [Splunk Deployment Server](https://docs.splunk.com/Documentation/Splunk/9.3.2/Updating/Aboutdeploymentserver).

## [Splunk Deployment Server](https://docs.splunk.com/Documentation/Splunk/9.3.2/Updating/Aboutdeploymentserver)
So what is the Splunk Deployment Server? Its a server that deploys configs, apps and content. So think of a magical server that deploys inputs.conf (what we want to log) and output.conf (where we want to send those logs).
Going back to my story of configuring the SUF individually for each endpoint. Imagine having to make config edits for 100s of endpoint devices. It would take forever to make those manual changes to each endpoint. The Splunk Deployment Server allows us to create a main config file and automatically deploy these configs to all the endpoints.

## Creating a Splunk Deployment Server
If you are already familiar of how to setup Splunk Enterprise trial then go ahead and create another one. I was really confused on how to install the Splunk Deployment Server until I realized that it was already baked into the Splunk Enterprise trial.
You can use your main Splunk server as the deployment server in my example I chose to separate it out and create another Splunk server and use it as my Deployment Server.

> IMPORTANT: If you are trying to deploy to both Windows and Linux devices you will need to install Splunk on a Linux device for compatibility.

## Connecting devices to the Splunk Deployment Server
When you install the SUF you are given the option to point to a deployment server. This is how you connect the SUF to the deployment server. Once the endpoint is connected to the deployment server you will be able to see it in you Forwarder Management tab.
![Screenshot of our Forwarder Management tab](/assets/forwarder-management.png)

## How things are organized
The Splunk Deployment Server uses the concept of apps, [server classes](https://docs.splunk.com/Documentation/Splunk/9.3.2/Updating/Definedeploymentclasses) and [deployment clients](https://docs.splunk.com/Splexicon:Deploymentclient).
![How things are organized in a Deployment Server](/assets/deployment-server.png)

## Server Class
Think of server classes as groupings mainly Windows vs Linux and deployment clients as your endpoints connected with SUF. You can associate a server class to multiple deployment clients.
For example in our production we identified 3 areas. One being our critical infrastructure, another the our workstations, and lastly virtual machines. So for our server classes we made a Windows and Linux server class for each of those 3 sections.
![Screenshot of Server Classes tab](/assets/server-classes.png)

## Deployment Clients
Think of deployment clients as the endpoints with SUFs installed connected to the server. Everytime you connect an endpoint to the deployment server you will see it here.
![Screenshot of Clients](/assets/clients.png)

## Apps
Think of apps as the config files to deploy to the deployment clients. Lets say we wanted to deploy a inputs.conf for any Windows machine. We would create a app containing the inputs.conf file monitoring the basic security, application and system logs. And we would also create an app containing a outputs.conf file telling the SUF where to send the logs. We would associate these 2 apps to our Windows server class. Now lets say you want to do the same for linux. You would create a different app containing an inputs.conf file monitoring the /var/log folder and associate it with a Linux server class.
![Screenshot of Apps](/assets/apps.png)

## Connecting it all together
After you define the apps and associate it with your server class you will then connect your deployment clients to the server classes. So lets say you created an inputs and outputs app for your Windows server class. You will then connect all your Windows endpoints to that server class and let the Splunk Deployment Server automate the deployments for you.
![Screenshot of soc_desktops_windows server class with associated apps and deployment clients](/assets/server-classes-apps.png)

## Automating the Splunk Universal Forwarder
Now you may have noticed that even though this saves us a lot of time on making config changes we still have to install the SUF on each endpoint and point it back to the server.
This is where our scripting skills come into play.

## Windows CMD/PowerShell
We can use the script below to automate the installation of the SUF and configuration. This script was used in CMD and can be easily converted to PowerShell.

- The first line will download the SUF
```
powershell.exe -Command wget -O C:\splunkforwarder-9.3.0-51ccf43db5bd-x64-release.msi "https://download.splunk.com/products/universalforwarder/releases/9.3.0/windows/splunkforwarder-9.3.0-51ccf43db5bd-x64-release.msi"
msiexec.exe /i C:\splunkforwarder-9.3.0-51ccf43db5bd-x64-release.msi DEPLOYMENT_SERVER="<IP>:8089" AGREETOLICENSE=Yes /quiet
netsh advfirewall firewall add rule name="Splunk TCP In" dir=in action=allow protocol=TCP localport=8089
netsh advfirewall firewall add rule name="Splunk TCP Out" dir=out action=allow protocol=TCP localport=9997, 8089
```
- The second line installs the SUF and configures it to point to a deployment server
- The last few lines open up the firewall to allow port 9997 and 8089 which are used for forwarding data and deployment communication respectively.
- Splunk only uses TCP to communicate and Port 9997 outbound is used to send logs by default.
- Port 8089 is used by the deployment server and needs to be open both inbound and outbound.

## Using PowerCli to run scripts on VMs
Our production system uses ESXi and VSphere and so we can use PowerCli to run our scripts on the VMs. PowerCli code below targets all Windows VMs in the 04_SOC location and runs the script from above.

```powershell
## Connect to elsa to start remote management. Admin required.
Connect-VIServer -Server elsa.sdc.cpp -User <username> -Password <password>

## Get all vms in resource group 04_SOC
Get-VM -Location "04_SOC"

## find all windows vms
Get-VM -Location "04_SOC" | Where-Object { $_.Guest.OSFullName -match "Windows" -or $_.Guest.OSFullName -match "Windows Server" }

## Run a script on vm
$vm = Get-VM -Name "04_Security_Engineer_S"
$script = "C:\Users\User\Desktop\forwarder.ps1"
$username = "<login username>"
$password = "<login password>"
$cred = New-Object System.Management.Automation.PSCredential ($username, (ConvertTo-SecureString $password -AsPlainText -Force))

Invoke-VMScript -VM $vm -ScriptText (Get-Content -Path $script -Raw) -GuestCredential $cred -ScriptType PowerShell
```
The same can be done for Linux in Bash
