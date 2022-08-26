# Skupper Hello World - Gateway

[![main](https://github.com/skupperproject/skupper-example-hello-world-gateway/actions/workflows/main.yaml/badge.svg)](https://github.com/skupperproject/skupper-example-hello-world-gateway/actions/workflows/main.yaml)

#### A minimal HTTP application deployed across Kubernetes and Container sites using Skupper

This example is part of a [suite of examples][examples] showing the
different ways you can use [Skupper][website] to connect services
across cloud providers, data centers, and edge sites.

[website]: https://skupper.io/
[examples]: https://skupper.io/examples/index.html

#### Contents

* [Overview](#overview)
* [Prerequisites](#prerequisites)
* [Step 1: Install the Skupper command-line tool](#step-1-install-the-skupper-command-line-tool)
* [Step 2: Set up your Kubernetes namespace](#step-2-set-up-your-kubernetes-namespace)
* [Step 3: Install Skupper in your Kubernetes namespace](#step-3-install-skupper-in-your-kubernetes-namespace)
* [Step 4: Deploy and expose the backend on your local machine](#step-4-deploy-and-expose-the-backend-on-your-local-machine)
* [Step 5: Deploy and expose the frontend on Kubernetes](#step-5-deploy-and-expose-the-frontend-on-kubernetes)
* [Step 6: Test the application](#step-6-test-the-application)
* [Accessing the web console](#accessing-the-web-console)
* [Cleaning up](#cleaning-up)
* [About this example](#about-this-example)

## Overview

This example is a very simple multi-service HTTP application
deployed across a Kubernetes cluster and a bare-metal host or VM.

It contains two services:

* A backend service that exposes an `/api/hello` endpoint.  It
  returns greetings of the form `Hi, <your-name>.  I am <my-name>
  (<pod-name>)`.

* A frontend service that sends greetings to the backend and
  fetches new greetings in response.

With Skupper, you can run the backend as a container on your local
machine and the frontend in Kubernetes and maintain connectivity
between the two services without exposing the backend to the public
internet.

<!-- <img src="images/entities.svg" width="640"/> -->

## Prerequisites

* A working installation of Docker or Podman

* The `kubectl` command-line tool, version 1.15 or later
  ([installation guide][install-kubectl])

* Access to a Kubernetes cluster, from [any provider you
  choose][kube-providers]

[install-kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[kube-providers]: https://skupper.io/start/index.html#prerequisites

## Step 1: Install the Skupper command-line tool

The `skupper` command-line tool is the primary entrypoint for
installing and configuring Skupper.  You need to install the
`skupper` command only once for each development environment.

On Linux or Mac, you can use the install script (inspect it
[here][install-script]) to download and extract the command:

~~~ shell
curl https://skupper.io/install.sh | sh
~~~

The script installs the command under your home directory.  It
prompts you to add the command to your path if necessary.

For Windows and other installation options, see [Installing
Skupper][install-docs].

[install-script]: https://github.com/skupperproject/skupper-website/blob/main/docs/install.sh
[install-docs]: https://skupper.io/install/index.html

## Step 2: Set up your Kubernetes namespace

Use `kubectl create namespace` to create the namespace you wish
to use (or use an existing namespace).  Use `kubectl config
set-context` to set the current namespace for your session.

_**Console for hello:**_

~~~ shell
kubectl create namespace hello
kubectl config set-context --current --namespace hello
~~~

_Sample output:_

~~~ console
$ kubectl create namespace hello
namespace/hello created

$ kubectl config set-context --current --namespace hello
Context "minikube" modified.
~~~

## Step 3: Install Skupper in your Kubernetes namespace

The `skupper init` command installs the Skupper router and service
controller in the current namespace.

**Note:** If you are using Minikube, [you need to start `minikube
tunnel`][minikube-tunnel] before you install Skupper.

[minikube-tunnel]: https://skupper.io/start/minikube.html#running-minikube-tunnel

_**Console for hello:**_

~~~ shell
skupper init
~~~

_Sample output:_

~~~ console
$ skupper init
Waiting for LoadBalancer IP or hostname...
Skupper is now installed in namespace 'hello'.  Use 'skupper status' to get more information.
~~~

## Step 4: Deploy and expose the backend on your local machine

Use `docker` to run the backend service on your local machine.
In the `hello` namespace, use the `skupper gateway expose`
command to expose the database on the Skupper network.
Use `kubectl get service/database` to ensure the database
service is available.

_**Console for hello:**_

~~~ shell
docker run --name backend --detach --rm -p 8080:8080 quay.io/skupper/hello-world-backend
skupper gateway expose backend localhost 8080 --type docker
kubectl get service/backend
~~~

## Step 5: Deploy and expose the frontend on Kubernetes

Use `kubectl create deployment` to deploy the frontend service
in `hello`.

Use `kubectl expose` with `--type LoadBalancer` to open network
access to the frontend service.

_**Console for hello:**_

~~~ shell
kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend
kubectl expose deployment/frontend --port 8080 --type LoadBalancer
~~~

_Sample output:_

~~~ console
$ kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend
deployment.apps/frontend created

$ kubectl expose deployment/frontend --port 8080 --type LoadBalancer
service/frontend exposed
~~~

## Step 6: Test the application

Now we're ready to try it out.  Use `kubectl get service/frontend`
to look up the external IP of the frontend service.  Then use
`curl` or a similar tool to request the `/api/health` endpoint at
that address.

**Note:** The `<external-ip>` field in the following commands is a
placeholder.  The actual value is an IP address.

_**Console for hello:**_

~~~ shell
kubectl get service/frontend
curl http://<external-ip>:8080/api/health
~~~

_Sample output:_

~~~ console
$ kubectl get service/frontend
NAME       TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
frontend   LoadBalancer   10.103.232.28   <external-ip>   8080:30407/TCP   15s

$ curl http://<external-ip>:8080/api/health
OK
~~~

If everything is in order, you can now access the web interface by
navigating to `http://<external-ip>:8080/` in your browser.

## Accessing the web console

Skupper includes a web console you can use to view the application
network.  To access it, use `skupper status` to look up the URL of
the web console.  Then use `kubectl get
secret/skupper-console-users` to look up the console admin
password.

**Note:** The `<console-url>` and `<password>` fields in the
following output are placeholders.  The actual values are specific
to your environment.

_**Console for hello:**_

~~~ shell
skupper status
kubectl get secret/skupper-console-users -o jsonpath={.data.admin} | base64 -d
~~~

_Sample output:_

~~~ console
$ skupper status
Skupper is enabled for namespace "hello" in interior mode. It is connected to 1 other site. It has 1 exposed service.
The site console url is: <console-url>
The credentials for internal console-auth mode are held in secret: 'skupper-console-users'

$ kubectl get secret/skupper-console-users -o jsonpath={.data.admin} | base64 -d
<password>
~~~

Navigate to `<console-url>` in your browser.  When prompted, log
in as user `admin` and enter the password.

## Cleaning up

To remove Skupper and the other resources from this exercise, use
the following commands.

_**Console for hello:**_

~~~ shell
docker stop backend
skupper gateway delete
skupper delete
kubectl delete service/frontend
kubectl delete deployment/frontend
~~~

## Next steps

Check out the other [examples][examples] on the Skupper website.

## About this example

This example was produced using [Skewer][skewer], a library for
documenting and testing Skupper examples.

[skewer]: https://github.com/skupperproject/skewer

Skewer provides some utilities for generating the README and running
the example steps.  Use the `./plano` command in the project root to
see what is available.

To quickly stand up the example using Minikube, try the `./plano demo`
command.
