.. _elastic_server_deb:

Install Elastic Stack with Debian packages
===========================================

The DEB package is suitable for Debian, Ubuntu, and other Debian-based systems.

.. note:: Many of the commands described below need to be executed with root user privileges.

Preparation
-----------

1. Oracle Java JRE is required by Logstash and Elasticsearch:

  a) For Debian:

  .. code-block:: console

    # echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee /etc/apt/sources.list.d/webupd8team-java.list
    # echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list
    # apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886

  b) For Ubuntu:

  .. code-block:: console

	 # add-apt-repository ppa:webupd8team/java

2. Once the repository is added, install Java JRE:

  .. code-block:: console

  	# apt-get update
  	# apt-get install oracle-java8-installer

3. Install the Elastic repository and its GPG key:

  .. code-block:: console

  	# apt-get install curl apt-transport-https
  	# curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
  	# echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-6.x.list
  	# apt-get update

Elasticsearch
-------------

Elasticsearch is a highly scalable full-text search and analytics engine. For more info please see `Elasticsearch <https://www.elastic.co/products/elasticsearch>`_.

1. Install the Elasticsearch package:

  .. code-block:: console

    # apt-get install elasticsearch=6.1.2

2. Enable and start the Elasticsearch service:

  a) For Systemd:

  .. code-block:: console

    # systemctl daemon-reload
    # systemctl enable elasticsearch.service
    # systemctl start elasticsearch.service

  b) For SysV Init:

  .. code-block:: console

  	# update-rc.d elasticsearch defaults 95 10
  	# service elasticsearch start

3. Load Wazuh Elasticsearch templates:

  .. code-block:: console

  	# curl https://raw.githubusercontent.com/wazuh/wazuh/3.1/extensions/elasticsearch/wazuh-elastic6-template-alerts.json | curl -XPUT 'http://localhost:9200/_template/wazuh' -H 'Content-Type: application/json' -d @-

  .. code-block:: console

	# curl https://raw.githubusercontent.com/wazuh/wazuh/3.1/extensions/elasticsearch/wazuh-elastic6-template-monitoring.json | curl -XPUT 'http://localhost:9200/_template/wazuh-agent' -H 'Content-Type: application/json' -d @-

4. Insert sample alert:

  .. code-block:: console

  	# curl https://raw.githubusercontent.com/wazuh/wazuh/3.1/extensions/elasticsearch/alert_sample.json | curl -XPUT "http://localhost:9200/wazuh-alerts-3.x-"`date +%Y.%m.%d`"/wazuh/sample" -H 'Content-Type: application/json' -d @-

.. note::

    It is recommended to edit the default configuration to improve the Elasticsearch performance. To do so, please see :ref:`elastic_tuning`.

Logstash
--------

Logstash is the tool that will collect, parse, and forward to Elasticsearch for indexing and storage all logs generated by Wazuh server. For more info please see `Logstash <https://www.elastic.co/products/logstash>`_.

1. Install the Logstash package:

  .. code-block:: console

    # apt-get install logstash=1:6.1.2-1

2. Download the Wazuh config for Logstash:

  a) Local configuration:

    .. code-block:: console

    	# curl -so /etc/logstash/conf.d/01-wazuh.conf https://raw.githubusercontent.com/wazuh/wazuh/3.1/extensions/logstash/01-wazuh-local.conf

    Because the Logstash user needs to read the alerts.json file, please add it to OSSEC group by running:

    .. code-block:: console

      # usermod -a -G ossec logstash

  b) Remote configuration:

    .. code-block:: console

      # curl -so /etc/logstash/conf.d/01-wazuh.conf https://raw.githubusercontent.com/wazuh/wazuh/3.1/extensions/logstash/01-wazuh-remote.conf


3. Enable and start the Logstash service:

  a) For Systemd:

  .. code-block:: console

  	# systemctl daemon-reload
  	# systemctl enable logstash.service
  	# systemctl start logstash.service

  b) For SysV Init:

  .. code-block:: console

    # update-rc.d logstash defaults 95 10
    # service logstash start

.. note::

    If you are running Wazuh server and the Elastic Stack server on separate systems (**distributed architecture**), then it is important to configure encryption between Filebeat and Logstash. To do so, please see :ref:`elastic_ssl`.

Kibana
------

Kibana is a flexible and intuitive web interface for mining and visualizing the events and archives stored in Elasticsearch. More info at `Kibana <https://www.elastic.co/products/kibana>`_.

1. Install the Kibana package:

  .. code-block:: console

   # apt-get install kibana=6.1.2

2. Install the Wazuh App plugin for Kibana:

  2.1) Increase the default Node.js heap memory limit to prevent out of memory errors when installing the Wazuh App.
  Set the limit as follow:

  .. code-block:: console

      # export NODE_OPTIONS="--max-old-space-size=3072"

  2.2) Install Wazuh App:

  .. code-block:: console

      # /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/wazuhapp/wazuhapp.zip

  .. warning::

    The Kibana plugin installation process may take several minutes. Please wait patiently.

  .. note::

    If you want to download a different Wazuh App plugin for another version of Wazuh or the Elastic Stack, you can check the table available at `GitHub <https://github.com/wazuh/wazuh-kibana-app#installation>`_ and use the appropiate installation command.

3. **Optional.** Kibana will listen only on the loopback interface (localhost) by default. To set up Kibana to listen on all interfaces, edit the file ``/etc/kibana/kibana.yml``. Uncomment the setting ``server.host`` and change the value to:

  .. code-block:: yaml

    server.host: "0.0.0.0"

  .. note::

    It is recommended to set up an Nginx proxy for Kibana in order to use SSL encryption and to enable authentication. Instructions to set the proxy up can be found at :ref:`kibana_ssl`.

4. Enable and start the Kibana service:

  a) For Systemd:

  .. code-block:: console

  	# systemctl daemon-reload
  	# systemctl enable kibana.service
  	# systemctl start kibana.service

  b) For SysV Init:

  .. code-block:: console

   # update-rc.d kibana defaults 95 10
   # service kibana start

5. Disable the Elastic repository:

  We recommend to disable the Elasticsearch repository in order to prevent an upgrade to a newer Elastic Stack version due to possible breaking changes with our App, so you should do it as follow:

  .. code-block:: console

   # sed -i -r '/deb https:\/\/artifacts.elastic.co\/packages\/6.x\/apt stable main/ s/^(.*)$/#\1/g' /etc/apt/sources.list.d/elastic-6.x.list

Connecting the Wazuh App with the API
-------------------------------------

Follow the next guide in order to connect the Wazuh App with the API:

.. toctree::
	:maxdepth: 1

	connect_wazuh_app
