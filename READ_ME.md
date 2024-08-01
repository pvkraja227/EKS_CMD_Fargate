From EC2 OR from cmd line (make sure CLI, Helm, Eksctl, Kubectl are in the same path)

From EC2 Instance:

### (Install AWS CLI)

sudo apt install unzip

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

aws configure (provide acccess keys)

aws iam list-users (check)

### (Install EKSCTL)

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin

eksctl version

### (Install Kubectl)

curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.7/2020-07-08/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin

kubectl version --short --client

### (Install Helm)

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh

./get_helm.sh

helm (check)

### (Creating Cluster - takes min 12-15 mins)

eksctl create cluster --name test-cluster --region us-west-1 --fargate

aws eks update-kubeconfig --name test-cluster --region us-west-1

### (Create Fargate profile)

eksctl create fargateprofile --cluster test-cluster --region us-west-1 --name alb-sample-app --namespace game-2048

### (Deploy Deployment, Service and Ingress)

git clone https://github.com/pvkraja227/EKS_CMD_Fargate.git

kubectl apply -f test.yaml

kubectl get pods -n game-2048 (5)

kubectl get svc -n game-2048 (default, NodePort)

kubectl get ingress -n game-2048 (no ExternalIP)

### (Check if there is an IAM OIDC provider configured already)

eksctl utils associate-iam-oidc-provider --cluster test-cluster --region us-west-1 --approve

### How to setup alb add on

Download IAM policy

curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

Create IAM Policy

aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

Create IAM Role

eksctl create iamserviceaccount --cluster=test-cluster --region us-west-1 --namespace=kube-system --name=aws-load-balancer-controller --role-name amazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --approve

### Deploy ALB controller

(Add helm repo)

helm repo add eks https://aws.github.io/eks-charts

(Update the repo)

helm repo update eks

(Install alb load balancer controller)

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=test-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=<your-vpc-id>

kubectl get deployment -n kube-system aws-load-balancer-controller (2)

kubectl get deployment -n kube-system (4)

kubectl get ingress -n game-2048 (ExternalIP)

goto EC2/Load Balancer/copy DNS and paste in chrome


