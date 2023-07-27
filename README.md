# add-EFS-to-eks

### Prerequisites

#### Follow the requirements here: https://repost.aws/knowledge-center/eks-persistent-storage

## 1- Create OIDC
### To add OIDC while creating eks : simply add this flag
```
--with-oidc
````
### If the Cluster is create , follow the steps below
### You can simply run this command: 
```
eksctl utils associate-iam-oidc-provider --cluster my-cluster --approve
```
#### or follows the steps below:
#### Create OIDC ROle first: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html
#### For the OIDC setup follow this: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html
###### The Provider URL comme from : run this command: 
```
aws eks describe-cluster --name your_cluster_name --query "cluster.identity.oidc.issuer" --output text
```
![image](https://github.com/thedevopsguru1/add-EFS-to-eks/assets/126810742/70ebb446-a54b-4ced-b4e2-13c225d9d040)
######## Put the ouput in provider URL
![image](https://github.com/thedevopsguru1/add-EFS-to-eks/assets/126810742/d8a38785-9065-4b68-827b-9f7382b27912)
######## For the audience use: sts.amazonaws.com
![image](https://github.com/thedevopsguru1/add-EFS-to-eks/assets/126810742/9caae496-ed69-4711-82f2-00f492a3b5ea)

#### Check to make sure that the your IAM OIDC provider is configured:
```
aws iam list-open-id-connect-providers
```
you should see this:
![image](https://github.com/thedevopsguru1/add-EFS-to-eks/assets/126810742/6339b1bf-a568-41e9-9609-b8983c51075a)


## 2- Keep follow the steps here:
#### Follow the requirements here: https://repost.aws/knowledge-center/eks-persistent-storage
### A-  Run the following command to verify that your AWS IAM OpenID Connect (OIDC) provider exists for your cluster:
```
aws eks describe-cluster --name your_cluster_name --query "cluster.identity.oidc.issuer" --output text
```
### B- Run the following command to verify that your IAM OIDC provider is configured:
```
aws iam list-open-id-connect-providers
```
### C- Download the IAM policy document from GitHub:
```
curl -o iam-policy-example.json https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/master/docs/iam-policy-example.json
```
### D - Create an IAM policy:
```
aws iam create-policy \
    --policy-name AmazonEKS_EFS_CSI_Driver_Policy \
    --policy-document file://iam-policy-example.json
```
### E - Run the following command to determine your cluster's OIDC provider URL:
```
aws eks describe-cluster --name your_cluster_name --query "cluster.identity.oidc.issuer" --output text
```
##### In step 3, replace your_cluster_name with your cluster name.
### F- Create the following IAM trust policy, and then grant the AssumeRoleWithWebIdentity action to your Kubernetes service account. For example:
```
cat <<EOF > trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_AWS_ACCOUNT_ID:oidc-provider/oidc.eks.YOUR_AWS_REGION.amazonaws.com/id/<XXXXXXXXXX45D83924220DC4815XXXXX>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.YOUR_AWS_REGION.amazonaws.com/id/<XXXXXXXXXX45D83924220DC4815XXXXX>:sub": "system:serviceaccount:kube-system:efs-csi-controller-sa"
        }
      }
    }
  ]
}
EOF
```
###### Note: In step 4, replace YOUR_AWS_ACCOUNT_ID with your account ID. Replace YOUR_AWS_REGION with your Region. Replace XXXXXXXXXX45D83924220DC4815XXXXX with the value returned in step 3.

### H- Create an IAM role:
```
aws iam create-role \
  --role-name AmazonEKS_EFS_CSI_DriverRole \
  --assume-role-policy-document file://"trust-policy.json"
```
### I- Attach your new IAM policy to the role:
```
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AmazonEKS_EFS_CSI_Driver_Policy \
  --role-name AmazonEKS_EFS_CSI_DriverRole
```
### J- Save the following contents to a file named efs-service-account.yaml.
```
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app.kubernetes.io/name: aws-efs-csi-driver
  name: efs-csi-controller-sa
  namespace: kube-system
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<AWS_ACCOUNT_ID>:role/AmazonEKS_EFS_CSI_DriverRole
```
### K -   Create the Kubernetes service account on your cluster. The Kubernetes service account named efs-csi-controller-sa is annotated with the IAM role that you created

```
kubectl apply -f efs-service-account.yaml
```
## 3- Install Driver 
```
helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/
```
```
helm repo update aws-efs-csi-driver
```
```
helm upgrade --install aws-efs-csi-driver --namespace kube-system aws-efs-csi-driver/aws-efs-csi-driver --set controller.serviceAccount.create=false --set controller.serviceAccount.name=efs-csi-controller-sa
```
If you did not create SA,then run this
```
helm upgrade --install aws-efs-csi-driver --namespace kube-system aws-efs-csi-driver/aws-efs-csi-driver 
```
https://github.com/kubernetes-sigs/aws-efs-csi-driver aws-efs-csi-driver

## 4- Create encrypted EFS 

### A- Get the VPC ID for your Amazon EKS cluster:
```
aws eks describe-cluster --name your_cluster_name --query "cluster.resourcesVpcConfig.vpcId" --output text
```
###### Note: In step 3, replace your_cluster_name with your cluster name.
### B-  Get the CIDR range for your VPC cluster:
```
aws ec2 describe-vpcs --vpc-ids YOUR_VPC_ID --query "Vpcs[].CidrBlock" --output text
```
###### Note: In step 4, replace the YOUR_VPC_ID with the VPC ID from the preceding step 3.
### C-  Create a security group that allows inbound network file system (NFS) traffic for your Amazon EFS mount points:
```
aws ec2 create-security-group --description efs-test-sg --group-name efs-sg --vpc-id YOUR_VPC_ID
```
###### Note: Replace YOUR_VPC_ID with the output from the preceding step 3. Save the GroupId for later.
### D- Add an NFS inbound rule so that resources in your VPC can communicate with your Amazon EFS file system:
```
aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 2049 --cidr YOUR_VPC_CIDR
```
######  Note: Replace YOUR_VPC_CIDR with the output from the preceding step 4. Replace sg-xxx with the security group ID from the preceding step 5.
### E- Create EFS
```
aws efs create-file-system --creation-token eks-efs --encrypted
```
###### Note: Save the FileSystemId for later use.
### F- To create a mount target for Amazon EFS, run the following command:
```
aws efs create-mount-target --file-system-id FileSystemId --subnet-id SubnetID --security-group sg-xxx
```
## 5- Create NFS-CLIENT
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
```
```
helm repo update nfs-subdir-external-provisioner
```
```
helm upgrade --install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=fs-0e6c8b8e20cd5356f.efs.us-east-2.amazonaws.com  --set nfs.path=/ --set storageClass.defaultClass=default --set storageClass.name=second-nfs-client
```


#### For YOUR_AWS_ACCOUNT_ID : run the command
```
aws iam list-open-id-connect-providers
```
![image](https://github.com/thedevopsguru1/add-EFS-to-eks/assets/126810742/241a6271-eeff-48f5-bbdd-da1fcc83a74a)

####  Replace XXXXXXXXXX45D83924220DC4815XXXXX with the value
![image](https://github.com/thedevopsguru1/add-EFS-to-eks/assets/126810742/044c8017-611d-4e35-87ae-e8760ab0bfdc)
##### For  --file-system-id
```
aws efs describe-file-systems --query "FileSystems[*].FileSystemId" --output text
```
```

## 4- Add NFS LINK
#### Dynamic nfs
##### On AWS EFS , here it is
####### Create a EFS and then copy the DNS name of the EFS
![image](https://github.com/thedevopsguru1/dynamic-nfs-provisioing/assets/126810742/c2136ea6-eb7e-4f4c-bf39-3eed8f5ff764)

```
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=fs-0e4070b4c27a59f48.efs.us-east-2.amazonaws.com  --set nfs.path=/
```
#### the the storage class to the PVC :  nfs-client
https://github.com/thedevopsguru1/dynamic-nfs-provisioing





