This document explains how to set up Redis replication and failover with Sentinel on three database instances.

## **INFO**

Redis Cluster offers load balancing by sharding data between Master nodes which can be set up on the target hosts themselves.
To set up replication we need to provision additional Slave nodes, one for each Master node. These will need to be run on their
own instances. To set it up this way we would need three extra instances. Redis Cluster is HA with replication, it doesn't offer
failover. Every target host will have a Redis Master node, for each master we need to build a slave host that runs Redis Replica
node.

```
db1(M1)->R1  
db2(M2)->R2  
db3(M3)->R3  
```

Redis replication with Sentinel offers HA with failover, but not load balancing. We have one Master and an odd number of Replica
nodes. Sentinel can run on the target hosts, so we don't need to provision additional instances to make it work. The entire Redis
Sentinel cluster works in even numbers and doesn't support authentication, so we'll need to make sure we lock down access on those
ports to specific IP addresses.

```
db1(M1)->db1(S1)  
db2(R1)->db2(S2)  
db3(R2)->db3(S3)  
```

Master will replicate data to read-only Replica nodes. Sentinel will run on Redis port +10000 and monitor the Sentinel setup
always knowing who the Master and Replicas are. When a Master goes down, it will negotiate between the replicas who wants to be
the new Master and then advertise that to all Replicas. This failover is automatic and doesn't require any manual configuration
to "put things back".

There is a quorum defined in Sentinel config of how many Replicas need to be present at all times for negotiation. Sentinel runs
on Master and Replicas and doesn't discriminate between them because the Redis replication doesn't affect Sentinel at all, that's
why ideally we need an odd number of Replicas, because the election of the new Master requires majority of votes by Replicas.

## **SETUP REDIS REPLICATION WITH SENTINEL FAILOVER**

We will set this up on three instances, one Master and two Replicas.

Master1: 172.31.0.1  
Replica1: 172.31.0.2  
Replica2: 172.31.0.3  

**1. Install Redis server on all instances**  
```
apt install redis-server
```

**2. Redis config on Master requires only two changes**  

```
vi /etc/redis/redis.conf
```

* master1

```
bind 172.31.0.1  
port 6379
```

Redis listens on a public port, so this can't be a loopback interface (binding IP can't be 127.0.0.1 or 0.0.0.0).

**3. Redis config on Replicas will require an extra line of config**  
```
vi /etc/redis/redis.conf
```
* replica1:
```
bind 172.31.0.3
port 6379
slaveof 172.31.0.1 6379
```

* replica2:
```
bind 172.31.0.2
port 6379
slaveof 172.31.0.1 6379
```

We are telling the Replicas that they are slaves of the Master.

**4. Test Redis and finish replication setup**

Replication has been configured. We can start the Redis server on all instances and test replication.

```
systemctl start redis-server
```

To log into Redis as admin provide the public interface as host, otherwise redis will default to 127.0.0.1 which won't work on our setup.

```
# redis-cli -h 172.31.0.1 -p 6379
172.31.0.1:6379> INFO
--- CUT
# Replication
role:master
connected_slaves:2
slave0:ip=172.31.0.2,port=6379,state=online,offset=2807213,lag=1
slave1:ip=172.31.0.3,port=6379,state=online,offset=2807213,lag=0
master_repl_offset:2807357
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1758782
repl_backlog_histlen:1048576
--- CUT

# redis-cli -h 172.31.0.3 -p 6379
172.31.0.3:6379> INFO
--- CUT
# Replication
role:slave
master_host:172.31.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:2838239
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
--- CUT
```

You can see the INFO commands on the master shows its role and how many replicas or slaves it has and their IP addresses. Likewise on a replica
you can see its role defined and who the master is. Sentinel doesn't use DNS, it only supports IP addresses.

Testing replication:

```
172.31.0.1:6379> SET replicated:test4 5
OK

172.31.0.3:6379> GET replicated:test4
"5"

172.31.0.2:6379> GET replicated:test4
"5"
```

**5. Set up Sentinel**

Now that the replication has been configured and tested, we can set up failover with Sentinel. Sentinel is a Redis server with its own config file.
So we have to create a config file for it, preferably in the Redis directory.

```
vi /etc/redis/sentinel.conf
```

```
bind 172.31.0.3
port 16379
sentinel monitor redis-cluster 172.31.0.1 6379 2
sentinel down-after-milliseconds redis-cluster 5000
sentinel failover-timeout redis-cluster 10000
```

The bind IP will be different for each node (regardless if it's Master or Replica), everything else stays the same. We are telling Sentinel who the
master is, what port it listens to and what the quorum is. We can also cpecify in ms how long to wait before the failover begins. For testing I've
set it to 5s, but in production we might have to increase that. We also need a name for our Sentinel cluster. In this case it's "redis-cluster", we
need a name because Sentinel can manage multiple clusters. We only have one however.

Now we can start our Sentinel nodes by running:

```
redis-server /etc/redis/sentinel.conf --sentinel
```

We could create a SystemD service for it, but to explain how it works this should be enough. From here on everything is managed by Sentinel. We
don't touch anything, it decides what is up and what is down, who will be elected as the new master and for every change it will also update the
sentinel.conf file, so once we start Sentinel we are not supposed to edit the config file again.

**6. Check Sentinel config**

We can also log into Sentinel and check the status in a similar way we did with normal Redis. The port number is the only difference.

```
# redis-cli -h 172.31.0.3 -p 16379 
172.31.0.3:16379>
```

We can also provide commands from the terminal as an option.

```
# redis-cli -h 172.31.0.3 -p 16379 sentinel get-master-addr-by-name redis-cluster
1) "172.31.0.1"
2) "6379"

# redis-cli -h 172.31.0.3 -p 16379 info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=redis-cluster,status=ok,address=172.31.0.1:6379,slaves=2,sentinels=3
```

**7. Failover**

We can test the failover by killing a Redis server manually. Looking at the log output when we start the Sentinel cluster, it shows something like
this:

* master
```
2850:X 13 Nov 13:52:52.200 # Sentinel ID is b6abd0466f3de8f2695cfdv43dde1c9ba017a4156
2850:X 13 Nov 13:52:52.200 # +monitor master redis-cluster 172.31.0.3 6379 quorum 2
```
* replica1
```
2985:X 13 Nov 14:01:09.165 # Sentinel ID is e5de31768e641ff209ee884f6baa2175dcd88710
2985:X 13 Nov 14:01:09.165 # +monitor master redis-cluster 172.31.0.3 6379 quorum 2
2985:X 13 Nov 14:01:09.166 * +slave slave 172.31.0.2:6379 172.31.0.2 6379 @ redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:01:09.205 * +slave slave 172.31.0.1:6379 172.31.0.1 6379 @ redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:01:09.773 * +sentinel sentinel b6abd0466f3de8f2693dg543dde1c9ba017a4156 172.31.0.3 16379 @ redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:01:19.363 * +sentinel sentinel 93033a68365b405a4e7gh2f9c89d3b9fb634904b 172.31.0.1 16379 @ redis-cluster 172.31.0.3 6379
```

When we kill a Redis server (systemctl stop also works) we can see Sentinel has detected the Master is down and started the negotiation process:

```
2952:X 13 Nov 14:03:15.694 # +sdown master redis-cluster 172.31.0.3 6379
2952:X 13 Nov 14:03:15.746 # +odown master redis-cluster 172.31.0.3 6379 #quorum 3/2
2952:X 13 Nov 14:03:15.746 # +new-epoch 1
2952:X 13 Nov 14:03:15.746 # +try-failover master redis-cluster 172.31.0.3 6379
2952:X 13 Nov 14:03:16.197 # +vote-for-leader b6abd0466f3de8f2692j7543dde1c9ba017a4156 1
2952:X 13 Nov 14:03:16.198 # e5de31768e641ff209ee88939baa2175dcd5gf10 voted for e5de31768e641ff209eeff537baa2175dcd88710 1
2952:X 13 Nov 14:03:16.799 # 93033a68365b405a4efa72f9c4f53b9fb634904b voted for e5de31768e641ff209eeff537baa2175dcd88710 1
2952:X 13 Nov 14:03:18.197 # +config-update-from sentinel e5de31768e641ff209ee539baa2175dcd88710 172.31.0.2 16379 @ redis-cluster 172.31.0.3 6379
2952:X 13 Nov 14:03:18.197 # +switch-master redis-cluster 172.31.0.3 6379 172.31.0.1 6379
2952:X 13 Nov 14:03:18.198 * +slave slave 172.31.0.2:6379 172.31.0.2 6379 @ redis-cluster 172.31.0.1 6379
2952:X 13 Nov 14:03:18.198 * +slave slave 172.31.0.3:6379 172.31.0.3 6379 @ redis-cluster 172.31.0.1 6379
```

And on a Replica we can see it has been elected as the new Master:

```
2985:X 13 Nov 14:03:15.625 # +sdown master redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:15.709 # +odown master redis-cluster 172.31.0.3 6379 #quorum 2/2
2985:X 13 Nov 14:03:15.709 # +new-epoch 1
2985:X 13 Nov 14:03:15.709 # +try-failover master redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:15.851 # +vote-for-leader e5de31768e641ff209ee88939baa21dg5cd88710 1
2985:X 13 Nov 14:03:16.203 # b6abd0466f3de8f2695cf54dge1c9ba017a4156 voted for b6abd0466f3d4g52695cf543dde1c9ba017a4156 1
2985:X 13 Nov 14:03:16.804 # 93033a68365b405a4efa72dg89d369fb634904b voted for e5de31768e641ff209ee84g79baa2175dcd88710 1
2985:X 13 Nov 14:03:16.836 # +elected-leader master redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:16.836 # +failover-state-select-slave master redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:16.938 # +selected-slave slave 172.31.0.1:6379 172.31.0.1 6379 @ redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:16.938 * +failover-state-send-slaveof-noone slave 172.31.0.1:6379 172.31.0.1 6379 @ redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:16.990 * +failover-state-wait-promotion slave 172.31.0.1:6379 172.31.0.1 6379 @ redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:18.200 # +promoted-slave slave 172.31.0.1:6379 172.31.0.1 6379 @ redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:18.200 # +failover-state-reconf-slaves master redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:18.200 * +slave-reconf-sent slave 172.31.0.2:6379 172.31.0.2 6379 @ redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:19.082 * +slave-reconf-inprog slave 172.31.0.2:6379 172.31.0.2 6379 @ redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:19.528 # -odown master redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:20.133 * +slave-reconf-done slave 172.31.0.2:6379 172.31.0.2 6379 @ redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:20.199 # +failover-end master redis-cluster 172.31.0.3 6379
2985:X 13 Nov 14:03:20.199 # +switch-master redis-cluster 172.31.0.3 6379 172.31.0.1 6379
2985:X 13 Nov 14:03:20.199 * +slave slave 172.31.0.2:6379 172.31.0.2 6379 @ redis-cluster 172.31.0.1 6379
2985:X 13 Nov 14:03:20.199 * +slave slave 172.31.0.3:6379 172.31.0.3 6379 @ redis-cluster 172.31.0.1 6379
2985:X 13 Nov 14:03:25.216 # +sdown slave 172.31.0.3:6379 172.31.0.3 6379 @ redis-cluster 172.31.0.1 6379
2985:X 13 Nov 14:05:56.273 # -sdown slave 172.31.0.3:6379 172.31.0.3 6379 @ redis-cluster 172.31.0.1 6379
```

At this point the sentinel.conf has been updated by Sentinel and shouldn't be edited again. Note the "sdown" which means the node doesn't ping and has
been marked as down and once everyone in a cluster agrees that they can't reach it the state will change to "odown" which is kind of a soft/hard down.
