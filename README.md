# elasticsearch-cluster
Deploy elasticsearch cluster with security enabled


## Prerequisites
 - ubuntu 18.04
 - elasticsearch version 6.8.1 or newer (newer than v7 not tested)

## Environment setup

### Install java
If deploy on new machine, java should be installed by running following script:
```shell
sudo apt install openjdk-11-jdk
```
### Turn off swap
Running above script to turn off swap
```shell
sudo swapoff -a
```
If you want to disable swap on reboot, you can open file `/etc/fstab` and comment the line starting with `swapfile`

### Set memlock to unlimited and max open file
Open file `/etc/systemd/system.conf`, uncomment 2 fields:
```shell
DefaultLimitNOFILE=<some-big-value>
DefaultLimitMEMLOCK=infinity
```

Open file `/etc/security/limits.conf`, add those lines to the end of file:

```shell
* - nofile 100000
* - memlock unlimited
root - nofile 100000
root - memlock unlimited
```

### Set vm.max_map_count
Open file `/etc/sysctl.conf`, add this line at the end of file:

```shell
vm.max_map_count=300000
```

After setting up all configuration above, restart machine to apply changes

## Elasticsearch setup
### Configure jvm HEAP_SIZE
Edit file `config/jvm.options` and set 2 lines:
```shell
-Xms4g
-Xmx4g
```
You should edit the memory to be half the memory your machine has. Here I set 4g

### Configure elasticsearch
Open file `config/elasticsearch.yml` and set parameters
```shell
cluster.name: <cluster-name>
node.name: <node-name>

node.master: true		# set true if master
node.data: false		# set true if data
node.ingest: false		# set true if ingest (client)

bootstrap.memory_lock: true

network.publish_host: <your-private-ip>
network.host: <`0.0.0.0` or public-ip>

discovery.zen.minimum_master_nodes: <min-master-node-in-cluster>
discovery.zen.ping.unicast.hosts: <list-of-all-nodes-ip-in-cluster>

xpack.security.enabled: false	# here Im putting `false` for no authentication
```
In case you want to setup elasticsearch cluster with basic authentication, you must have SSL certificate as instructed [here](https://www.elastic.co/blog/elasticsearch-security-configure-tls-ssl-pki-authentication)

### Some optional configuration
#### Re-index from a remote host
This option allows you to re-index from a backup cluster
```shell
reindex.remote.whitelist: <remote-ip>:<remote-port>
```
#### Setup repository for snapshot
Your cluster should have a repo for backup
```shell
path.repo: <shared-repo>
```
This config is only needed for master and data nodes
##### Setup shared repo for snapshot