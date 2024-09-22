# Prerequsite
- aws cli
- kubectl
- eksctl (recommended way to create cluster)


# Create Cluster (use bash)
> cluster_name=my-cluster

> eksctl create cluster --name $cluster_name --region us-east-1 --nodegroup-name my-nodes --nodes 1

> aws eks describe-cluster --name $cluster_name

> oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)

> echo $oidc_id

> aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4

> eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve

> aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4

> aws eks update-kubeconfig --name $cluster_name

# Deploying AWS Load Balancer Controller:

> curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

> aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json

### Replace ${AWS_ACCOUNT_ID} with AWS Accout Number
> eksctl create iamserviceaccount \
  --cluster=$cluster_name \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::292484479472:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

> helm repo add eks https://aws.github.io/eks-charts
 helm repo update eks
 helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=$cluster_name \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller

> kubectl get deployment -n kube-system aws-load-balancer-controller

# Useful Commands:

> aws eks list-clusters
> eksctl delete cluster --name $cluster_name


# References

- https://medium.com/@inderjotsingh141/setup-ingress-controller-in-eks-65aef27ae396
- https://medium.com/@shivam77kushwah/integrate-application-load-balancer-with-aws-eks-using-aws-load-balancer-controller-7e9b7d178a79
