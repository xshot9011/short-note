# Versioning

- Graylog 4.1
- Elasticsearch 6.8, and version 7 up to 7.10
- MongoDB 3.6, 4.0, 4.2 or 4.4
- Oracle Java SE 8 (OpenJDK 8 also works; latest stable update is recommended)

# Installation (single server version)

- update and upgrade package

```bash
sudo apt-get update --yes && sudo apt-get upgrade
```

for some reason your package don't want to be upgraded so don't pu --yes follow

- Install software dependencies

```bash
sudo apt-get install apt-transport-https openjdk-8-jre-headless uuid-runtime pwgen
```

- If command above fail to install

```bash
sudo add-apt-repository universe
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install apt-transport-https openjdk-8-jre-headless uuid-runtime pwgen
```

## Install Mongodb

```bash
# add key from server to trusted key
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
```

- Restart service

```bash
sudo systemctl daemon-reload
sudo systemctl enable mongod.service
sudo systemctl restart mongod.service
sudo systemctl status mongod
```

## Install Elasticsearch

```bash
wget -q https://artifacts.elastic.co/GPG-KEY-elasticsearch -O myKey
sudo apt-key add myKey
echo "deb https://artifacts.elastic.co/packages/oss-7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update
sudo apt-get install elasticsearch-oss
```

- Configuration elasticsearch

```bash
sudo tee -a /etc/elasticsearch/elasticsearch.yml > /dev/null <<EOT
cluster.name: graylog
action.auto_create_index: false
EOT
```

set the cluster name to graylog and set auto_create_index = false

- Restart Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl restart elasticsearch.service
sudo systemctl elasticsearch
```

## Install graylog

```bash
wget https://packages.graylog2.org/repo/packages/graylog-4.1-repository_latest.deb
sudo dpkg -i graylog-4.1-repository_latest.deb
```

- If you are using graylog (free)

```bash
sudo apt-get update && sudo apt-get install graylog-server
```

- Else:

```bash
sudo apt-get update && sudo apt-get install graylog-server graylog-enterprise-plugins graylog-integrations-plugins graylog-enterprise-integrations-plugins
```

Now the graylog will not start yet; no password is set

Then, you have to set the password and the hash of password

- Configuration Graylog

at: /etc/graylog/server/server.conf -> password_secret, root_password_sha2 and http_bind_address

```bash
# to get hash password from password which lenght more than 16 chars
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```

```conf
password_secret = <your_password>
...
root_password_sha2 = <your_hash_password>
...
http_bind_address = 0.0.0.0:9000
```

- Restart service

```bash
sudo systemctl daemon-reload
sudo systemctl enable graylog-server.service
sudo systemctl restart graylog-server.service
sudo systemctl graylog-server
```

Now: you can access graylog via URL <ip_address>:<graylog_port>

# Installation (ha version)