# add-EFS-to-eks

### Prerequisites

#### Follow the requirements here: https://repost.aws/knowledge-center/eks-persistent-storage
## 1- Create OIDC
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

#### Dynamic nfs
##### On AWS EFS , here it is
####### Create a EFS and then copy the DNS name of the EFS
![image](https://github.com/thedevopsguru1/dynamic-nfs-provisioing/assets/126810742/c2136ea6-eb7e-4f4c-bf39-3eed8f5ff764)

```
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=fs-0e4070b4c27a59f48.efs.us-east-2.amazonaws.com  --set nfs.path=/
```
https://github.com/thedevopsguru1/dynamic-nfs-provisioing





