# consul
This is a demonstration of using Consul ( service mesh )
## 1. Prerequisites
* [git](https://git-scm.com/downloads)
* [awscli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
* [config-profile](https://docs.aws.amazon.com/cli/latest/reference/configure/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
## 2. Start
### 1. Clone repo
```
git clone https://github.com/haquocdat543/consul.git
cd consul
```
### 2. Initialize clusters
Initialize two eks cluster
``
aws cloudformation deploy --stack-name k8s1 --template-file k8s1.yaml
aws cloudformation deploy --stack-name k8s2 --template-file k8s2.yaml
```

