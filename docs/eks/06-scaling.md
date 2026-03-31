# EKS Scaling

## 1. Cluster Provisioning

``` bash title="Clone the aews repo and move to 3w directory"
git clone https://github.com/gasida/aews.git
cd aews/3w
```

??? note "What's changed compared to w2"

    - EKS version upgraded to [v1.35](https://github.com/gasida/aews/blob/main/3w/var.tf#L23) to support in-place pod resource(CPU/memory) upgrades.
    - Add [additional IAM roles](https://github.com/gasida/aews/blob/main/3w/eks.tf#L113-L118) to EC2 instance profile to allow SSM Session Manager to access EC2 instances.
    - Add [metrics-server, external-dns](https://github.com/gasida/aews/blob/main/3w/eks.tf#L286-L297) to addons

!!! warning
    Modify [TargetRegion](https://github.com/gasida/aews/blob/main/3w/var.tf#L47) and [availability_zones](https://github.com/gasida/aews/blob/main/3w/var.tf#L53) in `var.tf` to wherever you want to deploy.

``` bash title="Download lb controller IAM policy file"
curl -o aws_lb_controller_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/refs/heads/main/docs/install/iam_policy.json
```

``` bash title="Create external-dns controller IAM policy"
cat << EOF > externaldns_controller_policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets",
        "route53:ListResourceRecordSets",
        "route53:ListTagsForResources"
      ],
      "Resource": [
        "arn:aws:route53:::hostedzone/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
EOF
```

``` bash title="Create auto-scaling controller IAM policy"
cat << EOF > cas_autoscaling_policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeScalingActivities",
        "ec2:DescribeImages",
        "ec2:DescribeInstanceTypes",
        "ec2:DescribeLaunchTemplateVersions",
        "ec2:GetInstanceTypesFromInstanceRequirements",
        "eks:DescribeNodegroup"
      ],
      "Resource": ["*"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup"
      ],
      "Resource": ["*"]
    }
  ]
}
EOF
```

``` bash title="Confirm three IAM policies"
ls *.json
aws_lb_controller_policy.json      cas_autoscaling_policy.json        externaldns_controller_policy.json
```

These policy files are supplied into [`aws_iam_policy` resources](https://github.com/gasida/aews/blob/main/3w/eks.tf#L42-L60)


``` bash title="Deploy EKS cluster resources via terraform"
terraform init
terraform plan
nohup sh -c "terraform apply -auto-approve" > create.log 2>&1 &
tail -f create.log
```

It will take about 12 minutes to provision the entire cluster resources. After the deployment is done, check the state of key resources.

``` bash title="Check the terraform state"
cat terraform.tfstate
terraform show
terraform state list
```

Update myeks cluster with the new context.
``` bash title="Add a new context to ~/.kube/config and update myeks cluster with the new context"
$(terraform output -raw configure_kubectl)
kubectl config rename-context $(cat ~/.kube/config | grep current-context | awk '{print $2}') myeks
# Context "arn:aws:eks:us-east-1:080403789922:cluster/myeks" renamed to "myeks".
```

``` bash title="check the eks version"
kubectl get node -o wide
```

## 2. Access Instance with SSM

In addition to `ssh`, Session Manager(SSM) provides a way to access the instance cli.

``` bash title="Look up the instance list managed by SSM"
aws ssm describe-instance-information \
  --query "InstanceInformationList[*].{InstanceId:InstanceId, Status:PingStatus, OS:PlatformName}" \
  --output text # table

i-047205992d1f630f1     Amazon Linux    Online
i-0e33ef688a374b594     Amazon Linux    Online
```

``` bash title="Install session-manager-plugin"
# https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/install-plugin-macos-overview.html
brew install --cask session-manager-plugin # macOS - Disable date: 2026-09-01
```

``` bash title="Access to the instance cli"
aws ssm start-session --target i-047205992d1f630f1

Starting session with SessionId: admin-h32jgdpoznvu4n9d7lktisl3ie
sh-5.2$ bash
[ssm-user@ip-192-168-13-172 bin]$
```

??? info "What enables to access the instance using sesson manager?"

    `AmazonSSMManagedInstanceCore` policy is added to EKS managed node group IAM role.
    ![ssm-policy](../assets/img/eks/06-scaling/ssm-policy.png)

??? info "How could my terminal access the instance with no public IP?"

    ![no-public-ip](../assets/img/eks/06-scaling/no-public-ip.png)
    Public IP is not attached to each instance. Then how can the external traffic reach to the instance? 

    ![ssm](../assets/img/eks/06-scaling/ssm.jpeg)
    
    When the SSM Agent starts up on your EC2 instance (usually at boot), it immediately initiates an **outbound HTTPS (TLS) connection** to the Systems Manager service endpoints in the AWS Cloud. It uses a technology similar to WebSockets or Long Polling. When you start a session via the console or CLI, here's what happens:

    1. AWS Console/CLI sends a request to the Session Manager service.
    2. Session Manager looks at its list of active tunnels and finds the one belonging to your instance.
    3. It sends a **Start Session** signal down that existing outbound tunnel.
    4. SSM Agent receives the signal, spawns a local shell process (like `/bin/bash`), and begins streaming the input/output of that shell back through the same tunnel.

    ![sessions-terminal](../assets/img/eks/06-scaling/sessions-terminal.png)

    **Why does this matter?**

    * No Inbound Rules: Because the agent initiates the connection, your Security Group can have zero inbound rules and still work perfectly.
    * Private Subnets: This is why SSM works for instances in private subnets (provided they have a route to the internet via NAT Gateway or [VPC Endpoints](https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-create-vpc-endpoints.html)).
    * Identity-Based: The connection is only allowed if the Instance's IAM Role has the `AmazonSSMManagedInstanceCore` policy, which gives the agent permission to check-in with the AWS service.


## 3. Confirm Addons

``` bash title="List up installed addons"
aws eks list-addons --cluster-name myeks | jq
{
  "addons": [
    "coredns",
    "external-dns",
    "kube-proxy",
    "metrics-server",
    "vpc-cni"
  ]
}
```

``` bash title="Check metrics-server"
kubectl get deploy -n kube-system metrics-server
kubectl describe deploy -n kube-system metrics-server
kubectl get pod -n kube-system -l app.kubernetes.io/instance=metrics-server -o wide
kubectl get pdb -n kube-system metrics-server
kubectl get svc,ep -n kube-system metrics-server
```

``` bash title="Check metrics-server api"
kubectl api-resources | grep -i metrics
kubectl explain NodeMetrics
kubectl explain PodMetrics
kubectl api-versions | grep metrics
kubectl get apiservices |egrep '(AVAILABLE|metrics)'
```

``` bash title="Check cpu/mem usage"
kubectl top node
kubectl top pod -A
kubectl top pod -n kube-system --sort-by='cpu'
kubectl top pod -n kube-system --sort-by='memory'
```

``` bash title="Check external-dns"
kubectl get deploy,pod,svc,ep,sa -n external-dns 
kubectl get sa -n external-dns external-dns -o yaml

kubectl describe deploy -n external-dns external-dns
...
    Args:
      --log-level=info
      --log-format=text
      --interval=1m
      --source=service
      --source=ingress
      --policy=upsert-only # record should be manually deleted in route 53
      --registry=txt
      --txt-owner-id=myeks
      --provider=aws
...
```

## 4. Install AWS Load Balancer Controller

``` bash title="Add helm chart repo"
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

``` bash title="Install AWS LBC using helm chart"
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --version 3.1.0 \
  --set clusterName=myeks \
  --set region=us-east-1
```
!!! info

    Pod is able to get the EC2 Instance Profile(IAM Role) via IMDS.

``` bash title="Check deployed resources"
helm list -n kube-system
kubectl get pod -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
kubectl logs -n kube-system deployment/aws-load-balancer-controller -f
```