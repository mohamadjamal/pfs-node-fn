Login to GCP
 - https://console.cloud.google.com

Create a new GCP project 
 - pfs-demo-project

Navigate to Kubernetes Engine
 -  Kubernetes Engine API enabled (automatically)

Create a Cluster
 -  Name = pfs-demo-cluster-1, Zone = asia-south1-a (Mumbai)

Install gcloud SDK Shell and configure your account. All the following commands in this demo are executed in the gcloud SDK shell.

Set the current project as default

```sh
gcloud projects list
gcloud config set project <PROJECT_ID>
```

Install the pfs cli and added to windows path 

Download pfs think or thin distributions and unzip it.

create a new context with your credentials

```sh
gcloud container clusters get-credentials pfs-demo-cluster-1
```

Grant yourself cluster-admin permissions.

```sh
kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=mohamadjamal.aj@gmail.com
```

Verify cluster

```sh
kubectl config current-context
```

List all pods

```sh
kubectl get pods --all-namespaces
```
Install PFS to the containers

```sh
pfs system install -m C:\Work\tao-workspace\pfs-downloads\pfs-distro-thin\manifest.yaml
```

Create a service account with the required permission to push function images to GCR

```sh
gcloud iam service-accounts create pfs-demo-image
```
grant the pfs-demo-image account the storage.admin role

```sh
gcloud projects add-iam-policy-binding <PROJECT_ID> --member serviceAccount:pfs-demo-image@<PROJECT_ID>.iam.gserviceaccount.com --role roles/storage.admin

gcloud projects add-iam-policy-binding pfs-demo-project-20191009 --member serviceAccount:pfs-demo-image@pfs-demo-project-20191009.iam.gserviceaccount.com --role roles/storage.admin

```
Create a private authentication key for the push service account and store it in a local file

```sh
gcloud iam service-accounts keys create  --iam-account "pfs-demo-image@<PROJECT_ID>.iam.gserviceaccount.com" C:\Work\tao-workspace\gcr-storage-admin.json

gcloud iam service-accounts keys create  --iam-account "pfs-demo-image@pfs-demo-project-20191009.iam.gserviceaccount.com" C:\Work\tao-workspace\gcr-storage-admin.json
```
Use the pfs CLI to initialize PFS resources in a Kubernetes namespace.

```sh
pfs namespace init default -m C:\Work\tao-workspace\pfs-downloads\pfs-distro-thin\manifest.yaml --gcr C:\Work\tao-workspace\gcr-storage-admin.json
```

Create a simple java function

```sh
pfs function create greeting --git-repo https://github.com/mohamadjamal/pfs-simple-java-fn.git --verbose

pfs function create greeting --local-path C:\Work\tao-workspace\pfs-simple-java-fn --handler com.demo.tao.cloud.Greeting --verbose
pfs function update greeting -–verbose
```

Invoking the funciton

```sh
pfs service invoke greeting --text -- -d "mohamad jamal aj"
 - curl 34.93.190.40/ -H 'Host: greeting.default.example.com' -H 'Content-Type: text/plain' -d 'mohamad jamal aj'
```

Delete the function

```sh
pfs service delete pfs-simple-java-fn-greeting
```

Other functions used in this demo

```sh
pfs function create uppercase -–git-repo  https://github.com/mohamadjamal/pfs-spring-boot-fn.git --verbose
pfs service invoke uppercase --text -- -d "mohamad jamal aj"

pfs function create hello -–git-repo  https://github.com/mohamadjamal/pfs-node-fn --artifact hello.js --verbose
pfs service invoke hello --text -- -d "mohamad jamal aj"

pfs function create hello-async -–git-repo  https://github.com/mohamadjamal/pfs-node-fn --artifact hello-async.js --verbose
pfs service invoke hello-async --text -- -d "mohamad jamal aj"
```

Eventing

```sh
pfs channel create my-input-channel
pfs subscription create --channel my-input-channel --subscriber greeting
pfs subscription create --channel my-input-channel --subscriber uppercase

kubectl get channel my-input-channel -o jsonpath='{.status.address.hostname}'
```


#### Todos

 - PubSub invocation
 - Update the ppt
 