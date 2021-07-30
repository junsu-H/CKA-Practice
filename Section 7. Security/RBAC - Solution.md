# RBAC - Solution

# KEYWORD:
kubectl get role /
kubectl get rolebinding /
kubectl create role -h /
kubectl create rolebinding -h

```bash
role - rolebinding vs. clusterrole - clusterrolebinding 차이점
role - rolebinding은 하나의 **네임스페이스**에서만 권한이 주어지지만,
clusterrole - clusterrolebinding은 하나의 **클러스터**에서 권한이 주어진다.

**serviceaccount는 계정을 생성하고,**
**role은 권한을 부여하고,
rolebinding은 해당 권한을 부여받은 유저를 맵핑

# sa - role - rolebinding - pod는 한 세트
# XXX-user는 rolebinding이랑 연관되어 있음.**
```

- 환경을 검사하고 클러스터에 구성된 권한 부여 모드를 식별하십시오. kube-apiserver 설정 확인

    ```bash
    ps -aux | grep -i kubelet
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cd /etc/kubernetes/manifests
    ls
    cat kube-apiserver.yaml  | grep -i mode

    # 또는
    kubectl get all --all-namespcaes
    kubectl describe pod/kube-apiserver-master -n kube-system | grep -i mode

    # --authorization-mode=Node,RBAC
    ```

- default 네임 스페이스에는 몇 개의 역할이 있습니까?

    ```bash
    kubectl get roles
    ```

- 모든 네임 스페이스에 몇 개의 역할이 있습니까?

    ```bash
    kubectl get roles --all-namespaces 
    kubectl get roles --all-namespaces --no-headers | wc -l

    # 13개
    ```

- kube-system에서 weave-net 역할

    ```bash
    kubectl get roles --all-namespaces
    kubectl describe roles weave-net -n kube-system

    # Resources쪽 보면 configmaps
    ```

- weave-net역할이 수행 할 수있는 작업configmaps

    ```bash
    kubectl describe roles weave-net -n kube-system

    # Verbs쪽 보면 create
    ```

- 다음 중 weave-net의 역할은?

    ```bash
    weave-net이라는 이름으로만 configmap을 보고 업데이트 가능
    ```

- 어떤 계정에 weave-net역할이 할당되어 있습니까?

    ```bash
    kubectl describe rolebinding weave-net -n kube-system

    # ServiceAccount  weave-net
    ```

- 사용자 `dev-user`가 생성됩니다. kubeconfig 파일에 사용자 세부 정보가 추가되었습니다. 사용자에게 부여 된 권한을 검사하십시오. 사용자가 기본 네임 스페이스에 포드를 나열 할 수 있는지 확인하십시오. `--as dev-user`kubectl과 함께 옵션을 사용하여 dev-user로 명령을 실행 하십시오.

    ```bash
    kubectl get pods **--as** dev-user

    # Error from server (Forbidden): pods is forbidden: User "dev-user" cannot list resource "pods" in API group "" in the namespace "default"
    # dev-user는 포드를 나열할 권한이 없습니다.
    ```

- `**default` 네임 스페이스에서 `dev-user` 포드를 생성, 나열 및 삭제 하는 데 필요한 필수 역할 및 역할 바인딩 을 만듭니다 .**

    주어진 사양을 사용하십시오

    - Role: developer
    - Role Resources: pods
    - Role Actions: list
    - Role Actions: create
    - RoleBinding: dev-user-binding
    - RoleBinding: Bound to dev-user

    ```bash
    **# create role
    kubectl create role developer \
        --resource=pods \
        --verb=list,create \**
        --dry-run

    **# create rolebinding
    kubectl create rolebinding dev-user-binding \
        --user=dev-user \
        --role=developer \**
        --dry-run
    ```

    ```yaml
    # developer-role.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: default
      name: developer
    rules:
    - apiGroups: [""] # "" indicates the core API group
      resources: ["pods"]
      verbs: ["list", "create"]
    ```

    ```yaml

    # dev-user-role-binding.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: dev-user-binding
      namespace: default
    subjects:
    - kind: User
      name: dev-user
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role 
      name: developer
      apiGroup: rbac.authorization.k8s.io
    ```

    ```bash
    kubectl create -f developer-role.yaml
    kubectl create -f dev-user-role-binding.yaml
    ```

    ```bash
    kubectl auth can-i get pods --as dev-user
    # no

    kubectl auth can-i create pods --as dev-user
    # yes

    kubectl auth can-i list pods --as dev-user
    # yes
    ```

    [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

- **★`dev-user`는 `blue` 네임 스페이스에서 dark-blue-app 포드에 대한 세부 정보를 얻으려고합니다. 문제를 조사하고 수정하십시오. 필요한 role과 rolebinding을 만들었지만 문제가 있는 것 같습니다.**

    ```bash
    kubectl auth can-i get pods/dark-blue-app -n blue --as dev-user
    # no

    kubectl describe role -n blue developer
    # Resource Name 확인
    # blue-app

    kubectl describe rolebinding -n blue dev-user-binding
    # 이상 없음.

    kubectl get roles developer -n blue -o yaml > developer.yaml
    cat developer.yaml | grep -i -A5 resourcename
    ```

    ```yaml
    # vim developer.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: developer
      namespace: blue
    rules:
    - apiGroups:  
      - ""
      resourceNames:
      - **dark**-blue-app
      resources:
      - pods
      verbs:
      - get  
      - watch
      - create
      - delete
    ```

    ```bash
    kubectl get roles -n blue
    kubectl delete roles developer -n blue
    kubectl create -f developer.yaml
    ```

    ```bash
    kubectl auth can-i get pods/dark-blue-app -n blue --as dev-user
    # yes
    ```

    [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

- `dev-user`에서 blue 네임 스페이스에 deployments를 create 권한을 부여하십시오. 
`"apps"`및`"extensions"` 두 그룹을 추가해야 합니다.

    ```bash
    # 이름은 상관 없음.

    k create role deploy-role -n blue \
        --resource=deployments \
        --verb=create
    k edit role deploy-role -n blue
    > apiGroup에 extensions 추가

    k create rolebinding dev-user-deploy-binding -n blue \
        --role=deploy-role \
        --user=dev-user
    ```

    ```yaml
    # deploy-role.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: deploy-role # 이름은 뭐든 상관 없음.
      namespace: blue
    rules:
    **- apiGroups: ["apps", "extensions"]**
      resources: ["deployments"]
      verbs: ["create"]
    ```

    ```yaml
    # dev-user-deploy-binding.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: dev-user-deploy-binding # 이름은 뭐든 상관 없음.
      namespace: blue
    subjects:
    - kind: User
      name: dev-user
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role 
      name: deploy-role
      apiGroup: rbac.authorization.k8s.io
    ```

    ```bash
    kubectl create -f deploy-role.yaml
    kubectl create -f dev-user-deploy-binding.yaml
    ```

    ```bash
    kubectl auth can-i create deployments -n blue --as dev-user
    # yes

    kubectl auth can-i create deployments.apps -n blue --as dev-user
    # yes

    kubectl auth can-i create deployments.**extensions** -n blue --as dev-user
    # no, why?
    ```

    [Authorization Overview](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)