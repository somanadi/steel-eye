# Technical Test Instructions
1)  Launch the Appication layer of Infrastructure
2) GO Environment Setup
   2a.  Create a projects directory to place our sample application
   2b.  Modify **/etc/environment** file
3) Deploy the sample application to projects Directory  and compling the Application.
4) Launch the sample application as systemd service and testing the sample application from Application server.

### 1) Launch the Appication layer of Infrastructure
Launch 2 Ubuntu 16.04 EC2 Instances which acts as Application Nodes using AWS Web console:



## 2)GO Environment Setup
In the section we will download and install **GO** binaries and set appropiate Environment variables, so that **GO** can access its binaries and application code.
&nbsp;
**2a)** Once the system and instance passed the status checks has passed, SSH into the one of the 2 Application Nodes and execute below commands to download and install **GO**.
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

**2b)** Create a projects directory to place our sample application
```sh
mkdir $HOME/projects/steeleye
```
**2c)** Now we will edit **/etc/environment**   and set the  enviromental variable so that **GO** can access its binaries and also our application code.

- Open /etc/environment using **vim** 
``` sh
sudo vim /etc/environment
```
- Make sure **/etc/environment** file match below configuration
```
GOPATH="/root/projects/steeleye"
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/go/bin:$GOPATH/bin"
```
**3) Deploying the sample application to Projects Directory and Testing the Application.**

- Now create a file named app.go in **/root/projects/steeleye**
``` sh
Vim /root/projects/steeleye/app.go
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

- Complie the  sample application using below command
```sh 
cd /root/projects/steeleye
go build
```
### Launch the sample application as systemd service

Go language does not natively provide a reliable way to daemonize itself, so we need to run our sample app as a system service as shown below.

- Create a file named steeleye.service in **/lib/systemd/system/**  as shown below
``` sh
$ vim /lib/systemd/system/steeleye.service
```

- Copy and paste the following contents  to ***steeleye.service*** file 

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
- Now visit https://PUBLIC_IP_OF_APP_SERVER:8484 from your browser

![Alt text](https://raw.githubusercontent.com/iamsoman/steel-eye/master/app-server-output.PNG )





 


 

