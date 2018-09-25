[![Build Status](https://travis-ci.org/rennokki/lar8s.svg?branch=master)](https://travis-ci.org/rennokki/lar8s)

[![PayPal](https://img.shields.io/badge/PayPal-donate-blue.svg)](https://paypal.me/rennokki)

# Lar8s
Lar8s is a package that helps you scaffold better your Laravel app on Kubernetes. It is a great way to learn how Kubernetes works and how can it be used to power your Laravel app.

# Shipping your product using Docker & K8s
To find more about how to deploy your Laravel app in Kubernetes, check [this Medium article](https://medium.com/@alexrenoki/run-laravel-on-kubernetes-5259188b10ca) that explains the concepts and the methodology using this specific repo.
This readme provides only documentation on running microservices, like Redis, Postgres or MongoDB.

# Running on GCP, MacOS or Docker with Kubernetes on Windows
You need to set up a load balancer since it's not provided, so you'll end up with no IP address to connect to. To fix this, add the load balancer provided:
```
$ kubectl create -f kubernetes/nginx/load-balancer.yaml
```

Simply get the services and seek for your IP:
```bash
$ kubectl get svc
```

# Running Redis
Running specific services is easier. 

First, make sure you customize your `kubernetes/redis/deployment.yaml` with your requested resources counts.

To run Redis in a pod, alone, run the following commands, in this order:
```bash
$ kubectl create -f kubernetes/redis/service.yaml
$ kubectl create -f kubernetes/redis/deployment.yaml
$ kubectl create -f kubernetes/redis/config.yaml
```

If you wish to have persistent data, edit the `kubernetes/redis/volume.yaml` to fit your needs (in terms of space) and run:
```
$ kubectl create -f kubernetes/redis/volume.yaml
```

# Running Postgres
Running Postgres is easy. First, modify the `kubernetes/postgres/deployment.yaml` file and set your desired resources counts. Then, modify the `kubernetes/postgres/secrets.yaml` with your values. For persistent data, change the `kubernetes/postgres/volume.yaml` file.

Run the configurations and all should be fine:
```bash
$ kubectl create -f kubernetes/postgres/service.yaml
$ kubectl create -f kubernetes/postgres/volume.yaml
$ kubectl create -f kubernetes/postgres/secrets.yaml
$ kubectl create -f kubernetes/postgres/deployment.yaml
```

# MongoDB
MongoDB is a NoSQL service, meaning you'll never have a schema like you do in MySQL/Postgres. This unlocks you the ability to be fully flexible on the structure, since you don't need migrations, but the downside is that you'll never be consistent in data retrieved (you might end up sometimes with fields that do exist in other files, but in specific files won't) and joins dosn't work, since NoSQL relies on storing data and not on complex querying.

To run your pod with Mongo, run this. Before, make sure you modify your `secrets.yaml` file with your Base64-encoded data for the Mongo container and change your `volume.yaml`'s size the way you need:
```bash
$ kubectl create -f kubernetes/mongodb/service.yaml
$ kubectl create -f kubernetes/mongodb/volume.yaml
$ kubectl create -f kubernetes/mongodb/secrets.yaml
$ kubectl create -f kubernetes/mongodb/deployment.yaml
```

# Coming soon
* Elasticsearch
* MySQL/MariaDB
* Memcached
