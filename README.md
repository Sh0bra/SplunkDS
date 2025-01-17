# Introduction
When I first learned how to setup a Splunk Server I used to connect my endpoints one at a time using the Splunk Universal Forwarder(SUF). This was no problem for my little lab setup with 2 endpoints but this was practice. In our actual production environment I would have to deploy config changes to over 900 vms.
This daunting task gave me stressful nights of sleep as I pondered how I would take on this massive task ahead of me. Until I stumbled upon the [Splunk Deployment Server](https://docs.splunk.com/Documentation/Splunk/9.3.2/Updating/Aboutdeploymentserver).

# [Splunk Deployment Server](https://docs.splunk.com/Documentation/Splunk/9.3.2/Updating/Aboutdeploymentserver)
So what is the Splunk Deployment Server? Its a server that deploys configs, apps and content. So think of a magical server that deploys inputs.conf (what we want to log) and output.conf (where we want to send those logs).
Going back to my story of configuring the SUF individually for each endpoint. Imagine having to make config edits for 100s of endpoint devices. It would take forever to make those manual changes to each endpoint. The Splunk Deployment Server allows us to create a main config file and automatically deploy these configs to all the endpoints.

# Creating a Splunk Deployment Server
If you are already familiar of how to setup Splunk Enterprise trial then go ahead and create another one. I was really confused on how to install the Splunk Deployment Server until I realized that it was already baked into the Splunk Enterprise trial.
You can use your main Splunk server as the deployment server in my example I chose to separate it out and create another Splunk server and use it as my Deployment Server.

> IMPORTANT: If you are trying to deploy to both Windows and Linux devices you will need to install Splunk on a Linux device for compatibility.

# Connecting devices to the Splunk Deployment Server
When you install the SUF you are given the option to point to a deployment server. This is how you connect the SUF to the deployment server. Once the endpoint is connected to the deployment server you will be able to see it in you Forwarder Management tab.
![Screenshot](/assets/forwader-management)

# How things are organized
The Splunk Deployment Server uses the concept of apps, [server classes](https://docs.splunk.com/Documentation/Splunk/9.3.2/Updating/Definedeploymentclasses) and [deployment clients](https://docs.splunk.com/Splexicon:Deploymentclient).

# Server Class
Think of server classes as groupings mainly Windows vs Linux and deployment clients as your endpoints connected with SUF. You can associate a server class to multiple deployment clients.
For example in our production we identified 3 areas. One being our critical infrastructure, another the our workstations, and lastly virtual machines. So for our server classes we made a Windows and Linux server class for each of those 3 sections.

# Deployment Clients
Think of deployment clients as the endpoints with SUFs installed connected to the server. Everytime you connect an endpoint to the deployment server you will see it here.

# Apps
Think of apps as the config files to deploy to the deployment clients. Lets say we wanted to deploy a inputs.conf for any Windows machine. We would create a app containing the inputs.conf file monitoring the basic security, application and system logs. And we would also create an app containing a outputs.conf file telling the SUF where to send the logs. We would associate these 2 apps to our Windows server class. Now lets say you want to do the same for linux. You would create a different app containing an inputs.conf file monitoring the /var/log folder and associate it with a Linux server class.

# Connecting it all together
After you define the apps and associate it with your server class you will then connect your deployment clients to the server classes. So lets say you created an inputs and outputs app for your Windows server class. You will then connect all your Windows endpoints to that server class and let the Splunk Deployment Server automate the deployments for you.
