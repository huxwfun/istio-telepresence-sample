# istio-telepresence-sample
A sample project using telepresence to develop istio micro-service locally

## WHY
It's known to be difficult to set up environments for microservices.

Usually, microservices are developed/deployed by different groups of people, keeping tracking and maintaining configurations, dependencies of those services can easily become impossible for individuals.

There're tools to help. For example, Helm for dependency management. However, it's still difficult to run tens or even hundreds of docker containers on everybody's laptop. Lots of teams maintain a universal dev cluster, for every programmer to work on. Then the problems of remote development kick in.

I'm gona work you through a sample here: with the help of Telepresence and Istio, we can develop our services localy while taking advantadge of shared dev clusters

Here're some ideas/tools for micro-service development on k8s https://kubernetes.io/blog/2018/05/01/developing-on-kubernetes/

## Before you begin
I assume readers understand the basics of k8s, Istio, and have a working k8s cluster (including minikube, dockerForMac k8s), also, I'm using [https://istio.io/docs/examples/bookinfo/](Istio bookinfo application) as sample, so you need to know the services of this application

I'm on MAC, using docker for mac and k8s integrated. Following steps should work on remote k8s or minidocker as well, if not, PR welcome.

This is not a detailed installation guide for Istio and its bookinfo app. But if you're using dockerForMac too, it could be done fast with these reminders

Install Istio, and the bookinfo sample application. It's well documented on istio official site, so I'll only list the basic steps here:
```
curl -L https://git.io/getLatestIstio | sh -
cd istio-1.0.6 
export PATH=$PWD/bin:$PATH
kubectl apply -f install/kubernetes/istio-demo.yaml
```
To make it simpler, I'm using istio WITHOUT mutual TLS auth between sidecars.

Check [https://istio.io/docs/setup/kubernetes/quick-start/](this page) to make sure your Istio installation finished


```
kubectl label namespace default istio-injection=enabled
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```
I'm on dockerForMac, now http://localhost/productpage shall be accessible

[https://www.telepresence.io/reference/install](Install Telepresence), it's os dependent, so you need to check official guide if you're not on MAC.
```
brew cask install osxfuse
brew install datawire/blackbird/telepresence
```

## Setup local ratings service
I choose ratings service only because it's written in nodejs and I already have tools installed. It's perfectly okay for you to try details(in ruby) or reviews(in java), steps shall be similiar

Go to your clone of this repo, run
```
yarn install
yarn start 9080
```

verify your setup by visiting http://localhost:9080/ratings/0

## Start telepresence
```
telepresence --new-deployment ratings-local --expose 9080
```
Telepresence will change your local network(including dns)

try
```
curl -v ratings:9080/ratings/0
curl -v ratings-local:9080/ratings/0
```
Magically, ou shall be able to visit the ratings service in your k8s cluster and your local ratings service

Now it's easy to setup a Istio request routing to forword requests to your local ratings service

I'm gonna do it for user with name jason
```
kubectl apply -f ratings.yaml
```

Refresh your localhost/productpage a few times, the original ratings service returns 5, 4 for 2 reviewers, the local one return 2, 1.

You shall see 2, 1 if login as jason, if not, 5, 4.

Now you can change your local code, debug, whatever else you do for local development.

And your local service can visit resources inside your k8s cluster just like it's in the cluster too

For users other than jason, the k8s dev cluster works as normal. Other developers won't notice a thing

## TODO
1. route not just by user, but also other useful request info
2. use docker with Telepresence so your local network won't be affected
3. How to deal with local MQ consumer easily?
