# ASP.NET Core Web Application + Elastic APM in Docker

Docker is a set of Platform as a Service (PaaS) products that use
OS-level virtualization to deliver software in packages called
containers. Containers are isolated from one another and bundle their
own software, libraries and configuration files; they can communicate
with each other through well-defined channels. Because all of the
containers share the services of a single operating system kernel, they
use fewer resources than virtual machines.

ASP.NET is a popular web-development framework for building web apps and
web APIs on the .NET platform. ASP.NET Core is the open-source version
of ASP.NET, that runs on macOS, Linux, and Windows. ASP.NET Core was
first released in 2016 and is a re-design of earlier Windows-only
versions of ASP.NET.

\"ELK\" is the acronym for three open source projects: Elasticsearch,
Logstash, and Kibana. Elasticsearch is a search and analytics engine.
Logstash is a server‑side data processing pipeline that ingests data
from multiple sources simultaneously, transforms it, and then sends it
to a \"stash\" like Elasticsearch. Kibana lets users visualize data with
charts and graphs in Elasticsearch.

Elastic APM is an application performance monitoring system built on the
Elastic Stack. It allows you to monitor software services and
applications in real-time, by collecting detailed performance
information on response time for incoming requests, database queries,
calls to caches, external HTTP requests, and more. This makes it easy to
pinpoint and fix performance problems quickly.

Elastic APM also automatically collects unhandled errors and exceptions.
Errors are grouped based primarily on the stacktrace, so you can
identify new errors as they appear and keep an eye on how many times
specific errors happen.

In this project, we will build a development single-page ASP.NET core
application (Model-View-Controller) and set up Elastic APM on Docker for
the application.

Prerequisite

Ubuntu 20.04 LTS virtual machine (at least 8gb RAM) with
[Docker](https://docs.docker.com/engine/install/ubuntu/) installed, SSH
client.

### Step 1 - Create the single-page ASP.NET core application

Install [.NET
SDK](https://docs.microsoft.com/en-us/dotnet/core/install/linux-ubuntu#2004-)
5.0 in the virtual machine.

*\$ wget
https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb
-O packages-microsoft-prod.deb*

*\$ sudo dpkg -i packages-microsoft-prod.deb*

*\$ sudo apt-get update; \\*

*sudo apt-get install -y apt-transport-https && \\*

*sudo apt-get update && \\*

*sudo apt-get install -y dotnet-sdk-5.0*

Check .NET SDK version.

*\$ dotnet \--version*

Create a sample ASP.NET Core Web App (Model-View-Controller)

*\$ dotnet new mvc -o myWebApp \--no-https*

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image1.jpeg)

Navigate to the new directory created by the previous command.

*\$ cd myWebApp*

Files and folders of the sample application in the directory

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image2.png)

Start the WebApp.

*\$ dotnet run*

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image3.png)

Open another SSH client, connect to the VM and confirm the creation of
the single-page WebApp.

*\$ curl localhost:5000*

Stop the application with ctrl + c

Install Nginx webserver.

*\$ sudo apt install nginx*

Confirm that Port 80 is opened in the Inbound port rules of the VM's
Network Security Group. If it is not open, create a rule to open it.

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image4.png)

Confirm the status of UFW.

*\$ sudo ufw status*

If it is active, disable it.

*\$ sudo ufw disable*

Create a virtual host (reverse proxy) config file with a text editor.

*\$ sudo nano /etc/nginx/sites-enabled/asp.conf*

Copy the configuration in files/asp.conf in the repository into the file:

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image5.png)

Save the file and test if the configuration is okay.

*\$ sudo nginx -t*

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image6.png)

Restart Nginx.

*\$ sudo systemctl restart nginx*

Start the application.

*\$ dotnet run*

Open port 90 in the Inbound port rules of the Ubuntu VM's Network
Security Group

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image7.jpeg)

Visit the ASP.NET Core WebApp on a web browser with the virtual
machine's Public IP address and port 90 i.e. *http://\<public_IP\>:90*

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image8.png)

### Step 2 - Configure the application for Elasticsearch APM

Add Elastic's Application Performance Monitoring (APM) agent NuGet
package into the app.

*\$ dotnet add package Elastic.Apm.NetCoreAll \--version 1.8.1*

Edit the *Startup.cs* file in the myWebApp directory with a text editor.
Add *using Elastic.Apm.NetCoreAll;* and
*app.UseAllElasticApm(Configuration);* in the file as seen in the
screenshot:

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image9.png)

Save the file. Also edit *appsettings.json* file and add some
configurations as seen in the screenshot:

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image10.png)

Note: Watch out for where commas need to be added. Missing commas can
cause errors when trying to run the app.

After the configuration, run:

*\$ dotnet restore*

### Step 3 - Deploy Elastic APM

Open another SSH client, connect to the VM and create the docker
containers needed for Elastic APM.

Install docker-compose.

*\$ sudo curl -L
\"https://github.com/docker/compose/releases/download/1.28.5/docker-compose-\$(uname
-s)-\$(uname -m)\" -o /usr/local/bin/docker-compose*

*\$ sudo chmod +x /usr/local/bin/docker-compose*

*\$ sudo docker-compose -version*

Create a docker-compose.yml file with a text editor:

*\$ sudo nano docker-compose.yml*

Copy the configuration in files/docker-compose.yml in the repository into the file.

Run *docker-compose up*. Compose will download the official docker
containers and start Elasticsearch, Kibana, and APM Server.

*\$ sudo docker-compose up*

This could take between 3 - 5 minutes.

Kibana is the Elastic's GUI, its running container available on
localhost:5601.

Create a virtual host for localhost:5601 so that it can be visible over
the internet.

*\$ sudo nano /etc/nginx/sites-enabled/asp2.conf*

Copy the configuration in files/asp2.conf in the repository into the file:

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image11.png)

Open port 91 in the Inbound port rules of the VM's network security
group and restart Nginx.

*\$ sudo systemctl restart nginx*

Visit the KIbana on a web browser with the virtual machine's Public IP
address and port 91 i.e. http://\<public_IP\>:91

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image12.png)

Click on the menu and scroll down to Observability. Click on APM.

For now, there is no application to monitor:

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image13.png)

To test if the Elastic APM was properly configured, run the WebApp (in
the myWebApp directory).

*\$ dotnet run*

Visit the WebApp on a web browser with the virtual machine's Public IP
address and port 90 i.e. http://\<public_IP\>:90, then click on the
Refresh button on the Kibana page. The WebApp would show up on the APM
page. The application performance data can be viewed here.

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image14.png)

Activities on the WebAPp can now be monitored on Kibana. For example,
clicks on the Home and Privacy links on the page will have Latency,
Throughput, Transaction and other records:

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image15.png)

Stop the running WebApp with ctrl + c.

To view the Docker images of the Elastic APM setup created:

*\$ sudo docker images*

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image16.png)

View the Docker containers created for the Elastic APM.

*\$ sudo docker ps -a*

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image17.png)

ctrl+c will stop all the Elastic APM containers running. The Kibana
webpage will no longer be available.

To restart the three Elastic containers and enable application
monitoring:

*\$ sudo docker start \<container_name_or_ID\>*

Start the Elasticserach container first, followed by Kibana container,
then APM-server container. Kibana and APM-server require that
Elasticsearch is running first:

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image18.png)

To ensure that these three containers (and also the WebApp container)
will run anytime on booting up the VM, update each container with a
restart policy while the containers are running:

*\$ sudo* *docker update \--restart unless-stopped
\<container_name_or_ID\>*

![](https://github.com/osygroup/Images/blob/main/ASP.NET-ElasticAPM/image19.png)

### Conclusion

If it isn't monitored, it isn't production.

Elastic APM helps to avoid developing additional metering and
instrumentation in micro services. The APM agent takes care of
monitoring all the key metrics. The data can be immensely useful
detecting problems early and understand the state of the systems in
production.

### Credits:

<https://www.elastic.co/what-is/elk-stack>

<https://www.elastic.co/guide/en/apm/get-started/current/overview.html>

<https://docs.docker.com/engine/examples/dotnetcore/>

<https://docs.microsoft.com/en-us/dotnet/core/docker/build-container?tabs=linux>

<https://medium.com/logistimo-engineering-blog/how-did-i-use-apm-in-kubernetes-ecosystem-8f22d52beb03>
