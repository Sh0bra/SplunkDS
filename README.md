# Introduction
When I first learned how to setup a Splunk Server I used to connect my endpoints one at a time using the Splunk Universal Forwarder(SUF). This was no problem for my little lab setup with 2 endpoints but this was practice. In our actual production environment I would have to deploy config changes to over 900 vms.
This daunting task gave me stressful nights of sleep as I pondered how I would take on this massive task ahead of me. Until I stumbled upon the Splunk Deployment Server.

# Splunk Deployment Server
So what is the Splunk Deployment Server? Its a server that deploys configs, apps and content. So think of a magical server that deploys inputs.conf (what we want to log) and output.conf (where we want to send those logs).
Going back to my story of configuring the SUF individually for each endpoint. Imagine having to make config edits for 100s of endpoint devices. It would take forever to make those manual changes to each endpoint. The Splunk Deployment Server allows us to create a main config file and automatically deploy these configs to all the endpoints.

# Creating a Splunk Deployment Server
If you are already familiar of how to setup Splunk Enterprise trial then go ahead and create another one. I was really confused on how to install the Splunk Deployment Server until I realized that it was already baked into the Splunk Enterprise trial.
You can use your main Splunk server as the deployment server in my example I chose to separate it out and create another Splunk server and use it as my Deployment Server.

> IMPORTANT: If you are trying to deploy to both Windows and Linux devices you will need to install Splunk on a Linux device for compatibility.

# Connecting devices to the Splunk Deployment Server
When you install the SUF you are given the option to point to a deployment server. This is how you connect the SUF to the deployment server. Once the endpoint is connected to the deployment server you will be able to see it in you Forwarder Management tab.
