# suse-ai-lab

This repo explain in an easy way, how to deploy the `SUSE AI stack` with the help of `Fleet` in a local single-node cluster.

Remember, all the SUSE AI Apps are only available through `Rancher Application Collection`.


## Part 1: Prerequisites:

In order to deploy the complete SUSE AI stack, we do need to prepare a few basic settings, so we can use `Fleet` and `Rancher Application Collection` together in a seemless way! Most important here is getting access for `Helm Charts` and `Images` to pull them from `Rancher Application Collection`.

1. For accessing `Rancher Application Collection` you should create a Service Token (with `username` and `password`)
```
#shell:
APPCO_USERNAME=<your-appco-username>
APPCO_PASSWORD=<your-appco-password>
```

2. Creating a basic-auth secret to pull `helm charts` from `Rancher Application Collection`
```
cat << EOF > appco-secret.yaml 
apiVersion: v1
kind: Secret
metadata:
  name: application-collection
  namespace: fleet-local
type: kubernetes.io/basic-auth
stringData:
  username: $APPCO_USERNAME
  password: $APPCO_PASSWORD
EOF

kubectl apply -f appco-secret.yaml
```

3. Creating a namespace `suse-ai`
```
kubectl create namespace suse-ai
```

4. Creating a docker secret to pull `images` from `Rancher Application Collection`
```
kubectl create secret docker-registry application-collection \
   --docker-server=dp.apps.rancher.io \
   --docker-username=$APPCO_USERNAME \
   --docker-password=$APPCO_PASSWORD \
   -n suse-ai
```


## Part 2: Deploy all the SUSE AI Apps:

```
cat << EOF > suse-ai-apps.yaml
apiVersion: fleet.cattle.io/v1alpha1
kind: GitRepo
metadata:
  name: suse-ai-apps
  namespace: fleet-local
spec:
  repo: "https://github.com/dieterreuter/suse-ai-lab"
  branch: main
  helmSecretName: application-collection
  helmRepoURLRegex: (oci|https)://dp.apps.rancher.io/*
  paths:
  - apps/ollama
  - apps/milvus
  - apps/open-webui
EOF

kubectl apply -f suse-ai-apps.yaml
```

## Part 3: Just delete everything - cleanup:

```
kubectl delete namespace suse-ai
kubectl delete secret application-collection -n fleet-local
```
