# install other dependencies
kubectl create namespace monitoring
helm upgrade --install scalar-logging-loki grafana/loki-stack -n monitoring
helm upgrade --install scalar-monitoring prometheus-community/kube-prometheus-stack -n monitoring

# confirm I have a EKS cluster
aws eks list-clusters

# check my Kubernetes nodes
kubectl get node

# create an OIDC provider
eksctl utils associate-iam-oidc-provider --region ap-northeast-1 --cluster plenty-20240626-2 --approve

# create an IAM service account
eksctl create iamserviceaccount \
    --name plenty \
    --namespace default \
    --region ap-northeast-1 \
    --cluster plenty-20240626-2 \
    --attach-policy-arn arn:aws:iam::aws:policy/AWSMarketplaceMeteringFullAccess \
    --approve \
    --override-existing-serviceaccounts

# add our Helm repository
helm repo add scalar-labs https://scalar-labs.github.io/helm-charts

# check pod before deployment
kubectl get pod

# deploy our product via Helm
helm upgrade --install scalar-manager scalar-labs/scalar-manager \
  --set api.image.repository=709825985650.dkr.ecr.us-east-1.amazonaws.com/scalar/scalar-manager-api-aws-payg \
  --set api.image.tag=2.0.0 \
  --set web.image.repository=709825985650.dkr.ecr.us-east-1.amazonaws.com/scalar/scalar-manager-web-aws-payg \
  --set web.image.tag=2.0.0 \
  --set serviceAccount.serviceAccountName=plenty \
  --set serviceAccount.automountServiceAccountToken=true

# check pod after deployment
kubectl get pod
