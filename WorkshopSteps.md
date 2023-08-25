# Tracy Developers Workshop Steps
This document provides the steps we'll be walking through during the Tracy Developers workshop that was hosted on August 25th, 2023. It provides some basic explanation of Kubernetes concepts but may not be fully complete.

## Introducing kubectl
From a system perspective, Kubernetes is an elaborate system of APIs that coordinate to control servers. The more common object types (Pods, Namespaces, Persistent Volumes), belong to the same core API `v1`. Newer object types may have their own APIs, such as Networking Ingress and it's API `networking.k8s.io/v1`. Finally, features under development may have their own APIs.

### ACTIVTY 1 - Playing with kubectl
* Type `kubectl get ` and press tab twice. This will should all the resource types on the system. See any that interest you?
* Type `kubectl get pods -A` to show all the pods on the server. The existing pods are those needed for kubernetes to run, such as the API server and etcd.
* Type `kubectl get namespaces`. This will show you all the namespaces on the server.
* If we want to isolate just the pods from a single namespace, then use the `-n` flag by typing `kubectl get pods -n kube-system`
* Finally, type `kubectl get pods -A --v=7`. This output is a lot more verbose than the pretty output we got the first time. It shows us the raw curl commands, including the API endpoints, the message body and more.

Up unto this point, we've only been doing `GET` commands, but we can do other common actions on the object types.
* Type `kubectl ` and hit tab twice. This will show us all the actions we can take on objects.

### ACTIVITY 2 - Putting in our own pods
The existing pods are boring. They make the cluster run, but nothing else. Let's see how we can run our own pods on the server. There are two ways to do this: declaratively or imperitively. In the declarative method, we create a file with the complete definition of our object and the run a command like `kubectl apply -f object.yml`. In the imperitive method, we do everything at runtime with the `kubectl` utility. It can be helpful to create objects quickly, but can get verbose for more complex objects with many required fields.

* Type `less alpine_pod.yml` to open the file for reading.
* Once you understand the general layout of the file, run it in the cluster by typing `kubectl apply -f resources/alpine_pod.yml`
* Type `kubectl get pods` to see our new pod.
* Type `kubectl describe pod alpine-pod` to see a full description of the pod
* Finally, we can jump into the pod as if it were another server. Type `kubectl exec -it alpine-pod -- /bin/sh`
* Once you're done exploring, type `exit` to get back to our control server.

### ACTIVITY 3 - Writing Code and building the container
#### Write node code using express that returns some text
* From the guide [here][1]
  * Push the following code into `package.json`
  ```json
  {
    "name": "tracy_devs_web_app",
    "version": "1.0.0",
    "description": "Node.js on Docker",
    "author": "First Last <first.last@example.com>",
    "main": "server.js",
    "scripts": {
      "start": "node server.js"
    },
    "dependencies": {
      "express": "^4.18.2"
    }
  }
  ```
  * Push this to `server.js`
  ```js
  'use strict';

  const express = require('express');

  // Constants
  const PORT = 8080;
  const HOST = '0.0.0.0';

  // App
  const app = express();
  app.get('/', (req, res) => {
    res.send('Hello World');
  });

  app.listen(PORT, HOST, () => {
    console.log(`Running on http://${HOST}:${PORT}`);
  });
  ```
  * Push this into `Dockerfile`
  ```Dockerfile
  FROM node:18

  # Create app directory
  WORKDIR /usr/src/app

  # Install app dependencies
  # A wildcard is used to ensure both package.json AND package-lock.json are copied
  # where available (npm@5+)
  COPY package*.json ./

  RUN npm install
  # If you are building your code for production
  # RUN npm ci --omit=dev

  # Bundle app source
  COPY . .

  EXPOSE 8080
  CMD [ "node", "server.js" ]  
  ```
  * Push this to `.dockerfile`
  ```
  node_modules
  npm-debug.log
  ```
#### Build the container
* Same guide
  * `docker build . -t aaronchamberlain/node-web-app`
  * `docker images`
  * `docker run -p 49160:8080 -d <your username>/node-web-app`
  * `docker ps`
  * `docker logs <container_id>`
  * We get a response like `Running on http://localhost:49160`

We would normally go through a process to authenticate with our repository and upload the image we have locally. However, to speed things up for the workshop, I've pushed the image to Docker Hub and we can use that in a pod definition.

* Type `less resources/nodejs_app.yml` to read what we have. It's very similar to the first pod definition that we had. But I've swapped the original image with the one we just built. I also had to expose some ports for networking so we can see it on the public IP address.
* Type `q` to exit reading the document.
* Type `kubectl apply -f resources/nodejs_app.yml` to install the container in our cluster.
* Now, let's find the IP of the service. This can be done with `minikube service list`
* Try verifying we can reach that endpoint with `curl http://192.168.49.2:32767`

But that's boring! I want to see it in my browser! If we were doing this on a production cluster, we'd probably configure Nginx Ingress or a LoadBalancer service. I have an example of the configuration to redirect to our NodeJS app, but we won't use it here. Instead, we'll just port forwarding. Pay particular attention to the `--address 0.0.0.0` flag, which redirects it to localhost, which will also resolve from the default public IP.

* Type `kubectl port-forward --address 0.0.0.0 service/nodejs-service 1234:8080`
* Now, we can finally see it in our browser. Go to `http://34.213.244.58:1234/`


### ACTIVITY 4 - Installing Software with Helm
Helm is a package manager for Kubernetes, just like npm or yarn for NodeJS, Cargo for Rust, or pip for Python. Their basic component is called a "Helm Chart". In reality, it's just yet another large YAML file that defines multiple Kubernetes objects like pods, networks, etc. We can look for helm charts and their relevant documentation here: https://artifacthub.io/

The steps we'll be doing are a part of this tutorial from SysDig (here)[2]. To install something, we can just run a command like `helm install <release name> <package name>`, which could look like `helm install our_prometheus prometheus-community/prometheus`. Let's do just that, starting first with setting up a repository.

* Type `helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`
* Type `helm repo add stable https://charts.helm.sh/stable`
* Type `helm repo update`
* Want to see what packages we have available now with those repos? Type `helm search repo prometheus-community` to see what's available to install.
* Finally, install it with `helm install prometheus prometheus-community/prometheus`
* Per the information that it output, we can also port forward this service to see what it created.
* Type `export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus-pushgateway,component=pushgateway" -o jsonpath="{.items[0].metadata.name}")`
* And then, `kubectl --namespace default port-forward --address 0.0.0.0 $POD_NAME 9090`
* Now, just like the last NodeJS app, we can visit it in the browser at `http://34.213.244.58:9090/`

As one final learning expirement, what if we don't like how Prometheus was configured out of the box? Like I said earlier, a helm chart is just a large YAML file with detailed configuration. If you don't provide any configuration yourself, you accept the defaults for everything.
* To see the values that will be used, type `helm show values prometheus-community/prometheus | less`
* If we wanted to change any of the values, save that output to a file, make the changes, and then pass it in to the install command.
  * For example, type `helm install prometheus prometheus-community/prometheus -f overrides.yml`


[1]: https://nodejs.org/en/docs/guides/nodejs-docker-webapp
[2]: https://sysdig.com/blog/kubernetes-monitoring-prometheus/