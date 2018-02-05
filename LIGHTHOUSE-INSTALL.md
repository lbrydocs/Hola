
## Prerequisites:

The Lighthouse search engine depends on version 5.x of the Elasticsearch service and does not support 6.x at this time.

apt install curl apt-transport-https
curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-5.x.list
apt update
apt install elasticsearch

Enable the Elasticsearch service with these commands:

> systemctl daemon-reload
> systemctl enable elasticsearch.service
> systemctl start elasticsearch.service

The service does not start instantly, but should come up in less than a minute. You can check Elasticsearch's status like this:

> netstat -plant | grep "9[23]00"

Lighthouse accesses the Elasticsearch API via TCP/9200. The service on port TCP/9300 is Elasticsearch's inter-cluster communiations port, which we do not use, but which will remain visible so long as the daemon is running.





## Running Lighthouse
### Prerequisites
* Node v8
* Yarn 
* Python2.7
* [Elasticsearch](https://www.elastic.co/downloads/elasticsearch)
