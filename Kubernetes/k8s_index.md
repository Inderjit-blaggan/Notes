# Core Kubernetes topics:

1. [Scheduling In k8s](#scheduling-in-k8s)



# Miscelanious Topics and Best practises:

1. [Postgres DB in Kubernetes](#postgres-db-in-kubernetes)
2. 
---


## Scheduling In k8s
- Doc link: https://medium.com/@josip.cloud/understand-scheduling-in-kubernetes-1c7b24050638 



### Postgres DB in Kubernetes

- Youtube link to intro: https://www.youtube.com/watch?v=Ny9RxM6H6Hg
- Documentation:

### Installation using cloudnativePostgress
```
# Source: https://gist.github.com/vfarcic/8301efb15748af1da3e376b7132e519e

###################################################################
# Should We Run Databases In Kubernetes? CloudNativePG PostgreSQL #
# https://youtu.be/Ny9RxM6H6Hg                                    #
###################################################################

# Additional Info:
# - CloudNativePG: https://cloudnative-pg.io
# - EDB: https://enterprisedb.com

#########
# Setup #
#########

# Create a Kubernetes cluster

git clone https://github.com/vfarcic/cloud-native-pg-demo

cd cloud-native-pg-demo

helm repo add cnpg https://cloudnative-pg.github.io/charts

helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts

helm repo update

helm upgrade --install cnpg cnpg/cloudnative-pg \
    --namespace cnpg-system --create-namespace --wait

helm upgrade --install prometheus-community \
    prometheus-community/kube-prometheus-stack \
    --namespace observability --create-namespace \
    --values https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/main/docs/src/samples/monitoring/kube-stack-config.yaml \
    --wait

kubectl --namespace observability apply \
    --filename grafana-configmap.yaml

kubectl create namespace demo

########################################################
# Running PostgreSQL In Kubernetes Using CloudNativePG #
########################################################

cat cluster.yaml

kubectl --namespace demo apply --filename cluster.yaml

kubectl --namespace demo get clusters

kubectl --namespace demo get pods

kubectl --namespace demo get statefulsets

kubectl --namespace demo get services

kubectl --namespace demo get secrets

kubectl --namespace demo get clusters

kubectl --namespace observability port-forward \
    service/prometheus-community-grafana 8080:80

# Open http://localhost:8080 in a browser

# Use `admin` as the username and `prom-operator` as the password

# Open https://cloudnative-pg.io/documentation/1.20/samples/cluster-example-full.yaml

###########
# Destroy #
###########

# Press Ctrl+C to stop port forwarding
# Destroy or reset the cluster

```
