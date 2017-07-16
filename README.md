# Ubuntu16.04 Jupyter Notebook with python 3.6, July 15th, 2017

### 1.create an instance from https://digitalocean.com (for easy and cheap($5 per month) to start, no worry about firewall issue)

###### ssh root@ip
type password

```
#Create ubuntu user
sudo useradd --create-home -s /bin/bash ubuntu
sudo adduser ubuntu sudo
sudo passwd ubuntu
#type new password for ubuntu user

#if you have ~/.ssh/id_rsa.pub in your local file, then you should copy it /home/ubuntu/.ssh/id_rsa.pub, if you don't have you should create one by run following command, then copy it to your /home/ubuntu/.ssh/id_rsa.pub
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

#Enter a file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
#Enter passphrase (empty for no passphrase): [Type a passphrase]
#Enter same passphrase again: [Type passphrase again]
#ssh-add ~/.ssh/id_rsa
```
ssh ubuntu@ip

```sh
#install anaconda, use default path and yes export to bash

wget https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh

bash Anaconda3-4.4.0-Linux-x86_64.sh

# add this line to .bashrc
source ~/.bashrc

# create py3.6 environment
conda create -n py3.6 python=3.6 anaconda

# activate the environment
source activate py3.6

# install juypterhub
conda install -c conda-forge jupyterhub=0.7.2

```
jupyter notebook
```

### you should see following message

```txt
[I 23:44:09.730 NotebookApp] Serving notebooks from local directory: /home/deploy
[I 23:44:09.731 NotebookApp] 0 active kernels
[I 23:44:09.731 NotebookApp] The Jupyter Notebook is running at: http://localhost:8888/?token=a9b26de95e061e401ea302xxx
[I 23:44:09.731 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[W 23:44:09.732 NotebookApp] No web browser found: could not locate runnable browser.
[C 23:44:09.732 NotebookApp]

    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://localhost:8888/?token=a9b26de95e061e401ea302xxx
```
then open another terminal, following step is expose your localhost:8888 to the world

###### ssh ubuntu@ip

run
```
curl http://localhost:8888/?token=a9b26de95e061e401ea302xxx
```

### open your port 8888 by using nginx server

cp follow text to a file call ~/my.conf
```
server {
    listen 80 default_server;

    location / {
        proxy_pass            http://localhost:8888;
        proxy_set_header      Host $host;
    }

    location /api/kernels/ {
        proxy_pass            http://localhost:8888;
        proxy_set_header      Host $host;
        # websocket support
        proxy_http_version    1.1;
        proxy_set_header      Upgrade "websocket";
        proxy_set_header      Connection "Upgrade";
        proxy_read_timeout    86400;
    }
}
```
Install nginx and setup
```
# install nginx server
sudo apt-get update
sudo apt-get install nginx

# copy my.conf to nginx configuration location
cp ~/my.conf /etc/nginx/sites-available/default

# start proxy server
service nginx restart

# see if any error log, if empty, good to go
tail /var/log/nginx/error.log
```

### last step, start server
```sh
jupyter notebook --allow-root
```
open your ip and type password, you should good to go.
an exmaple will be http://162.243.155.204/


Reference: 
* https://github.com/jupyter/notebook/issues/625#issue-112465411
* https://github.com/jupyter/notebook/issues/625

###### Author: Ruochen Xu 徐若尘
###### Source: https://github.com/xxu46/ubuntu_notebook