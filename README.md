# consul
This is a demonstration of using Consul ( service mesh )
## 1. Prerequisites
* [git](https://git-scm.com/downloads)
* [awscli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
* [config-profile](https://docs.aws.amazon.com/cli/latest/reference/configure/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
* [eksctl](https://eksctl.io/installation/)
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
```
eksctl utils associate-iam-oidc-provider --cluster=EKS1 --region ap-northeast-1 --approve
eksctl utils associate-iam-oidc-provider --cluster=EKS2 --region ap-northeast-1 --approve
```
```
eksctl create iamserviceaccount --name ebs-csi-controller-sa --namespace kube-system --cluster EKS1 --role-name AmazonEKS_EBS_CSI_DriverRole-K8S1 --role-only --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy --approve 
eksctl create iamserviceaccount --name ebs-csi-controller-sa --namespace kube-system --cluster EKS2 --role-name AmazonEKS_EBS_CSI_DriverRole-K8S2 --role-only --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy --approve 
```

Replace `YOUR_AWS_ACCOUNT` to your custom
```
aws eks create-addon --cluster-name EKS1 --addon-name aws-ebs-csi-driver --service-account-role-arn arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/AmazonEKS_EBS_CSI_DriverRole-K8S1
aws eks create-addon --cluster-name EKS2 --addon-name aws-ebs-csi-driver --service-account-role-arn arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/AmazonEKS_EBS_CSI_DriverRole-K8S2
```
### 3. Get output
```
aws cloudformation describe-stacks --query Stacks[].Outputs[*].[OutputKey,OutputValue] --output text
```
### 4. Add kubeconfig
```
aws eks update-kubeconfig --name EKS1
```
### 5. Install consul on EKS1 cluster
```
helm repo add hashicorp https://helm.releases.hashicorp.com
```
```
helm install eks1 hashicorp/consul --version 1.0.0 --values consul-values.yaml --set global.datacenter=eks1
```
Apply 11 tiers resources
```
kubectl apply -f config-consul.yaml
```
Access service
```
kubectl get svc
```
* Copy `consul-ui` `loadbalancerDNS` and access it on browser. `https://<loadbalancerDNS>`
* Copy `frontend-external` `loadbalancerDNS` and access it on browser. 
### 7. Install consul on EKS2 cluster
```
aws eks update-kubeconfig --name EKS2
```
```
helm install eks2 hashicorp/consul --version 1.0.0 --values consul-values.yaml --set global.datacenter=eks2
```
Apply 11 tiers resources
```
kubectl apply -f config-consul.yaml
```
Access service
```
kubectl get svc
```
* Copy `consul-ui` `loadbalancerDNS` and access it on browser. `https://<loadbalancerDNS>`
* Copy `frontend-external` `loadbalancerDNS` and access it on browser. 
### 8. Install mesh gateway
```
kubectl apply -f consul-mesh-gateway.yaml
```
* Open `eks1` consul ui > left corner > `Peers` ( at bottom ) > `Add peer connection` > `Generate token` > Enter `peer name` > `Generate token` > Copy it
* Open `eks2` consul ui > left corner > `Peers` ( at bottom ) > `Add peer connection` > `Establish peering` > Enter `Name of peer` > Paste `token` > `Add peer`

## 3. Destroy
### 1. Patch loadbalancer
* Patch all loadbalancer to Cluster IP
#### 1. EKS1
```
kubectl patch svc frontend-external -n default -p '{"spec": {"type": "ClusterIP"}}'
kubectl patch svc eks1-consul-ui -n default -p '{"spec": {"type": "ClusterIP"}}'
kubectl patch svc eks1-consul-mesh-gateway -n default -p '{"spec": {"type": "ClusterIP"}}'
```
#### 2. EKS2
```
kubectl patch svc frontend-external -n default -p '{"spec": {"type": "ClusterIP"}}'
kubectl patch svc eks2-consul-ui -n default -p '{"spec": {"type": "ClusterIP"}}'
kubectl patch svc eks2-consul-mesh-gateway -n default -p '{"spec": {"type": "ClusterIP"}}'
```
* Or you can go to AWS console and delete all loadbalancer used by two EKS cluster
### 2. Delete role stack
```
aws cloudformation delete-stack --stack-name eksctl-EKS1-addon-iamserviceaccount-kube-system-ebs-csi-controller-sa
aws cloudformation delete-stack --stack-name eksctl-EKS2-addon-iamserviceaccount-kube-system-ebs-csi-controller-sa
```
### 3. Delete eks stack
```
aws cloudformation delete-stack --stack-name k8s1
aws cloudformation delete-stack --stack-name k8s2
```
