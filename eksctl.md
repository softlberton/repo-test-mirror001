# eksctl

### Usage

Get EKS clusters:
```shell
eksctl get cluster
```

Get labels from EKS nodegroup:
```shell
eksctl get labels --cluster <eks-cluster-name> --nodegroup <eks-ng-name>
```

Get accessentry from EKS cluster:
```shell
eksctl get accessentry --cluster <eks-cluster-name> -o yaml
```
