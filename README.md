# Prometgheus-Grafana-NodeExporter

```text
Note- Prometheus will run on port [9090]
      Grafana will run on port [5000]
      Node Exporter will run on port [9100]
```      
## What is Prometheus?

Prometheus is a time series database. Prometheus is designed to monitor targets. Servers, databases, standalone virtual machines, pretty much everything can be monitored with Prometheus.

When does it fit?

Prometheus works well for recording any purely numeric time series
It fits both machine-centric monitoring and highly dynamic service-oriented architectures.
Prometheus is designed for reliability, during an system outage it allow you to quickly diagnose problems. 
Each Prometheus server is standalone, does not depend on any network storage or other remote services.
When does it not fit?

If you need 100% accuracy, such as for per-request billing, Prometheus is not a good choice

![image](https://user-images.githubusercontent.com/47140557/221348919-74c3c5db-cad1-47ab-89dd-0452e6953996.png)


## Install Prometheus

We will install the latest version of Prometheus on our server.

Before you start, you will need a Linux server. Preferably an unrestricted Ubuntu 16.04 LTS Server with root access, since all the commands demonstrated in this are executed on Ubuntu 16.04 LTS Server.

You can use other operating systems, such as Centos 7, but all commands in the course are prepared for Ubuntu 18, so you will experience some differences in syntax or equivalent commands which you may need to research yourself if I can't help you.

Once you have an Ubuntu 18 server ready, you can find the latest version of the Prometheus binary to download at https://prometheus.io/download/

Or just copy/paste this pre created command below for prometheus-2.14.
```cmd
wget https://github.com/prometheus/prometheus/releases/download/v2.18.1/prometheus-2.18.1.linux-amd64.tar.gz
```
If all is ok, then we can untar the gz archive.
```cmd
tar xvfz prometheus-2.14.0.linux-amd64.tar.gz
```
Now CD into the folder
```cmd
cd prometheus-2.14.0.linux-amd64
```
And start it
```cmd
./prometheus --config.file=prometheus.yml
```
Prometheus should now be running.

You can visit it at ```text http://[your ip address]:9090 ```

## Start Prometheus as a Service

Stop the running Prometheus process that we left running in the previous lecture, and copy the new files to bin folder.
```cmd
cp -r . /usr/local/bin/prometheus
```
Create a file called prometheus.service
```cmd
 sudo nano /etc/systemd/system/prometheus.service
```
Add the script and save

```yaml
[Unit]
Description=Prometheus Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/prometheus/prometheus --config.file=/usr/local/bin/prometheus/prometheus.yml

[Install]
WantedBy=multi-user.target
```

Now start and check the service is running.

```cmd
sudo service prometheus start
sudo service prometheus status
```

We can now leave the new Prometheus service running. If you ever need to stop the new Prometheus service, then type

```cmd
sudo service prometheus stop
sudo service prometheus status
```
## Install Node Exporter and Start as a Service

Now we will install the Prometheus Node Exporter on our existing server and configure it as a Service so that it keeps running in the background.

We will download the node exporter binary from the Prometheus downloads page at https://prometheus.io/download/#node_exporter

Download Prometheus Node Exporter Binary SSH into your Prometheus server, and run
```cmd
wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
```
Untar it,
```cmd
tar xzf node_exporter-0.18.1.linux-amd64.tar.gz
```
Copy it to the /usr/local/bin/ folder
```cmd
cp node_exporter-0.18.1.linux-amd64/node_exporter /usr/local/bin/node_exporter
```
Configure Prometheus Node Exporter as a Service, Create a file called node-exporter.service
```cmd
sudo vi /etc/systemd/system/node-exporter.service
```
Add the script and save
```yaml
[Unit]
Description=Prometheus Node Exporter Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Now start and check the service is running.

```cmd
systemctl daemon-reload
sudo service node-exporter start
sudo service node-exporter status
```

Node exporter will now be running on http://[your domain or ip]:9100/metrics

Add the new scrape config for the new node exporter.
```cmd
sudo vi /usr/local/bin/prometheus/prometheus.yml Scroll down to the bottom and add a new scrape config
```
```yml
  - job_name: 'node-exporter'
    static_configs:
    - targets: ['localhost:9100']
```
and restart the prometheus service.
```cmd
sudo service prometheus restart
sudo service prometheus status
```

PS- To monitor a different server, you simply need to set up node exporter as a service on that server and modify the prometheus.yaml file on the machine where Prometheus is installed. The alterations you'll need to make are:
```yaml

 - job_name: 'Name of the Job'
    static_configs:
    - targets: ['ip of your another server whre new node exporter is running:9100']
```  
``` text
This configuration can be added to the prometheus.yaml file on the machine where Prometheus is installed to enable monitoring of the new server.

``` 
