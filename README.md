# Steel-Eye Infrastructure Technical Test Deployment

## A) Application Tier Setup
In this section we willl launching, configuring and testing the application tier EC2 instances.
1)  Launch the Appication layer of Infrastructure
2)  **GO** Environment Setup
3) Deploy the sample application to projects Directory and compling the sample Application.
4) Launch the sample application as systemd service and testing the sample application deployed in the  Application server.

## B) Web Tier setup
In this section we will be launching our web server and configuring it to route the traffic in Round-Robin fashion to the Application servers we created in previous step.
1) Launch one Ubuntu 16.04 EC2 instance from AWS console.
2) Installing and configuring a NGINX in the WEB node and testing our final setup


___

## A) Application Tier Setup

### 1) Launch the Application layer of Infrastructure
Launch one Ubuntu 16.04 EC2 Instances which acts as a Application Node using AWS Web console and open TCP port **8484** from instance security group. As of now we are launching only 1 Application server, once this server is fully configured and tested we will create a Amazon Machine Image(AMI) and launch one more Application server in the coming steps to increase the count of Application Servers to 2.



### 2) GO Environment Setup
In the section we will download and install **GO** binaries and set appropiate Environment variables, so that **GO** can access its binaries and our application code.
&nbsp;

- Once the system and instance status checks has passed, SSH into the Application server and execute below commands to download and install **GO**.
&nbsp;
``` sh
$ sudo su
$ cd ~
$ apt-get update 
$ apt-get upgrade -y
$ wget https://storage.googleapis.com/golang/go1.8.1.linux-amd64.tar.gz
$ tar xvf go1.8.1.linux-amd64.tar.gz
$ chown -R root:root ./go
$ mv go /usr/local
```

- Create a projects directory to place our sample application 
```sh
mkdir -p $HOME/projects/steeleye
```
- Now we will edit **/etc/environment**   and set the  enviromental variables so that **GO** can access its binaries and also our application code.

- Open **/etc/environment** using **vim** 
``` sh
vim /etc/environment
```
- Make sure **/etc/environment** file match below configuration
```
GOPATH="/root/projects/steeleye"
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/go/bin:$GOPATH/bin"
```
### 3) Now run the below for the environment variables changes you make in the previous to take effect

```sh
source /etc/environment
```

### 4) Deploy the sample application to projects Directory and compling the sample Application.

- Now create a file named **app.go** at **/root/projects/steeleye** directory
``` sh
vim /root/projects/steeleye/app.go
```

- Copy below sample **GO** application to **/root/projects/steeleye/app.go** file
 which you created in previous step
```
import (
        "fmt"
        "net/http"
        "os"
)
func handler(w http.ResponseWriter, r *http.Request) {
        h, _ := os.Hostname()
        fmt.Fprintf(w, "Hi there, I'm served from %s!", h)
}
func main() {
        http.HandleFunc("/", handler)
        http.ListenAndServe(":8484", nil)
}
```

- Compile the  sample application using below command
```sh 
cd /root/projects/steeleye
go build
```
### 5) Launch the sample application as systemd service and testing the sample application deployed in the  Application server.

Go language does not natively provide a reliable way to daemonize itself, so we need to run our sample application as a systemd service as shown below.

- Create a file named **steeleye.service** at **/lib/systemd/system/**  as shown below
``` sh
$ vim /lib/systemd/system/steeleye.service
```

- Copy and paste the following contents  to **steeleye.service** file 

```
[Unit]
Description=steeleye sample application
ConditionPathExists=/root/projects/steeleye/steeleye
[Service]
Restart=always
RestartSec=3
ExecStart==/root/projects/steeleye/steeleye
[Install]
WantedBy=multi-user.target
```

- Now start steeleye service
``` sh
Systemctl enable steeleye.service
systemctl start steeleye
```
- Now visit https://PUBLIC_IP_OF_APP_SERVER:8484 from your browser and verify your getting expected results as shown below

<p align="center">
  <img width="460" height="300" src="https://raw.githubusercontent.com/iamsoman/steel-eye/master/app-server-output.PNG">
</p>


### 6) Launch the second Application server.
Now that we have fully configured and tested our first Application server, we can now create a AMI of it and Launch the second application server to increase the count of Application server to 2 and also open TCP port **8484** from instance security group.
&nbsp;
&nbsp;

## B) Web Tier Setup
In this section lets launch a  **Ubuntu 16.04** EC2 instance and configure it to be our **WEB node**

### 1) Launch the web node from AWS console.
Launch a  ubuntu 16.04 EC2 instance from AWS web console which we will be configuring as our **WEB** node and also open **HTTP PORT** in the instance security group.

### 2) Installing and configuring NGINX in the WEB node and testing our final setup
In this section we will install and configure NGinx to route traffic to our Application servers in Round-Robin fashion.

- Once the Web EC2 instance passes system and instance health check, **SSH** in to the instance and execute the below commands to install **NGINX** 

```sh
$ sudo su
$ cd ~
$ apt-get update
$ apt-get install nginx -y
$ systemctl enable nginx
$ systemctl start nginx
```

- Open **/etc/nginx/sites-enabled/default** file 
``` sh
$ vim /etc/nginx/sites-enabled/default
```
- Modify the file to match the below configuration, so that traffic is routed to Application server in Round-Robin fashion, Make sure you add appropiate values in **Upstream** block
``` 
    upstream steeleye {
        server Private_IP_OF_Application_Server_1:8484;
        server Private_IP_OF_Application_Server_1:8484;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://steeleye;
        }
    }
```

- Now reload  NGiNX for new configuration changes to reflect
```sh
$ systemctl reload nginx
``` 
- Now note visit Web Nodes **PUBLIC_IP_ADDRESS** from your Browser(preferably FireFox) and hard refresh mutiple times to see how the WEB node routes the traffic to Application Nodes in Round-Robin fashion. 

#### App_Server_1
<p align="center">
  <img width="460" height="300" src="https://raw.githubusercontent.com/iamsoman/steel-eye/master/round-robin-1.PNG">
</p>

&nbsp;

#### App_Server_2

<p align="center">
  <img width="460" height="300" src="https://raw.githubusercontent.com/iamsoman/steel-eye/master/round-robin-2.PNG">
</p>


 

