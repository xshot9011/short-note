# Installation (ha version)

## Overview Installation

- keep all server time sync via some methods

### We choose

- Graylog 4.1.5
- Elasticsearch 7.10.2
- MongoDB 4.4.9
- Oracle Java SE 8 (OpenJDK 8 also works; latest stable update is recommended)

### Mongodb replica set

Install follow by mobgodb documentation [here](https://docs.mongodb.com/v2.6/tutorial/deploy-replica-set-with-auth/)

- You can create replicaset from stanalone mongodb [here](https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/)
- odd number of replica
- provide authentication access to mongodb [here](https://docs.mongodb.com/v2.6/tutorial/deploy-replica-set-with-auth/)

## Deploy Mongodb Replicaset

### Create keyfile

If you don't use anthentication step, skip this section

This keyfile will be used to authenticate the member in the deployment

Only mongod instances with the correct keyfile can join replicaset

1. Generate the key

    ```bash
    # generate key
    openssl rand -base64 756 > <path_to_keyfile>
    # change to owner readable only
    chmod 400 <path_to_keyfile>
    ```

### Install mongodb (normal)

[Installation documentation](https://docs.mongodb.com/manual/installation/#std-label-tutorial-installation)

```bash
# import public key used by package management system
sudo apt-get install gnupg
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
# Create list file with ubuntu version
## touch /etc/apt/sources.list.d/mongodb-org-4.4.list
# Write package to file
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
# Reload
sudo apt-get update
# Install mongodb
sudo apt-get install mongodb-org --yes
```

Reload service

```bash
# Reload config
sudo systemctl daemon-reload
sudo systemctl enable mongod.service
sudo systemctl restart mongod.service
sudo systemctl status mongod
```

### Install mongosh 

- This can done by any host; we will use mongosh connect to mongod instance and initiate replicaset

note: We select only node1 will install this tool
note: If you use "mongo" instead no need to install this tool

[Installation documentation](https://docs.mongodb.com/mongodb-shell/install/#std-label-mdb-shell-install)

```bash
# import public key used by package management system
sudo apt-get install gnupg
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
# Create list file with Ubuntu version focal
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
# Reload local package
sudo apt-get update
# Install
sudo apt-get install -y mongodb-mongosh
```

### Copy the keyfile created above

Copy the keyfile to all machine that run mongod and make sure that the keyfile are read only by user mongo

```bash
# list the content of file
ls -al
# change owner to user mongodb
sudo chown mongodb:mongodb <path_to_keyfile>
# change permission to user mongodb
sudo chmod 400 <path_to_keyfile>
```

### Configuration for mongodb

For each mongod instance will have the same configurations as here

host: localhost make mongodb only accept connection that are running on the same machine.

When possible, use a logical DNS hostname instead of an ip address, particularly when configuring replica set members or sharded cluster members. The use of logical DNS hostnames avoids configuration changes due to ip address changes.

1. via commandline

    ```bash
    # Bind IP to mongodb for accessing over network
    # use , to seperate the host, lovcalhost = only access from that machine can connect to database
    # mongod --bind_ip localhost,mv_ip address
    mongod --bind_ip localhost,<hostname(s)|IP_address(es)>
    # Create replicaset
    mongod --replSet "rs0"
    # If authentication is required
    mongod --keyFile <path_to_keyfile>
    ```

2. via configuration file at /etc/mongod.conf

    ```conf
    ...
    security:
        keyFile: <path-to-keyfile>
    ...
    replication:
        replSetName: "rs0"
    ...
    net:
        bindIp: localhost,<hostname(s)|ip address(es)>
    ...
    ```

    Then, start mongod with configuration file 

    ```bash
    sudo mongod --config <path_to_configuration_file>
    ```

I'm not sure wheter we have to restart service or not, just restart nothing

```bash
sudo systemctl daemon-reload
sudo systemctl restart mongod.service
sudo systemctl status mongod
```

### Initial replicaset mongodb over localhost interface

localhost interface is done when you already create first user account, so user account need to has permission to create others user account too

Connect to mongod (one mongod instance only !!!!) 

```bash
# 1. Mongosh 
mongosh "<mongodb://<host>:<port>, ...>/[?replicaSet=<replicaset name>]&[tls=<true|false>]" [--username <username>] [--authenticationDatabase <database>]
# 2. Service
mongo
```

1. Initiate replicaset
   ```bash
   rs.initiate({
      _id : "<replicaset_name>",
      members: [
         { _id: 0, host: "<host1:[port]>" },
         { _id: 1, host: "<host2:[port]>" },
         { _id: 2, host: "<host3:[port]>" }
      ]
   })
   ```
   Example usage
   ```bash
   rs.initiate({_id : "<replicaset_name>",members: [{ _id: 0, host: "<host1:[port]>" },{ _id: 1, host: "<host2:[port]>" },{ _id: 2, host: "<host3:[port]>" }]})
   ```
2. After that
   ```bash
   # To ensure that the replicaset has primary node
   rs.status()
   ```

### Create the user administrator

If you don't use anthentication step, skip this section

In mongosh shell

note: passwordPrompt() allow the mongosh prompts for the password.

```bash
# get getSiblingDB -> used to return another database without modifying the db variable in the shell environment.
# Require minimum role for user "userAdminAnyDatabase"
db.getSiblingDB("admin").createUser(
  {
    user: "admin-big",  # admin of database
    pwd: passwordPrompt(), // or cleartext password
    roles: [ 
      { role: "userAdminAnyDatabase", db: "admin" }, 
      { role: "root", db: "admin"}
    ]
  }
)
```

One line command

```bash
db.getSiblingDB("admin").createUser({user: "admin-big",pwd: passwordPrompt(), roles: [{role: "userAdminAnyDatabase", db: "admin"}, {role: "root", db: "admin"}]})
```

### Authenticate as user administrator and create the cluster administrator

If you don't use anthentication step, skip this section

Authenticate created user, with linux shell

```bash
# with mongosh shell
mongosh -u "<created_username>" -p  --authenticationDatabase "admin"
```

Create cluster adminsitrator

The clusterAdmin role grants access to replication operations, such as configuring the replica set.

```bash
db.getSiblingDB("admin").createUser(
  {
    "user" : "big",
    "pwd" : passwordPrompt(),     // or cleartext password
    roles: [ { "role" : "clusterAdmin", "db" : "admin" } ]
  }
)
```

One line command

```bash
db.getSiblingDB("admin").createUser({"user": "big", "pwd": passwordPrompt(), roles: [{"role": "clusterAdmin", "db": "admin"}]})
```

### Create graylog accessment

1. authenticate as admin to set up database for graylog
    ```bash
    mongosh -u "admin-big" -p  --authenticationDatabase "admin" 
    ```
2. Create database for graylog
    [doc](https://docs.mongodb.com/v4.4/reference/method/db.createCollection/)
    ```bash
    db.getSiblingDB("admin").createCollection("<graylog_database_name>")
    # To verify
    use admin
    show collections
    ```
3. Create user with authorization to database (readWrite, dbAdmin)
    ```bash
    use <graylog_database_name>
    db.createUser(
      {
        user: "<graylog_username>",
        pwd: "<graylog_password>",
        roles: [
          { role: "readWrite", db: "<graylog_database_name>" },
          { role: "dbAdmin", db: "<graylog_database_name>" }
        ]
      }
    )
    # To verify
    use admin
    show users
    ```
    One line command
    ```bash
    db.createUser({user: "<graylog_username>", pwd: "<graylog_password>", roles: [{ role: "readWrite", db: "<graylog_database_name>"},{role: "dbAdmin", db: "<graylog_database_name>" }]})
    # To verify
    use <graylog_database_name>
    show collections
    ```
4. Verify authentication
    ```bash
    # Exit from above command line and login as graylog user to graylog database
    mongosh -u "<graylog_username>" -p  --authenticationDatabase "<graylog_database_name>" 
    use graylogs
    show collections
    ```

## Deploy elasticsearch cluster

### Install elasticsearch

```bash
# Add and install public key
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
# Add es to source list
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
# Reload
sudo apt-get update
# Install es
sudo apt-get install elasticsearch=7.10.2
```

### Config elasticsearch

***** Future add: authentication to elasticsearch

Then, we will config es to access over the network at /etc/elasticsearch/elasticsearch.yml

It is important to name the Elasticsearch cluster not simply named elasticsearch to avoid accidental conflicts with Elasticsearch nodes using the default configuration. Just choose anything else (we recommend graylog), because this is the default name and any Elasticsearch instance that is started in the same network will try to connect to this cluster.

```yml
...
cluster.name: <cluster-name>  # the same
node.name: <description_node_name>  # up to the node
network.host: <host_IP_address>  # node IP address
discovery.seed_hosts: ["<host_IP_address>"] # master ip address
cluster.initial_master_nodes: ["<master_node_name>"]  # master node name(s)
...
```

You can show more log with deleting --quite in ExecStarts of systemd service

Then, Reload configurations

```bash
sudo systemctl daemon-reload
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
sudo systemctl status elasticsearch
```

### Verfify usage

```bash
# send data to node1 
curl -XPUT 'http://<node1>:9200/mybucket/post/1' \
-H 'Content-Type: application/json' \
-d '
{
    "user": "rahul",
    "postDate": "01-16-2015",
    "body": "Adding Data in ElasticSearch Cluster" ,
    "title": "ElasticSearch Cluster Test"
}'
# get data from node2
curl 'http://<noed2>:9200/mybucket/post/_search?q=user:rahul&pretty=true'
```

### Note:

If secure elasticsearch with user authentication [docs](https://www.elastic.co/guide/en/x-pack/5.4/xpack-security.html#preventing-unauthorized-access), need to tell graylog [docs](https://github.com/Graylog2/graylog2-server/blob/2.3.0-beta.1/misc/graylog.conf#L172-L178)

## Deploy Graylog

### Install graylog

```bash
wget https://packages.graylog2.org/repo/packages/graylog-4.1-repository_latest.deb
sudo dpkg -i graylog-4.1-repository_latest.deb
sudo apt update
sudo apt install -y apt-transport-https openjdk-8-jre-headless uuid-runtime pwgen curl dirmngr
```

If you are using free graylog

```bash
sudo apt-get update && sudo apt-get install graylog-server
```

Else

```bash
sudo apt-get update && sudo apt-get install graylog-server graylog-enterprise-plugins graylog-integrations-plugins graylog-enterprise-integrations-plugins
```

### Config graylog

First, generate password for graylog

```bash
# This is for password_secret in graylog
echo "Password is: $(pwgen -N 1 -s 96)"
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```

at: /etc/graylog/server/server.conf

```conf
password_secret = <password_above>
root_password_sha2 = <hashed_password>
is_master = true  # only 1 node another is false
http_bind_address = <host_IP_address:9000>
# # List of Elasticsearch hosts Graylog should connect to.
elasticsearch_hosts = <http://[user:password@]<node_host>[:port]>, ...
# Set the number of log messages to keep per index; it is recommended to have several smaller indices instead of larger ones.
elasticsearch_max_docs_per_index = 20000000
# The following parameter defines to have a total number of indices if this number is reached old index will be deleted.
elasticsearch_max_number_of_indices = 20
# Shards setting rely on the number of nodes in the particular Elasticsearch cluster. If you have only one node, set it as 1.
elasticsearch_shards = 3
# This setting defines the number of replicas for your indices. If you have only one node in the Elasticsearch cluster, then set it as 0.
# I'm not sure so I  set it to 3
elasticsearch_replicas = 3
# See https://docs.mongodb.com/manual/reference/connection-string/ for details
mongodb_uri = mongodb://USERNAME:PASSWORD@mongodb-node01:27017,mongodb-node02:27017,mongodb-node03:27017/graylog?replicaSet=rs01
```

Reload config

```bash
sudo systemctl restart graylog-server
sudo systemctl enable graylog-server
sudo systemctl status graylog-server
```

Read the log

```bash
sudo tail -f /var/log/graylog-server/server.log
```