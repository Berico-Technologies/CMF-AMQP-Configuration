# Configuring *iptables* for a RabbitMQ Cluster

In this tutorial, we will configure *iptables* and RabbitMQ to allow RabbitMQ brokers to coordinate with each other.  This example builds on the previous tutorial *Configuring **iptables** for a single instance of RabbitMQ*, so we will only demonstrate how to configure *iptables* using the `iptables` command.

More importantly, the steps in this tutorial need to be performed on every node in the RabbitMQ cluster.

1.  Edit the RabbitMQ configuration file to force Erlang to use the specified port range when using the "Erlang Distribution Protocol".

  >  Erlang has a special "inter-process communication" framework allowing processes on the same machine, or across a network, to communicate with each other.  It is this mechanism that RabbitMQ uses to coordinate between brokers.  However, unless told not to do so, Erlang will randomly choose the ports it will use to communicate.

  a.  Edit/Create the RabbitMQ configuration file.

  >  RabbitMQ configuration is stored in the `/etc/rabbitmq/` directory in the `rabbitmq.config` file.  On virgin installs, this file will not exist and will have to be created.  RabbitMQ will detect the presence of `rabbitmq.config` when it is started and configure itself accordingly.  Think of this file as an `overrides` file.  You do not have to fully configure RabbitMQ in this file, merely override the settings you need changed.

  `sudo vi /etc/rabbitmq/rabbitmq.config`

  b.  Instruct the Erlang kernel to use the following port range (9100-9105) for the Erlang Distribution protocol:

  ```[{kernel, [ {inet_dist_listen_min, 9100}, 
                {inet_dist_listen_max, 9105} ]}].```

  > RabbitMQ's configuration is quite literally an Erlang *tuple*.  That's the reason why it has a *funky* syntax.

  c.  Restart RabbitMQ.

  `sudo service rabbitmq-server restart`

2.  Add rules to *iptables* to allow RabbitMQ to communicate with other RabbitMQ brokers.

  a.  First we need to unblock the EPMD port (Erlang Port Mapper Daemon); EPMD is a registry Erlang uses to determine which applications are running and on what port.

  `sudo iptables -I INPUT 1 -p tcp --dport 4369 -j ACCEPT`

  b.  Next, we need to unblock port range RabbitMQ will use for interprocess communication:

  `sudo iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport 9100:9105 -j ACCEPT`

  c.  Save the configuration.

  `sudo service iptables save`

  d.  Finally, restart *iptables*.

  `sudo service iptables restart`

3.  As a final precautionary step, reboot the server.

  `sudo reboot`

  >  The Erlang Distribution protocol requires services to register with a unique name (RabbitMQ uses "rabbit").  Sometimes the Erlang process registered with the "rabbit" handle will fail to stop.  Rebooting the machine will force the process to end and EPMD to restart, solving this problem.

## Verification

We'll verify that this works when we *Cluster Rabbit*, in the next tutorial.

> Thank you to the author of the [Loose XAML](http://loosexaml.wordpress.com/) blog for an excellent [article](http://loosexaml.wordpress.com/2012/08/06/rabbitmq-clustering-on-centos-6-12/) on configuring RabbitMQ and *iptables* for clustering.
