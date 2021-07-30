# Mock Exam - 3 - Solution

- 이름이 `pvviewer` 인 service account를 만들어라.  
이름이 `pvviewer-role`인 ClusterRole, 이름이 `pvviewer-role-binding`인 ClusterRoleBinding을 만들고 service account에 모든 PersistentVolume을 볼 수 있는 `list` 권한을 부여하시오. 
그 후 기본 네임 스페이스에 이름이 `pvviewer`이고, 이미지가 `redis`, serviceAccount가 `pvviewer`인 파드를 만드시오.
`Weight: 12`
    - ServiceAccount: pvviewer
    - ClusterRole: pvviewer-role
    - ClusterRoleBinding: pvviewer-role-binding
    - Pod: pvviewer
    - Pod configured to use ServiceAccount pvviewer ?

    ```bash
    kubectl get nodes
    kubectl create serviceaccount pvviewer
    ```

    ```yaml
    # pvviewer-sa.yaml

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: pvviewer
    ```

    ```bash
    kubectl get sa
    ```

    [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

    ---

    ```bash
    kubectl create clusterrole pvviewer-role \
        --resource=persistentvolumes \
        --verb=list
    ```

    ```yaml
    # pvviewer-role.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: pvviewer-role
    rules:
    - apiGroups: [""]
      resources: ["persistentvolumes"] # 다 소문자임
      verbs: ["list"]
    ```

    ```bash
    kubectl get clusterrole
    ```

    [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

    ---

    ```bash
    kubectl create clusterrolebinding pvviewer-role-binding \
        --clusterrole=pvviewer-role \
        --serviceaccount=default:pvviewer \
        --dry-run -o yaml > pvviewer-role-binding.yaml

    # 정답
    kubectl create clusterrolebinding pvviewer-role-binding \
        --clusterrole=pvviewer-role \
        --serviceaccount=default:pvviewer
    ```

    ```yaml
    # pvviewer-role-binding.yaml

    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      creationTimestamp: null
      name: pvviewer-role-binding
    subjects:
    - kind: ServiceAccount
      name: pvviewer
      namespace: default
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: pvviewer-role
    ```

    ```bash
    kubectl create -f pvviewer-role-binding.yaml
    kubectl get clusterrolebinding
    ```

    [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

    ---

    ```bash
    kubectl run --generator=run-pod/v1 pvviewer \
        --image=redis \
        --dry-run \
        -o yaml > pvviewer-pod.yaml
    ```

    ```yaml
    # pvviewer-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: pvviewer
    spec:
      serviceAccountName: pvviewer
      containers:
      - name: pvviewer-con
        image: redis
    ```

    ```bash
    kubectl create -f pvviewer-pod.yaml
    kubectl get pods
    kubectl describe pods pvviewer | grep -i -A5 volume
    ```

- 클러스터의 모든 노드 `InternalIP`를 나열하시오. 결과는 /root/node_ips에 저장하시오. 
결과 포맷은 아래와 같다.
`InternalIP of master`<space> 
`InternalIP of node1`<space> 
`InternalIP of node2`<space> 
`InternalIP of node3`(한 줄에)
`Weight: 12`

    ```bash
    kubectl get nodes
    kubectl edit nodes

    kubectl get nodes -o jsonpath="{.items[].status.addresses}"

    # 정답1
    kubectl get nodes -o jsonpath=\
    "{.items[*].status.addresses[0].address}" \
    > /root/node_ips

    # 정답2, 따옴표 주의
    kubectl get nodes -o jsonpath=\
    '{.items[*].status.addresses[?(@.type=="InternalIP")].address}' \
    > /root/node_ips

    cat /root/node_ips
    ```

    [JSONPath Support](https://kubernetes.io/docs/reference/kubectl/jsonpath/)

- 이름이 `multi-pod`이고, 두 개의 컨테이너를 사용하는 파드를 만들어라.
컨테이너 1: 이름 `alpha`, 이미지 `nginx`
컨테이너 2: 이름 `beta`, 이미지 `busybox`, 명령어 `sleep 4800`
환경 변수 :
컨테이너 1 :`name: alpha`
컨테이너 2 :`name: beta
Weight: 12`
    - Pod Name: multi-pod
    - Container 1: alpha
    - Container 2: beta
    - Container beta commands set correctly?
    - Container 1 Environment Value Set
    - Container 2 Environment Value Set

    ```bash
    kubectl run --generator=run-pod/v1 alpha \
        --image=nginx \
        --dry-run \
        -o yaml > multi-pod.yaml

    vim multi-pod.yaml
    ```

    ```yaml
    # multi-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: multi-pod
    spec:
      containers:
      - name: alpha
        image: nginx
        env:
        - name: name
          value: alpha

      - name: beta
        image: busybox
        command: ["sleep", "4800"]
        env:
        - name: name
          value: beta
    ```

    ```bash
    kubectl create -f multi-pod.yaml
    kubectl get pods
    # multipod 2/2이어야 함.
    ```

    [Communicate Between Containers in the Same Pod Using a Shared Volume](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)

    [Define Environment Variables for a Container](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)

- 아래 설정과 함께 이름이`non-root-pod`이고, 이미지가 `redis:alpine`인 파드를 생성하시오.
runAsUser : 1000
fsGroup : 2000
`Weight: 8`
    - Pod `non-root-pod` fsGroup configured
    - Pod `non-root-pod` runAsUser configured

    ```yaml
    # non-root-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: non-root-pod
    spec:
      securityContext:
        runAsUser: 1000
    #    runAsGroup: 3000
        fsGroup: 2000
      containers:
      - name: non-root-con
        image: redis:alpine
    ```

    [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

- 파드 `np-test-1`와 `np-test-service`이라는 서비스가 배포됐다. 이 서비스로 들어오는 연결이 작동하지 않는다. 해당 이슈를 해결하시오.
이름이 `ingress-to-nptest`이고 `80` 포트를 통해  서비스에 들어오는 연결을 허용하는 NetworkPolicy를 만드시오.
중요: 배포된 현재 개체를 삭제하지 말 것.
`Weight: 14`
    - Important: Don't Alter Existing Objects!
    - NetworkPolicy: Applied to All sources (Incoming traffic from all pods)?
    - NetWorkPolicy: Correct Port?
    - NetWorkPolicy: Applied to correct Pod?

    ```yaml
    kubectl get pods
    kubectl describe pods np-test-1 
    # np-test-1 확인, labels이랑 image, name 확인

    kubectl get svc
    kubectl describe svc np-test-service
    # np-test-service 확인, ip와 port 확인
    ```

    ```bash
    kubectl run --generator=run-pod/v1 test-np \
        --image=busybox:1.28 \
        --rm -it \
        -- sh
    nc -z -v -w 2 np-test-service 80
    # nc: np-test-service (10.109.252.70:80): Connection timed out
    # -z connection을 이루기 위한 최소한의 데이터 외에는 보내지 않음
    # -v verbosity를 증가 시켜 더 많은 정보를 얻음. 없으면 connected time out 안 나옴.
    # -w 2 2초 동안 연결 안 되면 연결대기 종료.

    kubectl get netpol
    kubectl describe netpol default-deny
    kubectl api-versions | grep -i network
    # networking.k8s.io/v1
    ```

    ```yaml
    # ingress-to-nptest.yaml
    # 정답

    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: ingress-to-nptest
      namespace: default
    spec:
      podSelector:
        matchLabels:
          run: np-test-1
      policyTypes:
      - Ingress
      ingress:
      - ports:
        - protocol: TCP
          port: 80
    ```

    ```yaml
    # 오답

    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: ingress-to-nptest
      namespace: default
    spec:
      # 얘는 서비스하려는 파드
      podSelector:
        matchLabels:
          run: np-test-1
      policyTypes:
      - Ingress
      ingress:
      # 얘는 다른 파드, 생략하면 모든 파드가 들어올 수 있다.
      - from:
        - podSelector:
            matchLabels:
              run: np-test-1
        ports:
        - protocol: TCP
          port: 80
    ```

    ```bash
    # 확인
    kubectl run --generator=run-pod/v1 test-np \
        --image=busybox:1.28 \
        --rm -it \
        -- sh
    nc -z -v -w 2 np-test-service 80 # 포트도 적어줘야 함.
    # np-test-service (10.109.252.70:80) open
    ```

    [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

- 워커 노드 node01을 Unschedulable 되도록 Taint해라.
작업이 완료되면 이름이 `dev-redis`이고, 이미지가 `redis:alpine`인 파드를 생성해 node01에 스케줄되지 않도록 하라.
마지막으로 이름이 `prod-redis`이고, 이미지가  `redis:alpine` 파드를 생성해 node01에서 스케줄되도록 하라.
key: env_type, value:production, operator: Equal and effect:NoSchedule
`Weight: 12`
    - Key = env_type
    - Value = production
    - Effect = NoSchedule
    - pod 'dev-redis' (no tolerations) is not scheduled on node01?
    - Create a pod 'prod-redis' to run on node01

    ```bash
    kubectl get nodes
    kubectl taint nodes node01 env_type=production:NoSchedule
    kbuectl describe nodes | grep -i taint
    ```

    ```yaml
    # dev-redis-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: dev-redis
    spec:
      containers:
      - name: dev-redis-con
        image: redis:alpine 
    ```

    ```bash
    kubectl create -f dev-redis-pod.yaml
    kubectl get pods -o wide
    ```

    ---

    ```yaml
    # prod-redis-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: prod-redis
    spec:
      containers:
      - name: prod-redis-con
        image: redis:alpine 
      tolerations:
      - key: "env_type"
        value: "production"
        operator: "Equal"
        effect: "NoSchedule"
    ```

    ```bash
    kubectl create -f prod-redis-pod.yaml
    kubectl get pods -o wide
    ```

    ```bash
    taint는 tolerations 조건에 부합하는 파드를 해당 노드에 생성

      tolerations:
      - key: "env_type"
        value: "production"
        operator: "Equal"
        effect: "NoSchedule"
    ```

- 이름이 `hr-pod`이고, 네임 스페이스가 `hr`, 이미지가 `redis:alpine` environment: `production`, tier: `frontend`에 속하는  파드를 만들어라.
시스템에 존재하지 않는 경우 적절한 레이블을 사용하고 필요한 오브젝트를 모두 작성해야 한다.
`Weight: 8`
    - hr-pod labeled with environment production?
    - hr-pod labeled with frontend tier?

    ```bash
    kubectl get ns
    # hr namespace 없음

    kubectl create ns hr

    kubectl run --generator=run-pod/v1 hr-pod \
        --namespace=hr \
        --image=redis:alpine \
        --labels=environment=production,tier=frontend \
        --dry-run \
        -o yaml > hr-pod.yaml
    ```

    ```yaml
    # hr-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: hr-pod
    #  namespace: hr
    # namespace랑 labels랑 동시에 쓰면 안 됨.
      labels:
        environment: production
        tier: frontend
    spec:
      containers:
      - name: hr-con
        image: redis:alpine
    ```

    ```bash
    kubectl create -f hr-pod.yaml
    kubectl get pods -n hr
    ```

- kubeconfig 파일 `super.kubeconfig`이 /root에 작성되었다.
해당 kubeconfig 파일은 문제가 있다. 문제를 해결하시오.
`Weight: 8`
    - /root/super.kubeconfig 수정

    ```bash
    ls

    kubectl get nodes

    cat $HOME/.kube/config
    kubectl config view $HOME/.kube/config
    # server: https://172.17.0.109:6443
    kubectl config current-context
    # kubernetes-admin@kubernetes
    kubectl config get-clusters
    # NAME
    # kubernetes
    kubectl config get-contexts
    # CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
    # *         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin

    cat /root/super.kubeconfig
    # server 확인
    kubectl cluster-info --kubeconfig=/root/super.kubeconfig
    # Kubernetes master is running at https://172.17.0.109:2379

    vim /root/super.kubeconfig
    # 2379 ---> 6443 교체

    # 확인
    kubectl cluster-info --kubeconfig=/root/super.kubeconfig
    ```

    [Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config)

- 이름이 `nginx-deploy`인 deployment를 배포되었다. replicas를 3으로 변경하시오. 변경이 불가할 경우 트러블 슈팅을 통해 해결하시오.
`Weight: 14`
    - deployment has 3 replicas

    ```bash
    kubectl get deploy
    kubectl scale deploy nginx-deploy --replicas=3
    kubectl describe deploy nginx-deploy

    kubectl get pods --all-namespaces
    kubectl describe pods kube-contro1ler-manager-master -n kube-system

    kubectl logs kube-contro1ler-manager-master -n kube-system
    # Error from server (BadRequest): container "kube-contro1ler-manager" in pod "kube-contro1ler-manager-master"
    # is waiting to start: trying and failing to pull image
    ```

    ```bash
    ps -aux | grep -i config
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cd /etc/kubernetes/manifests
    ls
    vim kube-controller-manager.yaml
    :%s/contro1ler/controller/g # 숫자를 영어로
    ```

    ```bash
    kubectl get pods --all-namespaces
    watch "kubectl get deploy"
    # 3개 되는지 확인
    ```