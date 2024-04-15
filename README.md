# poc-eks-github-arc

## GitHub Actions Runner Controller

<!-- TOC -->

- [IAM role](#EKS-iam-role)
- [EKS cluster](#EKS-deploy-with-eksctl)

### EKS - IAM role

Create IAM role for EKS:
```shell
aws iam create-role --role-name GithubRunnerEksRole --assume-role-policy-document file://"eks-iam-policy/eks-iam-role-trust-policy.json"
```

Attach IAM policy role to the new EKS role created in the previous step:
```shell
aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy --role-name GithubRunnerEksRole
```

Show the new IAM role created for EKS:
```shell
aws iam get-role --role-name GithubRunnerEksRole
```

### EKS - Deploy with eksctl

Create EKS cluster using `eksctl`:
```shell
eksctl create cluster -f ./eks-cluster-config/eks-cluster.yaml --timeout 15m
```

Enable EKS logging `cloudwatch`:
```shell
eksctl utils update-cluster-logging --enable-types all --cluster "<eks-cluster-name>" --approve
```

### GitHub Runner - Actions Runner Controller

Deploy Actions Runner Controller operator:
```shell
helm install arc \
 --namespace arc-systems \
 --create-namespace oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

Create new namespace for arc runners:
```shell
kubectl create ns arc-runners
```

Create new kubernetes secret for arc scale set:
```shell
kubectl create secret generic pre-defined-secret \
   --namespace="arc-runners" \
   --from-literal=github_app_id="<github-app-id>" \
   --from-literal=github_app_installation_id="<github-app-installation-id>" \
   --from-literal=github_app_private_key='<github-app-private-key>'
```

Deploy actions runner controller scale set:
```shell
helm install arc-runner-set \
 --namespace arc-runners \
 --create-namespace \
 --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
 --set githubConfigSecret="pre-defined-secret" \
 oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```
