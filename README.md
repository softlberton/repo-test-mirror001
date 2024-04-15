# poc-eks-github-arc

### GitHub Actions Runner Controller

<!-- TOC -->

- [IAM role](https://github.com/Iberia-Ent/commercial--management--ancillaries--devsecops/tree/main/automation)
- [EKS cluster](https://github.com/Iberia-Ent/commercial--management--ancillaries--devsecops/tree/main/database)

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

Enable EKS logging:
```shell
eksctl utils update-cluster-logging --enable-types all --cluster "<eks-cluster-name>" --approve
```

