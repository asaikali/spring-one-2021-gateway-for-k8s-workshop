# tanzu-spring-cloud-gateway

A guided tutorial for experimenting with Tanzu Spring Cloud Gateway on minikube.

   
## Launch minikube 

We will use minikube for this lab, configured to run with 8 CPUs and 16GB of RAM. 

1. Open a terminal window
2. rung the shell script `./start-minkube.sh` to launch minikube, this step can take a couple of minutes, you know it worked when
you see output similar to below.

```
😄  minikube v1.22.0 on Ubuntu 20.04 (xen/amd64)
🆕  Kubernetes 1.21.2 is now available. If you would like to upgrade, specify: --kubernetes-version=v1.21.2
✨  Using the docker driver based on existing profile
❗  Your cgroup does not allow setting memory.
    ▪ More information: https://docs.docker.com/engine/install/linux-postinstall/#your-kernel-does-not-support-cgroup-swap-limit-capabilities
❗  Your cgroup does not allow setting memory.
    ▪ More information: https://docs.docker.com/engine/install/linux-postinstall/#your-kernel-does-not-support-cgroup-swap-limit-capabilities
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔄  Restarting existing docker container for "minikube" ...
🐳  Preparing Kubernetes v1.20.10 on Docker 20.10.7 ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

3. Validate that minikube is running by executing `kubectl get nodes` you should see output similar to below
```
NAME       STATUS   ROLES                  AGE   VERSION
minikube   Ready    control-plane,master   29m   v1.20.10
```

## Point docker cli at the docker registry in minikube

To keep things simple for the lab we are going to use a the docker regstiry running
inside minikube. We are going to need to tell the docker cli to use this registry.

1. execute the command `minikube docker-env` you will see output similat to the one below
```
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.49.2:2376"
export DOCKER_CERT_PATH="/home/ubuntu/.minikube/certs"
export MINIKUBE_ACTIVE_DOCKERD="minikube"

# To point your shell to minikube's docker-daemon, run:
# eval $(minikube -p minikube docker-env)
```
The environment variables printed by this command point the docker cli to minikube.

2. Execute the command `eval $(minikube -p minikube docker-env)` to point the docker cli to the registry inside minikube

3. validate the configured by executing the command `docker images` you should the images listed below in the output below.

```
REPOSITORY                                TAG        IMAGE ID       CREATED         SIZE
k8s.gcr.io/kube-proxy                     v1.20.10   945c9bce487a   2 weeks ago     99.7MB
k8s.gcr.io/kube-controller-manager        v1.20.10   2f450864515d   2 weeks ago     116MB
k8s.gcr.io/kube-apiserver                 v1.20.10   644cadd07add   2 weeks ago     122MB
k8s.gcr.io/kube-scheduler                 v1.20.10   4c9be8dc650b   2 weeks ago     47.3MB
gcr.io/k8s-minikube/storage-provisioner   v5         6e38f40d628d   5 months ago    31.5MB
kubernetesui/dashboard                    v2.1.0     9a07b5b4bfac   8 months ago    226MB
k8s.gcr.io/etcd                           3.4.13-0   0369cf4303ff   12 months ago   253MB
k8s.gcr.io/coredns                        1.7.0      bfe3a36ebd25   14 months ago   45.2MB
kubernetesui/metrics-scraper              v1.0.4     86262685d9ab   17 months ago   36.9MB
k8s.gcr.io/pause                          3.2        80d28bedfe5d   18 months ago   683kB
```


## Download Spring Cloud Gateway for Kubernetes 

We have downladed [Spring Cloud Gateway for Kubernetes from Tanzu Net](https://network.pivotal.io/products/spring-cloud-gateway-for-kubernetes) you can 
find it extracted in `/home/ubuntu/Downloads/spring-cloud-gateway-k8s-1.0.2` 

2. execute the command  `cd /home/ubuntu/Downloads/spring-cloud-gateway-k8s-1.0.2`

3. Explore the directory structure using by executing the `tree` command you should see output below

```text
├── dashboards
│   ├── grafana-spring-cloud-gateway-for-kubernetes.json
│   └── wavefront-spring-cloud-gateway-for-kubernetes.json
├── helm
│   ├── scg-image-values.yaml
│   └── spring-cloud-gateway-1.0.2.tgz
├── images
│   ├── gateway-1.0.2.tar
│   └── scg-operator-1.0.2.tar
├── load-images.sh
└── scripts
    ├── install-spring-cloud-gateway.sh
    └── relocate-images.sh
```
3. Inspect the contents of the `images` folder. Notice it contains two container images packaged as `.tar` files. 
   the script located in `scripts/relocate-images.sh` is used to publish `gateway-1.0.2.tar` and 
   `scg-operator-1.0.2.tar` images into a private container registry accessible from the kubernetes cluster you 
   deploy the spring cloud gateway on.

4. Inspect the `helm` chart folder and notice that it contains a gzipped helm chart. This will 
   be used to install the spring cloud gateway operator. 
   
5. Inspect the `scripts` folder notice that it contains two shell scripts, one for publishing the private 
   container images to a registry, and the second scripts runs a helm to install the operator. 

## Load the SCGW container images into docker 

In a production depolyment you will publish the operator image and the gateway image into a private regstiry.
For this lab we are going to use the registry that is native to minikube to keep things simple, we have 
prepared shell script to load the images to the registry for you. 

1. make sure you are in the directory `~/Downloads/spring-cloud-gateway-k8s-1.0.2` 

1. execute the shell script `./load-images.sh` this can take a few minutes to complete. you will see output
   similar to the one below

```text
21639b09744f: Loading layer [==================================================>]  65.51MB/65.51MB
b6ae40cff175: Loading layer [==================================================>]  3.072kB/3.072kB
82c4776d33ef: Loading layer [==================================================>]  26.32MB/26.32MB
ca8be400e8e5: Loading layer [==================================================>]  414.2kB/414.2kB
1779dee469f5: Loading layer [==================================================>]  3.391MB/3.391MB
e2109e6b9be5: Loading layer [==================================================>]  4.304MB/4.304MB
ec0381c8f321: Loading layer [==================================================>]  6.656kB/6.656kB
207a6aa9b3e2: Loading layer [==================================================>]  144.2MB/144.2MB
0b18b1f120f4: Loading layer [==================================================>]   3.05MB/3.05MB
7fbc97c38fad: Loading layer [==================================================>]   5.12kB/5.12kB
f0ee16238e8a: Loading layer [==================================================>]   1.61MB/1.61MB
aa7edff7133e: Loading layer [==================================================>]  56.32kB/56.32kB
fcc507beb4cc: Loading layer [==================================================>]  4.096kB/4.096kB
273d10576e70: Loading layer [==================================================>]  60.19MB/60.19MB
6ca77678afd9: Loading layer [==================================================>]    299kB/299kB
5f70bf18a086: Loading layer [==================================================>]  1.024kB/1.024kB
80165ef615c5: Loading layer [==================================================>]  916.5kB/916.5kB
888ed16fa8d4: Loading layer [==================================================>]  2.518MB/2.518MB
c0b1daf33de4: Loading layer [==================================================>]  23.55kB/23.55kB
1dc94a70dbaa: Loading layer [==================================================>]  3.584kB/3.584kB
Loaded image: registry.pivotal.io/spring-cloud-gateway-for-kubernetes/scg-operator:1.0.2
19f70bace1fd: Loading layer [==================================================>]   5.12kB/5.12kB
f0ee16238e8a: Loading layer [==================================================>]   1.61MB/1.61MB
aa7edff7133e: Loading layer [==================================================>]  56.32kB/56.32kB
fcc507beb4cc: Loading layer [==================================================>]  4.096kB/4.096kB
85952c6e2cb5: Loading layer [==================================================>]  60.81MB/60.81MB
6ca77678afd9: Loading layer [==================================================>]    299kB/299kB
5f70bf18a086: Loading layer [==================================================>]  1.024kB/1.024kB
9bb4ef4745e1: Loading layer [==================================================>]  348.7kB/348.7kB
888ed16fa8d4: Loading layer [==================================================>]  2.518MB/2.518MB
95ae23bf8138: Loading layer [==================================================>]  34.82kB/34.82kB
1dc94a70dbaa: Loading layer [==================================================>]  3.584kB/3.584kB
Loaded image: registry.pivotal.io/spring-cloud-gateway-for-kubernetes/gateway:1.0.2
```

3. check your local docker images `docker images` and look for spring cloud gateway images. You should output similar
to below.
   
```text
REPOSITORY                                                             TAG        IMAGE ID       CREATED         SIZE
k8s.gcr.io/kube-proxy                                                  v1.20.10   945c9bce487a   2 weeks ago     99.7MB
k8s.gcr.io/kube-apiserver                                              v1.20.10   644cadd07add   2 weeks ago     122MB
k8s.gcr.io/kube-controller-manager                                     v1.20.10   2f450864515d   2 weeks ago     116MB
k8s.gcr.io/kube-scheduler                                              v1.20.10   4c9be8dc650b   2 weeks ago     47.3MB
gcr.io/k8s-minikube/storage-provisioner                                v5         6e38f40d628d   5 months ago    31.5MB
kubernetesui/dashboard                                                 v2.1.0     9a07b5b4bfac   8 months ago    226MB
k8s.gcr.io/etcd                                                        3.4.13-0   0369cf4303ff   12 months ago   253MB
k8s.gcr.io/coredns                                                     1.7.0      bfe3a36ebd25   14 months ago   45.2MB
kubernetesui/metrics-scraper                                           v1.0.4     86262685d9ab   17 months ago   36.9MB
k8s.gcr.io/pause                                                       3.2        80d28bedfe5d   18 months ago   683kB
registry.pivotal.io/spring-cloud-gateway-for-kubernetes/gateway        1.0.2      d2eb85620d05   41 years ago    308MB
registry.pivotal.io/spring-cloud-gateway-for-kubernetes/scg-operator   1.0.2      83bf3d63ef02   41 years ago    308MB
```

## Install Spring Cloud Gateway Operator

Spring Cloud Gateway operator is installed from a helm chart. We have configured the helm chart to use 
the images we loaded in the previous section. 

1. execute the command `cat helm/scg-image-values.yaml` you should see the output below
```yaml
gateway:
   image: "registry.pivotal.io/spring-cloud-gateway-for-kubernetes/gateway:1.0.2"
scg-operator:
   image: "registry.pivotal.io/spring-cloud-gateway-for-kubernetes/scg-operator:1.0.2"
```
4. Execute `./scripts/install-spring-cloud-gateway.sh` you should see output similar to below 
```text
chart tarball: spring-cloud-gateway-1.0.2.tgz
chart name: spring-cloud-gateway
Waiting up to 2m for helm installation to complete
Release "spring-cloud-gateway" does not exist. Installing it now.
NAME: spring-cloud-gateway
LAST DEPLOYED: Wed Sep  1 04:36:56 2021
NAMESPACE: spring-cloud-gateway
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
This chart contains the Kubernetes operator for Spring Cloud Gateway.
Install the chart spring-cloud-gateway-crds before installing this chart

Checking Operator pod state 
✔ Operator pod is running

Checking deployment replicas 
✔ Operator deployment is ready

Checking custom resource definitions 
✓ springcloudgatewaymappings.tanzu.vmware.com successfully installed
✓ springcloudgatewayrouteconfigs.tanzu.vmware.com successfully installed
✓ springcloudgateways.tanzu.vmware.com successfully installed

```

5. execute the command  `kubectl get all -n spring-cloud-gateway` you should see a pod running the spring cloud gateway 
operator as shown below. 
   
```text
NAME                               READY   STATUS    RESTARTS   AGE
pod/scg-operator-679f77dbb-8w9bt   1/1     Running   0          3m42s

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/scg-operator   ClusterIP   10.96.32.243   <none>        80/TCP    3m42s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/scg-operator   1/1     1            1           3m42s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/scg-operator-679f77dbb   1         1         1       3m42s
```

6. The spring cloud gateway operator installed three kubernetes custom resources definitions. Execute the command 
   `kubectl get crds` and you will see all the created custom CRDs shown below. These CRDs will be used to configure 
   the gateway. 
   
```text
 NAME                                              CREATED AT
springcloudgatewaymappings.tanzu.vmware.com       2021-03-04T05:47:20Z
springcloudgatewayrouteconfigs.tanzu.vmware.com   2021-03-04T05:47:20Z
springcloudgateways.tanzu.vmware.com              2021-03-04T05:47:20Z
```

## Define a gateway instance 

1. Inspect the `demo/my-gateway.yml` file it contains the YAML shown below which defines 
a spring cloud gateway instance.
   
```yaml
apiVersion: "tanzu.vmware.com/v1"
kind: SpringCloudGateway
metadata:
  name: my-gateway
```

2. Execute the command `kubectl apply -f demo/my-gateway.yml` which will submit a request to the cluster
to deploy an instance of spring cloud gateway. 

3. execute the command `kubectl get all` you should see a pod of the spring cloud gateway running
or being launched in the cluster's default namespace as shown in the output below.
   
```text
NAME               READY   STATUS    RESTARTS   AGE
pod/my-gateway-0   1/1     Running   0          3m36s

NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP    3h12m
service/my-gateway            ClusterIP   10.96.178.167   <none>        80/TCP     3m36s
service/my-gateway-headless   ClusterIP   None            <none>        5701/TCP   3m36s

NAME                          READY   AGE
statefulset.apps/my-gateway   1/1     3m36s
```

4. Inspect the file `demo/route-config.yml` it contains gateway configuration CRD that proxies requests
set the gateway to github. Notice that this route configuration is generic.  

```yaml
apiVersion: "tanzu.vmware.com/v1"
kind: SpringCloudGatewayRouteConfig
metadata:
  name: my-gateway-routes
spec:
  routes:
    - uri: https://github.com
      predicates:
        - Path=/**
      filters:
        - StripPrefix=1
```

5. run the command `kubectl apply -f demo/route-config.yml` you 

6. Inspect the file `demo/mapping.yml` notice that it points at the gateway instance we already deployed
at the configuration defined in `route-config.yml`
```yaml
apiVersion: "tanzu.vmware.com/v1"
kind: SpringCloudGatewayMapping
metadata:
  name: test-gateway-mapping
spec:
  gatewayRef:
    name: my-gateway
  routeConfigRef:
    name: my-gateway-routes
```

7. run the command `kubectl apply -f demo/mapping.yml` this will configure the already deployed 
   instance to pass proxy requests to github.com 
   
8. We don't have a load balancer configured with our kind cluster, so we will use port forwarding to
   test the gateway. run the command `kubectl port-forward service/my-gateway 8080:80` you will see 
   output like
```text
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

9. Using a browser go to `http://localhost:8080` you should see the github site. The request are 
   going to spring cloud gateway which is then sending them to github.com. Congrats you have 
   managed to deploy a spring cloud gateway instance using a CRD. There are many more things
   that you can do with spring cloud gateway that we will discuss in the rest of the workshop this is 
   just the start. 
   

## Deploy Animal Rescue Sample Application (SKIP we will doing a demo of this)

The [animal rescue](https://github.com/spring-cloud-services-samples/animal-rescue/) sample 
application demonstrate many  commonly used features of spring cloud gateway. To deploy 
the application and use the login with Auth0 feature you will need a client secret provided
by the workshop instructor, look for it in the workshop Slack channel. 

1. Edit the file `animal-rescue/overlays/sso-secret-for-gateway/secrets/test-sso-credentials.txt` 
   set the `client-secret` value to the one provided on the workshop slack 
   
2. Deploy the app using the command `kustomize build ./animal-rescue/ | kubectl apply -f -`

3. Check the animal rescue components that are deployed into the cluster using the command 
   `kubectl get all -n animal-rescue` you should see output similar to the one below.
   
```text
2021-03-04 21:30:18 ⌚  asaikali-a01 in ~/workshops/tanzu-spring-cloud-gateway
± |main S:14 U:1 ✗| → kubectl get all -n animal-rescue
NAME                                          READY   STATUS    RESTARTS   AGE
pod/animal-rescue-backend-74c54b577f-n7zxn    1/1     Running   0          6m54s
pod/animal-rescue-frontend-76cf7899b9-s2pxz   1/1     Running   0          6m54s
pod/gateway-demo-0                            1/1     Running   0          6m54s
pod/gateway-demo-1                            1/1     Running   0          6m15s

NAME                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/animal-rescue-backend    ClusterIP   10.103.68.238    <none>        80/TCP     6m54s
service/animal-rescue-frontend   ClusterIP   10.106.109.195   <none>        80/TCP     6m54s
service/gateway-demo             ClusterIP   10.110.12.46     <none>        80/TCP     6m54s
service/gateway-demo-headless    ClusterIP   None             <none>        5701/TCP   6m54s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/animal-rescue-backend    1/1     1            1           6m54s
deployment.apps/animal-rescue-frontend   1/1     1            1           6m54s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/animal-rescue-backend-74c54b577f    1         1         1       6m54s
replicaset.apps/animal-rescue-frontend-76cf7899b9   1         1         1       6m54s

NAME                            READY   AGE
statefulset.apps/gateway-demo   2/2     6m54s
```

4. Notice that there are two instances of the gateway deployed, this makes the gateway 
   highly available within the k8s cluster. The gateway instances replicate data between 
   each other in order to track who is logged in into the application. 
   
5. Notice that there are two pods one running the front end application and one running the 
   backend api.
   
6. Notice that the front end and backend services are ClusterIP, so they can be reached 
   inside the k8s cluster but not from outside. We are going to need to reach the gateway to 
   direct traffic to the frontend and backend.
   
7. Since we are running on a laptop, and we don't have a loadbalancer we are going to use k8s
 port forwarding to direct traffic from localhost to the gateway service. Execute the command
   `kubectl port-forward service/gateway-demo 9090:80 -n animal-rescue`
   
8. Using a browser visit `http://localhost:9090/rescue` you will see the home page for the 
   application. click around on the application and explore it.
   
9. Click the login button in the top right corner, you will be redirected to Auth0 you can login
   with a Google account and then you will be sent back to the application where you will 
   be able to see the userid in the top right corner. 

## Delete spring cloud gateway (informational optional)

1. delete the demo gateway `kubectl delete -f demo`
1. delete the demo gateway `kustomize build ./animal-rescue/ | kubectl delete -f -`
2. Uninstall the operator `helm uninstall spring-cloud-gateway -n spring-cloud-gateway`

## Resources

* [GA release blog post](https://tanzu.vmware.com/content/blog/vmware-spring-cloud-gateway-kubernetes-distributed-api-gateway-generally-available)
* [documentation](https://docs.pivotal.io/scg-k8s/1-0/)
