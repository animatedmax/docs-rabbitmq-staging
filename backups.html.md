---
breadcrumb: RabbitMQ for PCF Documentation
title: RabbitMQ for PCF Documentation
---

# Backups

## Data

The recommended way to backup queues & messages on RabbitMQ is to replicate the messages to another offsite cluster, instead of backing up the files on disk which is a more invasive process.

The recommended setup is to replicate the exchanges & queues to another cluster located elsewhere, potentially offsite.

Dependent upon your use case you could choose to use `Federation` or `Shovel`. [Read here for more details](http://www.rabbitmq.com/distributed.html)

This can be configured through the management dashboard. 

Mirroring to this second cluster will give you a duplicated set of messages, which will fill up the queues. So this cluster needs to be monitored. Your business logic in your application should be able to handle a potentially duplicated message in a fail over scenario, if the messages from this cluster are not being consumed and kept in sync with the source cluster.

## Configuration

To backup the configuration:

* obtain the admin username & password configured in OpsManager for the RabbitMQ tile
* [Follow these steps to login to your OpsManager installation and target the RabbitMQ tile deployment](http://docs.pivotal.io/pivotalcf/customizing/trouble-advanced.html#ssh)
* Change to root with `sudo -i`
* Update your shell path with `export PATH=$PATH:/var/vcap/packages/rabbitmq-server/bin`
* and `export PATH=$PATH:/var/vcap/packages/erlang/bin`
* then run `rabbitmqadmin export rabbit.config`
* copy the `rabbit.config` file to your desired safe location

[See here for more details](https://www.rabbitmq.com/management-cli.html)

# Restore

## Data

Assuming you are restoring to the RabbitMQ tile deployed on PCF, you will first need to re-create your service instances. You can then login into the management dashboard and setup a `Federation` or `Shovel` to consume from your secondary cluster.

## Configuration

To restore the configuration:

* Navigate to the RabbitMQ tile in OpsManager
* On the `Configuration` tab, scroll down and paste in the full config file
  * Note: This will be applied to the whole cluster so ensure there are no instance specific values in there if restoring from elsewhere / a previous rabbitmq deployment.
