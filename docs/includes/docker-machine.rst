#. **Non-Docker for Mac Windows Users:** Create and configure the Docker Machine

   #. Create a VirtualBox instance running a Docker container named ``confluent`` with 6 GB of memory.

      .. sourcecode:: bash

        docker-machine create --driver virtualbox --virtualbox-memory 6000 confluent

   #. Configure your terminal window to attach it to your new Docker Machine:

      .. sourcecode:: bash

         docker-machine env confluent
