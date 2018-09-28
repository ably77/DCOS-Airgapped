# Setting Up a pre-1.12 Local Universe

In order to have a properly functioning air-gapped cluster, it is common to deploy a Local Universe in order to both manage and control access to specific DC/OS packages, as well as provide all packages with their dependencies baked in since there is no access to the Public internet

Instructions on [Deploying a Local Universe Containing Selected Packages](https://docs.mesosphere.com/1.11/administering-clusters/deploying-a-local-dcos-universe/#deploying-a-local-universe-containing-selected-packages) are already a part of Mesosphere Documentation, but for the purpose of this guide we will be importing a pre-built Local Universe package that I have provided of both latest Certified Universe packages as well as select commonly used/tested Community packages so that as an SE you can get a test cluster up and running ASAP.

The `make` build for your reference:
```
sudo make DCOS_VERSION=1.11.6 DCOS_PACKAGE_INCLUDE="cassandra:2.3.0-3.0.16,confluent-kafka:2.3.0-4.0.0e,datastax-dse:2.3.0-5.1.2,datastax-ops:2.3.0-6.1.2,dcos-enterprise-cli:1.4.5,elastic:2.4.0-5.6.9,hdfs:2.3.0-2.6.0-cdh5.11.0,jenkins:3.5.1-2.107.2,kafka:2.3.0-1.1.0,kibana:2.4.0-5.6.9,kubernetes:1.3.0-1.10.8,marathon:1.6.535,spark:2.3.1-2.2.1-2,beta-tensorflow:0.2.0-1.5.0-beta,gitlab:1.0-9.1.0,grafana:5.5.0-5.1.3,jupyterlab:1.2.0-0.33.7,marathon-lb:1.12.3,mysql:5.7.12-0.3,prometheus:0.1.1-2.3.2" local-universe
```

## Prerequisites
- A running DC/OS Cluster
- Access to SSH into Masters/Agents

### Step 1: SSH into Master(s)

Few methods:
```
ssh-add <key.pem>
ssh -A <user>@<MASTER_IP>
```

```
ssh -i </path/to/key.pem> <user>@<MASTER_IP>
```

```
dcos node ssh --master-proxy --leader --user=<user>
```

### Step 2: Retrieve pre-built local-universe.tar.gz

With access to Public Internet, download the pre-built Local Universe TAR from S3 bucket:

```
```

If you are following the Air-Gapped guide and don't have access to Public Internet, use SCP instead:

**Step 2a:** Navigate to the Mesosphere AWS Console --> EC2 Console

For Mesosphere Sales Engineers, the link to AWS is [here](https://aws.mesosphere.com/awsconsole)

**Step 2b:** Search for Local-Universe-Builder

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/LocalU-step2b.png)

**Step 2c:** SSH into Instance and navigate to the DCOS-Airgapped directory

This instance is accessible using the default Mesosphere key provided (via OneLogin)[https://mesosphere.onelogin.com/notes/41130] and is using CentOS as the OS

```
ssh -A centos@<IP_HERE>
```

```
cd DCOS-Airgapped
```

**Step 2d:** SCP the files in the directory to your Master Nodes

Note: If you are using the default Mesosphere key provided (via OneLogin)[https://mesosphere.onelogin.com/notes/41130] then this instance is pre-configured to work accordingly

```
scp -i ~/.ssh/Mesosphere local-universe.tar.gz  <user>@<master_IP>:~
scp -i ~/.ssh/Mesosphere dcos-local-universe-http.service  <user>@<master_IP>:~
scp -i ~/.ssh/Mesosphere dcos-local-universe-registry.service  <user>@<master_IP>:~
```

Remember that the `local-universe.tar.gz` file is pretty large, so it may take some time to transfer. It may also be useful to review [How to split a large TAR file](https://www.tecmint.com/split-large-tar-into-multiple-files-of-certain-size/) if the file size is too big for your network policies. Because this Local Universe instance is located on AWS, it will be most efficient to use an AWS DC/OS cluster to leverage the AWS intranet for quicker transfer speeds.

### Step 3: In each Master Node, move the systemd files into the /etc/systemd/system directory

```
sudo mv dcos-local-universe-registry.service /etc/systemd/system/
sudo mv dcos-local-universe-http.service /etc/systemd/system/
```

Confirm that the files were successfully copied:

```
ls -la /etc/systemd/system/dcos-local-universe-*
```

### Step 4: Load the Local Universe into the local agent's Docker

```
sudo docker load < local-universe.tar.gz
```

Note: This may take some time to complete

### Step 5: Restart the systemd daemon and enable dcos-local-universe-http and dcos-local-universe-registry services:

Restart the systemd daemon:

```
sudo systemctl daemon-reload
```

Enable the dcos-local-universe-http and dcos-local-universe-registry services:

```
sudo systemctl enable dcos-local-universe-http
sudo systemctl enable dcos-local-universe-registry
```

Start the dcos-local-universe-http and dcos-local-universe-registry services:

```
sudo systemctl start dcos-local-universe-http
sudo systemctl start dcos-local-universe-registry
```

Confirm that the services are up and running:

```
sudo systemctl status dcos-local-universe-http
sudo systemctl status dcos-local-universe-registry
```

Note: Although the systemd component may have the status ACTIVE, it may take some time for the Docker Registry and NGINX http service to spin up and start serving

### Step 6: Validate that the Local Universe is up and serving

Once the Local Universe is up and serving, you can run a `sudo docker ps` to see that the Registry and NGINX services are up and running. Below is an example output:

```
```

### Step 7: In each agent, download a copy of the DC/OS certificate locally and set as trusted:

This will allow for DC/OS agents to correctly pull images from the Docker registry

```
sudo mkdir -p /etc/docker/certs.d/master.mesos:5000

sudo curl -o /etc/docker/certs.d/master.mesos:5000/ca.crt http://master.mesos:8082/certs/domain.crt
```

Restart Docker:

```
sudo systemctl restart docker
```

### Step 8: In each agent, Configure the Apache Mesos fetcher to trust the downloaded Docker certificate:

This will allow for DC/OS agents to correctly pull images from the Docker registry using the Mesos fetcher instead of Docker Daemon

Copy the Certificate:

```
sudo cp /etc/docker/certs.d/master.mesos:5000/ca.crt /var/lib/dcos/pki/tls/certs/docker-registry-ca.crt
```

Generate a hash:

```
cd /var/lib/dcos/pki/tls/certs/

openssl x509 -hash -noout -in docker-registry-ca.crt
```

Create a soft link:

```
sudo ln -s /var/lib/dcos/pki/tls/certs/docker-registry-ca.crt /var/lib/dcos/pki/tls/certs/<hash_number>.0
```

Exit when completed:

```
exit
```

Note: You may need to create the ~/pki/tls/certs directory on the public agent.

### Step 9: On a machine with DC/OS CLI installed and authenticated:

Remove default DC/OS Universe:

```
dcos package repo remove Universe
```

Add reference to the local Universes that you added to each Master from your DC/OS CLI enabled Client:

```
dcos package repo add local-universe  http://master.mesos:8082/repo
```

Validate in Local Universe in the DC/OS UI:

If successful we should see the local-universe as the only Package Repository listed under DC/OS --> System --> Settings --> Package Repositories

## Congratulations, your Local Universe is now complete!
