---
title: RabbitMQ for Pivotal Cloud Foundry&reg;
title: Clustering and Network Partitions
owner: London Services
---

The RabbitMQ tile uses the `pause_minority` option for handling cluster partitions. 
This ensures data integrity by pausing the partition of the cluster in the minority and resuming it with the data from the majority partition. 
You must maintain more than two nodes because if there is a partition when you have just two nodes both nodes immediately pause.

##<a id="detect-partition"></a> Detecting a Network Partition

When a network partition occurs, a log message is written to the RabbitMQ node log:

```
=ERROR REPORT==== 15-Oct-2012::18:02:30 ===
Mnesia(rabbit@da3be74c053640fe92c6a39e2d7a5e46): ** ERROR ** mnesia_event got
    {inconsistent_database, running_partitioned_network, rabbit@21b6557b73f343201277dbf290ae8b79}
```

You can also run the `rabbitmqctl cluster_status` command on any of the RabbitMQ nodes to see the network partition. To run `rabbitmqctl cluster_status`, do the following:

1. `$ sudo su -` 
1. `$ cd /var/vcap/packages`
1. `$ export ERL_DIR=$PWD/erlang/bin/`
1. `$ cd rabbitmq-server/bin/`
1. `$ ./rabbitmqctl cluster_status`

    <pre class="terminal"> [...
     {partitions,
         [{rabbit@da3be74c053640fe92c6a39e2d7a5e46,
              [rabbit@21b6557b73f343201277dbf290ae8b79]}]}]
    </pre> 

##<a id="recovering"></a> Recovering

Because the RabbitMQ tile uses the `pause_minority` option, minority nodes recover automatically after the partition is resolved. 
After a node recovers, it resumes accessing the queue along with data from the queues on the other nodes.
However, if your queues use `ha-mode: all`, they only synchronize fully after consuming all the messages created while the node was down.
This is similar to how messages synchronize when you create a new queue.

###<a id="manual-sync"></a> Manually Synchronizing after a Partition

After a network partition, a queue on a minority node synchronizes after consuming all the messages created while it was down. 
You can also run the `sync_queue` command to synchronize a queue manually. To run `sync_queue`, do the following on each node:

1. `$ sudo su -` 
1. `$ cd /var/vcap/packages`
1. `$ export ERL_DIR=$PWD/erlang/bin/`
1. `$ cd rabbitmq-server/bin/`
1. `$ ./rabbitmqctl list_queues`
1. `$ ./rabbitmqctl sync_queue name`
