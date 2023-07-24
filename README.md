# add-EFS-to-eks

### Prerequisites
## 1- Create OIDC
#### Follow the requirements here: https://repost.aws/knowledge-center/eks-persistent-storage
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






######