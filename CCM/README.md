# Setting Up an Airgapped CCM Cluster

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
dcos marathon app add 
