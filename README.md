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
kubectl apply -f https://raw.githubusercontent.com/nigel-falco/ai-workload-security/main/tensorflow.yaml
```

#### Version Pinning:
If you require a specific vulnerable version of TensorFlow, specify it in your pip install command, like ```pip install tensorflow==2.4.0```.

```
kubectl get pods
```
If the Jupyter Notebook is running correctly, you should see logs indicating that the server has started and is listening on ```port 8888```:
```
kubectl logs -f $(kubectl get pods -n default -o jsonpath='{.items[0].metadata.name}') | grep tensorflow
```

## Verifying the Service
```
kubectl apply -f https://raw.githubusercontent.com/nigel-falco/ai-workload-security/main/tensorflow-service.yaml
```
```
kubectl get service tensorflow-service
```
Once the Service is up, you can access your TensorFlow application at:
```
http://tensorflow-service.default.svc.cluster.local:8888
```

You can also use port-forwarding to access the Jupyter Notebook locally:

```
kubectl port-forward $(kubectl get pods -n default -o jsonpath='{.items[0].metadata.name}') 8888:8888
```

Then access it via ```http://localhost:8888``` in your browser.

## DDoS Attack / Data Exfilitration / Data Destruction from AI Workload

Shell into the pod if you want to generate some suspicious activity:
```
kubectl exec -it $(kubectl get pods -n default -o jsonpath='{.items[0].metadata.name}') -- /bin/bash
```
```
curl 49.12.80.40:80
```

Alternatively, run it as a single line command:
```
kubectl exec -it $(kubectl get pods -n default -o jsonpath='{.items[0].metadata.name}') -- curl http://39.107.213.229:6666
```

```
kubectl exec -it $(kubectl get pods -n default -o jsonpath='{.items[0].metadata.name}') -- curl http://104.237.10.164:45700
```

Lists the [C2 IPs and Port](https://eu1.app.sysdig.com/secure/#/policies/rules/falco/Detect%20outbound%20connections%20to%20common%20miner%20pool%20ports?filter=port&details=list-edit&name=miners_ip) addresses associated with the mining pool networks.

### Proper DDoS Workflow
```
kubectl apply -f https://raw.githubusercontent.com/nigel-falco/ai-workload-security/main/siege-job.yaml
```

View the logs of the Siege test:
```
kubectl logs -f $(kubectl get pods --selector=job-name=siege-job --output=jsonpath='{.items[0].metadata.name}')
```

After reviewing the test results, clean up the job to free resources:

```
kubectl delete job siege-job
```

## Scaling the cluster

```
eksctl get nodegroups --cluster sysdig-cluster
```

```
eksctl scale nodegroup --cluster sysdig-cluster --name ng-d93efc25 --nodes 0
```
