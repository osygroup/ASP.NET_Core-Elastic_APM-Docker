# ASP.NET Core Web Application + Elastic APM in Docker

Docker is a set of platform as a service (PaaS) products that use
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

### Prerequisite

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

![C:\\Users\\osygroup\\Documents\\ASP.NET Core with
ELK\\mvc4.JPG](github\ASP\media\image1.jpeg){width="5.989583333333333in"
height="1.6139599737532808in"}

Navigate to the new directory created by the previous command.

*\$ cd myWebApp*

Files and folders of the sample application in the directory

![](github\ASP\media\image2.png){width="5.833333333333333in"
height="0.6666666666666666in"}

Start the WebApp.

*\$ dotnet run*

![](github\ASP\media\image3.png){width="4.84375in" height="1.84375in"}

Open another SSH client, connect to the VM and confirm the creation of
the single-page WebApp.

*\$ curl localhost:5000*

Stop the application with ctrl + c

Install Nginx webserver.

*\$ sudo apt install nginx*

Confirm that Port 80 is opened in the Inbound port rules of the VM's
Network Security Group. If it is not open, create a rule to open it.

![](github\ASP\media\image4.png){width="6.171570428696413in"
height="2.375in"}

Confirm the status of UFW.

*\$ sudo ufw status*

If it is active, disable it.

*\$ sudo ufw disable*

Create a virtual host (reverse proxy) config file with a text editor.

*\$ sudo nano /etc/nginx/sites-enabled/asp.conf*

Copy the below configuration into the file:

*server {*

*listen 90;*

*location / {*

*proxy_pass http://localhost:5000;*

*proxy_set_header Upgrade \$http_upgrade;*

*proxy_set_header Connection \'upgrade\';*

*proxy_set_header Host \$host;*

*proxy_cache_bypass \$http_upgrade;*

*}*

*}*

![](github\ASP\media\image5.png){width="6.15625in" height="3.34375in"}

Save the file and test if the configuration is okay.

*\$ sudo nginx -t*

![](github\ASP\media\image6.png){width="5.6875in"
height="0.6666666666666666in"}

Restart Nginx.

*\$ sudo systemctl restart nginx*

Start the application.

*\$ dotnet run*

Open port 90 in the Inbound port rules of the Ubuntu VM's Network
Security Group

![C:\\Users\\osygroup\\Documents\\ASP.NET Core with
ELK\\nsg.JPG](github\ASP\media\image7.jpeg){width="4.622589676290464in"
height="3.3333333333333335in"}

Visit the ASP.NET Core WebApp on a web browser with the virtual
machine's Public IP address and port 90 i.e. *http://\<public_IP\>:90*

![](github\ASP\media\image8.png){width="6.5in"
height="2.5708333333333333in"}

### Step 2 - Configure the application for Elasticsearch APM

Add Elastic's Application Performance Monitoring (APM) agent NuGet
package into the app.

*\$ dotnet add package Elastic.Apm.NetCoreAll \--version 1.8.1*

Edit the *Startup.cs* file in the myWebApp directory with a text editor.
Add *using Elastic.Apm.NetCoreAll;* and
*app.UseAllElasticApm(Configuration);* in the file as seen in the
screenshot:

![](github\ASP\media\image9.png){width="6.5in"
height="4.391666666666667in"}

Save the file. Also edit *appsettings.json* file and add some
configurations as seen in the screenshot:

![](github\ASP\media\image10.png){width="6.5in"
height="3.3256944444444443in"}

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

Copy and paste in the following:

*version: \'2.2\'*

*services:*

*apm-server:*

*image: docker.elastic.co/apm/apm-server:7.11.1*

*depends_on:*

*elasticsearch:*

*condition: service_healthy*

*kibana:*

*condition: service_healthy*

*cap_add: \[\"CHOWN\", \"DAC_OVERRIDE\", \"SETGID\", \"SETUID\"\]*

*cap_drop: \[\"ALL\"\]*

*ports:*

*- 8200:8200*

*networks:*

*- elastic*

*command: \>*

*apm-server -e*

*-E apm-server.rum.enabled=true*

*-E setup.kibana.host=kibana:5601*

*-E setup.template.settings.index.number_of_replicas=0*

*-E apm-server.kibana.enabled=true*

*-E apm-server.kibana.host=kibana:5601*

*-E output.elasticsearch.hosts=\[\"elasticsearch:9200\"\]*

*healthcheck:*

*interval: 10s*

*retries: 12*

*test: curl \--write-out \'HTTP %{http_code}\' \--fail \--silent
\--output /dev/null http://localhost:8200/*

*elasticsearch:*

*image: docker.elastic.co/elasticsearch/elasticsearch:7.11.1*

*environment:*

*- bootstrap.memory_lock=true*

*- cluster.name=docker-cluster*

*- cluster.routing.allocation.disk.threshold_enabled=false*

*- discovery.type=single-node*

*- ES_JAVA_OPTS=-XX:UseAVX=2 -Xms1g -Xmx1g*

*ulimits:*

*memlock:*

*hard: -1*

*soft: -1*

*volumes:*

*- esdata:/usr/share/elasticsearch/data*

*ports:*

*- 9200:9200*

*networks:*

*- elastic*

*healthcheck:*

*interval: 20s*

*retries: 10*

*test: curl -s http://localhost:9200/\_cluster/health \| grep -vq
\'\"status\":\"red\"\'*

*kibana:*

*image: docker.elastic.co/kibana/kibana:7.11.1*

*depends_on:*

*elasticsearch:*

*condition: service_healthy*

*environment:*

*ELASTICSEARCH_URL: http://elasticsearch:9200*

*ELASTICSEARCH_HOSTS: http://elasticsearch:9200*

*ports:*

*- 5601:5601*

*networks:*

*- elastic*

*healthcheck:*

*interval: 10s*

*retries: 20*

*test: curl \--write-out \'HTTP %{http_code}\' \--fail \--silent
\--output /dev/null http://localhost:5601/api/status*

*volumes:*

*esdata:*

*driver: local*

*networks:*

*elastic:*

*driver: bridge*

Run *docker-compose up*. Compose will download the official docker
containers and start Elasticsearch, Kibana, and APM Server.

*\$ sudo docker-compose up*

This could take between 3 - 5 minutes.

Kibana is the Elastic's GUI, its running container available on
localhost:5601.

Create a virtual host for localhost:5601 so that it can be visible over
the internet.

*\$ sudo nano /etc/nginx/sites-enabled/asp2.conf*

![](github\ASP\media\image11.png){width="5.0625in"
height="2.9270833333333335in"}

Open port 91 in the Inbound port rules of the VM's network security
group and restart Nginx.

*\$ sudo systemctl restart nginx*

Visit the KIbana on a web browser with the virtual machine's Public IP
address and port 91 i.e. http://\<public_IP\>:91

![](github\ASP\media\image12.png){width="6.5in"
height="3.3979166666666667in"}

Click on the menu and scroll down to Observability. Click on APM.

For now, there is no application to monitor:

![](github\ASP\media\image13.png){width="6.5in"
height="3.191666666666667in"}

To test if the Elastic APM was properly configured, run the WebApp (in
the myWebApp directory).

*\$ dotnet run*

Visit the WebApp on a web browser with the virtual machine's Public IP
address and port 90 i.e. http://\<public_IP\>:90, then click on the
Refresh button on the Kibana page. The WebApp would show up on the APM
page. The application performance data can be viewed here.

![](github\ASP\media\image14.png){width="6.5in"
height="2.932638888888889in"}

Activities on the WebAPp can now be monitored on Kibana. For example,
clicks on the Home and Privacy links on the page will have Latency,
Throughput, Transaction and other records:

![](github\ASP\media\image15.png){width="6.5in"
height="3.451388888888889in"}

Stop the running WebApp with ctrl + c.

To view the Docker images of the Elastic APM setup created:

*\$ sudo docker images*

![](github\ASP\media\image16.png){width="6.5in"
height="0.9902777777777778in"}

View the Docker containers created for the Elastic APM.

*\$ sudo docker ps -a*

![](github\ASP\media\image17.png){width="6.5in"
height="0.9888888888888889in"}

ctrl+c will stop all the Elastic APM containers running. The Kibana
webpage will no longer be available.

To restart the three Elastic containers and enable application
monitoring:

*\$ sudo docker start \<container_name_or_ID\>*

Start the Elasticserach container first, followed by Kibana container,
then APM-server container. Kibana and APM-server require that
Elasticsearch is running first:

![](github\ASP\media\image18.png){width="5.489583333333333in"
height="1.1666666666666667in"}

To ensure that these three containers (and also the WebApp container)
will run anytime on booting up the VM, update each container with a
restart policy while the containers are running:

*\$ sudo* *docker update \--restart unless-stopped
\<container_name_or_ID\>*

![](github\ASP\media\image19.png){width="6.5in"
height="1.0006944444444446in"}

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
