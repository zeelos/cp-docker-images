.. _docker_quickstart:

|cp| Docker Quick Start
=======================

This quick start provides a basic guide for deploying a Kafka cluster along with all |cp| components in your Docker
environment.  By the end of this quick start, you will have a functional |cp| deployment that you can use to run applications.

This quick start builds a single node Docker environment. You will configure Kafka and |zk| to store data locally in their
Docker containers.

Prerequisites
    - .. include:: includes/docker-version.rst

    - `curl <https://curl.haxx.se/>`_

.. contents:: Contents
    :local:
    :depth: 1

.. _quickstart_compose:

=====================================
Step 1: Setup Your Docker Environment
=====================================

You can install |cp| using either :ref:`Docker Compose <quickstart_compose>` or the :ref:`Docker client <quickstart_engine>`.
Docker Compose is recommended since you can launch |cp| with a single invocation of a configuration file.

.. contents::
    :local:
    :depth: 1

Docker Compose Environment
--------------------------

Prerequisite
    - Docker Compose is `installed <https://docs.docker.com/compose/install/>`_. It is installed by default with Docker
      for Mac and Windows.
    - Non-Docker for Mac Windows Users: VirtualBox is `installed <https://www.virtualbox.org/wiki/Downloads>`_.

---------------------------------
Configure your Docker Environment
---------------------------------

.. include:: includes/docker-machine.rst

#. Clone the |cp| Docker Images Github Repository.

   .. sourcecode:: bash

     git clone https://github.com/confluentinc/cp-docker-images

#. Navigate to examples directory (``/examples/kafka-single-node/``) and start the |zk| and Kafka containers in detached
   mode (``-d``). 

   .. sourcecode:: bash

       cd <path-to-cp-docker-images>/examples/kafka-single-node/
       docker-compose up -d

   Your output should resemble the following:

   .. sourcecode:: bash

        Pulling kafka (confluentinc/cp-kafka:latest)...
        ...
        Creating kafkasinglenode_zookeeper_1 ...
        Creating kafkasinglenode_zookeeper_1 ... done
        Creating kafkasinglenode_kafka_1 ...
        Creating kafkasinglenode_kafka_1 ... done

#. Optional: Verify that your environment is up and running.

   * Run this command to verify that the services are up and running:

     .. sourcecode:: bash

       docker-compose ps

     You should see the following:

     .. sourcecode:: bash

                  Name                        Command            State   Ports
       -----------------------------------------------------------------------
       kafkasinglenode_kafka_1       /etc/confluent/docker/run   Up
       kafkasinglenode_zookeeper_1   /etc/confluent/docker/run   Up

     If the state is not `Up`, rerun the ``docker-compose up -d`` command.

   * Check the |zk| logs to verify that |zk| is healthy.

     .. sourcecode:: bash

       $ docker-compose logs zookeeper | grep -i binding

     You should see the following:

     .. sourcecode:: bash

       zookeeper_1  | [2016-07-25 03:26:04,018] INFO binding to port 0.0.0.0/0.0.0.0:32181 (org.apache.zookeeper.server.NIOServerCnxnFactory)

   * Check the Kafka logs to verify that broker is healthy.

     .. sourcecode:: bash

       $ docker-compose logs kafka | grep -i started

     You should see the following:

     .. sourcecode:: bash

       kafka_1      | [2017-08-31 00:31:40,244] INFO [Socket Server on Broker 1], Started 1 acceptor threads (kafka.network.SocketServer)
       kafka_1      | [2017-08-31 00:31:40,426] INFO [Replica state machine on controller 1]: Started replica state machine with initial state -> Map() (kafka.controller.ReplicaStateMachine)
       kafka_1      | [2017-08-31 00:31:40,436] INFO [Partition state machine on Controller 1]: Started partition state machine with initial state -> Map() (kafka.controller.PartitionStateMachine)
       kafka_1      | [2017-08-31 00:31:40,540] INFO [Kafka Server 1], started (kafka.server.KafkaServer)

-------------------------------
Create a Topic and Produce Data
-------------------------------

In this step you will create a topic and produce data to it. You'll use the client tools directly from another Docker
container.

#. Create a topic named ``foo`` and give it one partition and one replica.

   .. sourcecode:: bash

    $ docker-compose exec kafka  \
      kafka-topics --create --topic foo --partitions 1 --replication-factor 1 \
      --if-not-exists --zookeeper localhost:32181

   Your output should resemble:

   .. sourcecode:: bash

    Created topic "foo".

#. Optional: Verify that the topic was created successfully:

   .. sourcecode:: bash

    $ docker-compose exec kafka  \
      kafka-topics --describe --topic foo --zookeeper localhost:32181

   Your output should resemble:

   .. sourcecode:: bash

    Topic:foo   PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: foo  Partition: 0    Leader: 1    Replicas: 1  Isr: 1

#. Publish some data to the ``foo`` topic. This command uses the built-in Kafka Console Producer to produce 42 simple
   messages to the topic.

   .. sourcecode:: bash

    $ docker-compose exec kafka  \
      bash -c "seq 42 | kafka-console-producer --request-required-acks 1 --broker-list \
      localhost:29092 --topic foo && echo 'Produced 42 messages.'"

   Your output should resemble:

   .. sourcecode:: bash

    >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>Produced 42 messages.

#. Read back the message using the built-in Console consumer:

   .. sourcecode:: bash

    $ docker-compose exec kafka  \
      kafka-console-consumer --bootstrap-server localhost:29092 --topic foo \
      --from-beginning --max-messages 42

   Your output should resemble:

   ::

    1
    2
    3
    4
    ....
    42
    Processed a total of 42 messages



=======
Cleanup
=======

After you're done, cleanup is simple. Run the command ``docker rm -f $(docker ps -a -q)`` to delete all the containers
you created in the steps above, ``docker volume prune`` to remove any remaining unused volumes, and
``docker network rm confluent`` to delete the network that was created.

If you are running Docker Machine, you can remove the virtual machine with this command: ``docker-machine rm confluent``.

You must explicitly shut down Docker Compose. For more information, see the `docker-compose down <https://docs.docker.com/compose/reference/down/>`_
documentation. This will delete all of the containers that you created in this quick start.

.. sourcecode:: bash

   $ docker-compose down

==========
Next Steps
==========

- For examples of how to add mounted volumes to your host machines, see :ref:`external_volumes`. Mounted volumes provide a
  persistent storage layer for deployed containers. With a persistent storage layer, you can stop and restart Docker images,
  such as ``cp-kafka`` and ``cp-zookeeper``, without losing their stateful data.
- For examples of more complex target environments, see the :ref:`tutorials_overview`.
- For more information about |sr|, see :ref:`schemaregistry_intro`.
- For a more in-depth Kafka Connect example, see :ref:`connect_quickstart_avro_jdbc`.
