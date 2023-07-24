# add-EFS-to-eks

### Prerequisites
#### Follow the requirements here: https://repost.aws/knowledge-center/eks-persistent-storage
#### Create OIDC ROle first: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html
#### For the OIDC setup follow this: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html
###### The Provider URL comme from : run this command: aws eks describe-cluster --name your_cluster_name --query "cluster.identity.oidc.issuer" --output text
![image](https://github.com/thedevopsguru1/add-EFS-to-eks/assets/126810742/d8a38785-9065-4b68-827b-9f7382b27912)
![image](https://github.com/thedevopsguru1/add-EFS-to-eks/assets/126810742/a9ab9e6a-93a8-4df7-8fc7-7bca59d254be)



######
