# add-EFS-to-eks

### Prerequisites

#### Follow the requirements here: https://repost.aws/knowledge-center/eks-persistent-storage
###Create encrypted EFS 
```
aws efs create-file-system --creation-token eks-efs --encrypted
```
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
###C- Download the IAM policy document from GitHub:
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
### F- Create the following IAM trust policy, and then grant the AssumeRoleWithWebIdentity action to your Kubernetes service account. For example:
```
1
I want to use persistent storage in Amazon Elastic Kubernetes Service (Amazon EKS).

Short description
Set up persistent storage in Amazon EKS using either of the following options:

Amazon Elastic Block Store (Amazon EBS) Container Storage Interface (CSI) driver
Amazon Elastic File System (Amazon EFS) Container Storage Interface (CSI) driver
To use one of these options, complete the steps in either of the following sections:

Option A: Deploy and test the Amazon EBS CSI driver
Option B: Deploy and test the Amazon EFS CSI driver
The commands in this article require kubectl version 1.14 or greater. To see your version of kubectl, run the following command:

kubectl version --client --short
Note: It's a best practice to make sure you install the latest version of the drivers. For more information, see in the GitHub repositories for the Amazon EBS CSI driver and Amazon EFS CSI driver.

Resolution
Note: If you receive errors when running AWS Command Line Interface (AWS CLI) commands, make sure that you're using the most recent version of the AWS CLI.

Before you complete the steps in either section, you must:

1.    Install the AWS CLI.

2.    Set AWS Identity and Access Management (IAM) permissions for creating and attaching a policy to the Amazon EKS worker node role CSI Driver Role.

3.    Create your Amazon EKS cluster and join your worker nodes to the cluster.
Note: Run the kubectl get nodes command to verify your worker nodes are attached to your cluster.

4.    Run the following command to verify that your AWS IAM OpenID Connect (OIDC) provider exists for your cluster:

aws eks describe-cluster --name your_cluster_name --query "cluster.identity.oidc.issuer" --output text
Note: Replace your_cluster_name with your cluster name.

5.    Run the following command to verify that your IAM OIDC provider is configured:

`aws iam list-open-id-connect-providers | grep <ID of the oidc provider>`;
Note: Replace ID of the oidc provider with your OIDC ID. If you receive a No OpenIDConnect provider found in your account error, you must create an IAM OIDC provider.

6.    Install or update eksctl.

7.    Run the following command to create an IAM OIDC provider:

eksctl utils associate-iam-oidc-provider --cluster my-cluster --approve 
Note: Replace my-cluster with your cluster name.

Option A: Deploy and test the Amazon EBS CSI driver
Deploy the Amazon EBS CSI driver:

1.    Create an IAM trust policy file, similar to the one below:

cat <<EOF > trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_AWS_ACCOUNT_ID:oidc-provider/oidc.eks.YOUR_AWS_REGION.amazonaws.com/id/<your OIDC ID>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.YOUR_AWS_REGION.amazonaws.com/id/<XXXXXXXXXX45D83924220DC4815XXXXX>:aud": "sts.amazonaws.com",
          "oidc.eks.YOUR_AWS_REGION.amazonaws.com/id/<XXXXXXXXXX45D83924220DC4815XXXXX>:sub": "system:serviceaccount:kube-system:ebs-csi-controller-sa"
        }
      }
    }
  ]
}
EOF
Note: Replace YOUR_AWS_ACCOUNT_ID with your account ID. Replace YOUR_AWS_REGION with your AWS Region. Replace your OIDC ID with the output from creating your IAM OIDC provider.

2.    Create an IAM role named Amazon_EBS_CSI_Driver:

aws iam create-role \
 --role-name AmazonEKS_EBS_CSI_Driver \
 --assume-role-policy-document file://"trust-policy.json"
3.    Attach the AWS managed IAM policy for the EBS CSI Driver to the IAM role you created:

aws iam attach-role-policy \
--policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
--role-name AmazonEKS_EBS_CSI_DriverRole
4.    Deploy the Amazon EBS CSI driver:
Note: You can deploy the EBS CSI driver using Kustomize, Helm, or an Amazon EKS managed add-on. In the example below, the driver is deployed using the Amazon EKS add-on feature. For more information, see the aws-ebs-csi-driver installation guide.

aws eks create-addon \
 --cluster-name my-cluster \
 --addon-name aws-ebs-csi-driver \
 --service-account-role-arn arb:aws:iam::
YOUR_AWS_ACCOUNT_ID:role/AmazonEKS_EBS_CSI_DriverRole
Note: Replace my-cluster with your cluster name and YOUR_AWS_ACCOUNT_ID with your account ID.

Test the Amazon EBS CSI driver:

You can test your Amazon EBS CSI driver with a sample application that uses dynamic provisioning for the pods. The Amazon EBS volume is provisioned on demand.

1.    Clone the aws-ebs-csi-driver repository from AWS GitHub:

git clone https://github.com/kubernetes-sigs/aws-ebs-csi-driver.git
2.    Change your working directory to the folder that contains the Amazon EBS driver test files:

cd aws-ebs-csi-driver/examples/kubernetes/dynamic-provisioning/
3.    Create the Kubernetes resources required for testing:

kubectl apply -f manifests/
Note: The kubectl command creates a StorageClass (from the Kubernetes website), PersistentVolumeClaim (PVC) (from the Kubernetes website), and pod. The pod references the PVC. An Amazon EBS volume is provisioned only when the pod is created.

4.    Describe the ebs-sc storage class:

kubectl describe storageclass ebs-sc
5.    Watch the pods in the default namespace and wait for the app pod's status to change to Running. For example:

kubectl get pods --watch
6.    View the persistent volume created because of the pod that references the PVC:

kubectl get pv
7.    View information about the persistent volume:

kubectl describe pv your_pv_name
Note: Replace your_pv_name with the name of the persistent volume returned from the preceding step 6. The value of the Source.VolumeHandle property in the output is the ID of the physical Amazon EBS volume created in your account.

8.    Verify that the pod is writing data to the volume:

kubectl exec -it app -- cat /data/out.txt
Note: The command output displays the current date and time stored in the /data/out.txt file. The file includes the day, month, date, and time.

Option B: Deploy and test the Amazon EFS CSI driver
Before deploying the CSI driver, create an IAM role that allows the CSI driver's service account to make calls to AWS APIs on your behalf.

1.    Download the IAM policy document from GitHub:

curl -o iam-policy-example.json https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/master/docs/iam-policy-example.json
2.    Create an IAM policy:

aws iam create-policy \
    --policy-name AmazonEKS_EFS_CSI_Driver_Policy \
    --policy-document file://iam-policy-example.json
3.    Run the following command to determine your cluster's OIDC provider URL:

aws eks describe-cluster --name your_cluster_name --query "cluster.identity.oidc.issuer" --output text
Note: In step 3, replace your_cluster_name with your cluster name.

4.    Create the following IAM trust policy, and then grant the AssumeRoleWithWebIdentity action to your Kubernetes service account. For example:
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





