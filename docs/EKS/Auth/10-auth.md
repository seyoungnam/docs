# AuthN/AuthZ

## 1. Cluster Provisioning

### terraform apply
``` bash title="Clone the aews repo and move to 4w directory"
git clone https://github.com/gasida/aews.git
cd aews/4w
```

!!! note "What's changed compared to 3w"

    - Enables [control plane logs](https://github.com/gasida/aews/blob/main/4w/eks.tf#L80-L86)
    - [Enables IRSA](https://github.com/gasida/aews/blob/main/4w/eks.tf#L74) to use OIDC provider
    - Excludes [AWSLoadBalancerControllerPolicy](https://github.com/gasida/aews/blob/main/4w/eks.tf#L110) 
    - [Keep `external-dns`'s policy `sync`](https://github.com/gasida/aews/blob/main/4w/eks.tf#L165), meaning A record will be auto-deleted
    - Add [additional addons](https://github.com/gasida/aews/blob/main/4w/eks.tf#L168-L173)
        - `eks-pod-identity-agent`: When a pod makes a request to an AWS service, the agent intercepts the call to the AWS STS (Security Token Service) and provides the temporary credentials associated with the IAM role assigned to the pod's ServiceAccount.

!!! warning
    Modify [TargetRegion](https://github.com/gasida/aews/blob/main/4w/var.tf#L47) and [availability_zones](https://github.com/gasida/aews/blob/main/4w/var.tf#L53) in `var.tf` to wherever you want to deploy.

``` bash title="Deploy EKS cluster resources via terraform"
terraform init
terraform plan
nohup sh -c "terraform apply -auto-approve" > create.log 2>&1 &
tail -f create.log
```

``` bash title="Rename the cluster context and set up variables"
$(terraform output -raw configure_kubectl)
[ "$(kubectl config current-context)" != "myeks" ] && \
(kubectl config delete-context myeks 2>/dev/null || true; \
 kubectl config rename-context $(kubectl config current-context) myeks)

export CLUSTER_NAME=myeks
export ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text)
```

``` bash hl_lines="2 4" title="Confirm the deployment"
kubectl get ds,pod -n kube-system \
-l app.kubernetes.io/instance=eks-pod-identity-agent # (1)!

kubectl describe deploy -n external-dns external-dns | grep Args: -A10 # (2)!
kubectl get all -n cert-manager
```

1.  :octicons-code-review-16:
    ``` text
    NAME                                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    daemonset.apps/eks-pod-identity-agent   2         2         2       2            2           <none>          14m

    NAME                               READY   STATUS    RESTARTS   AGE
    pod/eks-pod-identity-agent-b7j4g   1/1     Running   0          14m
    pod/eks-pod-identity-agent-sl5gt   1/1     Running   0          14m
    ~/Documents/GitHub/aews/4w  week4 !1 ?2  kubectl describe deploy -n external-dns external-dns | grep Args: -A10 
    ```
2.  :octicons-code-review-16:
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
        Limits:
    ```

``` bash title="Get instances managed by SSM"
aws ssm describe-instance-information \
  --query "InstanceInformationList[*].{InstanceId:InstanceId, Status:PingStatus, OS:PlatformName}" \
  --output text # table

export NODE1=i-0b64a233926c6200c
export NODE2=i-0e7d7b7325528e294

aws ssm start-session --target $NODE1
aws ssm start-session --target $NODE2
```

### install krew tools

``` bash title="Install krew tools"
kubectl krew install access-matrix rbac-tool rolesum whoami
```

``` bash title="Run commands"
kubectl whoami # (1)!

# Show an RBAC access matrix for server resources
kubectl access-matrix
kubectl access-matrix --namespace default # (2)!

# RBAC Lookup by subject (user/group/serviceaccount) name
kubectl rbac-tool lookup
kubectl rbac-tool lookup system:masters # (3)!
kubectl rbac-tool lookup system:nodes # (4)!
kubectl rbac-tool lookup system:bootstrappers # eks:node-bootstrapper
kubectl describe ClusterRole eks:node-bootstrapper # (5)!

# RBAC List Policy Rules For subject (user/group/serviceaccount) name
kubectl rbac-tool policy-rules
kubectl rbac-tool policy-rules -e '^system:.*'
kubectl rbac-tool policy-rules -e '^system:authenticated'

# Summarize RBAC roles for subjects : ServiceAccount(default), User, Group
kubectl rolesum -h
kubectl rolesum aws-node -n kube-system    # sa
kubectl rolesum -k User system:kube-proxy  # user
kubectl rolesum -k Group system:masters    # group
kubectl rolesum -k Group system:nodes
kubectl rolesum -k Group system:authenticated # (6)!
```

1.  :octicons-code-review-16:
    ``` text
    arn:aws:iam::0804:user/admin
    ```
2.  :octicons-code-review-16:
    ``` text
    NAME                                            LIST  CREATE  UPDATE  DELETE
    applicationnetworkpolicies.networking.k8s.aws   ✔     ✔       ✔       ✔
    bindings                                              ✔               
    certificaterequests.cert-manager.io             ✔     ✔       ✔       ✔
    certificates.cert-manager.io                    ✔     ✔       ✔       ✔
    challenges.acme.cert-manager.io                 ✔     ✔       ✔       ✔
    ...
    ```
3.  :octicons-code-review-16:
    ``` text
      SUBJECT        | SUBJECT TYPE | SCOPE       | NAMESPACE | ROLE          | BINDING        
    -----------------+--------------+-------------+-----------+---------------+----------------
      system:masters | Group        | ClusterRole |           | cluster-admin | cluster-admin  
    ```
4.  :octicons-code-review-16:
    ``` text
      SUBJECT      | SUBJECT TYPE | SCOPE       | NAMESPACE | ROLE                  | BINDING                
    ---------------+--------------+-------------+-----------+-----------------------+------------------------
      system:nodes | Group        | ClusterRole |           | eks:node-bootstrapper | eks:node-bootstrapper   
    ```
5.  :octicons-code-review-16:
    ``` text
    Name:         eks:node-bootstrapper
    Labels:       eks.amazonaws.com/component=node
    Annotations:  <none>
    PolicyRule:
      Resources                                                      Non-Resource URLs  Resource Names  Verbs
      ---------                                                      -----------------  --------------  -----
      certificatesigningrequests.certificates.k8s.io/selfnodeclient  []                 []              [create]
      certificatesigningrequests.certificates.k8s.io/selfnodeserver  []                 []              [create]
    ```
6.  :octicons-code-review-16:
    ``` text
    Group: system:authenticated

    Policies:
    • [CRB] */system:basic-user ⟶  [CR] */system:basic-user
      Resource                                       Name  Exclude  Verbs  G L W C U P D DC  
      selfsubjectaccessreviews.authorization.k8s.io  [*]     [-]     [-]   ✖ ✖ ✖ ✔ ✖ ✖ ✖ ✖   
      selfsubjectreviews.authentication.k8s.io       [*]     [-]     [-]   ✖ ✖ ✖ ✔ ✖ ✖ ✖ ✖   
      selfsubjectrulesreviews.authorization.k8s.io   [*]     [-]     [-]   ✖ ✖ ✖ ✔ ✖ ✖ ✖ ✖   


    • [CRB] */system:discovery ⟶  [CR] */system:discovery


    • [CRB] */system:public-info-viewer ⟶  [CR] */system:public-info-viewer 
    ```


## 2. Identity and Access Management

### controlling access to the k8s API

![access-control-overview](https://kubernetes.io/images/docs/admin/access-control-overview.svg)
/// caption
https://kubernetes.io/docs/concepts/security/controlling-access/
///

Users and service accounts access the Kubernetes API through a multi-stage process. Every request must pass through several checkpoints before being executed and stored in **etcd**.

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

1. **Transport Security**: By default, the API server is protected by **TLS**. Clients must trust the cluster's CA to establish a secure connection (typically on port 6443).
2. **Authentication (AuthN)**: Verifies **who** is making the request. The API server uses authenticator modules (client certs, tokens, JWTs). If authentication fails, it returns an **HTTP 401** error.
3. **Authorization (AuthZ)**: Determines **what** the authenticated user is allowed to do. Kubernetes checks attributes like user, verb (get, create, etc.), and resource. Typically managed via **RBAC**. If denied, it returns an **HTTP 403** error.
4. **Admission Control**: The final gate for requests that modify objects (create, update, delete). Admission controllers can **mutate** (modify) the request or **validate** it. If any controller rejects the request, the entire request fails immediately.

### EKS Authentication Process

The following diagram illustrates how authentication and authorization work in Amazon EKS using the `aws-iam-authenticator`:

<div style="overflow-x: auto;" markdown="1">

``` mermaid
sequenceDiagram
    %%{init: {'themeVariables': { 'fontSize': '16px' }, 'sequence': {'useMaxWidth': false} }}%%
    actor User
    participant K as kubectl
    participant C as Auth Client
    participant API as API Server
    participant S as Auth Server
    participant STS
    participant M as aws-auth
    participant RBAC

    User->>K: 1. K8s Action
    K->>C: 2. Issue Token (Pre-signed URL)
    C-->>K: Return Token
    K->>API: 3. Action + Token
    API->>S: 4. Check Id Token
    S->>STS: 5. sts:GetCallerIdentity
    STS-->>S: 6. Success
    S->>M: 7. Check K8s User Mapping
    M-->>API: Return Identity
    API->>RBAC: 8. Check K8s Role
    RBAC-->>API: Confirm Permissions
    API->>K: 9. Allow / Deny
```

</div>

#### 1. K8s Action & 2. Issue Token (Pre-signed URL)

`aws eks get-token --cluster-name {cluster-name}` is the command to generate a token for authentication with an Amazon EKS cluster. This token is generated **locally** and attached to the `kubectl` request for authentication.


``` bash hl_lines="2 15-18" title="Where aws eks get-token command comes from?"
# get user info
aws sts get-caller-identity --query Arn # (1)!

# kubeconfig
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
      env: null
```

1.  :octicons-code-review-16:
    ``` text
    arn:aws:iam::{my-account-id}:user/admin
    ```
  
2.  :information_source: `aws eks get-token --cluster-name myeks` gets a token for authentication with an Amazon EKS cluster. Please run `aws eks get-token help` for more info.

Is `aws eks get-token --cluster-name {cluster-name}` executed everytime `kubectl` command is executed? The answer is no. The minted token is cached and used until `expirationTimestamp`. A new token is minted again when the token expires. Keep in mind that **the minting process happens in the local environment**.


``` bash hl_lines="1" title="What the token looks like?"
export CLUSTER_NAME=myeks
aws eks get-token --cluster-name $CLUSTER_NAME | jq # (1)!
aws eks get-token --cluster-name $CLUSTER_NAME | jq -r '.status.token'
aws eks get-token --cluster-name $CLUSTER_NAME --debug | jq
...
2026-04-08 00:00:55,430 - MainThread - botocore.regions - DEBUG - Calling endpoint provider with parameters: {'Region': 'us-east-1', 'UseDualStack': False, 'UseFIPS': False, 'UseGlobalEndpoint': False}
2026-04-08 00:00:55,431 - MainThread - botocore.regions - DEBUG - Endpoint provider result: https://sts.us-east-1.amazonaws.com
...
```

1.  :octicons-code-review-16:
    ``` json
    {
      "kind": "ExecCredential",
      "apiVersion": "client.authentication.k8s.io/v1beta1",
      "spec": {},
      "status": {
        "expirationTimestamp": "2026-04-08T02:48:17Z",
        "token": "k8s-aws-v1.<REDACTED_TOKEN>"
      }
    }    
    ```

``` bash hl_lines="1" title="JWT payload decoding (Header, Payload, Signature)"
TOKEN_DATA=$(aws eks get-token --cluster-name myeks | jq -r '.status.token')
IFS='.' read header payload signature <<< "$TOKEN_DATA" # (1)!

echo "$payload" | fold -w 4 | sed '$ d' | tr -d '\n' | base64 --decode
https://sts.us-east-1.amazonaws.com/?Action=GetCallerIdentity&
Version=2011-06-15&X-Amz-Algorithm=AWS4-HMAC-SHA256&
X-Amz-Credential=AKIARFODQPBRHTK56HEE%2F20260408%2Fus-east-1%2Fsts%2Faws4_request&
X-Amz-Date=20260408T041921Z&X-Amz-Expires=60&
X-Amz-SignedHeaders=host%3Bx-k8s-aws-id&
X-Amz-Signature=<REDACTED> # (2)!

```

1.  :information_source: This command splits a JWT into three variables—`header`, `payload`, and `signature`—using the period (`.`) as the delimiter. By setting `IFS='.'` (Internal Field Separator) just for this command, the read command assigns the text before the first `.` to `$header`, the text between `.` to `$payload`, and the rest to `$signature`.
2.  :information_source: the signature is the outcome of multiple `SHA256` hashing with a secret key and other additional parameters.


In sum, everytime we run a `kubectl` command, `aws eks get-token` command is also triggered to mint a new token or to use the existing token. This token is attached to the `kubectl` request and used for authentication at the `kube-apiserver`. 

#### 3. Action + Token

[`client-go` package](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins) is used in the `kubectl` command to transform **Pre-Signed URL** to **Bearer Token**, attach it to the request header, and make the request to `kube-apiserver`.

``` bash title="Call to kube-apiserver without kubectl"
kubectl get node -v=10
...
curl -v -XGET  -H "Accept: application/json;as=Table;v=v1;g=meta.k8s.io,application/json;as=Table;v=v1beta1;g=meta.k8s.io,application/json" -H "User-Agent: kubectl/v1.35.3 (darwin/arm64) kubernetes/6c1cd99" 'https://518F6F31299E0538B4621B38C98FA4BF.gr7.us-east-1.eks.amazonaws.com/api/v1/nodes?limit=500' # (1)!
...

TOKEN_DATA=$(aws eks get-token --cluster-name myeks | jq -r '.status.token')
curl -k -v -XGET \
  -H "Authorization: Bearer $TOKEN_DATA" \
  -H "Accept: application/json" \
  'https://518F6F31299E0538B4621B38C98FA4BF.gr7.us-east-1.eks.amazonaws.com/api/v1/nodes?limit=500'

...
> Authorization: Bearer k8s-aws-v1.aHR0cHM<REDACTED>
...
```

1.  :information_source: `kubectl` masks any security-assciated data(e.g., `Authorization` header) in logs, which is the reason we can't see the bearer token in the `curl` command.