# Setting Up a pre-1.12 Local Universe

In order to have a properly functioning air-gapped cluster, it is common to deploy a Local Universe in order to both manage and control access to specific DC/OS packages, as well as provide all packages with their dependencies baked in since there is no access to the Public internet

Instructions on [Deploying a Local Universe Containing Selected Packages](https://docs.mesosphere.com/1.11/administering-clusters/deploying-a-local-dcos-universe/#deploying-a-local-universe-containing-selected-packages) are already a part of Mesosphere Documentation, but for the purpose of this guide we will be importing a pre-built Local Universe package that I have provided of both latest Certified Universe packages as well as select commonly used/tested Community packages so that as an SE you can get a test cluster up and running ASAP.

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

### Step 2: Retrieve Certified local-universe.tar.gz from Mesosphere S3 Bucket:

With access to Public Internet, download the pre-built Local Universe TAR from S3 bucket

```
curl -O https://downloads.mesosphere.com/universe/public/local-universe.tar.gz

or 

wget https://downloads.mesosphere.com/universe/public/local-universe.tar.gz
```

If you are following the Air-Gapped guide and don't have access to Public Internet, use SCP instead:

**Step 2a:** Navigate to the Mesosphere AWS Console --> EC2 Console

For Mesosphere Sales Engineers, the link to AWS is [here](https://aws.mesosphere.com/awsconsole)

**Step 2b:** Search for Local-Universe-Builder

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/LocalU-step2b.png)

**Step 2c:** SSH into Instance and navigate to the CertifiedUniverse directory

This instance is accessible using the default Mesosphere key provided (via OneLogin)[https://mesosphere.onelogin.com/notes/41130] and is using CentOS as the OS

```
ssh -A centos@<IP_HERE>
```

Depending on which packages you want, `cd` into the correct directory. Here is a list of packages in each `local-universe.tar.gz`

Certified Universe Download:


Custom Built Local Universe (Includes Kubernetes, Jupyterlab, Beta-Tensorflow):
- cassandra:2.3.0-3.0.16
- dcos-enterprise-cli:1.4.5
- hdfs:2.3.0-2.6.0-cdh5.11.0
- kafka:2.3.0-1.1.0
- kubernetes:1.3.0-1.10.8
- marathon:1.6.535
- spark:2.3.1-2.2.1-2
- beta-tensorflow:0.2.0-1.5.0-beta
- jupyterlab:1.2.0-0.33.7
- marathon-lb:1.12.3
- mysql:5.7.12-0.3


```
cd CertifiedUniverse

or

cd CustomUniverse
```

**Step 2d:** SCP the files in the directory to your Master Nodes

Note: If you are using the default Mesosphere key provided (via OneLogin)[https://mesosphere.onelogin.com/notes/41130] then this instance is pre-configured to work accordingly

```
sudo scp -i ~/.ssh/Mesosphere local-universe.tar.gz  <user>@<master_IP>:~
sudo scp -i ~/.ssh/Mesosphere dcos-local-universe-http.service  <user>@<master_IP>:~
sudo scp -i ~/.ssh/Mesosphere dcos-local-universe-registry.service  <user>@<master_IP>:~
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

Expected Output:

```
$ sudo systemctl status dcos-local-universe-http
● dcos-local-universe-http.service - DCOS: Serve the local universe (HTTP)
   Loaded: loaded (/etc/systemd/system/dcos-local-universe-http.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2018-10-01 23:46:09 UTC; 6s ago
  Process: 20352 ExecStartPre=/usr/bin/docker rm %n (code=exited, status=1/FAILURE)
  Process: 20342 ExecStartPre=/usr/bin/docker kill %n (code=exited, status=1/FAILURE)
 Main PID: 20358 (docker)
   Memory: 4.0M
   CGroup: /system.slice/dcos-local-universe-http.service
           └─20358 /usr/bin/docker run --rm --name dcos-local-universe-http.service -p 8082:80 mesosphere/universe nginx -g daemon off;

Oct 01 23:46:09 ip-10-0-1-40.us-west-2.compute.internal systemd[1]: Starting DCOS: Serve the local universe (HTTP)...
Oct 01 23:46:09 ip-10-0-1-40.us-west-2.compute.internal docker[20342]: Error response from daemon: Cannot kill container dcos-local-universe-http.service: No such container: dcos-local-universe-http.service
Oct 01 23:46:09 ip-10-0-1-40.us-west-2.compute.internal docker[20352]: Error response from daemon: No such container: dcos-local-universe-http.service
Oct 01 23:46:09 ip-10-0-1-40.us-west-2.compute.internal systemd[1]: Started DCOS: Serve the local universe (HTTP).
```

```
$ sudo systemctl status dcos-local-universe-registry
● dcos-local-universe-registry.service - DCOS: Serve the local universe (Docker registry)
   Loaded: loaded (/etc/systemd/system/dcos-local-universe-registry.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2018-10-01 23:46:09 UTC; 11s ago
  Process: 20386 ExecStartPre=/usr/bin/docker rm %n (code=exited, status=1/FAILURE)
  Process: 20379 ExecStartPre=/usr/bin/docker kill %n (code=exited, status=1/FAILURE)
 Main PID: 20393 (docker)
   Memory: 4.1M
   CGroup: /system.slice/dcos-local-universe-registry.service
           └─20393 /usr/bin/docker run --rm --name dcos-local-universe-registry.service -p 5000:5000 -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key mesosphere/universe registry serve /etc/...

Oct 01 23:46:09 ip-10-0-1-40.us-west-2.compute.internal systemd[1]: Starting DCOS: Serve the local universe (Docker registry)...
Oct 01 23:46:09 ip-10-0-1-40.us-west-2.compute.internal docker[20379]: Error response from daemon: Cannot kill container dcos-local-universe-registry.service: No such container: dcos-local-universe-registry.service
Oct 01 23:46:09 ip-10-0-1-40.us-west-2.compute.internal docker[20386]: Error response from daemon: No such container: dcos-local-universe-registry.service
Oct 01 23:46:09 ip-10-0-1-40.us-west-2.compute.internal systemd[1]: Started DCOS: Serve the local universe (Docker registry).
```

**Note:** Although the systemd component may have the status ACTIVE, it may take some time for the Docker Registry and NGINX http service to spin up and start serving (Up to 10-15 minutes)

### Step 6: Validate that the Local Universe is up and serving

Once the Local Universe is up and serving, you can run a `sudo docker ps` to see that the Registry and NGINX services are up and running. Below is an example output:

```
$ sudo docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                     NAMES
fcf263740f3d        mesosphere/universe   "registry serve /e..."   8 minutes ago       Up 2 minutes        80/tcp, 443/tcp, 0.0.0.0:5000->5000/tcp   dcos-local-universe-registry.service
7b22352d4e83        mesosphere/universe   "nginx -g 'daemon ..."   8 minutes ago       Up 2 minutes        443/tcp, 5000/tcp, 0.0.0.0:8082->80/tcp   dcos-local-universe-http.service
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

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/LocalU-step9.png)

## Congratulations, your Local Universe is now complete!

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/LocalU-finished.png)

### Step 10: Testing your Local Universe Deployment

A good package to test deploy is the Spark framework. Modify the package to pull its docker image from `master.mesos:5000` instead of Public DockerHub

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/LocalU-step10a.png)

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/LocalU-step10a.png)

