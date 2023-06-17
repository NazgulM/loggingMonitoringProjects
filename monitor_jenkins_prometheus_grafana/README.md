# Monitoring Jenkins Using Prometheus and Grafana

1. Jenkins is an open-source Continuous Integration/Continuous Delivery and deployment toll written in Java. it is used to implement CI/CD Workflows, called pipelines. 

2. Prometheus is a time series Database. it is designed to monitor targets. Servers, Databases, standalone virtual machines, and many others can be monitored with prometheus. 

3. Grafana is a multi-platform open-source analytics and interactive visualization tool. It provides charts, graphs, alerts when connected to supported data sources. 

Jenkins, will be using an AWS EC2 Instance as Virtual Machine to host Jenkins and monitor that using Prometheus and Grafana.

Create an EC2 Instance with the Security group attached opening all required ports such as 8080 for Jenkins, 3000 for Grafana, and 9090 for Prometheus...

Once an EC2 instance is launched, follow the below commands to install Jenkins on it.

```

sudo yum update –y
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
sudo amazon-linux-extras install java-openjdk11 -y
sudo dnf install java-11-amazon-corretto -y   #(Amazon Linux 2023)
sudo yum install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Connect to http://<your_EC2_Instance_Public_IP>:8080 from your browser. You will be able to access Jenkins through its management interface.

![jenkins](jenkins1.png)

As prompted, enter the password found in /var/lib/jenkins/secrets/initialAdminPassword.

Use the following command to display this password:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

The Jenkins installation script directs you to the Starting Jenkins page. Click Install suggested plugins.

Once the installation is complete, the Create First Admin User will open. Enter your information, and then select Save and Continue.

![admin](jenkins2.png)

![plugins](jenkins3.png)

Also, you can customize the Prometheus like path, namespace and duration in which Prometheus should collect its metrics,.. by going through the following path in Jenkins. i.e., Manage Jenkins -> System

Prometheus and Grafana:
We will be installing Prometheus and Grafana through Docker as it will be a simple and straight-forward compared to installing them via commands.

```
docker run -d --name prometheus -p 9090:9090 prom/prometheus
docker run -d --name grafana -p 3000:3000 grafana/grafana

docker ps
CONTAINER ID   IMAGE     COMMAND    CREATED        STATUS          PORTS                                       NAMES
6d229255bb04   grafana/grafana   "/run.sh"                45 seconds ago   Up 45 seconds   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   grafana
c2780f8ddcec   prom/prometheus   "/bin/prometheus --c…"   55 seconds ago   Up 55 seconds   0.0.0.0:9090->9090/tcp, :::9090->9090/tcp   prometheus
```

We now need to changes in Prometheus configuration files to access Jenkins data.

Now, go to the Prometheus container by entering the following command.

After that, you need to change a few things in prometheus.yml file.

![prometheus](jenkins5.png)

Add the following commands at the end of prometheus.yml file, These commands implies that we are adding Jenkins as a target to collect metrics for Prometheus.

```
- job_name: jenkins
  metrics_path: /prometheus
  static_configs:
    - targets: ['ip_address_of_Jenkins']
```
After doing these changes, restart the Prometheus docker container using docker cmd like 'docker restart prometheus'

You can see the Prometheus running at <Ip_address_of_server:9090>

![prometheus](jenkins6.png)

Now access Grafana though rul: <Ip_address:3000>, default username and password will be admin.

![grafana](jenkins7.png)

After successfully logging in, add the Data Source as Prometheus.

![add data](jenkins8.png)

Click on Data Sources, and select Prometheus.

Give the Prometheus url and click on Save and Test at the bottom.

![prom](jenkins9.png)

