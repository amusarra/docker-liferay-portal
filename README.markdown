# Liferay 7.1 Community Edition GA1 (Cluster Configuration) Docker Compose Project

On October 23, 2017, [Liferay GA5 was released](https://community.liferay.com/news/liferay-portal-7-0-ce-ga5-release/). The great news was the return of the cluster.

This repository contains a [Docker Compose](https://docs.docker.com/compose/overview/) project that allows you to get within a few minutes a Liferay cluster composed of two working nodes.

This Docker Compose contains this services:

1. **lb-haproxy**: HA Proxy as Load Balancer
2. **liferay-portal-node-1**: Liferay 7.1 GA1 (with cluster support) node 1
3. **liferay-portal-node-2**: Liferay 7.1 GA1 (with cluster support) node 2
4. **postgres**: PostgreSQL 10 database
5. **es-node-1** and **es-node-2**: Elasticsearch 6.1.4 Cluster nodes

As for the shared directory for the Liferay document library, I decided to use a shared dock volume instead of NSF.

_Consider that this project is intended for cluster development and testing._


For more information about _Liferay Cluster_ and _Configure Liferay Portal to Connect to your Elasticsearch Cluster (6.1.4)_
 you can read [Liferay Portal Clustering](https://dev.liferay.com/discover/deployment/-/knowledge_base/7-1/liferay-clustering) and [Connect to your Elasticsearch Cluster](https://dev.liferay.com/ca/discover/deployment/-/knowledge_base/7-1/installing-elasticsearch#step-four-configure-liferay-to-connect-to-your-elastic-cluster) on Liferay Developer Network.

The **liferay** directory contains the following items:

1. **Cluster OSGi Bundle** (inside deploy directory)
    * com.liferay.portal.cache.ehcache.multiple.jar (version: [2.0.3](https://mvnrepository.com/artifact/com.liferay/com.liferay.portal.cache.ehcache.multiple/2.0.3))
    * com.liferay.portal.cluster.multiple.jar (version: [2.0.1](https://mvnrepository.com/artifact/com.liferay/com.liferay.portal.cluster.multiple/2.0.1))
    * com.liferay.portal.scheduler.multiple.jar (version: [2.0.2](https://mvnrepository.com/artifact/com.liferay/com.liferay.portal.scheduler.multiple/2.0.2))
2. **OSGi configs** (inside configs directory)
    * BundleBlacklistConfiguration.config: contains the list of bundles that need not be installed
    * ElasticsearchConfiguration.config: contains elastic cluster configuration
    * AdvancedFileSystemStoreConfiguration.cfg: contains the configuration of the document library
3. **Portal properties** (inside configs directory)
    * portal-ext.properties: contains common configurations for Liferay, such as database connection, cluster enabling, document library, etc.

The **haproxy** directory contains the following items:

1. **HA Proxy**
    * haproxy.cfg: It contains the configuration to expose an endpoint http which balances the two Liferay nodes.

The **elastic** directory contains the optional configuration files for customizing cluster configuration.

## Usage
To start a services from this Docker Compose, please run following `docker-compose` command, which will start a Liferay Portal 7.1 GA1 with cluster support running on Tomcat 9.0.6:

For the first start, proceed as follows:
```bash
$ docker-compose up -d liferay-portal-node-1
```

You can view output from containers following `docker-compose logs` or `docker-compose logs -f` for follow log output.

After the first Liferay node is on (liferay-portal-node-1), then run:
```bash
$ docker-compose up -d liferay-portal-node-2
```

After the two Liferay nodes are up, then pull the HA Proxy:
```bash
$ docker-compose up -d lb-haproxy
```

For the next start, you can run the only command:
```bash
$ docker-compose up -d
```

If you encounter (ERROR: An HTTP request took too long to complete) this issue regularly because of slow network conditions, consider setting COMPOSE_HTTP_TIMEOUT to a higher value (current value: 60).

```bash
$ COMPOSE_HTTP_TIMEOUT=200 docker-compose up -d
```

### Check Liferay and Elasticsearch services
To check the successful installation of cluster bundles, you can connect to the Gogo Shell of each node (via telnet) and run the command:

The telnet ports of the GogoShell exposed by the two nodes are respectively: 21311 and 31311

```bash
g! lb -s multiple
```

and you should get the following output. Be careful that the three bundle must be in the active state.

```bash
START LEVEL 20
   ID|State      |Level|Symbolic name
   52|Active     |   10|com.liferay.portal.cache.ehcache.multiple (2.0.3)
   53|Active     |   10|com.liferay.portal.cluster.multiple (2.0.1)
   54|Active     |   10|com.liferay.portal.scheduler.multiple (2.0.2)
```

On the logs of every Liferay instance you should see logs similar to those shown below.

```bash
liferay-portal-node-1_1  | 2018-10-02 08:58:57.681 INFO  [main][BundleStartStopLogger:35] STARTED com.liferay.portal.cache.ehcache.multiple_2.0.3 [52]
liferay-portal-node-1_1  | 2018-10-02 08:58:58.440 INFO  [main][BundleStartStopLogger:35] STARTED com.liferay.portal.cluster.multiple_2.0.1 [53]
liferay-portal-node-1_1  | 2018-10-02 08:58:58.484 INFO  [main][JGroupsClusterChannelFactory:141] Autodetecting JGroups outgoing IP address and interface for www.google.com:80
liferay-portal-node-1_1  | 2018-10-02 08:58:58.524 INFO  [main][JGroupsClusterChannelFactory:180] Setting JGroups outgoing IP address to 172.19.0.5 and interface to eth0
liferay-portal-node-1_1  |
liferay-portal-node-1_1  | -------------------------------------------------------------------
liferay-portal-node-1_1  | GMS: address=liferay-portal-node-1-29833, cluster=liferay-channel-control, physical address=172.19.0.5:43578
liferay-portal-node-1_1  | -------------------------------------------------------------------
liferay-portal-node-1_1  | 2018-10-02 08:59:00.935 INFO  [main][JGroupsReceiver:85] Accepted view [liferay-portal-node-1-29833|0] (1) [liferay-portal-node-1-29833]
```

From this log we see the join between the two cluster Liferay nodes (liferay-portal-node-1 and liferay-portal-node-2).

```bash
liferay-portal-node-2_1  | -------------------------------------------------------------------
liferay-portal-node-2_1  | GMS: address=liferay-portal-node-2-3742, cluster=liferay-channel-transport-0, physical address=172.19.0.6:51427
liferay-portal-node-2_1  | -------------------------------------------------------------------
liferay-portal-node-1_1  | 2018-10-02 09:37:09.679 INFO  [Incoming-1,liferay-channel-transport-0,liferay-portal-node-1-41590][JGroupsReceiver:85] Accepted view [liferay-portal-node-1-41590|1] (2) [liferay-portal-node-1-41590, liferay-portal-node-2-3742]
liferay-portal-node-2_1  | 2018-10-02 09:37:09.686 INFO  [main][JGroupsReceiver:85] Accepted view [liferay-portal-node-1-41590|1] (2) [liferay-portal-node-1-41590, liferay-portal-node-2-3742]
```

To check the correct installation of the Elasticsearch cluster, just check the status of the cluster and the presence of the Liferay indexes. We can use REST services to get this information.

```bash
curl http://localhost:9200/_cluster/health?pretty
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
  "active_shards" : 6,
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
curl curl http://localhost:9200/_cat/indices
```

In output you should also get Liferay indices.

```bash
green open liferay-20099               6tfpxVt7Td6Sc_UY1TUpfA 1 0    0 0  264b  264b
green open liferay-0                   zSf1E3mEQjSGmWPjm7pczg 1 0  149 0 228kb 228kb
green open .monitoring-es-6-2018.10.02 s9jxYLtyS46Bi_okocuVIA 1 1 4772 6 7.7mb 3.7mb
green open .monitoring-es-6-2018.10.01 -gJo6qMARbWjHwo4HGmjhA 1 1 2359 4 2.5mb 1.2mb
```

After all the services are up, you can reach Liferay this way:

1. Via HA Proxy or Load Balancer at URL http://localhost
2. Accessing directly to the nodes:
  * Liferay Node 1: http://localhost:6080
  * Liferay Node 2: http://localhost:7080

You can access the HA Proxy statistics report in this way: http://localhost:8181 (username/password: liferay/liferay)

In my case, I inserted the following entries on my /etc/ hosts file:

```bash
##
# Liferay 7.1 CE GA1 Cluster
##
127.0.0.1	liferay-portal-node-1.local
127.0.0.1	liferay-portal-node-2.local
127.0.0.1	liferay-portal.local
127.0.0.1	liferay-lb.local
```
To access Liferay through HA Proxy goto your browser at http://liferay-lb.local


## JGroups cluster support
This cluster support is limited to EhCache RMI replication. RMI is known to not scale well when increasing the number of cluster nodes. It creates more threads when adding more nodes to the cluster. They can cause server nodes to decrease its performance and even to crash.

[Juan Gonzalez](https://twitter.com/gonpinju) made the needed changes from Liferay 7 CE GA5 sources to change RMI to JGroups.

For more detailed information, I suggest you read [Liferay Portal 7 CE GA5 with JGroups cluster support]( https://web.liferay.com/community/forums/-/message_boards/message/97704861)


# License
These docker images are free software ("Licensed Software"); you can redistribute it and/or modify it under the terms of the GNU Lesser General Public License as published by the Free Software Foundation; either version 2.1 of the License, or (at your option) any later version.

These docker images are distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; including but not limited to, the implied warranty of MERCHANTABILITY, NONINFRINGEMENT, or FITNESS FOR A PARTICULAR PURPOSE. See the GNU Lesser General Public License for more details.

You should have received a copy of the GNU Lesser General Public License along with this library; if not, write to the Free Software Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA
