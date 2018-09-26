# Setting Up an Airgapped CCM Cluster

## Deploy a CCM Cluster:

### Step 1: Navigate to CCM

Using OneLogin, navigate to the CCM Homepage and select New Cluster

### Step 2: Configure your CCM Cluster  

For our exercise it is recommended to deploy at least 1 Public Agent and 3 Private Agents so that we can deploy Framework packages in later exercises

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/CCM-step2a.png) 

Once the deployment is complete, you should be able to view all cluster details as shown below:

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/CCM-step2b.png)

### Step 3: Navigate to the DC/OS UI and Install the DC/OS CLI

To access your DC/OS Cluster, navigate to `http://<MASTER_IP>` and logon using provided username/password:

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/CCM-step3a.png)

Install the DC/OS CLI on your Local Machine:

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/CCM-step3b.png)

### Step 4: Ensure that the Cluster is working properly and is connected to the Internet by deploying the simple NGINX marathon application pulling from Docker Hub below

```
{
  "id": "/nginx",
  "instances": 1,
  "portDefinitions": [],
  "container": {
    "type": "DOCKER",
    "volumes": [],
    "docker": {
      "image": "nginx",
      "forcePullImage": true
    }
  },
  "cpus": 0.1,
  "mem": 128,
  "requirePorts": false,
  "networks": [],
  "healthChecks": [],
  "fetch": [],
  "constraints": []
}
```

Deploy the application:

```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Airgapped/master/resources/nginx.json
```

Application should be successfully pulled from DockerHub and deployed in the DC/OS Services page:

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/CCM-step4a.png)

Remove Application:

```
dcos marathon app remove nginx --yes
```

## Setting Up Airgap in AWS

Step 1: Navigate to the AWS Console

For Mesosphere Sales Engineers, the link to AWS is [here](https://aws.mesosphere.com/awsconsole)

Step 2: Navigate to EC2 --> Security Groups and Search for your CCM Cluster
![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/AWS-step2a.png)

Step 3: Modify all Outbound Security Group Rules

Change all Outbound Traffic Rules from 0.0.0.0/0 to an internal 10.0.0.0/16 CIDR block

- <Cluster_Name>-PublicSlaveSecurityGroup
- <Cluster_Name>-AdminSecurityGroup
- <Cluster_Name>-MasterSecurityGroup
- <Cluster_Name>-SlaveSecurityGroup
- <Cluster_Name>-LBSecurityGroup

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/AWS-step3a.png)

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/AWS-step3b.png)


## Test Airgapped Deployment

### Step 1: Re-run the nginx example from above

```
dcos marathon app add https://raw.githubusercontent.com/ably77/DCOS-Airgapped/master/resources/nginx.json
```

### Step 2: Validate

Since the json definition has the parameter `"forcePullImage": true,` we should see that the NGINX deployment hangs because the cluster cannot pull from the Public Dockerhub:

Expected behavior is that the NGINX service will remain staging until timeout and then fail:

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/test-step2a.png)

If you navigate to the NGINX service --> Debug tab you should see the error below under `Last Task Failure`:

```
Failed to launch container: Failed to run 'docker -H unix:///var/run/docker.sock pull nginx': exited with status 1; stderr='Network timed out while trying to connect to https://index.docker.io/v1/repositories/library/nginx/images. You may want to check your internet connection or if you are behind a proxy. '
```

![](https://github.com/ably77/DCOS-Airgapped/blob/master/resources/test-step2b.png)

### Step 3: Remove the NGINX Service

```
dcos marathon app remove nginx
```

## Finished!

You now have a basic DC/OS Cluster setup to simulate an Airgapped (No external internet access) Environment. Next step is to install a Local Universe or the 1.12 Package Repository so that you can deploy Framework packages from the DC/OS Catalog!
