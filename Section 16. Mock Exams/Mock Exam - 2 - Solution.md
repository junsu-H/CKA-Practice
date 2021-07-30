# Mock Exam - 2 - Solution

- etcd 클러스터를 백업하여 /tmp/etcd-backup.db에 저장하시오.
`Weight: 10`

    ```bash
    ps -aux | grep -i kubelet
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cd /etc/kubernetes/manifests
    cat etcd.yaml | grep -i etcd

    # 또는 
    kubectl describe pods etcd-master -n kube-system | grep -i etcd

    # cacert
    # --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    # --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt 

    # --cert
    # --cert-file=/etc/kubernetes/pki/etcd/server.crt

    # --key
    # --key-file=/etc/kubernetes/pki/etcd/server.key

    # --endpoints
    kubectl describe pods etcd-master -n kube-system | grep -i listen-client-urls
    # --listen-client-urls=https://127.0.0.1:2379,https://172.17.0.48:2379

    ```

    ```bash
    # 버전 확인
    ETCDCTL_API=3 etcdctl version

    # 도움말
    ETCDCTL_API=3 etcdctl snapshot save -h

    # --dry-run 같은 거
    ETCDCTL_API=3 etcdctl member list \
        --cacert="/etc/kubernetes/pki/etcd/ca.crt" \
        --cert="/etc/kubernetes/pki/etcd/server.crt" \
        --key="/etc/kubernetes/pki/etcd/server.key" \
        --endpoints=127.0.0.1:2379 \
        /tmp/etcd-backup.db

    # 정답
    ETCDCTL_API=3 etcdctl snapshot save \
        --cacert="/etc/kubernetes/pki/etcd/ca.crt" \
        --cert="/etc/kubernetes/pki/etcd/server.crt" \
        --key="/etc/kubernetes/pki/etcd/server.key" \
        --endpoints=127.0.0.1:2379 \
        /tmp/etcd-backup.db

    # 테이블 확인
    ETCDCTL_API=3 etcdctl snapshot status -w table \
        --cacert="/etc/kubernetes/pki/etcd/ca.crt" \
        --cert="/etc/kubernetes/pki/etcd/server.crt" \
        --key="/etc/kubernetes/pki/etcd/server.key" \
        --endpoints=127.0.0.1:2379 \
        /tmp/etcd-backup.db
    ```

    [Operating etcd clusters for Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

- 이름이 `redis:alpine`이고, 이미지가 `redis-storage` volume type이 `emptyDir`인 파드를 생성하시오.
`Weight: 10`
    - Pod named 'redis-storage' created
    - Pod 'redis-storage' uses Volume type of emptyDir
    - Pod 'redis-storage' uses volumeMount with mountPath = /data/redis

    ```yaml
    # redis-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: redis-storage
    spec:
      containers:
      - name: redis-con
        image: redis:alpine  
        volumeMounts:
        - mountPath: /data/redis
          name: redis-volume
      volumes:
      - name: redis-volume
        emptyDir: {}
    ```

    ```bash
    kubectl create -f redis-pod.yaml
    ```

    [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

- 이름이 `super-user-pod`이고, 이미지가  `busybox:1.28`이고 system_time을 설정하고 명령어가 sleep 4800인 파드를 만드시오.
`Weight: 8`
    - Pod: super-user-pod
    - Container Image: busybox:1.28
    - SYS_TIME capabilities for the conatiner?

    ```yaml
    # super-user-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: super-user-pod
    spec:
      containers:
      - name: super-user-con
        image: busybox:1.28
        command: ["sleep", "4800"]
        securityContext:
          capabilities:
            add: ["SYS_TIME"]
    ```

    ```bash
    kubectl create -f super-user-pod.yaml
    ```

    [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

- 파드 정의 파일이 `/root/use-pv.yaml`에 있다 . 이 파일을 사용하고 `pv-1`라는 PV를 마운트하시오. 파드가 실행 중이고 PV가 바인드되어 있는지 확인하시오.
mountPath : /data 
permanentVolumeClaim name : my-pvc
`Weight: 12`
    - persistentVolume Claim configured correctly
    - pod using the correct mountPath
    - pod using the persistent volume claim?

    ```bash
    kubectl get pv --show-labels
    # CLAIM 없음, LABELS 없음
    ```

    ```yaml
    # my-pvc.yaml

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: my-pvc
    spec:
      accessModes:
      - ReadWriteOnce # pv랑 일치
      volumeMode: Filesystem
      resources:
        requests:
          storage: 10Mi

    # 얘는 pv랑 pvc를 이어주는 매개체임.
    # pv가 Labels이 없으므로 없어야 됨.
    # 아래 추가하면 pvc는 Pending 상태임.
    # selector:
    #    matchLabels:
    #       run: use-pv
    ```

    ```bash
    kubectl create -f my-pvc.yaml
    kubectl get pvc
    kubectl get pv
    # CLAIM default/my-pvc로 할당.

    vim use-pv.yaml
    ```

    ```yaml
    # use-pv.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null  
      labels:
        run: use-pv
      name: use-pv
    spec:
      containers:
      - image: nginx
        name: use-pv
        # pv 영역
        volumeMounts:
        - mountPath: /data
          # pv 이름
          name: pv-1

      # pv 영역
      volumes:  
        # pv 이름
      - name: pv-1
        # pvc 영역
        persistentVolumeClaim:
          claimName: my-pvc
    ```

    ```bash
    kubectl create -f use-pv.yaml

    kubectl describe pods use-pv | grep -i -A4 volumes
    # Volumes:
    #  pv-1:
    #    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    #    ClaimName:  my-pvc
    #    ReadOnly:   false
    ```

- 이름이 `nginx-deploy`이고, 이미지가 `nginx:1.16`이고 replicas `1`인 deployment를 만드시오. 만들 때 버전을 기록하시오. 이후 롤링 업데이트를 사용하여 `nginx:1.17`로 deployment를 업그레이드하시오. 버전 업그레이드가 기록되어 있는지 확인하시오.
`Weight: 15`
    - Deployment : nginx-deploy. Image: nginx:1.16
    - Image: nginx:1.16
    - Task: Upgrade the version of the deployment to 1:17
    - Task: Record the changes for the image upgrade

    ```bash
    kubectl create deploy nginx-deploy --image=nginx:1.16 -o yaml --record --dry-run
    kubectl create deploy nginx-deploy --image=nginx:1.16

    # 만약에 replicas를 2 이상으로 수정해야 된다면
    # kubectl edit deploy nginx-deploy에 들어가서 replicas: 2 수정

    # 또는
    # kubectl scale deploy nginx-deploy  --replicas=2 
    ```

    ```bash
    # rollupdate 확인
    kubectl rollout history deploy nginx-deploy
    ```

    ```bash
    kubectl set -h
    kubectl set image \
        [pod,rs,deploy] \
        [pod,rs,deploy name] \
        [container name]=[update할 image] \
        --record \
        --dry-run

    kubectl set image \
        deploy \
        nginx-deploy \
        nginx=nginx:1.17 \
        --record
    ```

    ```bash
    # rollupdate 확인
    kubectl rollout history deploy nginx-deploy

    # 이미지가 변경 되었는지 확인

    kubectl get deploy
    kubectl describe deploy nginx-deploy | grep -i image
    # nginx:1.17

    kubectl get pods
    kubectl describe pods nginx-deploy-5b7764d7c6-gc2l4 | grep -i image
    # nginx:1.17
    ```

- `john`라는 새 사용자를 만들고 클러스터에 대한 액세스 권한을 부여하시오.  `john`은 네임 스페이스가 `development`에서  `create, list, get, update and delete pods`에 대한 권한이 있어야 한다. 개인 키는  `/root/john.key`  CSR는 `/root/john.csr`  위치에 존재한다.
`Weight: 15`
    - CSR: john-developer Status:Approved
    - Role Name: developer, namespace: development, Resource: Pods
    - Access: User 'john' has appropriate permissions

    ```bash
    ls -al john.*
    kubectl api-versions | grep certif
    # certificates.k8s.io/v1beta1

    cat john.csr | base64 | tr -d '\n' > csr.txt

    # 두 개 같은지 비교
    cat john.csr
    base64 -d csr.txt
    ```

    ```yaml
    # john-csr.yaml

    apiVersion: certificates.k8s.io/v1beta1
    kind: CertificateSigningRequest
    metadata:
      name: john-developer
      # csr은 namespace 없음.
    spec:
      # cat john.csr | base64 | tr -d '\n'
      request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUx4Ung1NzhGbXJLZmo0T2NkME1ZMGNtcGxRRUQ1VkZmWE9KNnpiQ1RHc3dLOGFxCnJBMzgrbDk3cTJ3SXJXSHlCMXp2R2RxeDAwSERzVGdyMjVMT3lobHJNa0VMMW9USGVVamVSVGIvVnpDY0ZROHoKcjUvVFIwRThFR3R2Y3phVjU0cHdqUmIvZVNDQW5nM1hEb3NlS0R3Uk1qV0RubTVFRnhXTCtNQ0dJcW50dkx5SgpXc1pPanNlSG5jN2lQOGh0aFdVTlpUSTBPYmtZb2lJVXphZnJrYlhWQzhYSUxSWGh5T09DQUJxMnZlelQ5L0grCk9TMU1saDJPRlNvc3cvbkdPYUlnVXA5bXZ6d0RzZTBLZ0p4c010NU9zMXZtZnRhbDJiUzdLMnR0UWhqdnlWT1YKcHVSaFhMQjI4Wno3RGZhVDFVNHBsbXRMNnFjaUYzaEFnWU1Qd05rQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQTBpODhJRTVNWUs1eXRSVTB2MzFRTXlpMUtVS3FCcWhqaG1ZNnhlTFFtY1RXR2ZBMW1sMVExCjJ6OGVvU3NlcWQ5S2hMV0V5TVVIWnVFUE41Skw5TmthT2h2RmhUKzNsK2ZnY000RCt0NU1PdUdveCtScG9uMnEKanJHMVVpMzgrQ3ZmVTlVSk50ZStDMHFGNXVtZGl4UmlzQW5ERDVkWU0zUEkrSHVPUUY4VnlsVWFia3BYV0R1NQpmQkFMWk9obVRSSzV3Rnk1dS9xTmFBYU9ZVCt5RTBMQXZhWTZ2cVZZT1pRY09zend3N3RkR1RRUElEWWNUWUFoCml6dndRS3VmMUZuQ25xUk5KbzREc3c0c2dFcVp4VWVMZ3NFc0gxdHUzcDhpdllQSExmelBkb0RHSHFsaFllVmsKQXVIVkJGYjFieTJsMmRFUTVQcFN5ZWpZbm1KdDlxb04KLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
      usages:
      - digital signature
      - key encipherment
      - server auth
    ```

    ```yaml
    kubectl create -f john-csr.yaml

    kubectl get csr
    # CONDITION Pending

    kubectl certificate approve john-developer
    # CONDITION Approve
    ```

    [Manage TLS Certificates in a Cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)

    ---

    ---

    ---

    ```bash
    kubectl create role \
        developer \
        --resource=pods \
        --verb=create,list,get,update,delete \
        --namespace=development

    kubectl create rolebinding \
        developer-role-binding \
        --role=developer \
        --user=john \
        --namespace=development
    ```

    ```yaml
    # developer-role.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: developer
      namespace: development
    rules:
    - apiGroups: [""] # "" indicates the core API group
      resources: ["pods"]
      verbs: ["create", "list", "get", "update", "delete"]

    ```

    ```yaml
    # # developer-role-binding.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: developer-role-binding
      namespace: development
    subjects:
    - kind: User
      name: john # "name" is case sensitive
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role #this must be Role or ClusterRole
      name: developer # kubectl get role 해서 나온 role name
      apiGroup: rbac.authorization.k8s.io
    ```

    ```bash
    kubectl create -f developer-role.yaml
    kubectl create -f developer-role-binding.yaml

    # 제대로 생성되었는지 확인
    kubectl describe \
        role.rbac.authorization.k8s.io/developer \
        rolebinding.rbac.authorization.k8s.io/read-pods \
        -n development

    # john이 접근 가능한지 확인
    kubectl auth can-i update pods
    kubectl auth can-i update pods --as=john

    # 구체적으로 확인, 전부 yes
    kubectl auth can-i create pods -n development --as=john
    kubectl auth can-i list pods -n development --as=john
    kubectl auth can-i get pods -n development --as=john
    kubectl auth can-i update pods -n development --as=john
    kubectl auth can-i delete pods -n development --as=john

    # 얘는 no
    kubectl auth can-i watch pods -n development --as=john
    ```

    ```bash
    # csr는 role이랑 세트
    # role은 rolebinding이랑 세트
    # 결론적으로 인증이 필요한 사람이 있다는 건
    # csr, role, rolebinding 세 개를 만들어야 함.
    ```

- 이름이 `nginx-resolver` 이고, 이미지가 `nginx` 파드를 expose 시킨다. 서비스 이름은 `nginx-resolver-service` 이다. 클러스터 내에서 서비스 및 포드 이름을 찾을 수 있는지 테스트하시오. dns 조회할 때 `busybox:1.28` 이미지를 사용하시오. dns 조회 결과를 `/root/nginx.svc`와`/root/nginx.pod`에 저장하시오.
`Weight: 15`
    - Pod: nginx-resolver created
    - Service DNS Resolution recorded correctly
    - Pod DNS resolution recorded correctly

    ```bash
    kubectl run --generator=run-pod/v1 nginx-resolver \
        --image=nginx --dry-run

    kubectl run --generator=run-pod/v1 nginx-resolver \
        --image=nginx

    kubectl expose pods nginx-resolver \
        --name=nginx-resolver-service \
        --port=80
    #    --target-port=80 \
    #    --type=ClusterIP

    kubectl get svc -o wide
    # CLUSTER-IP 10.101.230.125

    kubectl describe svc nginx-resolver-service
    # 한 번 더 IP 확인
    ```

    ```bash
    kubectl get pods nginx-resolver -o wide
    # nginx-resolver 10.44.0.4

    # 또는
    kubectl get ep
    # 10.44.0.4

    # 정답1
    # --rm은 실행 후 test-nslookup pod 삭제
    kubectl run --generator=run-pod/v1 test-nslookup1 \
        --image=busybox:1.28 \
        --rm -it -- nslookup nginx-resolver-service > /root/nginx.svc

    #   아래도 가능
    #   --rm -it -- nslookup nginx-resolver-service.default \
    #   --rm -it -- nslookup nginx-resolver-service.default.cluster.local \

    # 정답2
    kubectl run --generator=run-pod/v1 test-nslookup2 \
        --image=busybox:1.28 \
        --rm -it \
        -- nslookup 10.44.0.4 > /root/nginx.pod

    #   아래도 가능
    #   -- nslookup 10-44-0-4.default.pod
    #   -- nslookup 10-44-0-4.default.pod.cluster.local

    # 조금 기다리면 됨.

    cat /root/nginx.svc
    cat /root/nginx.pod
    ```

    [Exposing an External IP Address to Access an Application in a Cluster](https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/)

- 이름이 `nginx-critical` 이고, 이미지가  `nginx` 인 파드를  `node01` 에 정적 파드로 만든다 . `node01` 에서 파드를 작성하고 실패시 자동으로 다시 작성/시작하시오. 정적 파드의 경로는 `/etc/kubernetes/manifests` 이다.
`Weight: 15`
    - Kubelet Configured for Static Pods
    - Pod nginx-critical-node01 is Up and running

    ```bash
    # master에서 작성하는 거 아님!!!

    ssh node01

    systemctl status kubelet
    # --config=/var/lib/kubelet/config.yaml

    ps -aux | grep -i config
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cd /etc/kubernetes/
    mkdir manifests
    cd manifests
    vim nginx-critical-pod.yaml
    ```

    ```yaml
    # nginx-critical-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-critical
    spec:
      containers:
      - name: nginx-critical
        image: nginx
    #  nodeName: node01
    ```

    ```bash
    docker ps | grep -i nginx-critical
    kubectl get pods
    ```