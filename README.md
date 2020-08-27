# Boutique Misconfigurations
This repository contains several Istio/K8S misconfigurations as applied to the [GCP Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo). This repository allows users to examine common misconfigurations as applied to a familiar microservice example. 
## Prerequisites/Versioning
### Prerequisites
These prerequisites are mostly from [GCP](https://github.com/BoutiqueMisconfigurations/microservices-demo/tree/3983672bef190fd51eb6388ffb9a710eae96a81e#prerequisites):
   - `gcloud` see [here](
https://cloud.google.com/kubernetes-engine/docs/quickstart)).
   - kubectl (can be installed via `gcloud components install kubectl`)
   - Local Kubernetes cluster deployment tool: (This demo has only been tested with Docker)
        - [Docker for Desktop (recommended for Mac/Windows)](https://www.docker.com/products/docker-desktop)
          - It provides Kubernetes support as [noted
     here](https://docs.docker.com/docker-for-mac/kubernetes/)
          - **Ensure that docker daemon is running!**
   - [skaffold]( https://skaffold.dev/docs/install/) ([ensure version ≥v1.10](https://github.com/GoogleContainerTools/skaffold/releases))

### Detailed Versioning Info
We use the following versions:
- [Istio 1.6.6](https://istio.io/latest/news/releases/1.6.x/announcing-1.6.6/)
```
$ istioctl version
client version: 1.6.6
control plane version: 1.6.6
data plane version: 1.6.6 (65 proxies)
```
- [Kubernetes 1.18](https://kubernetes.io/docs/setup/release/notes/)
```
$ kubectl version -o yaml
clientVersion:
  buildDate: "2020-04-16T23:35:15Z"
  compiler: gc
  gitCommit: 52c56ce7a8272c798dbc29846288d7cd9fbae032
  gitTreeState: clean
  gitVersion: v1.18.2
  goVersion: go1.14.2
  major: "1"
  minor: "18"
  platform: darwin/amd64
serverVersion:
  buildDate: "2020-06-01T22:20:10Z"
  compiler: gc
  gitCommit: fb7add51f767aae42655d39972210dc1c5dbd4b3
  gitTreeState: clean
  gitVersion: v1.15.12-gke.2
  goVersion: go1.12.17b4
  major: "1"
  minor: 15+
  platform: linux/amd64
```
## Installation: Setting Up the Service Mesh
We will be following a mixture of the Option 2 Configuration for GKE detailed in `microservices-demo/README.md` and with the Istio 1.6 installation [guide](https://istio.io/latest/docs/setup/getting-started/#install). Here is a step-by-step guide:

0) First, clone this repo with `git clone https://github.com/BoutiqueMisconfigurations/boutique-misconfigurations.git --recursive`. 
1) This step is similar to step 1 of Option 2 for GKE setup. Run these commands from the `gcp-src/` directory:
```
gcloud services enable container.googleapis.com
```

Next, run this command **instead of the create command in that README.:**

```
gcloud container clusters create demo --enable-autoupgrade --enable-autoscaling --min-nodes=3 --max-nodes=10 --num-nodes=5 --zone=us-central1-a --enable-network-policy --machine-type=n1-standard-4
```
The `--machine-type` flag ensures that we will have enough CPU power to actually run the mesh, the `--enable-network-policy` flag is needed to use NetworkPolicies, and the `--machine-type` flag is to ensure there is enough CPU to run the whole mesh.

```
kubectl get nodes
```
You should see several nodes up and running.

2) Next, configure Docker.
```
gcloud auth configure-docker -q
```
3) Now, we will download Istio following the Istio guide. First, download Istio to the root directory of this repository.  *Note, this demo uses [Istio v1.6.6](https://istio.io/latest/news/releases/1.6.x/announcing-1.6.6/)*
```
$ curl -L https://istio.io/downloadIstio | sh -
$ cd istio-1.6.6
$ export PATH=$PWD/bin:$PATH
```
4) Now, we will install Istio:
```
istioctl install
```
```
kubectl label namespace default istio-injection=enabled
```
5) Now, we need to apply the Istio manifests:
```
kubectl apply -f ./istio-manifests
```
6) Now, we go back to the GKE instructions, and use `skaffold` to apply the Kube manifests.
```
skaffold run --default-repo=gcr.io/[PROJECT_ID]
```
7) Now, we can verify the Services are up and running, and ensure they have sidecars. This is an example of a 
correct setup:
```
$ kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
adservice-5896c9c8f7-khftg               2/2     Running   0          114s
cartservice-795cdfdf5d-smrk8             2/2     Running   2          114s
checkoutservice-f8b7f88cf-8cwcw          2/2     Running   0          113s
currencyservice-6667649b-x264g           2/2     Running   0          113s
emailservice-877bd649b-6p78w             2/2     Running   0          112s
frontend-69946666bc-rqkfx                2/2     Running   0          111s
loadgenerator-57656b7885-7rh4g           2/2     Running   3          111s
paymentservice-7774fd9655-gq5pc          2/2     Running   0          110s
productcatalogservice-5cc7fc4c8c-hgpk4   2/2     Running   0          110s
recommendationservice-b994687f4-8srw5    2/2     Running   0          109s
redis-cart-578f55b7d5-4gp42              2/2     Running   0          109s
shippingservice-685cd97f5b-xc4mp         2/2     Running   0          108s
```
You should see `2/2 Running` for every service. If they are 1/1, you are missing the Envoys.

8) To access the Boutique, run:
```
INGRESS_HOST="$(kubectl -n istio-system get service istio-ingressgateway \
   -o jsonpath='{.status.loadBalancer.ingress[0].ip}')"
echo "$INGRESS_HOST"
```
and direct your web browser to that IP address.

## System Specs
### Docker Configuration
Docker was allocated 3 CPUs, 6.50 GB of Memory, 1 GB Swap, and 32 GB for the Disk image size.
### Local Computer Specs
All tests were written and tested using a 2019 MacBook Pro. Here are the detailed specs:

#### System Hardware Overview:
```
  Model Name:	MacBook Pro
  Model Identifier:	MacBookPro16,1
  Processor Name:	6-Core Intel Core i7
  Processor Speed:	2.6 GHz
  Number of Processors:	1
  Total Number of Cores:	6
  L2 Cache (per Core):	256 KB
  L3 Cache:	12 MB
  Hyper-Threading Technology:	Enabled
  Memory:	16 GB
  Boot ROM Version:	1037.120.87.0.0 (iBridge: 17.16.15300.0.0,0)
```
#### System Software Overview:
```
  System Version:	macOS 10.15.5 (19F101)
  Kernel Version:	Darwin 19.5.0
  Boot Volume:	Macintosh HD
  Boot Mode:	Normal
  Secure Virtual Memory:	Enabled
  System Integrity Protection:	Enabled
```

## Structure
This repo has the following sub-directories:
- `microservices-demo/` contains a git submodule of our fork of GCP's [source repository](https://github.com/GoogleCloudPlatform/microservices-demo/). All changes are explicitly stated in the files that were modified, and the `README.md` contains a list of all changed files.
- `misconfigs/` contains several nested directories, each containing a particular misconfiguration of the Online Boutique. Each subdirectory of `misconfigs/` contains a `README.md` that explains what bug that directory exposes.
