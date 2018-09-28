# Setting Up a pre-1.12 Local Universe

In order to have a properly functioning air-gapped cluster, it is common to deploy a Local Universe in order to both manage and control access to specific DC/OS packages, as well as provide all packages with their dependencies baked in since there is no access to the Public internet

Instructions on [Deploying a Local Universe Containing Selected Packages](https://docs.mesosphere.com/1.11/administering-clusters/deploying-a-local-dcos-universe/#deploying-a-local-universe-containing-selected-packages) are already a part of Mesosphere Documentation, but for the purpose of this guide we will be importing a pre-built Local Universe package that I have provided of both latest Certified Universe packages as well as select commonly used/tested Community packages

The `make` build for your reference:
```
sudo make DCOS_VERSION=1.11.6 DCOS_PACKAGE_INCLUDE="cassandra:2.3.0-3.0.16,chronos:2.5.1,confluent-kafka:2.3.0-4.0.0e,confluent-zookeeper:2.4.0-4.0.0e,datastax-dse:2.3.0-5.1.2,datastax-ops:2.3.0-6.1.2,dcos-enterprise-cli:1.4.5,elastic:2.4.0-5.6.9,hdfs:2.3.0-2.6.0-cdh5.11.0,jenkins:3.5.1-2.107.2,kafka:2.3.0-1.1.0,kafka-zookeeper:2.4.0-3.4.13,kibana:2.4.0-5.6.9,kubernetes:1.3.0-1.10.8,marathon:1.6.535,spark:2.3.1-2.2.1-2,beta-tensorflow:0.2.0-1.5.0-beta,confluent-connect:1.1.1-4.1.1,confluent-control-center:1.1.1-4.1.1,confluent-replicator:1.0.0-3.3.1,confluent-rest-proxy:1.1.1-4.1.1,confluent-schema-registry:1.1.1-4.1.1,gitlab:1.0-9.1.0,grafana:5.5.0-5.1.3,jupyterlab:1.2.0-0.33.7,marathon-lb:1.12.3,mysql:5.7.12-0.3,mysql-admin:4.6.4-0.2,postgresql:9.6-0.2,postgresql-admin:5.1-0.2,prometheus:0.1.1-2.3.2" local-universe
```


