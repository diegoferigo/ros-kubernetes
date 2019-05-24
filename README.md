<p align="center">
    <h1 align="center">ros-kubernetes</h1>
</p>

<p align="center">
<b><a href="https://github.com/diegoferigo/ros-kubernetes#what">What</a></b>
•
<b><a href="https://github.com/diegoferigo/ros-kubernetes#install">Install</a></b>
•
<b><a href="https://github.com/diegoferigo/ros-kubernetes#build-the-images">Build</a></b>
•
<b><a href="https://github.com/diegoferigo/ros-kubernetes#run-the-setup">Run</a></b>
•
<b><a href="https://github.com/diegoferigo/ros-kubernetes#resources">Resources</a></b>
</p>

## What

This repository stores a simple kubernetes setup to run the [ROS robotic middleware](http://ros.org/) on a distributed cluster. To our knowledge, most of the available tutorials and examples are either outdated or not well documented. Some of them are reported in the [Resources](#resources) section.

This setup is composed of three deployments, each of them running a single-container pod:

- **`roscore-deployment`** Runs the roscore process
- **`talker-deployment`** Publishes a string message to the `/chatter` topic
- **`listener-deployment`** Subscribes to the `/chatter` topic and echoes the string

## Install

The quicker way to run this kubernetes setup is creating a local cluster with tools such as [minikube](https://kubernetes.io/docs/setup/minikube/) or [kind](https://github.com/kubernetes-sigs/kind/). Since this is a very trivial setup, we'll be using `kind` which provides an easy and lightweight cluster. We assume that you already have `docker`, `docker-compose`, and `kubectl` installed, configured, and running in your machine.

### Install `kind`

Refer to the [kind repository](https://github.com/kubernetes-sigs/kind/) for the official installation instructions. We'll recap here below the steps we followed:

```bash
git clone --depth 1 https://github.com/kubernetes-sigs/kind
cd kind
make build
```

Then, either add the `<path-to-cloned-kind-repo>/bin/` folder to your `PATH` or add `alias kind='<path-to-cloned-kind-repo>/bin/kind'` to your `~/.bashrc`.

## Build the images

All the ROS pods will be running the [official `ros/melodic-ros-core` image](https://hub.docker.com/_/ros/). To simplify the kubernetes and kind setup, we extended the official image with minor modifications. You can find the `Dockerfile`s in the [docker](docker/) folder.

Execute the following commands to build the ROS images:

```bash
cd docker
docker-compose -f build-ros-cluster.yml build
```

After this, make sure with `docker images` that the following images have been successfully built:

- `roscluster/master:v0`
- `roscluster/node:v0`

## Run the setup

The first step to run the setup is starting a local cluster. Execute:

```bash
kind create cluster
export KUBECONFIG="$(kind get kubeconfig-path)"
```

Make sure executing `docker ps -a` that the `kindest/node:v1.14.2` container is up and running. Also check that the cluster is running:

```
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE                                                          
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   1d17h
```

Then, load the docker images we built into the local cluster: 

```bash
kind load docker-image roscluster/master:v0
kind load docker-image roscluster/node:v0
```

The kubernetes setup is stored in the [k8s](k8s/) folder. Execute the following to run the setup:

```
cd k8s
kubectl create -f .
```

The three deployments should now start. The `talker` and `listener` deployments will likely fail at the first attempt because they need to reach first `roscore`. Their deployment has an `initContainer` that tries to connect to `roscore` and restart the pod if it fails. Furthermore, if during their execution the `roscore` container fails or is restarted, a `livenessProbe` will restart the `talker` and `listener` containers.

Once that the cluster is running, you should have the following `kubectl get all` output:

```
NAME                                      READY   STATUS    RESTARTS   AGE
pod/listener-deployment-c7bfb7856-7lhz2   1/1     Running   1          37s
pod/roscore-deployment-7dd8db86d9-cxb7x   1/1     Running   0          37s
pod/talker-deployment-97cbb7744-b6gdr     1/1     Running   1          37s

NAME                              TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)           AGE
service/kubernetes                ClusterIP      10.96.0.1      <none>        443/TCP           3d17h
service/service-listener          ClusterIP      None           <none>        11311/TCP         37s
service/service-master            ClusterIP      None           <none>        11311/TCP         37s
service/service-talker            ClusterIP      None           <none>        11311/TCP         37s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/listener-deployment   1/1     1            1           37s
deployment.apps/roscore-deployment    1/1     1            1           37s
deployment.apps/talker-deployment     1/1     1            1           37s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/listener-deployment-c7bfb7856   1         1         1       37s
replicaset.apps/roscore-deployment-7dd8db86d9   1         1         1       37s
replicaset.apps/talker-deployment-97cbb7744     1         1         1       37s
```

You can get the `listener` log containing the topic message executing `kubectl logs -l node=listener` (note that the log is cut using label selectors, use the listener pod name for a complete log).

## Resources

- https://blog.zhaw.ch/icclab/challenges-with-running-ros-on-kubernetes/
- https://googlecloudrobotics.github.io/core
- https://rdbox-intec.github.io/homepage_en/