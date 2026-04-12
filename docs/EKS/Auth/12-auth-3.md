# Managing Access for New Administrative Identities

## 1. Scenario: Onboarding a New DevOps Engineer

This lab demonstrates the process of creating a new IAM identity, granting it administrative permissions, and verifying its access from a clean environment. This simulates the real-world workflow of onboarding a new engineer who needs to manage AWS resources and EKS clusters.

### Step 1: Provisioning the IAM User and Credentials

We begin by creating a new IAM user and generating long-lived access keys. For the purpose of this lab, we will grant the user broad administrative permissions.

``` bash hl_lines="2 5 12" title="Create and authorize testuser"
# 1. Create the new IAM user identity
aws iam create-user --user-name testuser # (1)!

# 2. Generate a programmatic access key pair (ID and Secret)
aws iam create-access-key --user-name testuser # (2)!

# 3. Attach the AdministratorAccess managed policy
aws iam attach-user-policy --user-name testuser \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# 4. Verify your current identity (should still be 'admin')
aws sts get-caller-identity --query Arn # (3)!
```

1.  :octicons-code-review-16: **IAM User Creation**:
    ``` json
    {
        "User": {
            "UserName": "testuser",
            "Arn": "arn:aws:iam::080403789922:user/testuser",
            "CreateDate": "2026-04-12T03:21:20+00:00"
        }
    }
    ```
2.  :octicons-code-review-16: **Access Key Issuance**: Note that the `SecretAccessKey` is only displayed during this initial creation step.
    ``` json
    {
        "AccessKey": {
            "UserName": "testuser",
            "AccessKeyId": "AKIARFODQPBRLIHWKM56",
            "SecretAccessKey": "****************************************",
            "Status": "Active"
        }
    }
    ```
3.  :octicons-code-review-16: **Context Verification**: At this stage, your local environment is still utilizing the original `admin` credentials.
    ``` text
    "arn:aws:iam::080403789922:user/admin"
    ```

---

## 2. Simulating a Fresh Workstation via Docker

To verify the new credentials without affecting your local environment, we use a Docker container. This ensures that no existing AWS profiles or environment variables interfere with the test.

### Step 2: Initialize the AWS CLI Container

Depending on your local hardware architecture, use the appropriate command to launch an ephemeral AWS CLI environment.

``` bash hl_lines="4" title="Launch isolated AWS CLI environment"
# For standard x86_64 architectures
docker run -it --rm --name aws-cli --entrypoint /bin/sh amazon/aws-cli

# For Apple Silicon (M1/M2/M3) users (1)
docker run -it --rm --platform linux/amd64 --name aws-cli --entrypoint /bin/sh amazon/aws-cli
```

1.  :information_source: **Architecture Compatibility**: The `amazon/aws-cli` image may require the `--platform linux/amd64` flag on ARM-based Macs to avoid `glibc` errors related to specific CPU extension requirements in newer Amazon Linux versions.

### Step 3: Configure and Validate the New Identity

Inside the container, we configure the CLI with the newly created `testuser` credentials and perform validation.

``` bash hl_lines="2 12 15" title="Verify access within the container"
# 1. Confirm the environment is currently unauthenticated
aws s3 ls # (1)!

# 2. Configure the new credentials
aws configure
# AWS Access Key ID [None]: AKIARFO...
# AWS Secret Access Key [None]: kLSlQrVw...
# Default region name [None]: us-east-1
# Default output format [None]: json

# 3. Verify the assumed identity
aws sts get-caller-identity --query Arn # (2)!

# 4. Test resource visibility
aws s3 ls # (3)!
```

1.  :octicons-code-review-16: **Expected Failure**: Since the container is fresh, it has no credentials configured.
    ``` text
    aws: [ERROR]: An error occurred (NoCredentials): Unable to locate credentials.
    ```
2.  :octicons-code-review-16: **Successful Context Switch**:
    ``` text
    "arn:aws:iam::080403789922:user/testuser"
    ```
3.  :information_source: **Resource Access**: The command now succeeds. If no buckets exist in the account, it will return an empty result instead of an error, confirming that authentication and authorization are active.

---

## 3. Kubernetes API Access Challenge

Although the `testuser` has full **AdministratorAccess** within AWS, this does not automatically grant access to the Amazon EKS cluster's internal API.

### Step 4: Configure kubectl and Test Access

We will now install `kubectl` and attempt to communicate with the EKS cluster using the `testuser` credentials.

``` bash hl_lines="9 10 15" title="Configure kubectl tool"
# 1. Configure kubeconfig in the container
CLUSTER_NAME=myeks
aws eks update-kubeconfig --name $CLUSTER_NAME --user-alias testuser

# 2. Confirm kubeconfig
cat ~/.kube/config

# 3. Install kubectl
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.35.2/2026-02-27/bin/linux/arm64/kubectl # (1)!
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.35.2/2026-02-27/bin/linux/amd64/kubectl # (2)!
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client=true

# 4. Attempt to list cluster nodes
kubectl get node -v6 # (3)!

# 5. Install rbac-tool: https://github.com/alcideio/rbac-tool
yum install tar gzip -y
curl https://raw.githubusercontent.com/alcideio/rbac-tool/master/download.sh | bash

# 6. Check the identity currently authenticating with the cluster
./bin/rbac-tool whoami
# Error: Failed to create kubernetes client - the server has asked for the client to provide credentials
```

1. :information_source: **ARM64**: For ARM-based architectures (e.g., Apple Silicon).
2. :information_source: **AMD64**: For standard x86_64 architectures.
3.  :octicons-code-review-16: **Authentication Failure**:
    ``` text hl_lines="10"
    I0412 04:07:08.366786      91 loader.go:405] Config loaded from file:  /root/.kube/config
    ...
    I0412 04:07:20.219973      91 round_trippers.go:632] "Response" verb="GET" url="https://.../api?timeout=32s" status="401 Unauthorized" milliseconds=11852
    E0412 04:07:20.221306      91 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server has asked for the client to provide credentials"
    ```

!!! failure "Why did the request fail with HTTP 401 Unauthorized?"
    The **401 Unauthorized** error occurs because Amazon EKS uses a decoupled authentication model. While the `testuser` is successfully authenticated by AWS IAM, the EKS cluster itself does not recognize this IAM identity. In EKS, an IAM principal must be explicitly mapped to Kubernetes users/groups via **EKS Access Entries** or the `aws-auth` ConfigMap before it can access the Kubernetes API. Simply having `AdministratorAccess` in IAM is insufficient for EKS cluster-level access.
    ![iam-access-entries](../../assets/img/eks/12-auth-3/iam-access-entries.png)

!!! tip "Security Best Practice"
    In production environments, avoid using `AdministratorAccess` for individual users. Instead, use permission sets or roles that adhere to the Principle of Least Privilege (PoLP).

---

## 4. Resolving Access with EKS Access Entries

To resolve the authorization failure, we must explicitly register the `testuser` IAM principal within the EKS cluster and associate it with a cluster-level access policy.

### Step 5: Provisioning the EKS Access Entry and Policy Association

``` bash hl_lines="7 10 16 20" title="Registering the new identity in EKS"
# 1. Initialize environment variables
export CLUSTER_NAME=myeks
export ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text)

# 2. Create the Access Entry for testuser
aws eks create-access-entry --cluster-name $CLUSTER_NAME \
  --principal-arn arn:aws:iam::$ACCOUNT_ID:user/testuser # (1)!

# 3. Verify the creation of the entry
aws eks list-access-entries --cluster-name $CLUSTER_NAME | jq -r .accessEntries[] # (2)!

# 4. Associate the 'AmazonEKSAdminPolicy' with the user
aws eks associate-access-policy --cluster-name $CLUSTER_NAME \
  --principal-arn arn:aws:iam::$ACCOUNT_ID:user/testuser \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSAdminPolicy \
  --access-scope type=cluster # (3)!

# 5. Validate the policy association and entry details
aws eks list-associated-access-policies --cluster-name $CLUSTER_NAME \
  --principal-arn arn:aws:iam::$ACCOUNT_ID:user/testuser | jq # (4)!

aws eks describe-access-entry --cluster-name $CLUSTER_NAME \
  --principal-arn arn:aws:iam::$ACCOUNT_ID:user/testuser | jq
```

1.  :octicons-code-review-16: **Access Entry Metadata**:
    ``` json
    {
        "accessEntry": {
            "clusterName": "myeks",
            "principalArn": "arn:aws:iam::080403789922:user/testuser",
            "kubernetesGroups": [],
            "accessEntryArn": "arn:aws:eks:us-east-1:080403789922:access-entry/myeks/user/080403789922/testuser/...",
            "type": "STANDARD"
        }
    }
    ```
2.  :octicons-code-review-16: **Updated Entry List**:
    ``` text
    arn:aws:iam::080403789922:role/aws-service-role/eks.amazonaws.com/AWSServiceRoleForAmazonEKS
    arn:aws:iam::080403789922:role/myeks-ng-1
    arn:aws:iam::080403789922:user/admin
    arn:aws:iam::080403789922:user/testuser
    ```
    ![testuser-access-entry](../../assets/img/eks/12-auth-3/testuser-access-entry.png)
3.  :octicons-code-review-16: **Policy Association Confirmation**:
    ``` json
    {
        "clusterName": "myeks",
        "principalArn": "arn:aws:iam::080403789922:user/testuser",
        "associatedAccessPolicy": {
            "policyArn": "arn:aws:eks::aws:cluster-access-policy/AmazonEKSAdminPolicy",
            "accessScope": { "type": "cluster" }
        }
    }
    ```
    ![testuser-access-policy](../../assets/img/eks/12-auth-3/testuser-access-policy.png)
4.  :octicons-code-review-16: **Detailed Policy View**:
    ``` json
    {
      "associatedAccessPolicies": [
        {
          "policyArn": "arn:aws:eks::aws:cluster-access-policy/AmazonEKSAdminPolicy",
          "accessScope": { "type": "cluster" }
        }
      ],
      "clusterName": "myeks",
      "principalArn": "arn:aws:iam::080403789922:user/testuser"
    }
    ```

### Step 6: Verify Resolved Access

Return to the isolated Docker container and execute the previously failing command.

``` bash title="Verifying successful cluster interaction"
# Attempt to list cluster nodes again
kubectl get node -v6 # Can now access cluster resources
kubectl get deploy -A # Can access deployment information
kubectl auth can-i delete pods --all-namespaces # Returns 'yes'
```

!!! note "Resources for Further Reading"
    For a detailed breakdown of the permissions granted by the `AmazonEKSAdminPolicy`, refer to the [Official Amazon EKS Access Policy Documentation](https://docs.aws.amazon.com/eks/latest/userguide/access-policy-permissions.html#access-policy-permissions-amazoneksadminpolicy).

### Step 7: Clean Up Access Entry

``` bash title="Resetting the environment for the next section"
# Run from within the container or a host with admin credentials
aws eks delete-access-entry --cluster-name $CLUSTER_NAME --principal-arn arn:aws:iam::$ACCOUNT_ID:user/testuser
aws eks list-access-entries --cluster-name $CLUSTER_NAME | jq -r .accessEntries[]
```

---

## 5. Implementing Granular Access with Custom RBAC

While AWS-managed EKS access policies provide a convenient way to grant broad permissions, production environments often require more granular control. In this section, we will define custom Kubernetes `ClusterRoles` and map an IAM identity to specific Kubernetes groups using EKS Access Entries.

### Step 8: Define Custom ClusterRoles and Group Bindings

We will create two distinct roles: one for viewing pods and another for full administrative control over pods. These will then be bound to specific Kubernetes groups.

``` bash hl_lines="23 30" title="Creating custom RBAC resources"
# 1. Create ClusterRoles for different access levels
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-viewer-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-admin-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["*"]
EOF

# 2. Verify the created ClusterRoles
kubectl get clusterrole | grep ^pod # (1)!

# 3. Create ClusterRoleBindings to map Groups to Roles
kubectl create clusterrolebinding viewer-role-binding --clusterrole=pod-viewer-role --group=pod-viewer
kubectl create clusterrolebinding admin-role-binding  --clusterrole=pod-admin-role  --group=pod-admin

# 4. Verify the ClusterRoleBindings
kubectl get clusterrolebinding | grep -E 'admin-role|viewer-role' # (2)!
```

1.  :octicons-code-review-16: **ClusterRole Verification**:
    ``` text
    pod-admin-role                           2026-04-12T05:52:45Z
    pod-viewer-role                          2026-04-12T05:52:45Z
    ```
2.  :octicons-code-review-16: **Binding Confirmation**:
    ``` text
    admin-role-binding             ClusterRole/pod-admin-role        7s
    viewer-role-binding            ClusterRole/pod-viewer-role       11s
    ```

### Step 9: Auditing the RBAC Configuration

Using `rbac-tool` and `rolesum`, we can audit the current RBAC state to ensure the groups are correctly associated with the intended roles and permissions.

``` bash hl_lines="2 6" title="Inspecting RBAC subjects and policies"
# 1. Look up subjects associated with the custom groups
kubectl rbac-tool lookup pod-viewer # (1)!
kubectl rbac-tool lookup pod-admin

# 2. Summarize permissions for each group
kubectl rolesum -k Group pod-viewer # (2)!
kubectl rolesum -k Group pod-admin
```

1.  :octicons-code-review-16: **Subject Lookup**:
    ``` text
    SUBJECT    | SUBJECT TYPE | SCOPE       | NAMESPACE | ROLE            | BINDING              
    -------------+--------------+-------------+-----------+-----------------+----------------------
    pod-viewer | Group        | ClusterRole |           | pod-viewer-role | viewer-role-binding  
    ```
2.  :octicons-code-review-16: **Permission Summary**:
    ``` text
    Group: pod-viewer

    Policies:
    • [CRB] */viewer-role-binding ⟶  [CR] */pod-viewer-role
      Resource  Name  Exclude  Verbs  G L W C U P D DC  
      pods      [*]     [-]     [-]   ✔ ✔ ✔ ✖ ✖ ✖ ✖ ✖   
    ```

### Step 10: Mapping IAM Principal to Kubernetes Groups

We will re-create the Access Entry for `testuser`. Instead of an AWS-managed policy, we associate the user directly with the `pod-viewer` Kubernetes group.

``` bash hl_lines="4 8" title="Bridging IAM to custom K8s groups"
# 1. Re-create the Access Entry with custom group mapping
aws eks create-access-entry --cluster-name $CLUSTER_NAME \
  --principal-arn arn:aws:iam::$ACCOUNT_ID:user/testuser \
  --kubernetes-groups pod-viewer # (1)!

# 2. Verify the Access Entry configuration
aws eks describe-access-entry --cluster-name $CLUSTER_NAME \
  --principal-arn arn:aws:iam::$ACCOUNT_ID:user/testuser | jq .accessEntry.kubernetesGroups # (2)!
```

1.  :information_source: **Group Mapping**: By specifying `--kubernetes-groups`, EKS includes these group memberships in the identity token presented to the Kubernetes API server.
    ``` json
    {
        "accessEntry": {
            "clusterName": "myeks",
            "principalArn": "arn:aws:iam::080403789922:user/testuser",
            "kubernetesGroups": [
                "pod-viewer"
            ],
            ...
            "type": "STANDARD"
        }
    }
    ```
    ![access-entry-pod-viewer](../../assets/img/eks/12-auth-3/access-entry-pod-viewer.png)
2.  :octicons-code-review-16: **Group Verification**:
    ``` json
    [
      "pod-viewer"
    ]
    ```

### Step 11: Final Validation of Granular Access

Return to the isolated container and verify that `testuser` can perform actions allowed by the `pod-viewer` group, but remains restricted from other administrative tasks.

``` bash hl_lines="2 3 4" title="Testing custom group permissions"
# 1. Verify identity and group membership
kubectl auth can-i list pods # (1)!
kubectl auth can-i delete pods # (2)!
kubectl auth can-i get nodes # (3)!
```

1.  :octicons-check-circle-16: **Authorized**: Returns `yes` (Mapped via `pod-viewer-role`).
2.  :octicons-x-circle-16: **Unauthorized**: Returns `no` (Restricted to read-only verbs).
3.  :octicons-x-circle-16: **Unauthorized**: Returns `no` (Restricted to `pods` resource).

---

## 6. Elevating Permissions with Group Updates

In many scenarios, you may need to escalate or modify a user's permissions. Since we have already defined the `pod-admin` group and its associated `ClusterRole`, we can simply update the EKS Access Entry to reflect this new group membership.

### Step 12: Update Access Entry Group to pod-admin

We will now modify the existing Access Entry for `testuser`, switching their group membership from `pod-viewer` to `pod-admin`.

``` bash hl_lines="4 8" title="Updating EKS group membership"
# 1. Update the Access Entry with the pod-admin group
aws eks update-access-entry --cluster-name $CLUSTER_NAME \
  --principal-arn arn:aws:iam::$ACCOUNT_ID:user/testuser \
  --kubernetes-groups pod-admin # (1)!

# 2. Verify the updated configuration
aws eks describe-access-entry --cluster-name $CLUSTER_NAME \
  --principal-arn arn:aws:iam::$ACCOUNT_ID:user/testuser | jq .accessEntry.kubernetesGroups # (2)!
```

1.  :information_source: **Group Update**: The `update-access-entry` command allows you to modify the Kubernetes groups associated with a principal without re-creating the entry.
2.  :octicons-code-review-16: **Group Verification**:
    ``` json
    [
        "pod-admin"
    ]
    ```

### Step 13: Final Validation of Administrative Access

Finally, we verify that the `testuser` has successfully inherited the broader permissions granted to the `pod-admin` group.

``` bash hl_lines="2 5" title="Verifying elevated pod permissions"
# 1. Test administrative capabilities on pods
kubectl auth can-i delete pods --all-namespaces # (1)!

# 2. Verify continued restriction on other resources
kubectl auth can-i get nodes # (2)!

# 3. List pods across all namespaces
kubectl get pods -A
```

1.  :octicons-check-circle-16: **Authorized**: Now returns `yes`, confirming the switch to the administrative group.
2.  :octicons-x-circle-16: **Unauthorized**: Still returns `no`, as the `pod-admin-role` is strictly limited to the `pods` resource.

---

## 7. Resource Cleanup

To avoid unnecessary costs and maintain security hygiene, ensure all lab resources are decommissioned.

``` bash title="Cleanup commands"
# 1. Remove the isolated Docker environment
docker rm -f aws-cli

# 2. Delete the IAM test user (Ensure you are using admin credentials)
aws iam delete-access-key --user-name testuser --access-key-id <YOUR_ACCESS_KEY_ID>
aws iam detach-user-policy --user-name testuser --policy-arn arn:aws:policy/AdministratorAccess
aws iam delete-user --user-name testuser
```

---

## 8. Conclusion

In this lab, we explored the complete lifecycle of managing administrative access in Amazon EKS. We progressed from initial IAM identity creation to troubleshooting authentication failures, and ultimately implementing both AWS-managed and custom granular RBAC policies. By leveraging **EKS Access Entries** and **Kubernetes RBAC**, we demonstrated how to bridge AWS identities with internal cluster permissions, ensuring that security and accessibility are balanced according to the Principle of Least Privilege.