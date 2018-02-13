# Prerequisites:

Clone this repository as it contains four configuration files as well as the instructions for installing Lighthouse.

### Install Requirements First
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

### Install Service Configuration Files

```
cp decorder.service /etc/systemd/system/
cp lbrycrdd.service /etc/systemd/system/
cp ligthouse.service /etc/systemd/system/
mkdir ~/.lbrycrd
cp lbrycrd.conf ~/.lbrycrd
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
mkdir ~/lbrycrd
cd ~/lbrycrd
wget https://github.com/lbryio/lbrycrd/releases/download/v0.12.1.0/lbrycrd-linux.zip
unzip lbrycrd-linux.zip
```

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
tcp        0      0 127.0.0.1:9245          0.0.0.0:*               LISTEN      1010/lbrycrdd   
tcp        0      0 0.0.0.0:9246            0.0.0.0:*               LISTEN      1010/lbrycrdd 
```

### Install Decoder

```
cd
git clone https://github.com/lbryio/lighthouse.git
cd ~/lighthouse/decoder
pip install -r requirements.txt
```

Active Decoder Service:

```
systemctl daemon-reload
systemctl enable decoder.service
systemctl start decoder.service

```

You can check the Decoder service like this:

```
netstat -plant | grep ":5000 "
```

And it should be found running at port 5000:

```
tcp        0      0 127.0.0.1:5000          0.0.0.0:*               LISTEN      1014/python   
```

### Install Lighthouse

```
cd ~/lighthouse
./gendb.sh
yarn install --production=false
```

Enable the service

```
systemctl daemon-reload
systemctl enable lighthouse.service
systemctl start lighthouse.service
```

Wait a minute or so, then test the service:

```
netstat -plant | grep "50005"
```

And you should have a running service on TCP/50005 which is accessible to the world.

```
tcp6       0      0 :::50005                :::*                    LISTEN      1043/node 
```

You can checked the status with curl:

```
curl http://127.0.0.1:50005/status
```

You should find the spaceUsed parameter climbing steadily, but it may take ten or fifteen minutes to reach its full amount.

```
{"status":"gettingClaimTrie","spaceUsed":"59.1MB","claimsInIndex":0,"totSearches":1}
```

### Final Inspection

The Lighthouse service is the most complex piece of LBRY. This is what you will find when the system is operational.

Check open ports, ignoring ssh on port 22:

```
netstat -plant | grep "LIST" | grep -v ":22"
```

Find Elasticsearch running as Java apps using ports 9200 & 9300 locally. Find the Decoder running as a Python app on port 5000 locally. Find lbrycrdd running on port 9245 locally and port 9246 globally. Finally the Ligthouse service will be running as a node (JavaScript) package on port 50005 globally.

```
tcp        0      0 127.0.0.1:9245          0.0.0.0:*               LISTEN      1010/lbrycrdd   
tcp        0      0 0.0.0.0:9246            0.0.0.0:*               LISTEN      1010/lbrycrdd   
tcp        0      0 127.0.0.1:5000          0.0.0.0:*               LISTEN      1014/python     
tcp6       0      0 127.0.0.1:9200          :::*                    LISTEN      1035/java       
tcp6       0      0 ::1:9200                :::*                    LISTEN      1035/java       
tcp6       0      0 127.0.0.1:9300          :::*                    LISTEN      1035/java       
tcp6       0      0 ::1:9300                :::*                    LISTEN      1035/java       
tcp6       0      0 :::50005                :::*                    LISTEN      1012/node       
```

The lbrycrd daemon will keep a full copy of the LBRY blockchain on disk. You can examine it here:

```
cd ~/.lbrycrd/
du -m
```

This is what disk usage was like the day this documentation was created - almost 1.8 gig of storage used.

```
74	./claimtrie
202	./chainstate
1	./database
209	./blocks/index
1509	./blocks
1789	.
```
