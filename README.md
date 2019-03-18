#Pre-requisites that must be installed:

Kubectl, kubeadm, kubelet, Aws cli, Aws iam autenticantion, Golang, Terraform


# Configure inside the terraform.tfvars file it must have the access credentials.


# terraform-aws-eks
Deploy a full AWS EKS cluster with Terraform

# Follow the resources that will be created
1. VPC
2. Internet Gateway (IGW)
3. Public and Private Subnets
4. Security Groups, Route Tables and Route Table Associations
5. IAM roles, instance profiles and policies
6. An EKS Cluster
7. Autoscaling group and Launch Configuration
8. Worker Nodes in a private Subnet
9. The ConfigMap required to register Nodes with EKS
10. KUBECONFIG file to authenticate kubectl using the heptio authenticator aws binary

# Variables to be changed 
You can configure the following input variables:

| Variables            | Description                       | Names         |
|----------------------|-----------------------------------|---------------|
| `cluster-name`       | The name of your EKS Cluster      | `my-cluster`  |
| `aws-region`         | The AWS Region to deploy EKS      | `us-east-2`   |
| `k8s-version`        | The desired K8s version to launch | `1.11`        |
| `node-instance-type` | Worker Node EC2 instance type     | `t3.meduim`   |
| `desired-capacity`   | Autoscaling Desired node capacity | `2`           |
| `max-size`           | Autoscaling Maximum node capacity | `5`           |
| `min-size`           | Autoscaling Minimum node capacity | `1`           |
| `vpc-subnet-cidr`    | Subnet CIDR                       | `10.0.0.0/16` |


# IAM
The AWS credentials must be associated with a user having at least the following AWS managed IAM policies

* IAMFullAccess
* AutoScalingFullAccess
* AmazonEKSClusterPolicy
* AmazonEKSWorkerNodePolicy
* AmazonVPCFullAccess
* AmazonEKSServicePolicy
* AmazonEKS_CNI_Policy
* AmazonEC2FullAccess

# This example: 
In addition, you will need to create the following managed policies

*EKS*

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "eks:*"
            ],
            "Resource": "*"
        }
    ]
}
```

#After Cluster Creation 
Create the archive:

curl -O https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-01-09/aws-auth-cm.yaml

#Edit the file aws-auth-cm.yaml with this RoleARN Outputs, showed in the end of the environment provision.
# This example: 
# Replace the <ARN of instance role (not instance profile)> with the "rolearn" value that you recorded in the previous procedure, and save the file.
    
    - rolearn: <ARN of instance role (not instance profile)>

#Replace to this: 

    - rolearn: arn:aws:iam::420056820328:role/my-cluster-eks-node-role

# This example refers to the last part of the environment provision: 

Outputs:

config-map = 
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::420056820328:role/my-cluster-eks-node-role
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes

#Execute this command:
kubectl apply -f aws-auth-cm.yaml

#Execute the command to update the kubeconfig of the cluster.

#Edit region and cluster_name in the command below.

aws eks --region region update-kubeconfig --name cluster_name

#The command result will generate a configuration file at the default kubeconfig path (.kube/config ) 
 

#Then execute the commands below:
kubectl get svc

kubectl get nodes    

# This example 
guilherme@work:~% kubectl get svc

NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   1h


guilherme@work:~% kubectl get nodes    

NAME                                       STATUS    ROLES     AGE       VERSION
ip-10-0-1-181.us-east-2.compute.internal   Ready     <none>    1h        v1.11.5

#Tutorial: Deploy the Kubernetes Web UI (Dashboard)
https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html