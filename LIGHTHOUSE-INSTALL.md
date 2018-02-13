# Prerequisites:

### Basics
Lighthouse requires Elasticsearch, Yarn, and Bitcoin libraries

```
apt install curl apt-transport-https
curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-5.x.list
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
add-apt-repository ppa:bitcoin/bitcoin
```

```
apt update
apt install elasticsearch yarn libdb4.8-dev libdb4.8++-dev python-pip nodejs-legacy unzip default-jre-headless
```

These steps are required to install Node version 8. The commands are correct, there is an apt update in the install script.

```
curl -sL https://deb.nodesource.com/setup_8.x -o nodesource_setup.sh
chmod 755 nodesource_setup.sh
./nodesource_setup.sh
apt install nodejs
```


### Elasticsearch
Enable the Elasticsearch service with these commands:

```
systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
```


The service does not start instantly, but should come up in less than a minute. You can check Elasticsearch's status like this:


```
netstat -plant | grep "9[23]00"
```

And the response should look something like this:

```
tcp6       0      0 127.0.0.1:9200          :::*                    LISTEN      1055/java       
tcp6       0      0 ::1:9200                :::*                    LISTEN      1055/java       
tcp6       0      0 127.0.0.1:9300          :::*                    LISTEN      1055/java       
tcp6       0      0 ::1:9300                :::*                    LISTEN      1055/java   
```

Lighthouse accesses the Elasticsearch API via TCP/9200. The service on port TCP/9300 is Elasticsearch's inter-cluster communiations port, which we do not use, but which will remain visible so long as the daemon is running.

You can examine the actual service using curl:

```
 curl -GET http://localhost:9200/
```


And your response should be similar to this:

```
{
  "name" : "s59WZF5",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "vyQdPqqSTWGKfxALBIH9hg",
  "version" : {
   "number" : "5.6.6",
    "build_hash" : "7d99d36",
    "build_date" : "2018-01-09T23:55:47.880Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```


### lbrycrd daemon

Lighthouse gathers information from the LBRY blockchain and enters it into Elasticsearch. Visit releases to find the latest version of the binaries.

```
cd
mkdir lbrycrd
cd lbrycrd
wget https://github.com/lbryio/lbrycrd/releases/download/v0.12.1.0/lbrycrd-linux.zip
unzip lbrycrd-linux.zip
mkdir ~/.lbrycrd
```

Once unzipped you configure the daemon

And then enable the service:

```
systemctl enable lbrycrdd.service
systemctl start lbrycrdd.service
```

You can check the lbrycrd service:

```
netstat -plant | grep "924" | grep "LISTEN"
```

And you should find these two ports active:

```
tcp        0      0 127.0.0.1:9245          0.0.0.0:*               LISTEN      1047/lbrycrdd   
tcp        0      0 127.0.0.1:9246          0.0.0.0:*               LISTEN      1047/lbrycrdd   
```




### Decoder & Lighthouse daemon

```
cd
git clone https://github.com/lbryio/lighthouse.git
cd ~/lighthouse/decoder
pip install -r requirements.txt
```

Configure the Decoder


```
cat > /etc/systemd/system/decoder.service
[Unit]
Description="Lighthouse claim decoder"
After=network.target

[Service]
Environment="HOME=/root"
ExecStart=/usr/bin/python /root/lighthouse/decoder/decoder.py
User=root
Group=root
Restart=on-failure
KillMode=process

[Install]
WantedBy=multi-user.target
ctrl-d
```

Configure Lighthouse:

```
cat > /etc/systemd/system/lighthouse.service
[Unit]
Description="Lighthouse service"
After=network.target

[Service]
Environment="HOME=/root"
Environment="PORT=50005"
ExecStart=/usr/bin/node /root/lighthouse/dist/index.js
User=root
Group=root
Restart=on-failure
KillMode=process

[Install]
WantedBy=multi-user.target
ctrl-d
```

Activate the services:

```
systemctl daemon-reload
systemctl enable decoder.service
systemctl start decoder.service
systemctl enable lighthouse.service
systemctl start lighthouse.service
```

Wait a minute or so, then test the service:

```
netstat -plant | grep "50005"
````

And you should have a running service on TCP/50005 which is accessible to the world.

```
tcp6       0      0 :::50005                :::*                    LISTEN      1043/node 
```

```
curl http://127.0.0.1:50005/status
```
