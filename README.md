# ai-workload-security
Test scenario for a bunch of AI workload packages

## Prerequisite Stuff

Set up an ```AWS-CLI Profile``` in order to interact with AWS services via my local workstation
```
aws configure --profile nigel-aws-profile
export AWS_PROFILE=nigel-aws-profile                                            
aws sts get-caller-identity --profile nigel-aws-profile
aws eks update-kubeconfig --region eu-west-1 --name sysdig-cluster
```

## Create EKS Cluster with Cilium CNI

```
eksctl create cluster --name sysdig-cluster --node-type t3.xlarge --nodes 1 --nodes-min=0 --nodes-max=3 --max-pods-per-node 58
```

## Installing Tensorflow AI Workload

```
wget https://raw.githubusercontent.com/nigel-falco/ai-workload-security/main/tensorflow.yaml
```

```
kubectl apply -f tensorflow.yaml
```

```
kubectl get pods
```

```
kubectl logs <pod-name>
```

## Verifying the Service
If the Jupyter Notebook is running correctly, you should see logs indicating that the server has started and is listening on port 8888. <br/>
You can also use port-forwarding to access the Jupyter Notebook locally:

```
kubectl port-forward <pod-name> 8888:8888
```

Then access it via ```http://localhost:8888``` in your browser.

## Scaling the cluster

```
eksctl get nodegroups --cluster sysdig-cluster
```

```
eksctl scale nodegroup --cluster falco-cluster --name ng-d93efc25 --nodes 0
```
