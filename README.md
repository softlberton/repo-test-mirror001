# poc-eks-github-arc

## GitHub Actions Runner Controller

<!-- TOC -->

- [IAM role](#eks---iam-role)
- [EKS cluster](#eks---deploy-with-eksctl)
- [Deploy Actions Runner Controller](#github-runner---actions-runner-controller)

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

Set up kubeconfig:
```shell
aws eks update-kubeconfig --name "<eks-cluster-name>"
```

### GitHub Runner - Actions Runner Controller

Deploy actions Runner controller `operator`:
```shell
helm install arc -n arc-systems \
 --create-namespace \
 oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

Check if the ARC operator is running:
```shell
kubectl get po -A | grep "arc"
```

To enable ARC to authenticate to GitHub, generate a personal access token or create GitHub App.

- [Authenticating ARC with a GitHub App](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller/authenticating-to-the-github-api#authenticating-arc-with-a-github-app)
- [Authenticating ARC with a personal access token (classic)](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller/authenticating-to-the-github-api#authenticating-arc-with-a-personal-access-token-classic)

// to do

Create new namespace for arc runner `scale set`:
```shell
kubectl create ns arc-runners
```

Create kubernetes secret for arc runner `scale set` using GitHub App:
```shell
kubectl create secret generic pre-defined-secret \
   --namespace="arc-runners" \
   --from-literal=github_app_id="<github-app-id>" \
   --from-literal=github_app_installation_id="<github-app-installation-id>" \
   --from-literal=github_app_private_key='<github-app-private-key>'
```

Deploy actions runner controller `scale set`:
```shell
GITHUB_CONFIG_URL="https://github.com/<github-org-name>"

helm install arc-runner-set -n arc-runners \
 --create-namespace \
 --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
 --set githubConfigSecret="pre-defined-secret" \
 oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

Check helm release installations:
```shell
helm ls -A -o yaml
```

Check controller and operator manager pods in the `arc-systems` namespace:
```shell
kubectl get po -n arc-systems
```






## Source

- [Quickstart for Actions Runner Controller](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller/quickstart-for-actions-runner-controller)
- [EKSCTL - Creating and managing clusters](https://eksctl.io/usage/creating-and-managing-clusters/)

