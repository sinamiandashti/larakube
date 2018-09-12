[![Build Status](https://travis-ci.org/rennokki/lar8s.svg?branch=master)](https://travis-ci.org/rennokki/lar8s)

[![PayPal](https://img.shields.io/badge/PayPal-donate-blue.svg)](https://paypal.me/rennokki)

# Lar8s
Lar8s is a package that helps you scaffold better your Laravel app on Kubernetes. It is a great way to learn how Kubernetes works and how can it be used to power your Laravel app.

# Shipping your product using Docker & K8s
The shipping is done by packing up your app into an `alpine` image and pushing it into a Docker registry. Usually, i use Gitlab because it has a built-in private Docker registry, but you can use whatever you want.

In the tests, you pull the image and test your app. After the tests pass, deploying your app to development, staging and production can be done by building your `alpine` image  that contains your app which was previously built and tested, pushed to the registry and triggering your K8s cluster's rolling update function to pull that specific image and create containers.

# Installing
Simply download the repo and put your `docker/`, `kubernetes/` and `Dockerfile` into your root.

# Building your Laravel app's image
Note: The app image is different from the rest of the containers, like nginx and php-fpm. This adds speed to your app by using separate image with your project and it's faster and portable.

To build your image with your app, simply run this in the root of your project:
```bash
$ docker build . -t yourname/container_name:optional_tag
```

Before building the container, make sure you ran all the commands, from `composer install` for dependencies to NPM assets build for frontend files and so forth. Everything you do now will be live on server.

Comparing to basic deployment, your container will contain all the data and no deployment script is available. Make sure you load everything in the container right after the testing phase ends (with success)

I recommend using tags since you'll have various versions of your images and it's better to keep them tracked using the tag. Usually, it can be done by assigning:
* `version.major.minor` or like `version.major.minor-rc0.1` for tagging releases (like `rennokki/lar8s:3.2.0`)
* push's head commit SHA (`rennokki/lar8s:c3422b7`, like you can use when building tests for commits and shipping your container to tests by pulling that sha ref)

```bash
$ docker login --username yourusername --password yourpassword
```

You can push your image to the registry `push` command:
```bash
$ docker push rennokki/lar8s:3.2.0
```
or
```bash
$ docker push rennokki/lar8s:c3422b7
```

# Building Laravel's PHP environment
Laravel is pretty pretentious on libs and extensions, so lar8s provides support for PHP-FPM environment container, that you can find in `docker/php-fpm/` folder. Go to that directory and build it.

Tagging and pushing this container is important since you'll be pulling later in the Kubernetes configuration file on `laravel-app` deployment. This should not be tagged on every commit, PR or release. It can also be upgraded to newer versions if you decide to change the PHP-FPM container's settings.

```bash
$ cd docker/php-fpm/
```
```bash
$ docker build . -t rennokki/lar8s-php-fpm:1.0.0
```

Then you can push it to the registry (after logging into Docker CLI):
```bash
$ docker push rennokki/lar8s-php-fpm:1.0.0
```

# Installing Minikube
For testing purposes, Minikube is perfect. [Make sure you install your own.](https://github.com/kubernetes/minikube)

You can start your Minikube:
```bash
$ minikube start
```

After starting, additionally, enable the ingress addon for Minikube if you plan to use the ingress configuration:
```bash
$ minikube addons enable ingress
```

You'll want to use the Minikube dashboard, so this will open up a dashboard to your browser:
```bash
$ minikube dashboard
```

# Configuring Kubernetes' Laravel app.
Deploying your app is based on Deployments. You create a deployment configuration for your Laravel app, NGINX and PHP-FPM, that all come into the same pod, to communicate easily between them - NGINX serves as a frontend proxy, PHP-FPM is your backend environment and your app lies in the last container.

When scaling using an autoscaler (which is provided), they all 3 will scale and it will be easy to load balance between them.

First of all, open the `kubernetes/app/deployment.yaml` and change the following:
* `rennokki/lar8s:1.0.0` with your app's container build previously
* `rennokki/lar8s-php-fpm:1.0.0` with your PHP-FPM container built previously
* In the `resources` of each container, adapt your resources needs.

Open `kubernetes/app/secrets.yaml` and, under the `data`, add your Laravel's .env variables. This will not operate by configuring your own `.env` file from `.env.example`, this is done by explicitly passing secrets. Make sure you encode your strings in Base64 before adding them to the YAML file.

If you want persistent data (note recommended since your app container will not get the latest data) open `kubernetes/app/volume.yaml` and change `5Gi` to suit your application's needs on persisting volume capacity.

Open `kubernetes/nginx/config.yaml` and change your NGINX's configuration. You can leave it like that, it's working too.

After you've done editing the Kubernetes configurations, run the following commands. This will take care of all the secrets, volumes and services for your PHP-FPM, NGINX and Laravel app containers:
```bash
$ kubectl create -f kubernetes/app/secrets.yaml
```
```bash
$ kubectl create -f kubernetes/nginx/config.yaml
$ kubectl create -f kubernetes/nginx/service.yaml
```

If you opted for persistent volume:
```bash
$ kubectl create -f kubernetes/app/volume.yaml
```

After this, hit the deployment files to start creating the pods needed:
```bash
$ kubectl create -f kubernetes/app/deployment.yaml
$ kubectl create -f kubernetes/nginx/deployment.yaml
```

If you go to the dashboard, under `Pods` section or if you run `kubectl get pods` you will see your pods are actually created.

When they're done, you can find your URL to your app by getting the list of services on Minikube:
```bash
$ minikube service list
```

On the `laravel-app` service, there is a corresponding URL with a port. Copy that to your browser and you'll see your app is running.

# Autoscaling
If you plan to autoscale automatically (horizontally), edit the `kubernetes/app/autoscaler.yaml` file to fit your needs on `minReplicas` and `maxReplicas`, as well as `targetCPUUtilizationPercentage`.

# Ingress
Ingress is a facade that binds your ugly IP:PORT service to a domain.

There are two ways you can create ingress using provided ingress configurations:
* ingress on HTTP (without TLS)
* ingress on HTTPS (requires certificates)

To create an ingress on HTTP, simply edit `kubernetes/nginx/ingress.yaml` and replace the `kubernetes.test` domain with your own. This steps occurs even on TLS ingress.

After this, add the Ingress configuration to the Minikube:
```bash
$ kubectl create -f kubernetes/nginx/ingress.yaml
```

Edit your computer's hosts file with your Minikube's IP and the domain you have just replaced in the ingress file. To get the Minikube IP, run:
```bash
$ minikube ip
```

For HTTPS ingress, create your own self-signed certificates. After this, you'll end up with two files: a private key and a certificate file. Copy their content and turn them into base64.

Open the `kubernetes/nginx/tls-secres.yaml` file and paste your base64-encoded key to `tls.key` and your base64-encoded certificate to `tls.crt`.

Make sure you have opened the `kubernetes/nginx/ssl-ingress.yaml` file and replaced the `kubernetes.test` with your domain.

Then you just have to add the secrets and the  ingress to Minikube, in this exact order:
```bash
$ kubectl create -f kubernetes/nginx/tls-secrets.yaml
$ kubectl create -f kubernetes/nginx/ingress-ssl.yaml
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
