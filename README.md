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

1. [Architecture](#1---architecture)
2. [Writing gRPC services in Go](#2---writing-grpc-services-in-go)
3. [Creating a Helm chart for deploying services](#3---creating-a-helm-chart-for-deploying-services)
4. [Creating an OpenShift Template for deploying services](#4---creating-an-openshift-template-for-deploying-services)
5. [Creating a Kubernetes Operator for deploying services](#5---creating-a-kubernetes-operator-for-deploying-services)
6. [Installing and operating Istio Service Mesh with OpenShift](#6---installing-and-operating-istio-service-mesh-with-openshift)
7. [Deploying the demo services using Istio](#7---deploying-the-demo-services-using-istio)
8. [Deploying the demo services without using Istio](#8---deploying-the-demo-services-without-using-istio)

## 1 - Architecture

This demo is composed of four microservices modeling how a person buy products in an eCommerce:

- [Account](https://github.com/drhelius/grpc-demo-account): Models a user account. The user can have many orders in it. The account also has a reference to user information.
- [Order](https://github.com/drhelius/grpc-demo-order): This is a group of products ordered by the user.
- [User](https://github.com/drhelius/grpc-demo-user): The user personal information.
- [Product](https://github.com/drhelius/grpc-demo-product): A description of a product in the store including price an details.

All four microservices are written in go using gRPC as the main communication framework. Additionally, an HTTP (REST) listener is also provided for each of them.

The relationships between the services look like this:

Three deployment methods are provided for demonstration purposes:

- Helm Chart
- Kubernetes Operator
- OpenShift Template

## 2 - Writing gRPC services in Go

TODO

## 3 - Creating a Helm chart for deploying services

TODO

`helm package helm-charts/grpc-demo-services`

`helm package helm-charts/grpc-demo-services-istio`

`helm repo index docs --url https://drhelius.github.io/grpc-demo/`

`helm fetch grpc-demo/grpc-demo-services`

`helm install --set account.route=account-grpc-demo.mycluster.com grpc-demo-istio grpc-demo/grpc-demo-services-istio`

`helm uninstall grpc-demo-istio`

## 4 - Creating an OpenShift Template for deploying services

OpenShift templates are not available in other Kubernetes distributions but they are very convenient for simple deployments if you are working with OpenShift.

These templates can be parameterized but the lack of dynamism (loops and conditionals) usually makes Helm a better option.

Two templates are provided in this repo for deploying the demo services both with Istio and without it:

- [grpc-demo-template-istio.yaml](openshift-templates/grpc-demo-template-istio.yaml)
- [grpc-demo-template.yaml](openshift-templates/grpc-demo-template.yaml)

These templates define all the manifests needed in order to get the services deployed and working.

The parameters allow you to configure the services image, version, replicas and resources.

The `APP_NAME` parameter is just an identifier to label all the manifests created by the templates and organize the view in the OpenShift Developer Console.

The `grpc-demo-template-istio.yaml` template expects an additional `ACCOUNT_ROUTE` parameter to expose the Account service using an [Ingress Gateway](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/). Make sure to provide a valid *fqdn* for this route that makes sense in your cluster. The default value `account-grpc-demo.mycluster.com` is just a placeholder and will not work out of the box.

## 5 - Creating a Kubernetes Operator for deploying services

TODO

## 6 - Installing and operating Istio Service Mesh with OpenShift

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

With all four operators installed in your cluster you are ready to deploy Istio.

First create a namespace for the control plane, the name `istio-system` is recommended:

`$ oc new-project istio-system`

### Deploy Service Mesh control plane

With the project ready we can create the control plane.

A Service Mesh Control Plane manifest is [provided in this repo](openshift-service-mesh/service-mesh-control-plane.yaml). Use it to bootstrap the installation of Istio in OpenShift:

`$ oc create -f openshift-service-mesh/service-mesh-control-plane.yaml -n istio-system`

Istio operator will then create all the deployments that conform the control plane. After a few minutes it should look like this:

## 7 - Deploying the demo services using Istio

You can choose three different methods to deploy the demo services and setup the service mesh:

- Helm Chart
- Kubernetes Operator
- OpenShift Template

But first you need to create a namespace for the services and tell Istio to start monitoring this namespace by adding it to the [Member Roll](https://docs.openshift.com/container-platform/4.5/service_mesh/service_mesh_install/installing-ossm.html#ossm-member-roll-create_installing-ossm).

### Create a project to deploy the demo services

`$ oc new-project grpc-demo-istio`

### Create the service mesh member roll

A Service Mesh Member Roll manifest is [provided in this repo](openshift-service-mesh/service-mesh-member-roll.yaml). It includes the `grpc-demo-istio` project just created. If you are using a different name for the project you should change it accordingly.

Use it to create the Member Roll:

`$ oc create -f openshift-service-mesh/service-mesh-member-roll.yaml -n istio-system`

### Deploying the demo services using a Helm Chart

If you wish to use the provided Helm chart follow this steps.

Make sure you are working with the right namespace:

`$ oc project grpc-demo-istio`

Add the chart repository to your helm client:

```bash

$ helm repo add grpc-demo https://drhelius.github.io/grpc-demo/
"grpc-demo" has been added to your repositories

$ helm repo list
NAME        URL
grpc-demo   https://drhelius.github.io/grpc-demo/
```

Make sure you get the latest list of charts:

```bash
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "grpc-demo" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

You can inspect the chart before you install it in your cluster:

```bash
$ helm show chart grpc-demo/grpc-demo-services-istio
apiVersion: v2
description: A group of interconnected GRPC demo services written in Go that run on
  OpenShift Service Mesh.
home: https://github.com/drhelius/grpc-demo
icon: https://raw.githubusercontent.com/openshift/console/master/frontend/public/imgs/logos/golang.svg
keywords:
- go
- grpc
- demo
- service
- istio
maintainers:
- email: isanchez@redhat.com
  name: Ignacio Sánchez
  url: https://twitter.com/drhelius
name: grpc-demo-services-istio
sources:
- https://github.com/drhelius/grpc-demo
version: 1.0.0
```

The chart can be paramterized. These are the default values for all the parameters:

```yaml
appName: grpc-demo-istio

account:
  image: quay.io/isanchez/grpc-demo-account
  version: v1.0.0
  replicas: 1
  route: account-grpc-demo.mycluster.com

order:
  image: quay.io/isanchez/grpc-demo-order
  version: v1.0.0
  replicas: 1

product:
  image: quay.io/isanchez/grpc-demo-product
  version: v1.0.0
  replicas: 1

user:
  image: quay.io/isanchez/grpc-demo-user
  version: v1.0.0
  replicas: 1

limits:
  memory: "200"
  cpu: "0.5"

requests:
  memory: "100"
  cpu: "0.1"
```

Note that you must provide a valid *fqdn* for the route that is going to expose the Account service using an Ingress Gateway. This *fqdn* should make sense in your cluster so change `account-grpc-demo.mycluster.com` for a route valid in your cluster.

Install the chart using a custom Account route:

`$ helm install --set account.route=account-grpc-demo.mycluster.com grpc-demo-istio grpc-demo/grpc-demo-services-istio`

After a few minutes the services should be up an running:

```bash
$ oc get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
account-v1.0.0   1/1     1            1           3m1s
order-v1.0.0     1/1     1            1           3m1s
product-v1.0.0   1/1     1            1           3m1s
user-v1.0.0      1/1     1            1           3m1s
```

You can uninstall all by running:

`$ helm uninstall grpc-demo-istio`

## 8 - Deploying the demo services without using Istio

TODO





