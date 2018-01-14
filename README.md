


Launch 2 Ubuntu 16.04 EC2 Instances which acts as Application Nodes using AWS Web console:

Once the instances passed the status checks, SSH into the one of the 2 Application Nodes and execute below commands to set up GO environment.

Installing Go

sudo su
cd ~
wget https://storage.googleapis.com/golang/go1.8.1.linux-amd64.tar.gz
tar xvf go1.8.1.linux-amd64.tar.gz
chown -R root:root ./go
mv go /usr/local


Setting Go Paths System Wide
Create a projects directory to place our sample application
mkdir $HOME/projects/steeleye

Set the environmental variable globally so that GO know where to look for its files using /etc/environment files


sudo vim /etc/environment

Modify the Environment Variable file as below
GOPATH="/root/projects/steeleye"
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/go/bin:$GOPATH/bin"

  

Now create a file name app.go in /root/projects/steeleye
Vim /root/projects/steeleye/app.go 

Copy below sample GO application to /root/projects/steeleye/app.go file
package main
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


Now run sample app using below command
cd /root/projects/steeleye
go build
Go language does not natively provide a reliable way to daemonize itself, so we need to run our sample app as a system service as shown below.
Create the system service file as shown below
vim /lib/systemd/system/steeleye.service

[Unit]
Description=steeleye sample application
ConditionPathExists=/root/projects/steeleye/steeleye
[Service]
Restart=always
RestartSec=3
ExecStart==/root/projects/steeleye/steeleye
[Install]
WantedBy=multi-user.target

Now start steeleye service
Systemctl enable steeleye.service
systemctl start steeleye

 




Now visit https://PUBLIC_IP_OF_APP_SERVER:8484 from your browser
  




 


 

