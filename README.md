# Tanzu Application Platform installation on minikube

## TL;DR

Accept all EULA's
[Tanzu Application Platform](https://network.tanzu.vmware.com/products/tanzu-application-platform/)
[Tanzu Cluster Essentials](https://network.tanzu.vmware.com/products/tanzu-cluster-essentials/)
[Tanzu Build Service](https://network.tanzu.vmware.com/products/build-service/)
[Tanzu Build Service Dependencies](https://network.tanzu.vmware.com/products/tbs-dependencies/)
[Tanzu Buildpacks Suite](https://network.tanzu.vmware.com/products/tanzu-buildpacks-suite)
[Tanzu Stacks Suite](https://network.tanzu.vmware.com/products/tanzu-stacks-suite)


Start minikube
`minikube start --kubernetes-version='1.22.8' --memory='12g' --cpus='8'  --driver='kvm2'`

Tunnel minikube (is needed to successfully deploy TAP)
`minikube tunnel`

Install [Tanzu-CLI](https://network.tanzu.vmware.com/products/tanzu-application-platform/)

Download [Cluster Essentials](https://network.tanzu.vmware.com/products/tanzu-cluster-essentials/)

Extract Cluster Essentials

Export Install Variables

`
export INSTALL_BUNDLE="registry.tanzu.vmware.com/tanzu-cluster-essentials/cluster-essentials-bundle@sha256:ab0a3539da241a6ea59c75c0743e9058511d7c56312ea3906178ec0f3491f51d"
export INSTALL_REGISTRY_HOSTNAME="registry.tanzu.vmware.com"
export INSTALL_REGISTRY_USERNAME='<VMWARE USER>' 
export INSTALL_REGISTRY_PASSWORD='<VMWARE USER PASSWORD>'
` 

Install Cluster Essentials
`cd <CLUSTER ESSENTIALS FOLDER> && ./install.sh`

Export TAP Install Params

`
export TAP_VERSION="1.1.0"
export TAP_NAMESPACE="tap-install"
export DOMAIN="example.com"
export APPS_DOMAIN="apps.example.com"
export DOCKER_SERVER="https://index.docker.io/v1/"
export DOCKER_USERNAME='<REGISTRY USER>'
export DOCKER_PASSWORD='<REGISTRY USER PASSWORD>'
`

Create K8s NS

`kubectl create ns $TAP_NAMESPACE`

Setup TAP

`
tanzu secret registry add tap-registry --username $INSTALL_REGISTRY_USERNAME --password $INSTALL_REGISTRY_PASSWORD --server $INSTALL_REGISTRY_HOSTNAME --namespace $TAP_NAMESPACE --export-to-all-namespaces --yes
tanzu package repository add tanzu-tap-repository --url $INSTALL_REGISTRY_HOSTNAME/tanzu-application-platform/tap-packages:$TAP_VERSION --namespace $TAP_NAMESPACE 
tanzu package repository get tanzu-tap-repository --namespace $TAP_NAMESPACE 
tanzu package available list --namespace $TAP_NAMESPACE
export minikubeip=$(minikube ip) &&  sed "s/MINIKUBEIP/$minikubeip/g" values_MINIKUBEIP.yaml > out.yaml
tanzu package install tap -p tap.tanzu.vmware.com -v $TAP_VERSION --values-file out.yaml --poll-timeout 45m --namespace $TAP_NAMESPACE
`

Browse TAP

`echo "https://`
