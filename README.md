#### Get the apps

```sh
git clone https://github.com/bcrabbe/grpcMath
git clone https://github.com/bcrabbe/iris
export PATH_TO_IRIS="./iris"
export PATH_TO_GRPCMATH="./grpcMath"
```

#### Install Kubernetes

```sh
brew install kubectl

brew cask install minikube
```

#### Start a local Kubernetes cluster

```sh
minikube start
```

#### Add our images

minikube starts it own docker process. To add the images for our apps to this docker daemon

```sh
eval $(minikube docker-env)

docker build -t grpc-math:latest $PATH_TO_GRPCMATH

docker build -t iris:latest $PATH_TO_IRIS
```

#### Run the apps

```sh
kubectl run --image iris --image-pull-policy Never --port 80 iris

kubectl run --image grpc-math --image-pull-policy Never --port 80  grpc-math
```

to see whats running:
```sh
kubectl get pods
```

<!-- Test the apps -->
<!-- ___ -->
<!-- in another shell run -->
<!-- ``` -->
<!-- kubectl proxy -->
<!-- ``` -->
<!-- pods run in an isolated, private network, this forwards external requests to our apps. -->

<!-- pull out the names from `kubectl get pods`: -->
<!-- ``` -->
<!-- eval $(kubectl get pods -o go-template --template '{{range $index, $element := .items}}export POD_{{$index}}={{.metadata.name}}{{"\n"}}{{end}}') -->
<!-- echo POD_0: $POD_0 -->
<!-- echo POD_1: $POD_1 -->
<!-- ``` -->

<!-- test IRIS: -->

<!-- ``` -->
<!-- curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_1/proxy/test -->
<!-- ``` -->
<!-- response: -->
<!-- ``` -->
<!-- {"predictions":[1,0,2,1,1,0,1,2,1,1,2,0,0,0,0,2,2,1,1,2,0,2,0,2,2,2,2,2,0,0,0,0,1,0,0,2,1,0,0,0,2,1,1,0,0],"score":0.9777777777777777,"truth":[1,0,2,1,1,0,1,2,1,1,2,0,0,0,0,1,2,1,1,2,0,2,0,2,2,2,2,2,0,0,0,0,1,0,0,2,1,0,0,0,2,1,1,0,0]} -->
<!-- ``` -->

#### Expose apps as a service

- grpc-math

```sh
kubectl expose deployment/grpc-math --type="NodePort" --port 80
```
 test:

```sh
export NODE_PORT=$(kubectl get services/grpc-math -o go-template='{{(index .spec.ports 0).nodePort}}')

node --experimental-modules $PATH_TO_GRPCMATH/math_client.mjs $(minikube ip):$NODE_PORT
```

- iris

 ```sh
kubectl expose deployment/iris --type="NodePort" --port 80
```

 test:

 ```sh
export NODE_PORT=$(kubectl get services/iris -o go-template='{{(index .spec.ports 0).nodePort}}')

curl -v $(minikube ip):$NODE_PORT/test
```

