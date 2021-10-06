# Upgrade graylog 3.2 to 4.1

This README is created for upgrading graylog

## Previous version

- mongodb version = 4.2.6
- elasticsearch version = 6.8.15
- graylog version = 3.2.4
- Oracle Java SE 8

## Target version

- mongodb version = 4.2.6 (same)
- elasticsearch version = 7.10.2
- graylog version = 4.1.5
- Oracle Java SE 8 (same)

## Graylog 4.1 system requirements

- Some modern Linux distribution (Debian Linux, Ubuntu Linux, or CentOS recommended)
- Elasticsearch 6.8, and version 7 up to 7.10
- MongoDB 3.6, 4.0, 4.2 or 4.4
- Oracle Java SE 8 (OpenJDK 8 also works; latest stable update is recommended)
- 4 CPU Cores.
- 8 GB RAM.
- SSD Hard Disk Space with High IOPS for Elasticsearch Log Storage.

## Reference

- upgrade to graylog 4.1 [doc](https://github.com/Graylog2/documentation/blob/4.1/pages/upgrade/graylog-4.1.rst)

## Step to upgrade graylog

1. Stop graylog-server
2. Stop mongod 
3. Upgrade graylog-server 
4. Upgrade mongod (skipped)
5. Start mongod 
6. Start graylog-server 
7. Upgrade elasticsearch

### Note

- have a good backup
- are you running a MongoDB Replica set between your nodes? That may require some special handling and might require you to upgrade MongoDB on both nodes first.
- are you running an ES cluster?
- make sure your journal size is adequate to buffer any messages
- Graylog 3.x supports ES 5.x and 6.x, which are you starting from? There may need to be some special handling of the indices depending on the version you are on and if you had Indices from 5.x.

# migration and upgrde step

## pre migration

### Backup mongodb

```bash
mongodump --host localhost:27017 -u graylog -p password -d graylog -o /backup/<file>.mongodump
```

### Backup elasticsearch

Snapshots: You cannot restore snapshots from later Elasticsearch versions into a cluster running an earlier Elasticsearch version. For example, you cannot restore a snapshot taken in 7.6.0 to a cluster running 7.5.0.

Indices: You cannot restore indices into a cluster running a version of Elasticsearch that is more than one major version newer than the version of Elasticsearch used to snapshot the indices. For example, you cannot restore indices from a snapshot taken in 5.0 to a cluster running 7.0.

1. make directory to store backup

    ```
    mkdir /backup/elasticsearch
    chmod 755 elasticsearch
    sudo chown elasticsearch:elasticsearch /backup/elasticsearch
    ```

2. configuration for elasticsearch [doc](https://help.hcltechsw.com/connections/v6/admin/install/es_install_backup_repos.html)

    ```
    sudo vi  /etc/elasticsearch/elasticsearch.yml
    # inside /etc/elasticsearch/elasticsearch.yml
    path.repo: ["/backup/elasticsearch"]
    ```

3. register snapshot repository [doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository)

    ```bash
    # create repo named "my_backup"
    curl -X PUT "10.88.50.1:9200/_snapshot/my_backup?pretty" -H 'Content-Type: application/json' -d'
    {
        "type": "fs",
        "settings": {
        "location": "/backup/elasticsearch"
        }
    }
    '
    ```

    to verify

    ```bash
    curl -X GET "10.88.50.1:9200/_snapshot/_all?pretty"
    ```

4. create snapshot [doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html)

    ```bash
    # create snapshot "first" in repo "my_backup"
    curl -XPUT "10.88.50.1:9200/_snapshot/my_backup/first?wait_for_completion=true"
    ```

## migraton

1. Stop graylog-server

    ```bash
    sudo systemctl stop graylog-server.service
    ```

2. Stop mongod

    ```bash
    sudo systemctl stop mongod.service
    ```

3. Upgrade graylog-server

    Repeat for each version 
    
    - version 3.2 to 3.3 [doc](https://github.com/Graylog2/documentation/blob/4.1/pages/upgrade/graylog-3.3.rst)
    ```bash
    wget https://packages.graylog2.org/repo/packages/graylog-3.3-repository_latest.deb
    sudo dpkg -i graylog-3.3-repository_latest.deb
    sudo apt-get update
    # to list all ava version
    sudo apt-cache policy graylog-server
    sudo apt-get install graylog-server
    # Package distributor has shipped an updated version. Choose "N"
    ```

    - version 3.3 to 4.0 [doc](https://github.com/Graylog2/documentation/blob/4.1/pages/upgrade/graylog-4.0.rst)
    
    NOTE: dashboard migration need to be rechecked 
    ```bash
    wget https://packages.graylog2.org/repo/packages/graylog-4.0-repository_latest.deb
    sudo dpkg -i graylog-4.0-repository_latest.deb
    sudo apt-get update
    # to list all ava version
    sudo apt-cache policy graylog-server
    sudo apt-get install graylog-server
    # Package distributor has shipped an updated version. Choose "N"
    ```

    - version 4.0 to 4.1 [doc](https://github.com/Graylog2/documentation/blob/4.1/pages/upgrade/graylog-4.1.rst)
    ```bash
    wget https://packages.graylog2.org/repo/packages/graylog-4.1-repository_latest.deb
    sudo dpkg -i graylog-4.1-repository_latest.deb
    sudo apt-get update
    # to list all ava version
    sudo apt-cache policy graylog-server
    sudo apt-get install graylog-server
    # Package distributor has shipped an updated version. Choose "N"
    ```

4. Upgrade mongod (skipped)

5. Start mongod

    ```bash
    sudo systemctl start mongod.service
    sudo systemctl status mongod.service
    ```

6. Start graylog-server

    ```bash
    sudo systemctl start graylog-server.service
    sudo systemctl status graylog-server.service
    ```

7. Upgrade elasticsearch [doc](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/restart-upgrade.html)

    1. Check the [deprecation log](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/logging.html#deprecation-logging) to see if you are using any deprecated features and update your code accordingly.
    2. Review breaking [change](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/migrating-7.10.html#breaking-changes-7.10)
        
        NOTE: [snapshotAPI](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/migrating-7.10.html#breaking_710_snapshot_restore_changes)
    3. If you use any plugins, make sure there is a version of each plugin that is compatible with Elasticsearch version 7.10.x
    4. Test in dev env

    - start upgrading es

    ```bash
    # Disable shard allocation.
    curl -X PUT "10.88.50.1:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d '
    {
        "persistent": {
        "cluster.routing.allocation.enable": "primaries"
        }
    }'
    # Stop indexing and perform a synced flush.
    curl -X POST "10.88.50.1:9200/_flush/synced?pretty"
    # Temporarily stop the tasks associated with active machine learning jobs and datafeeds.
    curl -X POST "10.88.50.1:9200/_ml/set_upgrade_mode?enabled=true&pretty"
    # Shutdown all nodes.
    sudo systemctl stop elasticsearch.service
    # Import key
    wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
    echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
    sudo apt-get update
    sudo apt-cache policy elasticsearch
    sudo apt-get upgrade elasticsearch=7.10.2
    # Reload
    sudo systemctl daemon-reload
    sudo systemctl enable elasticsearch.service
    sudo systemctl start elasticsearch.service
    # Verify upgrade
    curl -X GET "10.88.50.1:9200/?pretty"
    # Reenable allocation
    curl -X PUT "10.88.50.1:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
    {
        "persistent": {
            "cluster.routing.allocation.enable": null
        }
    }'
    curl -X GET "10.88.50.1:9200/_cat/health?pretty"
    curl -X GET "10.88.50.1:9200/_cat/recovery?pretty"
    # Restart machine learning jobs.
    curl -X POST "10.88.50.1:9200/_ml/set_upgrade_mode?enabled=false&pretty"
    sudo systemctl restart elasticsearch.service
    sudo systemctl restart graylog-server.service
    ```

## post migration

- **recheck dashboard**
- recheck stream
- recheck input
- check for deprecation function [issue from graylog 4.0](https://github.com/Graylog2/graylog2-server/issues/9690)
    
    ```bash
    curl -X GET "10.88.50.1:9200/_migration/deprecations?pretty"
    ```

    NOTE:

    ```json
    {
    "gl-events_0" : [
        {
            "level" : "critical",
            "message" : "Index created before 7.0",
            "url" : "https://www.elastic.co/guide/en/elasticsearch/reference/master/breaking-changes-8.0.html",
            "details" : "This index was created using version: 6.8.15"
        },
    ]
    }
    ```

    [solution nิw](https://github.com/Graylog2/graylog2-server/issues/9999#issuecomment-771546868)

# Rollback graylog 4.1 to 3.2

## Previous version

- mongodb version = 4.2.6
- elasticsearch version = 7.10.2
- graylog version = 4.1.5
- Oracle Java SE 8

## Target version

- mongodb version = 4.2.6 (same)
- elasticsearch version = 6.8.15
- graylog version = 3.2.4
- Oracle Java SE 8 (same)

## Graylog 3.2 system requirements

- Some modern Linux distribution (Debian Linux, Ubuntu Linux, or CentOS recommended)
- Elasticsearch 5 or 6
- MongoDB 3.6 or 4.0
- Oracle Java SE 8 (OpenJDK 8 also works; latest stable update is recommended)
- 4 CPU Cores.
- 8 GB RAM.
- SSD Hard Disk Space with High IOPS for Elasticsearch Log Storage.

## Soft rollback

### Rollback elasticsearch

[doc](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/modules-snapshots.html#_shared_file_system_repository)

since elasticsearch has no downgrade; I did was, removed the existing elasticsearch, and installed a new one.

The information stored in a snapshot is not tied to a particular cluster or a cluster name. Therefore it’s possible to restore a snapshot made from one cluster into another cluster. All that is required is registering the repository containing the snapshot in the new cluster and starting the restore process.

The new cluster doesn’t have to have the same size or topology. However, the version of the new cluster should be the same or newer (only 1 major version newer) than the cluster that was used to create the snapshot.

First of all it’s necessary to make sure that new cluster have enough capacity to store all indices in the snapshot. (you can select some indices to restore)

1. stop graylog and elasticsearch

```bash
sudo systemctl stop graylog-server.service
sudo systemctl stop elasticsearch.service
```

2. remove elasticsearch

```bash
# read for relate file
sudo cat /usr/lib/systemd/system/elasticsearch.service
# backup config file
cp /etc/elasticsearch/elasticsearch.yml ./elasticsearch.yml.bak
sudo apt-get purge elasticsearch
sudo rm -rf /var/lib/elasticsearch
```

3. reinstall elasticsearch 6.8.15

```bash
sudo apt-get install elasticsearch=6.8.15
cp ./elasticsearch.yml.bak /etc/elasticsearch/elasticsearch.yml
sudo systemctl daemon-reload
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
sudo systemctl status elasticsearch
```

4. Start graylog for checking connection 

```bash
sudo systemctl start graylog-server.service
# go to web browser and search for message; if u get nothing, u come to the right way
sudo systemctl stop graylog-server.service
```

5. Restore snapshot


```bash
# to create repo (actually this one is connect to the same repo)
curl -X PUT "10.88.50.1:9200/_snapshot/my_backup?pretty" -H 'Content-Type: application/json' -d'
    {
        "type": "fs",
        "settings": {
        "location": "/backup/elasticsearch"
        }
    }
    '
# check for what indices in snapshot in "my_backup" repo 
curl -X GET "10.88.50.1:9200/_snapshot/my_backup/_all?pretty"
# close all default index
curl -X POST "10.88.50.1:9200/_all/_close?pretty"

## choose wisely
# restore to "first" snapshot in repo "my_backup"
curl -X POST "10.88.50.1:9200/_snapshot/my_backup/first/_restore?pretty"
# restore selected indices
curl -X POST "10.88.50.1:9200/_snapshot/my_backup/first/_restore?pretty" -H 'Content-Type: application/json' -d'
{
  "indices": "<index_name>, ...",
  "ignore_unavailable": true,
  "include_global_state": true,
  "rename_pattern": "index_(.+)",
  "rename_replacement": "restored_index_$1"
}'
```

6. Start graylog for checking connection 

```bash
sudo systemctl start graylog-server.service
# go to web browser and search for message; if u get old things, u come to the right way
```

### Rollback graylog

[community](https://community.graylog.org/t/graylog-downgrade/17995)

Most of configuration is stored in mongodb

1. Stop graylog for preparing upgrading

```bash
sudo systemctl stop graylog-server.service
```

2. Backup server config file

```bash
sudo cp /etc/graylog/server/server.conf ./server.conf.bak
```

3. Downgrade version

```bash
sudo dpkg -r graylog-4.1-repository
sudo apt-get purge graylog-server
sudo apt-get update
sudo apt-cache policy graylog-server
sudo apt-get install graylog-server=3.2.4-1
```

4. Restore server.conf and verify setting

```bash
sudo cp ./server.conf.bak /etc/graylog/server/server.conf
sudo systemctl enable graylog-server.service
sudo systemctl start graylog-server.service
sudo systemctl status graylog-server.service
# wait untrill port 9000 open
watch -n 1 "sudo lsof -Pi | grep LISTEN"
```

### Post rollback

- check dashboard
- check input
- check msg
- check indices

## hard rollback

## revert snapshot