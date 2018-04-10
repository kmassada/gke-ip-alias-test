# IP aliasing

## Create a Network

```shell
gcloud beta container clusters create --enable-ip-alias --create-subnetwork name=my-cluster-subnet
```

## Create a cluster

```shell
gcloud beta container clusters create [CLUSTER_NAME] --enable-ip-alias \
--create-subnetwork="" [--cluster-ipv4-cidr [RANGE] --services-ipv4-cidr [RANGE] ]
```

## Create a workload

use github.com/kmassada/gke-ingress-test to create workloads.

## Identify urls to test

### Identify a nodeIP

```shell
kubectl get nodes -o=jsonpath='{.items[0].status.addresses[0]}'
```

### Identify a podIP:containerPort

note we are using as `metatada.labels` key and value `app: hey`, also this only works if you know your spec. assumes `containers[0].ports[0]` first container, first pod

```shell
 kubectl get pods -o jsonpath='{range .items[?(@.metadata.labels.app=="hey")]}{.status.podIP}{":"}{.spec.containers[0].ports[0].containerPort}{"\t"}{.metadata.name}{"\n"}'
```

this does the same

```shell
kubectl get endpoints
```

### nodePort

this also prints both nodeport, endpoint, TargetPort (same with containerPort) in our example.

```shell
kubectl describe services/hey-service
```

### Form urls

Pod IP: `10.60.0.6` targetPort: `8080`
Node IP: `10.0.0.2` nodePort: `30569`

## Create instance whithin network 

find cluster network

```shell
gcloud beta container clusters describe $CLUSTER_NAME --zone $ZONE --format json | jq '.| "\(.zone) \(.subnetwork) \(.network)"'
```

find image, ubunutu in our case

```shell
gcloud beta compute images list
```

create new instance

```shell
gcloud beta compute instances create instance-1 \
--zone=us-central1-c \
--machine-type=n1-standard-1 \
--subnet=$SUBNET \
--image=ubuntu-1710-artful-v20180405 \
--image-project=ubuntu-os-cloud \
--zone=$ZONE
```

ssh and setup instance with `curl`

```shell
gcloud beta compute ssh instance-1 --zone=$ZONE
nstance-1:~$ sudo apt-get update && sudo apt-get install curl
```

## hit endpoints

```shell
# Pod IP: `10.60.0.6` targetPort: `8080`
$ curl 10.60.0.6:8080
# Node IP: `10.0.0.2` nodePort: `30569`
$ curl 10.0.0.2:30569
```

## Doesn't expose service pod

this doesn't apply to service pod
