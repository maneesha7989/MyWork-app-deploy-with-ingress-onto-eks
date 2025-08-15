# App-deploy-with-ingress-onto-eks
(Mini project - app deployment into EKS along with Ingress)
--------------------------------------------------------------------------------------------------------------------------------------
EKS Pros:
- Managed Control Plane: In EKS, cloud provider AWS itself takes care of managing the Kubernetes control plane components, such as the API server, controller manager, and etcd. AWS handles upgrades, patches, and ensures high availability of the control plane, so we don’t need to bother about it. 

- Automated Updates: EKS automatically updates the Kubernetes version, eliminating the need for manual intervention and ensuring that the cluster stays up-to-date with the latest features and security patches.

- Scalability: EKS can automatically scale the Kubernetes control plane based on demand, ensuring the cluster remains responsive as the workload increases.

- AWS Integration: EKS seamlessly integrates with various AWS services, such as AWS IAM for authentication and authorization, Amazon VPC for networking, and AWS Load Balancers for service exposure.

- Monitoring and Logging: EKS integrates with AWS CloudWatch for monitoring cluster health and performance metrics, making it easier to track and troubleshoot issues.

EKS Cons:
- Cost: EKS is a managed service, and this convenience comes at a cost. Running an EKS cluster may be more expensive compared to self-managed Kubernetes, especially for large-scale deployments.

- Less Control: While EKS provides a great deal of automation, it also means that you have less control over the underlying infrastructure and some Kubernetes configurations.

Self-Managed Kubernetes on EC2 Instances Pros:
- Cost-Effective: Self-managed Kubernetes allows you to take advantage of EC2 spot instances and reserved instances, potentially reducing the overall cost of running Kubernetes clusters.

- Flexibility: With self-managed Kubernetes, you have full control over the cluster's configuration and infrastructure, enabling customization and optimization for specific use cases.

- Experimental Features: Self-managed Kubernetes allows you to experiment with the latest Kubernetes features and versions before they are officially supported by EKS.

Self-Managed Kubernetes on EC2 Instances Cons:
- Complexity: Setting up and managing a self-managed Kubernetes cluster can be complex and time-consuming, especially for those new to Kubernetes or AWS.

- Maintenance Overhead: Self-managed clusters require manual management of Kubernetes control plane updates, patches, and high availability.

- Scaling Challenges: Scaling the control plane of a self-managed cluster can be challenging, and it requires careful planning to ensure high availability during scaling events.

- Security and Compliance: Self-managed clusters may require additional effort to implement best practices for security and compliance compared to EKS, which comes with some built-in security      features.

- Lack of Automation: Self-managed Kubernetes requires more manual intervention and scripting for certain operations, which can increase the risk of human error.

# Project implementation:
------------------------------------------------------------------------------------------------------------------------------------
Create a new cluster or use any existing EKS cluster

Manual cluster creation command - AWS CLI

    $  eksctl create cluster --name demo-cluster-1 --region us-east-1 --fargate

Switching context to the newly created cluster

    $  aws eks update-kubeconfig --name demo-cluster-1 --region us-east-1

Creating FARGATE profile

    $  eksctl create fargateprofile \
        --cluster demo-cluster-1 \
        --namespace game-2048 \
        --region us-east-1 \
        --name alb-sample-app

Write eksInfra.yaml - single manifest file for multiple resources

APPLY the eksInfra.yaml

    $  kubectl apply -f eksInfra.yaml

Associating OIDC provider with the cluster

    $  eksctl utils associate-iam-oidc-providrer --cluster \
        demo-cluster-1 --approve

Copy-paste Alb-ingress controller JSON policy and edit is as required and store in our repo

Download into server using command "$ curl -O <custom-JSON-policy-github-link>

From that policy doc, Create JSON policy inside cluster

    $  aws iam create-policy \
        --policy-name AWSLoadBalancerControllerIAMPolicy
        --policy-document file://iam_policy.json 

Then create new Service Account or use an existing one

    $  eksctl create iamserviceaccount \
        --cluster=demo-cluster-1 \
        --namespace=kube-system \
        --name=aws-load-balancer-controller-sa \
        --role-name AmazonEKSLoadBalancerControllerRole \
        --attach-policy-arn=arn:aws:iam::<my-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
        --approve

Next is to update Helm repo "eks" and install controller

    $  helm repo update eks
    $  helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
            --set clusterName=demo-cluster-1 \
            --set serviceAccount.create=false \
            --set serviceAccount.name=aws-load-balancer-controller-sa \
            --set region=us-east-1 \
            --set vpcId=<my-vpc-id>

DNS ADDRESS is now reflecting in our Ingress

Executed it on browser, and that's it. App accessible for external users via internet !!

<img width="302" height="194" alt="image" src="https://github.com/user-attachments/assets/ba730349-7cca-44a2-9c4c-0ad5c78d46c8" />


!!  The End !!
