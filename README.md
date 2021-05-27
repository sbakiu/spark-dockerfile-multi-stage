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
    --conf spark.kubernetes.container.image=<spark-image> \
    local:///app/src/main/pi.py
```
