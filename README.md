# spark-dockerfile-multi-stage
Making use of Docker best practices, this repo uses a multi stage Dockerfile for Spark jobs. It is meant to be used as a starting point for projects deploying Apache Spark jobs in Kubernetes clusters. It can be used with `spark-submit` command or with `spark-operator`. Since many of the deployments are running in cloud environments, the `jars` to allow `S3` and `GCS` access are included.

## Build
Build the docker image by running:
```
docker build -t my-spark-image:latest .
```

To override Spark or Hadoop verson, in the build stage
```
docker build --build-arg HADOOP_VERSION_DEFAULT=2.7 -t my-spark-image:latest .
```

## Usage

### Scala/Java
The application `jar` file should be added in `/app` directory.
The `spark-submit` command looks like this:
```
spark-submit \
    --master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=1 \
    --conf spark.kubernetes.container.image=<spark-image> \
    local:///app/<jar-name>.jar
```

### Python

The application files should be added in `/app` directory.
The `spark-submit` command looks like this:
```
spark-submit \
    --master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
    --deploy-mode cluster \
    --name spark-pi \
    --conf spark.executor.instances=1 \
    --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark-sa \
    --conf spark.kubernetes.container.image=<spark-image> \
    local:///app/src/main/pi.py
```

## Testing

### Install minikube
MacOS:
```
brew install minikube
```

### Install kubectl
MacOS:
```
brew install kubectl
```

### Preconfiguration
Start minikube cluster:
```
minikube start --insecure-registry "10.0.0.0/24" --memory 8192 --cpus 4
minikube addons enable registry
```

Enable pushing images to `minikube` docker registry
```
docker run --rm -it --network=host alpine ash -c "apk add socat && socat TCP-LISTEN:5000,reuseaddr,fork TCP:$(minikube ip):5000"
```

More info [here](https://minikube.sigs.k8s.io/docs/handbook/registry/)

### Fix serivce account permissions
For Spark to work, the service account running the appliaction needs to have permissions to start pods in the cluster. Permissions are added in the `rbac.yaml` file. Execute:
```
kubectl apply -f rbac.yaml
```

### Building images
Build Docker image:
```
docker build -t localhost:5000/spark-local -f Dockerfile .
```

Push image to registry:
```
docker push localhost:5000/spark-local
```

### Run application
Execute
```
spark-submit \
    --master k8s://https://$(minikube ip):8443 \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=2 \
    --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark-sa \
    --conf spark.kubernetes.container.image=localhost:5000/spark-local \
    local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar
```