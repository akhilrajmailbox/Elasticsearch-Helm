# elasticsearch Helm Chart

* Installs the web dashboarding system [ElasticSearch Cluster](https://github.com/akhilrajmailbox/Elasticsearch-Helm/blob/master/README.md)

**This deployment using ES version 6.8.0, will not support version 7.x.x**

**All the YAML files are for Azure Deployment, but you can update the Yaml file to use any other cloud platform (gcp serviceaccount mounting configuration missing and the service.yaml file having azure related configuration for example....)**

[help](https://www.datadoghq.com/blog/elasticsearch-game-day/)

[readonlyrest-docs](https://github.com/beshu-tech/readonlyrest-docs/blob/master/elasticsearch.md)

## Current software

* Alpine Linux 3.8
* OpenJDK JRE 8u171
* Elasticsearch 6.8.0 -- 6.8.0

## Kubernetes Cluster Requirement

* cluster of 3 worker nodes (HA Configuration)
* medium / high IOPS Required for the vm type
* 2vcpu and 8 GB RAM for each worker ndoes
* premium Storageclass needed for better performance


## ES Cluster Features

* High Available (HA) Cluster on Kubernetes
* Self Healing Configuration for all nodes
* anytime can scale the cluster size (both horizontal and vertical)
* Authentication and Authorisation enabled with custom configuration and plugins
* The default cluster have the following components (can be increase the number anytime without downtime)
	* 3 master nodes
 	* 3 data nodes
	* 2 client nodes
	* 1 Kibana Server
	* 2 APM server


## Custom Values.yml files

* ES Cluster Deployment : `only-es-cluster-value.yml`
* ES Node Deployment : `only-node-value.yml`
* Kibana Deployment : `only-kibana-value.yml`
* APM Deployment : `only-apm-value.yml`
* Cronjob Deployment : `only-cronjob-value.yml`


## Authentication for ElasticSearch and Kibana

```
ES version 6.8.0 Docker image have plugin readonlyrest plugin, and so youy can enable the authentication for elasticsearch with the following environment Variables
```

This kubernetes deployment by default enabling the authetication for elasticsearch and kibana. If you don't need authetication you can simply remove the `AUTH_CONFIG` environment variable.

These are the default Credentials for the deployment until and unless you remove the `AUTH_CONFIG` environment variable.

The Usernames are hardcoded but you can override this credentials (only passwords) by updating these environment variables from Configmap (I already gave it as an example in the deployment part) or from secrets in kubernetes


**Note: API Server, Kibana Server and ES-HQ Server need kibanAdmin credentials for accessing the cluster**


| User Name | Variable Name | Default Value | Description |
|---------------|---------------|---------------|---------------|
| kibanAdmin | KIBANA_ADMIN_PASSWORD | ARHelmKibanaAdmiNSecreT | Have full permission on the ES cluster and kibana dashboard |
| kibanaUser | KIBANA_RO_PASSWORD | ARHelmKibanaRoSecreT | Have readonly access on ES cluster and kibana dashboard |
| LogAdmin | PUSHLOG_PASSWORD | ARHelmLogPusHSecreT | Password for elasticsearch authetication (used by log shippers) |
| - | APM_TOKEN | ARHelmApMTokeN | token for communicate with APM server |


**Note:** `x-pack-ml` module is forcibly disabled as it's not supported on Alpine Linux.

## Kubernetes Deployment 

**Note: These following environment and its default values are giving below, it is customised for dockerized deployment in K8s but will degrade the performance (MEMORY_LOCK false)**

```
MEMORY_LOCK false
```

kubernetes issue : [Not support new ES version](https://hub.helm.sh/charts/stable/elasticsearch)


## TL;DR;

```bash
kubectl create ns elastic-cluster
helm repo add ar-repo https://akhilrajmailbox.github.io/Elasticsearch-Helm

helm install elasticsearch ar-repo/elasticsearch -n elastic-cluster
```


## Installing the Chart

To install the chart with the release name `elasticsearch`:

```console
$ helm install elasticsearch ar-repo/elasticsearch -n elastic-cluster
```

## Uninstalling the Chart

To uninstall/delete the elasticsearch deployment:

```console
$ helm delete elasticsearch -n elastic-cluster
```


## Deployment Strategy


## ES Cluster Deployment (Master, Data and Ingest Nodes are running on each server)

![Cluster Topology](./src/es-cluster.png)

To install the chart with the release name `elasticsearch` to install ES Cluster Configuration Only :

```console
$ helm install elasticsearch ar-repo/elasticsearch -n elastic-cluster -f only-es-cluster-value.yml
```

## ES Node Deployment (Master, Data and Ingest Nodes are running on same server)

![Node Topology](./src/es-node.png)

To install the chart with the release name `elasticsearch` to install ES Node Configuration Only :

```console
$ helm install elasticsearch ar-repo/elasticsearch -n elastic-cluster -f only-node-value.yml
```


The command removes all the Kubernetes components associated with the chart and deletes the release.


## Configuration

| Parameter                                 | Description                                   | Default                                                 |
|-------------------------------------------|-----------------------------------------------|---------------------------------------------------------|
| `es.enabled`                        | enable cluster for es (client, data, master in separate node) | `true`                         |
| `node.enabled`                        | enable cluster for es (client, data, master in each node) | `true`                         |
| `apm.enabled`                        | enable apm server | `true`                         |
| `kibana.enabled`                        | enable kibana  | `true`                         |
| `image.repository`                        | common registry for es (client, data, master) | `akhilrajmailbox/elasticsearch`                         |
| `image.tag`                               | common tag for es (client, data, master)      | `elasticsearch-6.8.0`                                   |
| `image.pullPolicy`                        | common pullPolicy for es                      | `Always`                                                |
| `EsAuthAutoCreation.enabled`          | enable authetication  in helm                        | `true`                                                  |
| `EsAuthAutoCreation.auth_config`          | enable authetication                          | `true`                                                  |
| `EsAuthAutoCreation.apmToken`             | apm token                                     | `ARHelmApMTokeN`                                        |
| `EsAuthAutoCreation.kibanaAdminPass`      | Kibana Admin pass                             | `ARHelmKibanaAdmiNSecreT`                               |
| `EsAuthAutoCreation.kibanaRoUserPass`     | Kibana User Pass                              | `ARHelmKibanaRoSecreT`                                  |
| `EsAuthAutoCreation.esClientPass`         | ES Client Pass                                | `ARHelmLogPusHSecreT`                                   |
| `cluster_name`                            | cluster name                                  | `ar-es`                                                 |
| `EsSnapshot.enabled`                      | enable snapshot                               | `false`                                                 |
| `EsSnapshot.azureblob.enabled`            | enable azure blob snapshot in helm                   | `nil`                                                     |
| `EsSnapshot.azureblob.azureSnap`            | enable azure blob snapshot                    | `nil`                                                     |
| `EsSnapshot.azureblob.account_name`       | azure blob account name                       | `true`                                                     |
| `EsSnapshot.azureblob.account_key`        | azure blob account key                        | `nil`                                                     |
| `EsSnapshot.gcs.enabled`                  | enable gcp bucket snapshot in helm                    | `nil`                                                     |
| `EsSnapshot.gcs.gcsSnap`                  | enable gcp bucket snapshot                    | `true`                                                     |
| `EsSnapshot.s3.enabled`                   | enable s3 bucket snapshot in helm                    | `nil`                                                     |
| `EsSnapshot.s3.s3Snap`                   | enable s3 bucket snapshot                     | `true`                                                     |
| `EsSnapshot.s3.access_key`                | s3 access key                                 | `nil`                                                     |
| `EsSnapshot.s3.secret_key`                | s3 secret key                                 | `nil`                                                     |
| `EsSnapshotCronJob.enabled`               | enable cronjob deployment in k8s              | `false`                                                     |
| `EsSnapshotCronJob.image.repository`      | cronjob deployment image repository           | `akhilrajmailbox/elasticsearch`                                                     |
| `EsSnapshotCronJob.image.tag`             | image tag                                     | `cron-snapshots`                                                     |
| `EsSnapshotCronJob.image.pullPolicy`      | image pull policy                             | `Always`                                                     |
| `EsSnapshotCronJob.myenv`                 | deployment env production or development      | `development`                                                     |
| `EsSnapshotCronJob.host`                  | elasticsearch host                            | `nil`                                                     |
| `EsSnapshotCronJob.snapindices`           | indices for snapshot                          | `nil`                                                     |
| `EsSnapshotCronJob.indicesdeldays`        | delete indices older than indicesdeldays      | `90`                                                     |
| `EsSnapshotCronJob.snaprepo`              | snapshot repo name                            | `nil`                                                     |
| `EsSnapshotCronJob.snapdeldays`           | delete snapshot older than snapdeldays        | `365`                                                     |
| `node.enabled`                        | enable node cluster es | `false`                         |
| `node.replicaCount`                        | replicacount | `3`                         |
| `node.podDisruptionBudget.minAvailable`        | Pod disruption minimum available              | `nil`                                                   |
| `node.podDisruptionBudget.maxUnavailable`      | Pod disruption maximum unavailable            | `nil`                                                   |
| `node.priorityClassName`      | priorityClassName            | `nil`                                                   |
| `node.java_opts`      | java_opts for master           | `-Xms4G -Xmx4G`                                                   |
| `node.number_of_master`      | min required number of master            | `2`                                                   |
| `node.resources`                               | CPU/Memory resource requests/limits           | `{}`                                                    |
| `node.nodeSelector`                            | Node labels for pod assignment                | `{}`                                                    |
| `node.tolerations`                             | Toleration labels for pod assignment          | `[]`                                                    |
| `node.affinity`                                | Affinity settings for pod assignment          | `{}`                                                    |
| `node.service.discovery.port`                            | Kubernetes port where service is exposed      | `9300`                                                    |
| `node.service.discovery.protocol`                        | Kubernetes service protocol                   | `TCP`                                                   |
| `node.service.discovery.annotations`                     | Service annotations                           | `{}`                                                    |
| `node.service.client.port`                            | Kubernetes port where service is exposed      | `9300`                                                    |
| `node.service.client.protocol`                        | Kubernetes service protocol                   | `TCP`                                                   |
| `node.service.client.annotations`                     | Service annotations                           | `{}`                                                    |
| `node.service.type`                            | type of services      | `LoadBalancer`                                                    |
| `node.persistence.name`                        | name of persistence  | `es-data`                                                   |
| `node.persistence.enabled`                     | Use persistent volume to store data           | `true`                                                 |
| `node.persistence.size`                        | Size of persistent volume claim               | `50Gi`                                                  |
| `node.persistence.storageClassName`            | Type of persistent volume claim               | `nil`                                                   |
| `node.persistence.accessModes`                 | Persistence access modes                      | `[ReadWriteOnce]`                                       |
| `master.replicaCount`                        | replicacount | `3`                         |
| `master.podDisruptionBudget.minAvailable`        | Pod disruption minimum available              | `nil`                                                   |
| `master.podDisruptionBudget.maxUnavailable`      | Pod disruption maximum unavailable            | `nil`                                                   |
| `master.priorityClassName`      | priorityClassName            | `nil`                                                   |
| `master.number_of_master`      | min required number of master            | `2`                                                   |
| `master.java_opts`      | java_opts for master           | `-Xms1G -Xmx1G`                                                   |
| `master.resources`                               | CPU/Memory resource requests/limits           | `{}`                                                    |
| `master.nodeSelector`                            | Node labels for pod assignment                | `{}`                                                    |
| `master.tolerations`                             | Toleration labels for pod assignment          | `[]`                                                    |
| `master.affinity`                                | Affinity settings for pod assignment          | `{}`                                                    |
| `master.service.port`                            | Kubernetes port where service is exposed      | `9300`                                                    |
| `master.service.protocol`                        | Kubernetes service protocol                   | `TCP`                                                   |
| `master.service.annotations`                     | Service annotations                           | `{}`                                                    |
| `data.replicaCount`                        | replicacount | `3`                         |
| `data.podDisruptionBudget.minAvailable`        | Pod disruption minimum available              | `nil`                                                   |
| `data.podDisruptionBudget.maxUnavailable`      | Pod disruption maximum unavailable            | `nil`                                                   |
| `data.priorityClassName`      | priorityClassName            | `nil`                                                   |
| `data.java_opts`      | java_opts for master           | `-Xms4G -Xmx4G`                                                   |
| `data.resources`                               | CPU/Memory resource requests/limits           | `{}`                                                    |
| `data.nodeSelector`                            | Node labels for pod assignment                | `{}`                                                    |
| `data.tolerations`                             | Toleration labels for pod assignment          | `[]`                                                    |
| `data.affinity`                                | Affinity settings for pod assignment          | `{}`                                                    |
| `data.service.port`                            | Kubernetes port where service is exposed      | `9300`                                                    |
| `data.service.protocol`                        | Kubernetes service protocol                   | `TCP`                                                   |
| `data.service.annotations`                     | Service annotations                           | `{}`                                                    |
| `data.persistence.name`                        | name of persistence  | `es-data`                                                   |
| `data.persistence.enabled`                     | Use persistent volume to store data           | `true`                                                 |
| `data.persistence.size`                        | Size of persistent volume claim               | `50Gi`                                                  |
| `data.persistence.storageClassName`            | Type of persistent volume claim               | `nil`                                                   |
| `data.persistence.accessModes`                 | Persistence access modes                      | `[ReadWriteOnce]`                                       |
| `client.replicaCount`                        | replicacount | `2`                         |
| `client.podDisruptionBudget.minAvailable`        | Pod disruption minimum available              | `nil`                                                   |
| `client.podDisruptionBudget.maxUnavailable`      | Pod disruption maximum unavailable            | `nil`                                                   |
| `client.priorityClassName`      | priorityClassName            | `nil`                                                   |
| `client.number_of_master`      | min required number of master            | `2`                                                   |
| `client.java_opts`      | java_opts for master           | `-Xms1G -Xmx1G`                                                   |
| `client.resources`                               | CPU/Memory resource requests/limits           | `{}`                                                    |
| `client.nodeSelector`                            | Node labels for pod assignment                | `{}`                                                    |
| `client.tolerations`                             | Toleration labels for pod assignment          | `[]`                                                    |
| `client.affinity`                                | Affinity settings for pod assignment          | `{}`                                                    |
| `client.service.port`                            | Kubernetes port where service is exposed      | `9200`                                                    |
| `client.service.type`                            | type of services      | `LoadBalancer`                                                    |
| `client.service.protocol`                        | Kubernetes service protocol                   | `TCP`                                                   |
| `client.service.annotations`                     | Service annotations                           | `{}`                                                    |
| `apm.esAdmin`                        | elasticsearch username to connect to ES | `nil`                         |
| `apm.esPassword`                        | elasticsearch password | `nil`                         |
| `apm.esUrl`                        | elasticsearch URL | `nil`                         |
| `apm.apmToken`                        | APM Server Token | `nil`                         |
| `apm.image.repository`                        | common registry for apm | `akhilrajmailbox/elasticsearch`                         |
| `apm.image.tag`                               | common tag for apm      | `apm-server-6.8.0`                                   |
| `apm.image.pullPolicy`                        | common pullPolicy for apm                      | `Always`                                                |
| `apm.replicaCount`                        | replicacount | `2`                         |
| `apm.podDisruptionBudget.minAvailable`        | Pod disruption minimum available              | `nil`                                                   |
| `apm.podDisruptionBudget.maxUnavailable`      | Pod disruption maximum unavailable            | `nil`                                                   |
| `apm.priorityClassName`      | priorityClassName            | `nil`                                                   |
| `apm.resources`                               | CPU/Memory resource requests/limits           | `{}`                                                    |
| `apm.nodeSelector`                            | Node labels for pod assignment                | `{}`                                                    |
| `apm.tolerations`                             | Toleration labels for pod assignment          | `[]`                                                    |
| `apm.affinity`                                | Affinity settings for pod assignment          | `{}`                                                    |
| `apm.service.port`                            | Kubernetes port where service is exposed      | `80`                                                    |
| `apm.service.type`                            | type of services      | `LoadBalancer`                                                    |
| `apm.service.protocol`                        | Kubernetes service protocol                   | `TCP`                                                   |
| `apm.service.annotations`                     | Service annotations                           | `{}`                                                    |
| `kibana.esAdmin`                        | elasticsearch username to connect to ES | `nil`                         |
| `kibana.esPassword`                        | elasticsearch password | `nil`                         |
| `kibana.esUrl`                        | elasticsearch URL | `nil`                         |
| `kibana.image.repository`                        | common registry for kibana | `akhilrajmailbox/elasticsearch`                         |
| `kibana.image.tag`                               | common tag for kibana      | `kibana-6.8.0`                                   |
| `kibana.image.pullPolicy`                        | common pullPolicy for kibana                      | `Always`                                                |
| `kibana.replicaCount`                        | replicacount | `1`                         |
| `kibana.podDisruptionBudget.minAvailable`        | Pod disruption minimum available              | `nil`                                                   |
| `kibana.podDisruptionBudget.maxUnavailable`      | Pod disruption maximum unavailable            | `nil`                                                   |
| `kibana.priorityClassName`      | priorityClassName            | `nil`                                                   |
| `kibana.resources`                               | CPU/Memory resource requests/limits           | `{}`                                                    |
| `kibana.nodeSelector`                            | Node labels for pod assignment                | `{}`                                                    |
| `kibana.tolerations`                             | Toleration labels for pod assignment          | `[]`                                                    |
| `kibana.affinity`                                | Affinity settings for pod assignment          | `{}`                                                    |
| `kibana.service.port`                            | Kubernetes port where service is exposed      | `80`                                                    |
| `kibana.service.type`                            | type of services      | `LoadBalancer`                                                    |
| `kibana.service.protocol`                        | Kubernetes service protocol                   | `TCP`                                                   |
| `kibana.service.annotations`                     | Service annotations                           | `{}`                                                    |


## Kibana Server Only

To install the chart with the release name `elasticsearch` to install Kibana server Only :

```console
$ helm install elasticsearch ar-repo/elasticsearch -n elastic-cluster -f only-kibana-value.yml
```

## APM Server Only

To install the chart with the release name `elasticsearch` to install APM Server Only :

```console
$ helm install elasticsearch ar-repo/elasticsearch -n elastic-cluster -f only-apm-value.yml
``` 



## Scaling Considerations

### client nodes

We can deploy autoscalers for our client nodes depending upon our CPU thresholds. A sample HPA for client node might look something like this:

**NOTE : you have to install and configure matrics in your cluster**

```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: es-client
  namespace: elasticsearch
spec:
  maxReplicas: 5
  minReplicas: 2
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: es-client
  targetCPUUtilizationPercentage: 80
```

Whenever the autoscaler will kick in, we can watch the new client-node pods being added to the cluster, by observing the logs of any of the master-node pods.

### data nodes

In case of Data-Node Pods all we have to do it increase the number of replicas using the K8 Dashboard or GKE console. The newly created data node will be automatically added to the cluster and start replicating data from other nodes.

### master nodes

Master-Node Pods do not require autoscaling as they only store cluster-state information but in case you want to add more data nodes make sure there are no even number of master nodes in the cluster also the environment variable NUMBER_OF_MASTERS is updated accordingly.



## PVC Resizing Configuration

[here](https://kubernetes.io/blog/2018/07/12/resizing-persistent-volumes-using-kubernetes/)

1. Delete all the pods which are using pvc (delete the StatefulSet)
2. edit all pvc with as follows

```
kubectl -n elasticsearch edit  pvc storage-es-data-0
```

update the storage entry.

3. redeploy the `StatefulSet`



## The LOG Battle: Logstash and Fluentd

* source data processing pipeline
  * Logstash
  * Fluentd
* lightweight shippers
  * Filebeat
  * Fluentbit

[here](https://medium.com/tensult/the-log-battle-logstash-and-fluentd-c65f2f7c24b4)

### filebeat

[installation 1](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation.html)

[installation 2](https://crunchify.com/setup-guide-install-configure-filebeat/)

[configuration](https://www.elastic.co/guide/en/beats/filebeat/5.1/filebeat-configuration-details.html)


### Environment variables

This image can be configured by means of environment variables, that one can set on a `Deployment`.

* [CLUSTER_NAME](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#cluster.name)
* [NODE_NAME](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node.name)
* [NODE_MASTER](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#master-node)
* [NODE_DATA](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#data-node)
* [NETWORK_HOST](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#network-interface-values)
* [HTTP_ENABLE](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-http.html#_settings_2)
* [HTTP_CORS_ENABLE](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-http.html#_settings_2)
* [HTTP_CORS_ALLOW_ORIGIN](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-http.html#_settings_2)
* [NUMBER_OF_MASTERS](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-zen.html#master-election)
* [MAX_LOCAL_STORAGE_NODES](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#max-local-storage-nodes)
* [ES_JAVA_OPTS](https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html)
* [ES_PLUGINS_INSTALL](https://www.elastic.co/guide/en/elasticsearch/plugins/current/installation.html) - comma separated list of Elasticsearch plugins to be installed. Example: `ES_PLUGINS_INSTALL="repository-gcs,x-pack"`
* [SHARD_ALLOCATION_AWARENESS](https://www.elastic.co/guide/en/elasticsearch/reference/current/allocation-awareness.html#CO287-1)
* [SHARD_ALLOCATION_AWARENESS_ATTR](https://www.elastic.co/guide/en/elasticsearch/reference/current/allocation-awareness.html#CO287-1)
* [MEMORY_LOCK](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#bootstrap.memory_lock) - memory locking control - enable to prevent swap (default = `true`) .
* DISCOVERY_SERVICE (default = elasticsearch-discovery)
* [REPO_LOCATIONS](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html#_shared_file_system_repository) - list of registered repository locations. For example `"/backup"` (default = `[]`). The value of REPO_LOCATIONS is automatically wrapped within an `[]` and therefore should not be included in the variable declaration. To specify multiple repository locations simply specify a comma separated string for example `"/backup", "/backup2"`.
* [PROCESSORS](https://github.com/elastic/elasticsearch-definitive-guide/pull/679/files) - allow elasticsearch to optimize for the actual number of available cpus (must be an integer - default = 1)

### Backup
Mount a shared folder (for example via NFS) to `/backup` and make sure the `elasticsearch` user
has write access. Then, set the `REPO_LOCATIONS` environment variable to `"/backup"` and create
a backup repository:

`backup_repository.json`:
```
{
  "type": "fs",
  "settings": {
    "location": "/backup",
    "compress": true
  }
}
```

```bash
curl -XPOST http://<container_ip>:9200/_snapshot/nas_repository -d @backup_repository.json`
```

Now, you can take snapshots using:

```bash
curl -f -XPUT "http://<container_ip>:9200/_snapshot/nas_repository/snapshot_`date --utc +%Y_%m_%dt%H_%M`?wait_for_completion=true"
```



## azure blob storage and snapshots

For configuring the Azure blobe storage as snapshot registry, first create the blobe storage account in azure and then you have to configure the following environment variable for the es deployment


variable name	|	default value	| description
---------------------|---------------------|---------------------
AZURE_REPOSITORY_CONFIG	| -	| This value should be "true" to enable the snapshots |
AZURE_REPOSITORY_ACCOUNT_NAME	|	-	|	configure the blob storage name |
AZURE_REPOSITORY_ACCOUNT_KEY	|	-	|	configure the key1 or key2 value here |



## Google bucket and snapshots

### create secret `gcp-es-sa-token` with your serviceaccount.json file as follow :

```
kubectl -n elastic-cluster create secret generic gcp-es-sa-token --from-file=/path/to/serviceaccount.json
```

variable name	|	default value	| description
---------------------|---------------------|---------------------
GCS_REPOSITORY_CONFIG	| -	| This value should be "true" to enable the snapshots |

**Note** : Create one service account json file and mount it to "/opt/secrets/serviceaccount.json"


## AWS s3 and snapshots

variable name	|	default value	| description
---------------------|---------------------|---------------------
S3_REPOSITORY_CONFIG	| -	| This value should be "true" to enable the snapshots |
S3_ACCESS_KEY	|	-	|	configure the access key for s3 |
S3_SECRET_KEY	|	-	|	configure the secret key for s3 |


### register the snapshot repository


#### Azure 

**note : i am using the sub container name `backup-container` in here, you can give any name here but you have to create the container in the storage account before starting the elasticsearch**

```
curl -XPUT 'http://192.168.0.12:9200/_snapshot/es_backup' -H 'Content-Type: application/json' -d '{
  "type": "azure",
  "settings": {
    "account": "default",
    "container": "backup-container",
    "base_path": "backups",
    "chunk_size": "32MB",
    "compress": true
  }
}'
```

## GCS

```
curl -XPUT 'http://kibanAdmin:ARHelmKibanaAdmiNSecreT@34.71.225.186:9200/_snapshot/es_backup' -H 'Content-Type: application/json' -d '{
  "type": "gcs",
  "settings": {
    "bucket": "essnapshot",
    "client": "default",
    "base_path": "backups",
    "chunk_size": "32MB",
    "compress": true
  }
}'
```


## S3

```
curl -XPUT 'http://192.168.0.12:9200/_snapshot/es_backup' -H 'Content-Type: application/json' -d '{
  "type": "s3",
  "settings": {
    "bucket": "my_bucket",
    "client": "default",
    "base_path": "backups",
    "chunk_size": "32MB",
    "compress": true
  }
}'
```


## Deploy Snapshot Cronjob

**Note : Please update the only-cronjob-value.yml file with your confioguration before deploying it. This cronjob will automatically create the snapshot for every 1 hr if the myenv environment is production** 

```console
$ helm install es-cron ar-repo/elasticsearch -n elastic-cluster -f only-cronjob-value.yml
```


### create a sample snapshot

```
curl -XPUT 'http://kibanAdmin:ARHelmKibanaAdmiNSecreT@34.71.225.186:9200/_snapshot/es_backup/snapshot_1' -H 'Content-Type: application/json' -d '{ "indices":"*","include_global_state":false }'
```


## Backup and Restore Steps

### Backup

* Install plugin:-

```
sudo ES_PATH_CONF=/etc/elasticsearch/es-node-2 /usr/share/elasticsearch/bin/elasticsearch-plugin install repository-azure
```

* change the configuration:-

```
sudo nano /etc/elasticsearch/es-node-2/elasticsearch.yml
cloud.azure.storage.default.account: xxxxxxxxxxx
cloud.azure.storage.default.key: xxxxxx
```

* Restart ES Service:-

```
sudo systemctl restart es-node-2_elasticsearch.service
```

* create Repo:-

```
curl -XPUT 'http://localhost:9200/_snapshot/azurebackup' -H 'Content-Type: application/json' -d '{ "type": "azure", "settings": { "container": "elasticsearch-snapshots", "base_path": "sunbirddevtele"} }'
```

* create Snapshot:-

```
curl -XPUT 'http://localhost:9200/_snapshot/azurebackup/snapshot_1' -H 'Content-Type: application/json' -d '{ "indices":"*","include_global_state":false }'
```

* check status of backup:-

```
curl -XGET 'http://localhost:9200/_snapshot/azurebackup/_all'
```


### Restore

* Install plugin:-

```
sudo ES_PATH_CONF=/etc/elasticsearch/es-node-2 /usr/share/elasticsearch/bin/elasticsearch-plugin install repository-azure
```

* change the configuration:-

```
sudo nano /etc/elasticsearch/es-node-2/elasticsearch.yml
cloud.azure.storage.default.account: xxxxxxxxxxx
cloud.azure.storage.default.key: xxxxxx
```

* Restart ES Service:- 

```
sudo systemctl restart es-node-1_elasticsearch.service
```

* create Repo:-

```
curl -XPUT 'http://localhost:9200/_snapshot/azurebackup' -H 'Content-Type: application/json' -d '{ "type": "azure", "settings": { "container": "elasticsearch-snapshots", "base_path": "sunbirddevtele"} }'
```

* Delete unwanted indices:- 

```
curl -XDELETE http://localhost:9200/_all
```

* Restore from snapshot:-

```
curl -XPOST 'http://localhost:9200/_snapshot/azurebackup/snapshot_1/_restore'
```


### Reference links ::

[snapshots-restore](https://github.com/project-sunbird/sunbird-devops/wiki/elasticsearch-backup-and-restore-for-telemetry-and-composite-search-to-azure-blob)

[es-azure snapshots plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/6.8/repository-azure.html)

[es-azure plugin description](https://www.elastic.co/guide/en/elasticsearch/plugins/7.4/repository-azure-repository-settings.html)

[blob-storage and snapshots 1](https://azure.microsoft.com/en-in/blog/archive-elasticsearch-indices-to-azure-blob-storage-using-the-azure-cloud-plugin/)

[blob-storage and snapshots 2](https://stackoverflow.com/questions/54113059/elasticsearch-snapshot-creation-understanding-how-where-to-store-them-to)
