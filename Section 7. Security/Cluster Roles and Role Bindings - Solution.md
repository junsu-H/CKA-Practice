# Cluster Roles and Role Bindings

# KEYWORD:
kubectl get clusterroles /
kubectl get clusterrolebindings

```bash
role - rolebinding vs. clusterrole - clusterrolebinding 차이점
role - rolebinding은 하나의 네임스페이스에만 권한이 주어지지만,
clusterrole - clusterrolebinding은 하나의 클러스터에 권한이 주어진다.

serviceaccount는 계정을 생성하고,
clusterrole은 권한을 부여하고,
clusterrolebinding은 해당 권한을 부여받은 유저를 맵핑

# sa - clusterrole - clusterrolebinding - pod는 한 세트
```

- 클러스터에 몇 개의 ClusterRoles 가 정의되어 있습니까?

    ```bash
    kubectl get clusterroles --no-headers | wc -l

    # 55개
    ```

- 클러스터에 몇 개의 ClusterRoleBindings 가 있습니까?

    ```bash
    kubectl get clusterrolebindings --no-headers | wc -l

    # 43개
    ```

- cluster-admin클러스터 역할의 일부 네임 스페이스는 무엇입니까 ?

    ```bash
    kubectl get clusterroles cluster-admin
    kubectl describe clusterroles cluster-admin

    # clusterrole은 전체 클러스터이며 네임 스페이스의 일부가 아님
    ```

- 어떤 사용자/그룹이 `cluster-admin`역할에 바인딩되어 있습니까? 역할에 대한 ClusterRoleBinding의 이름이 동일합니다.

    ```bash
    kubectl get clusterrolebinding cluster-admin
    kubectl describe clusterrolebinding cluster-admin
    ```

    ```yaml
    Name:         cluster-admin
    Labels:       kubernetes.io/bootstrapping=rbac-defaults
    Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
    Role:
      Kind:  ClusterRole
      Name:  cluster-admin
    Subjects:
      Kind   Name            Namespace
      ----   ----            ---------
      Group  system:masters
    ```

- `cluster-admin`역할 은 어떤 수준의 권한 을 부여합니까?`cluster-admin`역할의 권한 검사

    ```bash
    kubectl describe clusterroles cluster-admin

    # 클러스터에 모든 자원 조치 실행
    ```

- 새로운 사용자 `michelle`가 팀에 합류했습니다. 그녀는 클러스터 `nodes`에 집중할 것 입니다. `nodes`에 액세스 할 수 있도록 필요한 ClusterRoles 및 ClusterRoleBindings를 작성하십시오. kubeconfig 파일이 자격 증명으로 구성되었습니다.

    ```bash
    # 이름은 뭐든 상관 없음.

    # kubectl config view

    # create clusterrole
    kubectl create clusterrole node-admin \
        --resource=nodes \
        --verb=list \
    #    --verb=get,watch,list,create,delete \
        --dry-run

    # create clusterrolebinding
    kubectl create clusterrolebinding michelle-binding \
        --user=michelle \
        --clusterrole=node-admin \
        --dry-run

    # user는 subject.name
    # clusterrole은 roleref.name
    ```

    ---

    ```yaml
    # name하고 verbs는 알아서 넣음.

    # node-admin-clusterrole.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: node-admin
    # "namespace" omitted since ClusterRoles are not namespaced
    rules:
    - apiGroups: [""]
      resources: ["nodes"] # 사용할 리소스
      verbs: ["list"]
    #  verbs: ["get", "watch", "list", "create", "delete"]
    ```

    ```yaml
    # michelle-clusterrole-binding.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: michelle-binding
    subjects:
    - kind: User
      name: michelle
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: ClusterRole
      name: node-admin # ClusterRole.metadata.name
      apiGroup: rbac.authorization.k8s.io
    ```

    ```bash
    kubectl create -f node-admin-clusterrole.yaml
    kubectl create -f michelle-clusterrole-binding.yaml
    ```

    ```bash
    kubectl auth can-i list nodes --as michelle
    # Warning: resource 'nodes' is not namespace scoped
    # yes
    ```

    [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

- `michelle`의 책임이 커지고 있으며 이제는 스토리지에 대한 책임도 있습니다. 스토리지에 액세스할 수 있도록 필요한 `ClusterRoles`및 `ClusterRoleBindings`를 작성하십시오. `kubectl api-resources` 명령에서 API 그룹 및 리소스 이름을 가져옵니다. 주어진 사양을 사용하십시오.
    - ClusterRole: storage-admin
    - Resource: persistentvolumes
    - Resource: storageclasses
    - ClusterRoleBinding: michelle-storage-admin
    - ClusterRoleBinding Subject: michelle
    - ClusterRoleBinding Role: storage-admin

    ```bash
    kubectl api-resources
    # persistentvolumes
    # storageclasses

    # create clusterrole
    kubectl create clusterrole storage-admin \
        --resource=nodes,persistentvolumes,storageclasses \
        --verb=get,watch,list,create,delete \
        --dry-run

    # create clusterrolebinding
    kubectl create clusterrolebinding michelle-storage-admin \
        --user=michelle \
        --clusterrole=storage-admin \
        --dry-run

    # user는 subject.name
    # clusterrole은 roleref.name
    ```

    ---

    ```yaml
    # storage-admin.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: storage-admin
    # "namespace" omitted since ClusterRoles are not namespaced
    rules:
    - apiGroups: [""]
      resources: ["nodes", "persistentvolumes", "storageclasses"] # 사용할 리소스
      verbs: ["get", "watch", "list", "create", "delete"]
    ```

    ```yaml
    # michelle-storage-admin.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: michelle-storage-admin
    subjects:
    - kind: User
      name: michelle
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: ClusterRole
      name: storage-admin # ClusterRole.metadata.name
      apiGroup: rbac.authorization.k8s.io
    ```

    ```bash
    kubectl create -f storage-admin.yaml
    kubectl create -f michelle-storage-admin.yaml
    ```

    [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)