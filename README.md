# consul
This is a demonstration of using Consul ( service mesh )
## 1. Prerequisites
* [git](https://git-scm.com/downloads)
* [awscli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
* [config-profile](https://docs.aws.amazon.com/cli/latest/reference/configure/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
* [helm](https://helm.sh/docs/intro/install/)
## 2. Start
### 1. Clone repo
```
git clone https://github.com/haquocdat543/consul.git
cd consul
```
### 2. Initialize clusters
Initialize two eks cluster
```
aws cloudformation deploy --stack-name k8s1 --template-file k8s1.yaml
aws cloudformation deploy --stack-name k8s2 --template-file k8s2.yaml
```
### 3. Get output
```
aws cloudformation describe-stacks --query Stacks[].Outputs[*].[OutputKey,OutputValue] --output text
```
### 4. Add kubeconfig
```
aws eks update-kubeconfig --name eks1
```
### 5. Install consul on eks cluster
```
helm repo add hashicorp https://helm.releases.hashicorp.com
```
```
helm install k8s1 hashicorp/consul --version 1.0.0 --values consul-mesh-gateway.yaml --set global.datacenter=k8s1
```
