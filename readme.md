[![Build Status](https://travis-ci.org/rennokki/lar8s.svg?branch=master)](https://travis-ci.org/rennokki/lar8s)

[![PayPal](https://img.shields.io/badge/PayPal-donate-blue.svg)](https://paypal.me/rennokki)

# Lar8s
Lar8s is a package that helps you scaffold better your Laravel app on Kubernetes. It is a great way to learn how Kubernetes works and how can it be used to power your Laravel app.

# Shipping your product using Docker & K8s
To find more about how to deploy your Laravel app in Kubernetes, check [this Medium article](https://medium.com/@alexrenoki/run-laravel-on-kubernetes-5259188b10ca) that explains the concepts and the methodology using this specific repo.
This readme provides only documentation on running microservices, like Redis, Postgres or MongoDB.

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
Running Postgres is easy. First, modify the `kubernetes/postgres/deployment.yaml` file and set your desired resources counts. Then, modify the `kubernetes/postgres/secrets.yaml` with your base64-encoded values. For persistent data, change the `kubernetes/postgres/volume.yaml` file.

Run the configurations and all should be fine:
```bash
$ kubectl create -f kubernetes/postgres/service.yaml
$ kubectl create -f kubernetes/postgres/volume.yaml
$ kubectl create -f kubernetes/postgres/secrets.yaml
$ kubectl create -f kubernetes/postgres/deployment.yaml
```

# Coming soon
* Elasticsearch
* MySQL/MariaDB
* Memcached
