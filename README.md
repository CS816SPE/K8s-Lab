# K8s-Lab

## Setup Kubernetes Cluster

We will be using **[Kubeadm-dind](https://github.com/kubernetes-sigs/kubeadm-dind-cluster)** to spin up Kubernetes cluster

- Pre-requisites
  - Make sure docker is installed and running
  - Allow non-root users to run `docker` following the instructions [here](https://docs.docker.com/install/linux/linux-postinstall/) 

- Download the shell script which has the necessary code to download and spin up Kubernetes cluster
  `wget https://github.com/kubernetes-sigs/kubeadm-dind-cluster/releases/download/v0.1.0/dind-cluster-v1.13.sh`
- Gves executable permission to the script `chmod +x dind-cluster-v1.13.sh`
- Spin up the cluster `sudo ./dind-cluster-v1.13.sh up`
- Install `kubectl` (Only for Ubuntu users)
  ```
  sudo apt-get update && sudo apt-get install -y apt-transport-https
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
  sudo apt-get update
  sudo apt-get install -y kubectl
  ```

- Get used to the following K8s commands
  - `kubectl get pods`
  - `kubectl get pods --all-namespaces`
  - `kubectl get nodes`
  - `kubectl get cs`

### Install WeaveScope

We will use WeaveScope on the cluster to get a visualisation of the cluster

- Use a remote Yaml configuration file to install Weavescope on the cluster 
  `kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-service-type=NodePort"`

- Get the port on which Weavescope is running
  `NODE_PORT=$(kubectl get -n weave svc weave-scope-app --output=jsonpath='{range.spec.ports[0]}{.nodePort}')`

- Get the pod name, so as to get the IP it is running on in the next step 
  `POD_NAME=$(kubectl get -n weave pod --selector=weave-scope-component=app -o jsonpath='{.items..metadata.name}')`
- Get the IP of the node, the pod is running on, using the name from the previous step
   `HOST_IP=$(kubectl get pods -n weave $POD_NAME --output=jsonpath='{range.status}{.hostIP}')`

The weavescope dashboard should now be reachable at `echo $HOST_IP:$NODE_PORT`.

Play around the dashboard for a while and get used to it.

### Deploy an App on the cluster

1. Run Nginx App
   `kubectl apply -f https://raw.githubusercontent.com/CS816SPE/K8s-Lab/master/deployment.yaml`

2. Expose the App
   `kubectl apply -f https://raw.githubusercontent.com/CS816SPE/K8s-Lab/master/service.yaml`

3. Access the App

   1. Choose one of the pod and find it's node by running `kubectl get pods -o wide`
   2. Get the IP of the node you chose by running `kubectl get nodes -o wide`
   3. Now, get the port of the app by running the command `kubectl get svc`. Pick the port on the right side of **':'** 
   4. Access the app using the IP and the port

4. Delete one of the instances from the WeaveScope dashboard and see what happens

5. Scale up the App, while monitoring the WeaveScope dashboard (Tip: Arrange your terminal & browser such that you can see the changes live)
   `kubectl scale deployment --replicas 7 nginx-dep`

6. Scale down the App
   `kubectl scale deployment --replicas 2 nginx-dep`

7. Delete the service & the app

   `kubectl delete -f https://raw.githubusercontent.com/CS816SPE/K8s-Lab/master/service.yaml`
   `kubectl delete -f https://raw.githubusercontent.com/CS816SPE/K8s-Lab/master/deployment.yaml`

   

### Clean your setup

- Delete Weavescope App `kubectl delete -f "https://cloud.weave.works/k8s/scope.yaml?k8s-service-type=NodePort"`
- Stop your cluster `./dind-cluster-v1.13.sh down`

- Remove the containers and volumes used to create the cluster `./dind-cluster-v1.13.sh clean` 
