# Istio gRPC Services Demo

---

STILL IN DEVELOPMENT

---

This is a demo to showcase the features of some of the technologies that may be involved in modern microservice development and operation within a Kubernetes platform.

This technologies include [Go](https://golang.org/), [gRPC](https://grpc.io/), [Istio](https://istio.io/), [Helm](https://helm.sh/) and [Kubernetes Operators](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/).

Note that this is just a demo and may not represent the real life, but it could help you to understand basic concepts that may be the building blocks to create more complex gRPC microservices in a service mesh.

Three different mechanisms are provided to deploy the demo. You can also choose wether you use Istio or not.

All the examples are provided for Red Hat OpenShift but could be applied to any Kubernetes distribution. If you want to run OpenShift on your laptop you may want to try [Red Hat CodeReady Containers](https://developers.redhat.com/products/codeready-containers/overview).

## Index

1. [Architecture](#architecture)
2. [Writing gRPC services in Go](#writing-grpc-services-in-go)
3. [Creating a Helm chart for deploying services](#creating-a-helm-chart-for-deploying-services)
4. [Creating an OpenShift Template for deploying services](#creating-an-openshift-template-for-deploying-services)
5. [Creating a Kubernetes Operator for deploying services](#writing-grpc-services-in-go)
6. [Installing and operating Istio Service Mesh with OpenShift](#writing-grpc-services-in-go)
7. [Deploying the demo services using Istio](#deploying-the-demo-services-using-istio)
8. [Deploying the demo services without using Istio](#deploying-the-demo-services-without-using-istio)

## Architecture

This demo is composed of four microservices modeling how a person buy products in an eCommerce:

- [Account](https://github.com/drhelius/grpc-demo-account): Models a user account. The user can have many orders in it. The account also has a reference to user information.
- [Order](https://github.com/drhelius/grpc-demo-order): This is a group of products ordered by the user.
- [User](https://github.com/drhelius/grpc-demo-user): The user personal information.
- [Product](https://github.com/drhelius/grpc-demo-product): A description of a product in the store including price an details.

All four microservices are written in go using gRPC as the main communication framework. Additionally, an HTTP (REST) listener is also provided for each of them.

Three deployment methods are provided for demonstration purposes:

- Helm Chart
- Kubernetes Operator
- OpenShift Template

## Writing gRPC services in Go

TODO

## Creating a Helm chart for deploying services

TODO

`helm package helm-charts/grpc-demo-services`

`helm package helm-charts/grpc-demo-services-istio`

`helm repo index docs --url https://drhelius.github.io/grpc-demo/`

`helm fetch grpc-demo/grpc-demo-services`

`helm install --set account.route=account-grpc-demo.mycluster.com grpc-demo-istio grpc-demo/grpc-demo-services-istio`

`helm uninstall grpc-demo-istio`

## Creating an OpenShift Template for deploying services

OpenShift templates are not available in other Kubernetes distributions but they are very convenient for simple deployments if you are working with OpenShift.

These templates can be parameterized but the lack of dynamism (loops and conditionals) usually makes Helm a better option.

Two templates are provided in this repo for deploying the demo services both with Istio and without it:

- [grpc-demo-template-istio.yaml](openshift-templates/grpc-demo-template-istio.yaml)
- [grpc-demo-template.yaml](openshift-templates/grpc-demo-template.yaml)

These templates define all the manifests needed in order to get the services deployed and working.

The parameters let you configure the services image, version, replicas and resources.

The *APP_NAME* parameter is just an identifier to label all the manifest created by the templates and organize the view in the Developer Console.

The `grpc-demo-template-istio.yaml` expects an additional parameter (*ACCOUNT_ROUTE*) to expose the account service using an Ingress Gateway. Make sure to provide a valid fqdn for this route that makes sense in your cluster. The default value (*account-grpc-demo.mycluster.com*) is just a placeholder and will not work out of the box.

## Installing and operating Istio Service Mesh with OpenShift

In order to install OpenShift Service Mesh you should go through the steps explained in the [official docs](https://docs.openshift.com/container-platform/4.5/service_mesh/service_mesh_install/preparing-ossm-installation.html). The following is a simplified guide.

Istio in OpenShift is installed by running a set of operators. Before installing the Red Hat Service Mesh operator you have to install the Elasticsearch, Jaeger and Kiali operators.

### Install Elasticsearch Operator

- Update Channel: 4.5
- Installation Mode: All namespaces
- Installed Namespace: openshift-operators
- Approval Strategy: Automatic

### Install Red Hat OpenShift Jaeger Operator

- Update Channel: stable
- Installation Mode: All namespaces
- Installed Namespace: openshift-operators
- Approval Strategy: Automatic

### Install Kiali Operator (provided by Red Hat)

- Update Channel: stable
- Installation Mode: All namespaces
- Installed Namespace: openshift-operators
- Approval Strategy: Automatic

### Install Red Hat OpenShift Service Mesh operator

- Update Channel: stable
- Installation Mode: All namespaces
- Installed Namespace: openshift-operators
- Approval Strategy: Automatic

### Create project istio-system

`oc new-project istio-system`

### Deploy Service Mesh control plane

A Service Mesh Control Plane (SMCP) is [provided in this repo](openshift-service-mesh/service-mesh-control-plane.yaml). Use it to bootstrap the installation of Istio in OpenShift.

`oc create -f openshift-service-mesh/service-mesh-control-plane.yaml -n istio-system`

## Deploying the demo services using Istio

### Create a project to deploy our services

`oc new-project grpc-demo-istio`

### Create the service mesh member roll

`oc create -f openshift-service-mesh/service-mesh-member-roll.yaml -n istio-system`

## Deploying the demo services without using Istio

TODO





