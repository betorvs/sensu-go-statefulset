# Sensu Statefulset

Travis CI: [![Build Status](https://travis-ci.org/betorvs/sensu-go-statefulset.svg?branch=master)](https://travis-ci.org/betorvs/sensu-go-statefulset)

## Deploy

You can use this direcly as a helm repository or download these and install manually.

### Usage

## Create credentials secret

First create using kubectl:

```sh
kubectl create secret generic credentials --from-literal=sensu_backend_cluster_admin_username="admin" \
    --from-literal=sensu_backend_cluster_admin_password='P@ssw0rd!2GO' \
    -n sensu --dry-run=client -o yaml > credentials-secret.yaml
```

Install the helm repository and install sensu chart:

```sh
helm repo add betorvs https://betorvs.github.io/sensu-go-statefulset/
helm repo update
kubectl create ns sensu
kubectl apply -f credentials-secret.yaml
helm upgrade --install sensu-backend --namespace sensu betorvs/sensu-go-statefulset
```


## Create Sensu Certificates using cfssl

```sh
cd sensu-certs/
cfssl gencert -initca sensu-ca.json | cfssljson -bare ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server sensu-backend.json | cfssljson -bare sensu-backend
```

### Create secrets from Certificates

```sh
kubectl create secret generic sensu-backend-pem --from-file=sensu-backend.pem=sensu-backend.pem \
    --from-file=sensu-backend-key.pem=sensu-backend-key.pem -n sensu --dry-run=client -o yaml > ../sensu-backend-secrets.yaml
kubectl create secret generic sensu-ca-pem --from-file=sensu-ca.pem=ca.pem -n sensu \
    --dry-run=client -o yaml > ../sensu-ca-secrets.yaml

```


### Deploy with ssl example

```sh
kubectl apply -f sensu-certs/sensu-backend-secrets.yaml
kubectl apply -f sensu-certs/sensu-ca-secrets.yaml
helm upgrade --install sensu-backend -f values-ssl.yaml --namespace sensu betorvs/sensu-go-statefulset
```


## Create "exported" confimap:

From an test environment, create a dump using sensuctl, like:

```sh
sensuctl dump all --format yaml --file exported.yaml

```

Then create an Kubernetes Secret using this:

```sh
kubectl create secret generic exported --from-file=exported.yaml=exported.yaml \
    -n sensu --dry-run=client -o yaml > exported-secret.yaml
```

Then enable `importResources` and run:

```sh
kubectl apply -f exported-secret.yaml
helm upgrade --install sensu-backend --namespace sensu .
```

### To test the deployment

To test deployment:
```sh
helm test sensu-backend
```

## To remove

```sh
helm delete sensu-backend --purge
```