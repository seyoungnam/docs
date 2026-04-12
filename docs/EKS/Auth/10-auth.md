# Amazon EKS Authentication and Authorization Mechanisms

## 1. Initial Cluster Infrastructure Provisioning

### Terraform-Based Deployment
``` bash title="Clone the laboratory repository and navigate to the Week 4 module"
git clone https://github.com/gasida/aews.git
cd aews/4w
```

!!! note "Key Architectural Enhancements (vs. Week 3)"

    - **Control Plane Logging**: Enabled [comprehensive logging](https://github.com/gasida/aews/blob/main/4w/eks.tf#L80-L86) for auditability and troubleshooting.
    - **IRSA Integration**: Enabled [IAM Roles for Service Accounts (IRSA)](https://github.com/gasida/aews/blob/main/4w/eks.tf#L74) via a dedicated OIDC provider.
    - **Optimized Policy Sets**: Excluded the [AWSLoadBalancerControllerPolicy](https://github.com/gasida/aews/blob/main/4w/eks.tf#L110) for lean configuration.
    - **External-DNS Synchronization**: Configured the `external-dns` policy to [`sync`](https://github.com/gasida/aews/blob/main/4w/eks.tf#L165) for automated record lifecycle management.
    - **Core Add-on Integration**: Deployed essential [add-ons](https://github.com/gasida/aews/blob/main/4w/eks.tf#L168-L173), including:
        - `eks-pod-identity-agent`: Facilitates secure, high-performance credential delivery to pods without using IRSA's OIDC overhead.

!!! warning "Configuration Prerequisites"
    Prior to deployment, ensure the [TargetRegion](https://github.com/gasida/aews/blob/main/4w/var.tf#L47) and [availability_zones](https://github.com/gasida/aews/blob/main/4w/var.tf#L53) in `var.tf` are correctly configured for your environment.

``` bash title="Deploying EKS infrastructure via Terraform"
terraform init
terraform plan
nohup sh -c "terraform apply -auto-approve" > create.log 2>&1 &
tail -f create.log
```

``` bash title="Establishing kubectl context and environment variables"
$(terraform output -raw configure_kubectl)
[ "$(kubectl config current-context)" != "myeks" ] && \
(kubectl config delete-context myeks 2>/dev/null || true; \
 kubectl config rename-context $(kubectl config current-context) myeks)

export CLUSTER_NAME=myeks
export ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text)
```

``` bash hl_lines="3 6" title="Validating core component deployment"
# Verify the EKS Pod Identity Agent status
kubectl get ds,pod -n kube-system \
-l app.kubernetes.io/instance=eks-pod-identity-agent # (1)!

# Inspect External-DNS configuration arguments
kubectl describe deploy -n external-dns external-dns | grep Args: -A10 # (2)!

# Validate Cert-Manager installation
kubectl get all -n cert-manager
```

1.  :octicons-code-review-16: **Agent Health Check**:
    ``` text
    NAME                                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    daemonset.apps/eks-pod-identity-agent   2         2         2       2            2           <none>          14m

    NAME                               READY   STATUS    RESTARTS   AGE
    pod/eks-pod-identity-agent-b7j4g   1/1     Running   0          14m
    pod/eks-pod-identity-agent-sl5gt   1/1     Running   0          14m
    ```
2.  :octicons-code-review-16: **Configuration Audit**:
    ``` text hl_lines="7"
        Args:
          --log-level=info
          --log-format=text
          --interval=1m
          --source=service
          --source=ingress
          --policy=sync
          --registry=txt
          --txt-owner-id=myeks
          --provider=aws
    ```

``` bash title="Managing Worker Nodes via AWS Systems Manager (SSM)"
# Identify instances registered with SSM
aws ssm describe-instance-information \
  --query "InstanceInformationList[*].{InstanceId:InstanceId, Status:PingStatus, OS:PlatformName}" \
  --output text # table

export NODE1=<INSTANCE_ID_1>
export NODE2=<INSTANCE_ID_2>

# Establish interactive shell sessions
aws ssm start-session --target $NODE1
aws ssm start-session --target $NODE2
```

---

## 2. Deep Dive: Identity and Access Management (IAM)

### The API Server Access Control Workflow

![access-control-overview](https://kubernetes.io/images/docs/admin/access-control-overview.svg)
/// caption
Source: [Kubernetes Official Documentation - Controlling Access to the Kubernetes API](https://kubernetes.io/docs/concepts/security/controlling-access/)
///

Interaction with the Kubernetes API is governed by a multi-stage security pipeline, ensuring every request is validated before persisting to **etcd**.

``` mermaid
graph LR
    User([User/ServiceAccount]) --> Trans[1. Transport Security]
    subgraph K8s_API_Server [K8s API Server]
        Trans --> AuthN[2. Authentication]
        AuthN --> AuthZ[3. Authorization]
        AuthZ --> AC[4. Admission Control]
        AC --> Registry[API Registry/etcd]
    end
    
    style AuthN fill:#f9f,stroke:#333,stroke-width:2px
    style AuthZ fill:#bbf,stroke:#333,stroke-width:2px
    style AC fill:#dfd,stroke:#333,stroke-width:2px
```

1.  **Transport Security**: All communication is encrypted via **TLS**. Clients must utilize certificates trusted by the cluster's Root CA.
2.  **Authentication (AuthN)**: Verifies the requester's identity. Failure at this stage results in an **HTTP 401 Unauthorized** response.
3.  **Authorization (AuthZ)**: Evaluates whether the authenticated identity has the necessary permissions to perform the requested action. Failure results in an **HTTP 403 Forbidden** response.
4.  **Admission Control**: Processes requests that modify cluster state. Admission controllers can perform data validation or mutation (e.g., injecting sidecars) before final commit.

### Amazon EKS Authentication Architecture

Amazon EKS integrates AWS IAM with Kubernetes RBAC through the `aws-iam-authenticator` (or the native EKS Access Entry service). This enables the use of IAM identities for cluster authentication.

<div style="overflow-x: auto;" markdown="1">

``` mermaid
sequenceDiagram
    %%{init: {'themeVariables': { 'fontSize': '16px' }, 'sequence': {'useMaxWidth': false} }}%%
    actor User
    participant K as kubectl
    participant C as Auth Client
    participant API as API Server
    participant S as Auth Service
    participant STS
    participant M as Identity Mapping
    participant RBAC as RBAC / Access Policy

    User->>K: 1. K8s Action
    K->>C: 2. Issue Token (Pre-signed URL)
    C-->>K: Return Token
    K->>API: 3. Action + Token
    
    Note over API,M: Authentication (AuthN) Phase
    API->>S: 4. Check Token (TokenReview)
    S->>STS: 5. sts:GetCallerIdentity
    STS-->>S: 6. IAM Identity
    S->>M: 7. Check Mapping (aws-auth / Access Entry)
    M-->>S: Return K8s Identity
    S-->>API: 8. Authenticated (User/Groups)
    
    Note over API,RBAC: Authorization (AuthZ) Phase
    API->>RBAC: 9. Check Permissions (SubjectAccessReview)
    RBAC-->>API: 10. Allowed / Denied
    
    API->>K: 11. Response
```

</div>

#### Token Issuance and the Pre-signed URL Model

EKS authentication tokens are essentially **Pre-signed URLs** for the AWS STS `GetCallerIdentity` action. These tokens are generated locally and have a short expiration window.

``` bash hl_lines="2 15-18" title="Analyzing the EKS Kubeconfig executive command"
# Identify the local IAM identity
aws sts get-caller-identity --query Arn # (1)!

# Inspect the kubeconfig 'exec' configuration
cat ~/.kube/config
...
users:
- name: arn:aws:eks:us-east-1:080403789922:cluster/myeks
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - --region
      - us-east-1
      - eks
      - get-token
      - --cluster-name
      - myeks # (2)!
      - --output
      - json
      command: aws
...
```

1.  :octicons-code-review-16: **Active Principal**:
    ``` text
    arn:aws:iam::{account-id}:user/admin
    ```
2.  :information_source: The `aws eks get-token` command facilitates the creation of the temporary session token.

#### Deciphering the Authentication Token

The token provided by the AWS CLI is a base64-encoded string containing the signed STS request.

``` bash title="Decoding the authentication payload"
TOKEN_DATA=$(aws eks get-token --cluster-name myeks | jq -r '.status.token')
IFS='.' read header payload signature <<< "$TOKEN_DATA"

# Decode the payload to reveal the signed GetCallerIdentity request
echo "$payload" | fold -w 4 | sed '$ d' | tr -d '\n' | base64 --decode
https://sts.us-east-1.amazonaws.com/?Action=GetCallerIdentity&
Version=2011-06-15&X-Amz-Algorithm=AWS4-HMAC-SHA256&
X-Amz-Credential=AKIARFODQPBRHTK56HEE%2F20260408%2Fus-east-1%2Fsts%2Faws4_request&
X-Amz-Date=20260408T041921Z&X-Amz-Expires=60&
X-Amz-SignedHeaders=host%3Bx-k8s-aws-id&
X-Amz-Signature=<SIG_HASH>
```

#### The TokenReview Process

The Kubernetes API server utilizes the **Webhook Token Authentication** plugin to validate tokens. It submits a `TokenReview` request to the EKS identity service, which then proxies the request to AWS STS to confirm the identity.

``` bash hl_lines="11" title="Executing a manual TokenReview validation"
TOKEN_DATA=$(aws eks get-token --cluster-name myeks | jq -r '.status.token')
cat > token-review.yaml << EOF
apiVersion: authentication.k8s.io/v1
kind: TokenReview
metadata:
  name: identity-check
spec:
  token: ${TOKEN_DATA}
EOF

kubectl create -f token-review.yaml -v=9 # (1)!
```

1.  :octicons-code-review-16: **Authentication Metadata**: The response confirms the mapping from the IAM user to the internal Kubernetes identity.
    ``` json hl_lines="3 7-8"
    {
      "status": {
        "authenticated": true,
        "user": {
          "username": "arn:aws:iam::{account-id}:user/admin",
          "groups": [
            "system:authenticated"
          ],
          ...
        }
      }
    }
    ```

#### Auditability via AWS CloudTrail

Every authentication attempt triggered by `aws-iam-authenticator` is logged in **AWS CloudTrail** as a `GetCallerIdentity` event, providing a robust audit trail for cluster access.

![get-caller-identity](../../assets/img/eks/10-auth/get-caller-identity.png)

---

## 3. Advanced RBAC Inspection and Auditing

To maintain a secure cluster, it is essential to audit the permissions associated with various subjects. We use the `rbac-tool` and `rolesum` Krew plugins for detailed analysis.

``` bash hl_lines="5 8" title="Auditing cluster permissions"
# Identify the current subject
kubectl whoami

# Summarize permissions for the 'system:nodes' group
kubectl rolesum -k Group system:nodes # (1)!

# Discover roles bound to a specific group
kubectl rbac-tool lookup system:masters # (2)!
```

1.  :octicons-code-review-16: **Node Permissions**: Typically managed by the Node Authorizer, nodes have limited explicit RBAC.
2.  :octicons-code-review-16: **Administrative Binding**:
    ``` text
      SUBJECT        | SUBJECT TYPE | SCOPE       | NAMESPACE | ROLE          | BINDING        
    -----------------+--------------+-------------+-----------+---------------+----------------
      system:masters | Group        | ClusterRole |           | cluster-admin | cluster-admin  
    ```

### Validating Authorization via SubjectAccessReview

We can manually test if a subject has specific permissions using the `SubjectAccessReview` resource.

``` bash hl_lines="14" title="Testing explicit authorization"
cat > sar-test.yaml << EOF
apiVersion: authorization.k8s.io/v1
kind: SubjectAccessReview
spec:
  user: "arn:aws:iam::${ACCOUNT_ID}:user/admin"
  groups:
    - system:masters
  resourceAttributes:
    namespace: "kube-system"
    verb: "get"
    resource: "pods"
EOF

kubectl create -f sar-test.yaml # (1)!
```

1.  :octicons-code-review-16: **Authorized**: Returns `allowed: true`, confirming the administrative principal has the requested access.

By utilizing **EKS Access Entries** and native **Access Policies**, you can manage these permissions directly through the AWS API, further centralizing security management.

![admin-policy](../../assets/img/eks/10-auth/admin-policy.png)
