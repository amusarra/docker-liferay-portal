# Liferay 7.2 Community Edition GA1 (Cluster Configuration) Docker Compose Project

[![Antonio Musarra's Blog](https://img.shields.io/badge/maintainer-Antonio_Musarra's_Blog-purple.svg?colorB=6e60cc)](https://www.dontesta.it)
[![Build Status](https://travis-ci.org/amusarra/docker-liferay-portal.svg?branch=7.2.0-ce-ga1-tomcat-postgres-cluster)](https://travis-ci.org/amusarra/docker-liferay-portal)
[![Twitter Follow](https://img.shields.io/twitter/follow/antonio_musarra.svg?style=social&label=%40antonio_musarra%20on%20Twitter&style=plastic)](https://twitter.com/antonio_musarra)

In June 2019 (about five months ago), [Liferay 7.2 GA1 was released](https://liferay.dev/news/liferay-portal-7-2-ce-ga1-release/). The great news was the return of the cluster OOTB, **without installing any other jar**.

This repository contains a [Docker Compose](https://docs.docker.com/compose/overview/) project that allows you to get within a few minutes a Liferay cluster composed of two working nodes.

This Docker Compose contains this services:

1. **lb-haproxy**: HA Proxy (version 2.0.7) as Load Balancer
2. **liferay-portal-node-1**: Liferay 7.2 GA1 (with cluster support) node 1
3. **liferay-portal-node-2**: Liferay 7.2 GA1 (with cluster support) node 2
4. **postgres**: PostgreSQL 10 database
5. **es-node-1** and **es-node-2**: Elasticsearch 6.8.1 Cluster nodes

As for the shared directory for the Liferay document library, I decided to use a shared dock volume instead of NSF.

_Consider that this project is intended for cluster development and testing._ The cluster configuration is the default one with JGroups's UDP multicast, but unicast and TCP are still available.

For more information about _Liferay Cluster_ and _Configure Liferay Portal to Connect to your Elasticsearch Cluster_
 you can read [Liferay Portal Clustering](https://dev.liferay.com/discover/deployment/-/knowledge_base/7-1/liferay-clustering) and [Connect to your Elasticsearch Cluster](https://dev.liferay.com/ca/discover/deployment/-/knowledge_base/7-1/installing-elasticsearch#step-four-configure-liferay-to-connect-to-your-elastic-cluster) on Liferay Developer Network.

The **liferay** directory contains the following items:

1. **OSGi configs** (inside osgi/configs directory)
    * ElasticsearchConfiguration.config: contains elastic cluster configuration
    * AdvancedFileSystemStoreConfiguration.cfg: contains the configuration of the document library
3. **Portal properties** (inside configs directory)
    * portal-ext.properties: contains common configurations for Liferay, such as database connection, cluster enabling, document library, etc.

The **haproxy** directory contains the following items:

1. **HA Proxy**
    * haproxy.cfg: It contains the configuration to expose an endpoint http which balances the two Liferay nodes.

The **elastic** directory contains the optional configuration files for customizing cluster configuration.

The **deploy** directory contains two directories, one for each liferay node, where inside it is possible to copy the bundles to be installed on the respective node.

```bash
deploy
├── liferay-portal-node-1
└── liferay-portal-node-2
```

The fragment of the docker-compose.yml to highlight the bind type volumes dedicated to the deployment of bundles on Liferay nodes.

```yaml
  liferay-portal-node-1:
    build:
      context: .
      dockerfile: Dockerfile-liferay
    hostname: liferay-portal-node-1.local
    volumes:
      - lfr-dl-volume:/data/liferay/document_library
      - ./deploy/liferay-portal-node-1:/etc/liferay/mount/deploy
  liferay-portal-node-2:
    build:
      context: .
      dockerfile: Dockerfile-liferay
    hostname: liferay-portal-node-2.local
    volumes:
      - lfr-dl-volume:/data/liferay/document_library
      - ./deploy/liferay-portal-node-2:/etc/liferay/mount/deploy
```



## 1. Usage

Install docker-compose and run docker with at least 8 GB of memory. The figure below shows the configuration of my Docker installation on macOS Catalina (v10.15).

![docker-configuration](docs/images/docker-configuration.png)



To start a services from this Docker Compose, please run following `docker-compose` command, which will start a Liferay Portal 7.2 GA1 with cluster support running on Tomcat 9.0.17:

For start Liferay Cluster proceed as follows:
```bash
$ docker-compose up -d
```

You can view output from containers following `docker-compose logs` or `docker-compose logs -f` for follow log output.

If you encounter (ERROR: An HTTP request took too long to complete) this issue regularly because of slow network conditions, consider setting COMPOSE_HTTP_TIMEOUT to a higher value (current value: 60).

```bash
$ COMPOSE_HTTP_TIMEOUT=200 docker-compose up -d
```

### 1.1 Check Liferay and Elasticsearch services

To check the successful installation of cluster bundles, you can connect to the Gogo Shell of each node (via telnet) and run the command:

The telnet ports of the GogoShell exposed by the two nodes are respectively: 21311 and 31311

```bash
g! lb -s multiple
```

and you should get the following output. Be careful that the three bundle must be in the active state.

```bash
START LEVEL 20
   ID|State      |Level|Symbolic name
  455|Active     |   10|com.liferay.portal.cache.multiple (3.0.5)|3.0.5
  461|Active     |   10|com.liferay.portal.scheduler.multiple (3.0.2)|3.0.2
  890|Active     |   10|com.liferay.portal.cluster.multiple (3.0.6)|3.0.6
```

On the logs of every Liferay instance you should see logs similar to those shown below.

```bash
liferay-portal-node-1_1  | 2019-11-05 23:38:56.039 INFO  [Start Level: Equinox Container: 1ad0069e-0f93-4499-aac6-bc7da430eb11][JGroupsClusterChannelFactory:158] Autodetecting JGroups outgoing IP address and interface for www.google.com:80
liferay-portal-node-1_1  | 2019-11-05 23:38:56.234 INFO  [Start Level: Equinox Container: 1ad0069e-0f93-4499-aac6-bc7da430eb11][JGroupsClusterChannelFactory:197] Setting JGroups outgoing IP address to 172.21.0.5 and interface to eth0
liferay-portal-node-1_1  |
liferay-portal-node-1_1  | -------------------------------------------------------------------
liferay-portal-node-1_1  | GMS: address=liferay-portal-node-1-51531, cluster=liferay-channel-control, physical address=172.21.0.5:34684
liferay-portal-node-1_1  | -------------------------------------------------------------------
liferay-portal-node-2_1  | 2019-11-05 23:38:56.908 INFO  [Incoming-2,liferay-channel-control,liferay-portal-node-2-29145][JGroupsReceiver:91] Accepted view [liferay-portal-node-2-29145|3] (2) [liferay-portal-node-2-29145, liferay-portal-node-1-51531]
liferay-portal-node-1_1  | 2019-11-05 23:38:56.953 INFO  [Start Level: Equinox Container: 1ad0069e-0f93-4499-aac6-bc7da430eb11][JGroupsReceiver:91] Accepted view [liferay-portal-node-2-29145|3] (2) [liferay-portal-node-2-29145, liferay-portal-node-1-51531]
liferay-portal-node-1_1  | 2019-11-05 23:38:56.957 INFO  [Start Level: Equinox Container: 1ad0069e-0f93-4499-aac6-bc7da430eb11][JGroupsClusterChannel:105] Create a new JGroups channel {channelName: liferay-channel-control, localAddress: liferay-portal-node-1-51531, properties: UDP(...)}
liferay-portal-node-1_1  |
liferay-portal-node-1_1  | -------------------------------------------------------------------
liferay-portal-node-1_1  | GMS: address=liferay-portal-node-1-63028, cluster=liferay-channel-transport-0, physical address=172.21.0.5:52103
liferay-portal-node-1_1  | -------------------------------------------------------------------
liferay-portal-node-2_1  | 2019-11-05 23:38:57.374 INFO  [Incoming-1,liferay-channel-transport-0,liferay-portal-node-2-44641][JGroupsReceiver:91] Accepted view [liferay-portal-node-2-44641|3] (2) [liferay-portal-node-2-44641, liferay-portal-node-1-63028]
liferay-portal-node-1_1  | 2019-11-05 23:38:57.384 INFO  [Start Level: Equinox Container: 1ad0069e-0f93-4499-aac6-bc7da430eb11][JGroupsReceiver:91] Accepted view [liferay-portal-node-2-44641|3] (2) [liferay-portal-node-2-44641, liferay-portal-node-1-63028]
liferay-portal-node-1_1  | 2019-11-05 23:38:57.389 INFO  [Start Level: Equinox Container: 1ad0069e-0f93-4499-aac6-bc7da430eb11][JGroupsClusterChannel:105] Create a new JGroups channel {channelName: liferay-channel-transport-0, localAddress: liferay-portal-node-1-63028, properties: UDP(..)}
```

From this log we see the join between the two cluster Liferay nodes (liferay-portal-node-1 and liferay-portal-node-2).

To check the correct installation of the Elasticsearch cluster, just check the status of the cluster and the presence of the Liferay indexes. We can use REST services to get this information:

1. General info on installation of the Elasticsearch
2. Cluster Status
3. Indexes Status



```bash
$ curl http://localhost:9200
```

In output you should get the general info of the Elasticsearch.

```bash
{
  "name" : "ExxDLIz",
  "cluster_name" : "docker-elasticsearch",
  "cluster_uuid" : "owMQWJH9SRq3e9u3WE00pw",
  "version" : {
    "number" : "6.8.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "1fad4e1",
    "build_date" : "2019-06-18T13:16:52.517138Z",
    "build_snapshot" : false,
    "lucene_version" : "7.7.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```



```bash
$ curl "http://localhost:9200/_cluster/health?pretty"
```

In output you should get the status of the cluster in green and the presence of two nodes.

```bash
{
  "cluster_name" : "docker-elasticsearch",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 2,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 4,
  "active_shards" : 5,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

```bash
$ curl http://localhost:9200/_cat/indices
```

In output you should also get Liferay indices.

```bash
green open liferay-20101               crHED_E6Qi6gkaampSf4XA 1 0  86  2 123.4kb 123.4kb
green open liferay-20099               0wMuMUAkRH6kzcNmlraJqg 1 0   0  0    262b    262b
green open liferay-0                   mRcT7AGVRSqVwHvBz1wHcg 1 0 183 14 945.6kb 945.6kb
green open .monitoring-es-6-2019.10.23 IY6EW_hDR0m8Gc5GmZUVLA 1 1 282 20 713.8kb 466.3kb
```

You can check if all the services have gone up using the `docker-compose ps` command.

![docker-compose-ps-command-output](docs/images/docker-compose-ps-command-output.png)

After all the services are up, you can reach Liferay this way:

1. Via HA Proxy or Load Balancer at URL http://localhost
2. Accessing directly to the nodes:
  * Liferay Node 1: http://localhost:6080
  * Liferay Node 2: http://localhost:7080
  
  

![liferay-portal-72-node-1](docs/images/liferay-portal-72-node-1.png)

![liferay-portal-72-node-2](docs/images/liferay-portal-72-node-2.png)

You can access the HA Proxy statistics report in this way: http://localhost:8181 (username/password: liferay/liferay)

![ha-proxy-statistics-report](docs/images/ha-proxy-statistics-report.png)

In my case, I inserted the following entries on my /etc/hosts file:

```bash
##
# Liferay 7.2 CE GA1 Cluster
##
127.0.0.1	liferay-portal-node-1.local
127.0.0.1	liferay-portal-node-2.local
127.0.0.1	liferay-portal.local
127.0.0.1	liferay-lb.local
```
To access Liferay through HA Proxy goto your browser at http://liferay-lb.local


# License
These project are free software ("Licensed Software"); you can redistribute it and/or modify it under the terms of the GNU Lesser General Public License as published by the Free Software Foundation; either version 2.1 of the License, or (at your option) any later version.

These docker images are distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; including but not limited to, the implied warranty of MERCHANTABILITY, NONINFRINGEMENT, or FITNESS FOR A PARTICULAR PURPOSE. See the GNU Lesser General Public License for more details.

You should have received a copy of the GNU Lesser General Public License along with this library; if not, write to the Free Software Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA
