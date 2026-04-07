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

<div style="overflow-x: auto;">
``` mermaid
sequenceDiagram
    %%{init: {'themeVariables': { 'fontSize': '20px' }, 'sequence': {'useMaxWidth': false} }}%%
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




