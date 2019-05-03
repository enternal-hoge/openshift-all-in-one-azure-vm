# openshift-all-in-one-vm-azure
OpenShift Container Platform 3.11 all-in-one Install on Azure VM


## Prerequiment
- use azure subscription 'pay-as-you-go'
- Red Hat Customer Portal Account
- Red Hat Subscription
- SSH Key generation

## Usecase
- development environment
- demo environment
- sandbox environment for version up 
- scrap and build environment
- full use oc command

## Confirm Quota, Location Name, VM Images
```
# az vm list-usage --location "japaneast" -o table
# az account list-locations -o table
# az vm image list --publisher RedHat --all -o table
```


## Create Resoure Group using Azure Cloud Shell
```
# az group create -l japaneast -n openshift-all-in-one
```

## Create VM using Azure Portal

Search Market Place 'Red Hat Linux 7.6'

| field | value | |
|:--|:--|:--|
| Subscription | pay-as-you-go | Even a free account may be ok |
| Resource Group | openshift-all-in-one | resource group name decide any value. |
| VM Name  | openshift-vm | vm name decide any value. |
| Location | japaneast | near the location |
| VM Size | Standard D2s v3 | Recommendation 'D4s_v3' |
| OS Username | [YourOsName] | any value |
| SSH Public key | [YourPublicKey] |  |
| Listener Port | 80,443,22,8443 | allow accept port of nsg |
| Disk | Standard HDD | Recommendation 'SSD' |
| DNS Name | [AnyValue].japaneast.cloudapp.azure.com | used as [yourAzureDnsName] |
| Public IP | static | Determined automatically |

## Restart VM

After changed vm dns name, stop and start target vm.

## Install Open Shift Container Platform

```
$ ssh -i ~/.ssh/<AnyValue>_rsa <YourName>@<YourPublicIP>
$ sudo su -
# rpm -e rhui-azure-rhel7
# subscription-manager register
Registering to: subscription.rhsm.redhat.com:443/subscription
Username: <yourCostmerPortalAccountName>
Password: <yourCostmerPortalPassword>
# subscription-manager list --available
# subscription-manager attach --pool=<YOUR_SUBSCRIPTION_MASTER_POOL_ID>
# subscription-manager repos --disable '*'
# subscription-manager repos --enable 'rhel-7-server-rpms'
# subscription-manager repos --enable 'rhel-7-server-extras-rpms'
# subscription-manager repos --enable 'rhel-7-server-ose-3.11-rpms'
# yum install atomic-openshift-clients docker git -y
# firewall-cmd --add-port 80/tcp --permanent
# firewall-cmd --add-port 8443/tcp --permanent
# firewall-cmd --add-port 443/tcp --permanent
# firewall-cmd --reload
# vi /etc/docker/daemon.json 
# cat /etc/docker/daemon.json 
{ "insecure-registries": ["172.30.0.0/16"] }
# systemctl enable docker
# systemctl start docker
# docker login https://registry.redhat.io
Username: <yourCostmerPortalAccountName>
Password: <yourCostmerPortalPassword>
# cd ~
# oc cluster up --enable '*,automation-service-broker,service-catalog,template-service-broker' --public-hostname=[yourAzureDnsName] --routing-suffix=[yourAzureDnsName].nip.io  
```

## OpenShift WEB UI

https://[AnyValue].japaneast.cloudapp.azure.com:8443/console

User:     developer
Password: [any value]

or 

User:     system
Password: admin

## Stop and Restart VM

After vm stop and start, Start the cluster with the following command.

```
# oc login -u system:admin
# oc cluster up
```

## Add ImageStreams / Templates

```
# subscription-manager repos --enable="rhel-7-server-ansible-2.6-rpms"
# yum install openshift-ansible -y
# oc login -u system:admin
# oc get is -n openshift
# oc get templates -n openshift
# oc cluster add centos-imagestreams
# XPAASSTREAMDIR=/usr/share/ansible/openshift-ansible/roles/openshift_examples/files/examples/x86_64/xpaas-streams
# oc create -f $XPAASSTREAMDIR/openjdk18-image-stream.json -n openshift
# oc create -f $XPAASSTREAMDIR/eap71-image-stream.json -n openshift
# oc create -f $XPAASSTREAMDIR/eap72-image-stream.json -n openshift
# oc create -f $XPAASSTREAMDIR/fis-image-streams.json -n openshift
# XPAASTEMPLATES=/usr/share/ansible/openshift-ansible/roles/openshift_examples/files/examples/x86_64/xpaas-templates
# oc create -f $XPAASTEMPLATES -n openshift
```
