# gsctl
A standalone CLI tool for creating, provisioning and installing GigaSpaces clusters

[![Watch the video](https://github.com/Gigaspaces/gsctl/raw/master/images/fleeq.png)](https://csm-gigaspaces.fleeq.io/l/jucegzvnxg-9x8bee587p)

## Prerequisites

* Java 8
* Amazon AWS account and .aws/credentials as described in the [docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) 
(should contain `aws_access_key_id`, `aws_secret_access_key` and `region`)

## Setting Up The Environment

* Download https://github.com/Gigaspaces/gsctl/raw/15.2.0-m11/gsctl.jar into your chosen directory

## Creating The GigaSpaces Cluster

* Run `java -jar gsctl.jar init --cluster-name=<cluster name>` where cluster name is a logical name for your cluster.\
  The command creates provision.yml file that can be edited to supply existing AWS resources (i.e vpc,keyName, securityGroups...).\
  If the file remains unedited all resources will be created when running _create_ command  

````BASH
yael@yael-pcu:~/nomad-workspace$ java -jar gsctl.jar init --cluster-name=GS_CLUSTER
Default provision.yml file and .gsctl config folder were created successfully

yael@yael-pcu:~/nomad-workspace$ cat provision.yml 
---
aws:
  keyName: null
  vpcId: null
  vpcSubnetId: null
  securityGroup: null
  amiId: null
  userName: null
servers:
  label: "GS Cluster [GS_CLUSTER] Server Group"
  groups:
  - type: "m4.xlarge"
    tags: null
    count: 3
clients:
  label: "GS Cluster [GS_CLUSTER] Client Group"
  groups:
  - type: "m4.xlarge"
    tags: null
    count: 3
gsManagers: 3
name: "GS_CLUSTER"
````
 

* Run `java -jar gsctl.jar create` to provision, install and run the GigaSpaces cluster.
````
yael@yael-pcu:~/nomad-workspace$ java -jar gsctl.jar create
2020-02-16 15:28:18,077 [main] INFO  com.gigaspaces.aws.AwsUtils -  Successfully created vpc: vpc-0e300ae2639bf2d20
2020-02-16 15:28:18,416 [main] INFO  com.gigaspaces.aws.AwsUtils -  Successfully created internet gateway igw-0c06082b6c28a0eff
2020-02-16 15:28:19,070 [main] INFO  com.gigaspaces.aws.AwsUtils -  Successfully created subnet subnet-08640c896fdc2729c
2020-02-16 15:28:19,952 [main] INFO  com.gigaspaces.aws.AwsUtils -  Successfully created route table rtb-0d8554c74b1705eb4
2020-02-16 15:28:21,061 [main] INFO  com.gigaspaces.aws.AwsUtils -  Successfully created security group sg-077ef69ed1fa681ee
2020-02-16 15:28:21,880 [main] INFO  com.gigaspaces.aws.AwsUtils -  Successfully created pem file /home/yael/nomad-workspace/.gsctl/GS_CLUSTER.pem
2020-02-16 15:28:23,137 [main] INFO  com.gigaspaces.aws.AwsUtils -  waiting for instances to become available ...
2020-02-16 15:28:24,425 [main] INFO  com.gigaspaces.aws.AwsUtils -  Successfully created 3 instances: 
id: i-0d00f5ea73c43e7a1 private ip: 172.31.11.235       public ip: 63.35.184.16
id: i-0de8edf41b544c67d private ip: 172.31.11.213       public ip: 34.254.196.25
id: i-081f2ab780e347bdf private ip: 172.31.13.72        public ip: 52.213.155.105

2020-02-16 15:28:26,070 [main] INFO  com.gigaspaces.aws.AwsUtils -  waiting for instances to become available ...
2020-02-16 15:28:27,316 [main] INFO  com.gigaspaces.aws.AwsUtils -  Successfully created 3 instances: 
id: i-07b652444829943b2 private ip: 172.31.10.158       public ip: 52.18.157.186
id: i-09b6b569408012bb9 private ip: 172.31.3.27 public ip: 34.251.140.90
id: i-04a5ece120d0e3a99 private ip: 172.31.8.189        public ip: 34.240.117.163

Waiting for all machines to be connected
Machine [34.254.196.25] is connected (took: 32s)
Machine [52.213.155.105] is connected (took: 32s)
Machine [34.240.117.163] is connected (took: 32s)
Machine [63.35.184.16] is connected (took: 33s)
Machine [34.251.140.90] is connected (took: 33s)
Machine [52.18.157.186] is connected (took: 33s)

Starting setup on machines
Setup finished on machine [63.35.184.16] (took: 100s)
Setup finished on machine [52.213.155.105] (took: 101s)
Setup finished on machine [34.240.117.163] (took: 101s)
Setup finished on machine [52.18.157.186] (took: 102s)
Setup finished on machine [34.251.140.90] (took: 102s)
Setup finished on machine [34.254.196.25] (took: 105s)

Cluster is ready, Nomad UI: http://63.35.184.16:4646
Deploy influxdb job
Deploy grafana job
Deploy telegraf job
Deploy manager job
Wait for all jobs allocations to run ...
.....................

Manager is ready, Manager UI: http://34.254.196.25:8090
Grafana is ready, Grafana UI: http://52.18.157.186:6066
Done

````

## Deploying GigaSpaces Services

* Run `java -jar gsctl.jar deploy stateful mySpace processor.jar` to deploy the built-in space example (from: https://github.com/Gigaspaces/gsctl/tree/master/services)
  
````
 yael@yael-pcu:~/nomad-workspace$ java -jar gsctl.jar deploy stateful mySpace processor.jar
 Deploying mySpace job
 .....
````

* Run `java -jar gsctl.jar deploy stateless myFeeder feeder.jar` to deploy the built-in feeder example (from: https://github.com/Gigaspaces/gsctl/tree/master/services)
  
````
 yael@yael-pcu:~/nomad-workspace$ java -jar gsctl.jar deploy stateless myFeeder feeder.jar
 Deploying myFeeder job
 .....
````

* Run `java -jar gsctl.jar service-address manager` to get Ops Manager UI url and browse to see deployed services.

````
yael@yael-pcu:~/nomad-workspace$ java -jar gsctl.jar service-address manager
http://34.254.196.25:8090
````

* Run `java -jar gsctl.jar service-address grafana` to get Grafana UI url to see deployed services dashboards.

````
yael@yael-pcu:~/nomad-workspace$ java -jar gsctl.jar service-address grafana
http://52.18.157.186:6066
```` 
## Undeploying GigaSpaces Services

* Run `java -jar gsctl.jar undeploy mySpace` to undeploy mySpace service.

## Tearing Down The GigaSpaces Cluster

* Run 'java -jar gsctl.jar destroy' to take the cluster down and delete all dynamically created resources.

````
yael@yael-pcu:~/nomad-workspace$ java -jar gsctl.jar destroy
GS_CLUSTER
Terminating instances: i-07b652444829943b2, i-09b6b569408012bb9, i-04a5ece120d0e3a99, i-0d00f5ea73c43e7a1, i-0de8edf41b544c67d, i-081f2ab780e347bdf
2020-02-16 15:44:27,270 [main] INFO  com.gigaspaces.aws.AwsUtils -  Successfully terminated 6 instances
Deleting route table: rtb-0d8554c74b1705eb4
Disassociating rtbassoc-03b4005b2d4e0fdc3 ...
Successfully deleted route table: rtb-0d8554c74b1705eb4
Deleting gateway: igw-0c06082b6c28a0eff
Successfully deleted gateway: igw-0c06082b6c28a0eff
Deleting subnet: subnet-08640c896fdc2729c
Successfully deleted subnet: subnet-08640c896fdc2729c
Deleting security group: sg-077ef69ed1fa681ee
Successfully deleted security group: sg-077ef69ed1fa681ee
2020-02-16 15:44:28,863 [main] INFO  com.gigaspaces.aws.AwsUtils -  Successfully deleted vpc: vpc-0e300ae2639bf2d20
Deleting key-pair with name: GS_CLUSTER
Successfully deleted key-pair with name: GS_CLUSTER
````
