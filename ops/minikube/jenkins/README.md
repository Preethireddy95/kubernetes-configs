# Jenkins master and slave on kubernetes

## Build the custom docker image for Jenkins by installing required plugins from plugins.txt
`docker build -t k8s-jenkins:lts -f Dockerfile_Jenkins .`

## Build the custom docker image for Maven Jenkins slave
`docker build -t preethireddy95/maven:3.6.2-jdk8 -f Docker_maven .`

## Build the custom docker image for docker + kubectl + awscli
`docker build -t preethireddy95/dockerkubeaws:latest -f Dockerfile_kubeaws .`

## Create Jenkins master on kubernetes
```
kubectl apply -f jenkins.yaml -n ops
minikube service jenkins -n ops
```

## create this clusterrolebinding to connect to kubernetes master from jenkins
```
kubectl create clusterrolebinding default-admin --clusterrole cluster-admin --serviceaccount=ops:default
minikube addons configure registry-creds
configure only with AWS ECR and docker hub registry 'registry.hub.docker.io'
minikube addons enable registry-creds
```

## Create Global credentials in Jenkins
Github username and password as id: `git-creds`
AWS access key and secret key of jenkins user as id: `aws-credentials`
Docker hub username and password as id: `dockerhub-creds`

## Add Global Tool configuration
* Add JDK installation with Name: JDK-1.8

## Add Config files
* Add npmrc config file with id: `npmrc`
* Add config files for client.key, client.crt, ca.crt in kubeconfig file only as data.  
* Add custom config file for minikube kubeconfig with id: `kubeconfig`

# Generate the <CA-DATA>
`cat ~/.minikube/ca.crt | base64 | tr -d '\n'`
# Generate the <CLIENT-CRT-DATA>
`cat ~/.minikube/client.crt | base64 | tr -d '\n'`
# Generate the <CLIENT-KEY-DATA>
`cat ~/.minikube/client.key | base64 | tr -d '\n'`
# kubeconfig file
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <CA-DATA>
    server: https://192.168.99.101:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate-data: <CLIENT-CRT-DATA>
    client-key-data: <CLIENT-KEY-DATA>
```
# validate kubeconfig from local
`kubectl get pods -n ops --kubeconfig=./config`

## Add jenkins slaves dynamic creation in kubernetes
### Add kubernetes cloud in jenkins configuration
* Get the kubernetes master URL and Jenkins pod internal IP
```
kubectl cluster-info
kubectl get pods -n ops
kubectl describe <jenkins-pod-name> -n ops
```
* Sample format
```
Kubernetes URL: https://192.168.99.105:8443
Kubernetes Namespace: ops
Jenkins URL: http://172.17.0.6:8080
```
