# Versioning

- Graylog 4.1
- Elasticsearch 6.8, and version 7 up to 7.10
- MongoDB 3.6, 4.0, 4.2 or 4.4
- Oracle Java SE 8 (OpenJDK 8 also works; latest stable update is recommended)

## We choose

- Graylog 4.1
- Elasticsearch 7.10
- MongoDB 4.4
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

## Overview Installation

- keep all server time sync via some methods

### Mongodb replica set

Install follow by mobgodb documentation [here](https://docs.mongodb.com/v2.6/tutorial/deploy-replica-set-with-auth/)

- You can create replicaset from stanalone mongodb [here](https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/)
- odd number of replica
- provide authentication access to mongodb [here](https://docs.mongodb.com/v2.6/tutorial/deploy-replica-set-with-auth/)

## Deploy Mongodb Replicaset

### We choose

- Graylog 4.1
- Elasticsearch 7.10
- MongoDB 4.4.9
- Oracle Java SE 8 (OpenJDK 8 also works; latest stable update is recommended)

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
2. Copy the keyfile to all machine that run mongod and make sure that the keyfile are read only by user mongo

    ```bash
    # list the content of file
    ls -al
    # change owner to user mongodb
    chown mongodb:mongodb <path_to_keyfile>
    # change permission to user mongodb
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
# Reload config
sudo systemctl daemon-reload
sudo systemctl enable mongod.service
sudo systemctl restart mongod.service
sudo systemctl status mongod
```

### Install mongosh 

- This can done by any host; we will use mongosh connect to mongod instance and initiate replicaset

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

### Configuration for mongodb

Copy the keyfile to machine that run mongod and make sure that the keyfile are read only by user mongo

```bash
# list the content of file
ls -al
# change owner to user mongodb
sudo chown mongodb:mongodb <path_to_keyfile>
# change permission to user mongodb
sudo chmod 400 <path_to_keyfile>
```

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

    Then, verify the connection

    ```bash
    mongosh --host <IP_address>
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
    mongod --config <path_to_configuration_file>
    ```

I'm not sure wheter we have to restart service or not, just restart nothing

```bash
sudo systemctl daemon-reload
sudo systemctl restart mongod.service
sudo systemctl status mongod
```

### Initial replicaset mongodb

Connect to mongod (one mongod instance only !) 

```bash
# Mongosh 
mongosh "<mongodb://<host>:<port>, ...>/[?replicaSet=<replicaset name>]&[tls=<true|false>]" [--username <username>] [--authenticationDatabase <database>]
# Service
mongo
```

from mongosh shell we wil initiate the replicaset

```bash
rs.initiate()

rs.initiate( {
   _id : "<replicaset_name>",
   members: [
      { _id: 0, host: <host1> },
      { _id: 1, host: <host2> },
      { _id: 2, host: <host3> }
   ]
})

rs.conf()
{
   "_id" : "rs0",
   "version" : 1,
   "protocolVersion" : NumberLong(1),
   "members" : [
      {
         "_id" : 0,
         "host" : "mongodb0.example.net:27017",
         "arbiterOnly" : false,
         "buildIndexes" : true,
         "hidden" : false,
         "priority" : 1,
         "tags" : {

         },
         "secondaryDelaySecs" : NumberLong(0),
         "votes" : 1
      },
      {
         "_id" : 1,
         "host" : "mongodb1.example.net:27017",
         "arbiterOnly" : false,
         "buildIndexes" : true,
         "hidden" : false,
         "priority" : 1,
         "tags" : {

         },
         "secondaryDelaySecs" : NumberLong(0),
         "votes" : 1
      },
      {
         "_id" : 2,
         "host" : "mongodb2.example.net:27017",
         "arbiterOnly" : false,
         "buildIndexes" : true,
         "hidden" : false,
         "priority" : 1,
         "tags" : {

         },
         "secondaryDelaySecs" : NumberLong(0),
         "votes" : 1
      }
      
   ],
   "settings" : {
      "chainingAllowed" : true,
      "heartbeatIntervalMillis" : 2000,
      "heartbeatTimeoutSecs" : 10,
      "electionTimeoutMillis" : 10000,
      "catchUpTimeoutMillis" : -1,
      "getLastErrorModes" : {

      },
      "getLastErrorDefaults" : {
         "w" : 1,
         "wtimeout" : 0
      },
      "replicaSetId" : ObjectId("585ab9df685f726db2c6a840")
   }
}
# To ensure that the replicaset has primary node
rs.status()
```

### Create the user administrator

If you don't use anthentication step, skip this section

In mongosh shell

note: passwordPrompt() allow the mongosh prompts for the password.

```bash
# Require minimum role for user "userAdminAnyDatabase"
db.createUser()

admin = db.getSiblingDB("admin")
admin.createUser(
  {
    user: "fred",
    pwd: passwordPrompt(), // or cleartext password
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```

### Authenticate as user administrator and create the cluster administrator

If you don't use anthentication step, skip this section

Authenticate created user

```bash
mongosh -u "<created_username>" -p  --authenticationDatabase "admin"
# Or in mongosh shell
db.getSiblingDB("admin").auth("<created_username>", passwordPrompt()) 
```

Create cluster adminsitrator

The clusterAdmin role grants access to replication operations, such as configuring the replica set.

```bash
db.getSiblingDB("admin").createUser(
  {
    "user" : "ravi",
    "pwd" : passwordPrompt(),     // or cleartext password
    roles: [ { "role" : "clusterAdmin", "db" : "admin" } ]
  }
)
```

### After installation

1. Create the replica set
2. Create database for graylog
3. Create user with authorization to database (read, write)
4. (optional) bind IP address for access over the network
