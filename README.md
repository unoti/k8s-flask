# Flask in K8s
**Goal**: Get a simple Flask app running in k8s to test out my local k8s minikube installation.
Along the way we'll document some of the commands needed.

## Reference
 * [Simple Flask App](https://dev.to/ibmdeveloper/deploying-a-python-flask-application-to-kubernetes-1n76)
   * I selected this Flask tutorial because most other resources I could find included complex apps including Redis, Postgres, or MySQL.

# Part 1: Build and Test the Container
We'll start by building a simple HTTP server in Flask and get it running in a container.  We'll test the container in isolation without K8s first.  Then in Part 2 we'll work on running that container inside our k8s cluster.

## Build
**NOTE**: On Windows, these commands need to be run either in Ubuntu or a Git Bash shell.  If you run these commands under a cmd Window you will probably run into failures because of [this issue](https://stackoverflow.com/questions/64221861/failed-to-resolve-with-frontend-dockerfile-v0).  The error you get in that situation is like `failed to solve with frontend dockerfile.v0`.
```
$ docker build --tag flask-hello .
```
* `--tag` puts a tag on the container of `flask-hello`.

## Run
```
$ docker run --detach --name flasksimple --publish 80:80 flask-hello
```
* `--detach` runs the container in the background and prints the container ID.
* `--name` Assigns a name to the container.
* `--publish` Publishes the container's ports to the host.
* The last argument is the name of the image, which comes from the tag in the build process.

You can see it running in multiple ways:
* Open a browser to `http://localhost` and you'll see the output from the application in your browser.
* Open docker and go to Containers/Apps.  You should see `flask-hello` listed there as running.

## View stdout
If you want to later attach to this process and see its output:
```
$ docker attach flasksimple --sig-proxy=false
```
This will let you see the HTTP requests going through Flask.
To stop viewing the logs, use ^C, but remember the container will still be running in the background. [Reference: How do you attach and detach from a Docker process?](https://stackoverflow.com/questions/19688314/how-do-you-attach-and-detach-from-dockers-process)

## Stop Running
```bash
$ docker stop flasksimple
 # Disassociates the name so you can run it next time
$ docker rm flasksimple
```

# Part 2: Run Flask App in K8s
In this section we will get our little Flask app running in Kubernetes.

References:
* [K8s Minicube docs, local container images](https://kubernetes.io/docs/tutorials/hello-minikube/#create-a-docker-container-image)
* [Can't pull image from private docker repository](https://stackoverflow.com/questions/49639280/kubernetes-cannot-pull-image-from-private-docker-image-repository)

## Install Local Kubernetes Cluster
If you haven't already, set up your local k8s environment in Minikube, and test that your local K8s cluster works.  I recommend [this tutorial](https://minikube.sigs.k8s.io/docs/start/).

## Create Deployment
First let's launch the dashboard UI for K8s:
```
$ minikube dashboard
```

First we need to add the docker image into our k8s repository.  We need to do this because we're using a private local repository instead of a public repository for our image.  References: [Minicube cache add docs](https://minikube.sigs.k8s.io/docs/handbook/pushing/#2-push-images-using-cache-command), [How to use local docker images with Minikube](https://stackoverflow.com/questions/42564058/how-to-use-local-docker-images-with-minikube)

You can see the available images using `docker image ls`, and there you will see our `flask-hello` image with `latest` for the tag.  Let's make this private image available to K8s:
```
$ minikube cache add flask-hello:latest
```

We will let K8s launch our container with our application and be responsible for restarting it if it fails.
```
$ kubectl create deployment hello-minikube --image=flask-hello:latest
$ kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

## Remove Deployment
```
$ kubectl delete deployment hello-minikube
```