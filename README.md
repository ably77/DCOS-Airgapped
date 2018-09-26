# DCOS-Airgapped

The purpose of this guide is to show a couple methods of how a Mesosphere Sales Engineer can set up an airgapped cluster for testing or lab purposes

## [Setting Up an Airgapped CCM Cluster]
## [Setting Up an Airgapped Terraform DC/OS Cluster]

# Local Universe

The default DC/OS Universe expects internet connectivity in order to pull auxillary package dependencies when needed. However, customers who have strict security requirements often operate in environments where DC/OS is unable to reach the Internet (airgapped environments, proxies, etc.) In this scenario, an operator can build a Local Universe that is housed and served without any direct connection to the Internet. This allows our customers to:
- Ability to oerate a fully air-gapped cluster
- Deploy DC/OS Services using intranet latency and bandwidth
- Manage and control access to specific DC/OS Packages

The purpose of this guide will be to continue forward after setting up our Airgapped cluster by adding a Local Universe for both 1.11 Local Universe as well as 1.12 Package Registry (beta)

## [Setting Up pre-1.12 Local Universe]
## [Setting Up 1.12 Package Registry]  
