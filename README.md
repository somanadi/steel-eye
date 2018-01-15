# Technical Test Instructions

##Application Tier
1)  Launch the Appication layer of Infrastructure
2)  GO Environment Setup
    *  Create a projects directory to place our sample application
    *  Modify **/etc/environment** file
3) Deploy the sample application to projects Directory  and compling the Application.
4) Launch the sample application as systemd service and testing the sample application from Application server.

##Web Tier

### 1) Launch the Application layer of Infrastructure
Launch 1 Ubuntu 16.04 EC2 Instances which acts as Application Nodes using AWS Web console. As of now we are launching only 1 Application server, once this server is fully configured and tested we will create a Amazon Machine Image(AMI) and launch one more Application server in the coming steps to make to count of Application Servers 2.



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

- Create a projects directory 
```sh
mkdir $HOME/projects/steeleye
```
- Now we will edit **/etc/environment**   and set the  enviromental variables so that **GO** can access its binaries and also our application code.

- Open /etc/environment using **vim** 
``` sh
vim /etc/environment
```
- Make sure **/etc/environment** file match below configuration
```
GOPATH="/root/projects/steeleye"
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/go/bin:$GOPATH/bin"
```
### 3) Deploying the sample application to the Projects Directory and  compiling

- Now create a file named **app.go** in **/root/projects/steeleye** directory
``` sh
vim /root/projects/steeleye/app.go
```

- Copy below sample GO application to **/root/projects/steeleye/app.go** file
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
### 4) Launch the sample application as systemd service

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
- Now visit https://PUBLIC_IP_OF_APP_SERVER:8484 from your browser and verify your getting expected results as below

![Alt text](https://raw.githubusercontent.com/iamsoman/steel-eye/master/app-server-output.PNG )

### 5) Launch the second Application server tier.
Now that we have fully configured and tested our first Application server, we can now create a AMI of it and Launch the second application server to increase the count of Application server to 2.




 


 

