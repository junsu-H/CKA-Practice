# Lightning Lab - 1 - Solution

# KEYWORD:
update /
custom-columns /
kubectl set /
ETCDCTL_API=3 etcdctl snapshot save -h

- `kubeadm`를 사용해 Kubernetes의 현재 버전 `1.16`에서 `1.17.0`로 업그레이드하시오. 마스터 노드부터 시작해서 한 번에 하나의 노드만 업그레이드해야 한다. 다운 타임을 최소화하려면 각 노드를 업그레이드하기 전에 대체 노드에 `gold-nginx` 파드를 배치해야 한다. `master` 노드를 먼저 업그레이드 하십시오. node01을 업그레이드하기 전에 배출하십시오. `gold-nginx`파드는 `master` 노드에서 실행되어야 한다.
`Weight: 15`

    ```bash
    # Master
    kubeadm version
    # GitVersion:"v1.16.0"

    apt-mark unhold kubeadm && \
    apt-get update && apt-get install -y kubeadm=1.17.0-00 -y && \
    apt-mark hold kubeadm

    kubectl drain master --ignore-daemonsets

    sudo kubeadm upgrade plan # 맨 아래에 있는 명령어가 kubeadm upgrade apply v1.17.0 -y

    apt-mark unhold kubelet kubectl && \
    apt-get update && apt-get install -y kubelet=1.17.0-00 kubectl=1.17.0-00 && \
    apt-mark hold kubelet kubectl

    sudo kubeadm upgrade apply v1.17.0 -y
    # [upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.17.0". Enjoy!
    # 이게 떠야 됨.

    # kubelet 재시작
    sudo systemctl restart kubelet

    kubectl uncordon master
    kubectl drain node01 --ignore-daemonsets
    ```

    ```bash
    # Node01
    ssh node01

    kubeadm version
    # GitVersion:"v1.16.0"

    apt-mark unhold kubeadm && \
    apt-get update && apt-get install -y kubeadm=1.17.0-00 -y && \
    apt-mark hold kubeadm

    apt-mark unhold kubelet kubectl && \
    apt-get update && apt-get install -y kubelet=1.17.0-00 kubectl=1.17.0-00 && \
    apt-mark hold kubelet kubectl

    kubeadm upgrade node --kubelet-version v1.17.0
    # [upgrade] The configuration for this node was successfully updated!
    # 이게 떠야 됨.

    sudo systemctl restart kubelet

    exit
    ```

    ```bash
    kubectl uncordon node01

    # Backup 확인

    # Master
    kubeadm version
    # GitVersion:"v1.17.0"

    ssh node01
    kubeadm version
    # GitVersion:"v1.17.0"
    exit

    kubectl get pods -o wide
    #(make sure this is scheduled on master node)
    ```

    [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

    [kubeadm upgrade](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/#cmd-upgrade-node)

- 네임 스페이스`admin2406`의 모든 deployment 이름을 다음 형식으로 인쇄하십시오.
`DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE
<deployment name> <container image used> <ready replica count> <Namespace>`. 
데이터는 `deployment name` 기준으로 오름차순으로 정렬
예 :`DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACEdeploy0 nginx:alpine 1 admin2406`
결과는 `/opt/admin2406_data`에 쓴다.
`Weight: 15`

    힌트 : 필요한 형식으로 데이터를 사용 `-o custom-columns`하고 `--sort-by`인쇄하십시오.

    ```bash
    kubectl get deploy -n admin2406 -o \
    custom-columns=\
    DEPLOYMENT:.metadata.name,\
    CONTAINER_IMAGE:.spec.template.spec.containers[*].image,\
    READY_REPLICAS:.spec.readyReplicas,\
    NAMESPACE:.metadata.namespace \
    --sort-by=.metadata.name > /opt/admin2406_data
    ```

    ```bash
    custom-columns=\

    # 이름(DEPLOYMENT), DEPLOYMENT은 변수명
    DEPLOYMENT:.metadata.name 

    # 이미지(CONTAINER_IMAGE), CONTAINER_IMAGE는 변수명
    CONTAINER_IMAGE:.spec.template.spec.containers[].image,\
    # CONTAINER_IMAGE:.spec.template.spec.containers[*].image 동일

    # REPLICAS 개수(READY_REPLICAS), READY_REPLICAS는 변수명
    READY_REPLICAS:.status.readyReplicas,\

    # 네임 스페이스(NAMESPACE), NAMESPACE는 변수명
    NAMESPACE:.metadata.namespace \

    --sort-by=.metadata.name > /opt/admin2406_data
    ```

    [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

    [Overview of kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)

- kubeconfig 파일 `admin.kubeconfig`이 /root에 작성되었습니다. 구성에 문제가 있습니다. 문제를 해결하고 수정하십시오.
`Weight: 8`

    ```bash
    # 이상 확인
    kubectl cluster-info --kubeconfig /root/admin.kubeconfig
    # Unable to connect to the server: x509: certificate signed by unknown authority

    kubectl get ep
    kubectl config view --kubeconfig=/root/admin.kubeconfig

    vim /root/admin.kubeconfig
    # https://172.17.0.53:6443로 포트 변경 후

    # 정상 동작 확인
    kubectl cluster-info --kubeconfig /root/admin.kubeconfig
    ```

- 이미지 `nginx:1.16`및  replicas `1` 을 사용하여 이름이  `nginx-deploy` 새 배포를 만듭니다. 그런 후 다음을 `1.17`사용하여 배포를 버전으로 업그레이드하십시오 `rolling update`. 버전 업그레이드가 리소스 주석에 기록되어 있는지 확인하십시오.
`Weight: 12`
    - Image: nginx:1.16
    - Task: Upgrade the version of the deployment to 1:17
    - Task: Record the changes for the image upgrade

    ```bash
    kubectl create deployment nginx-deploy --image=nginx:1.16 \
        --dry-run

    kubectl create deployment nginx-deploy --image=nginx:1.16

    kubectl create deployment nginx-deploy --image=nginx:1.16 \
        -o yaml --dry-run
    ```

    ```yaml
    # nginx:1.16-deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deploy
      name: nginx-deploy
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx-deploy
      strategy: {}  template:
        metadata:
          creationTimestamp: null
          labels:
            app: nginx-deploy
        spec:
          containers:
          - image: nginx:1.16
            name: nginx
            resources: {}
    status: {}
    ```

    ```yaml
    # kubectl set
    # 기존 응용 프로그램 리소스를 변경하는데 도움이 됩니다.

    # kubectl set image
    kubectl set image deployment/nginx-deploy nginx=nginx:1.17 \
        --record --dry-run

    kubectl set image deployment/nginx-deploy nginx=nginx:1.17 \
        --record

    kubectl set image deployment/nginx-deploy nginx=nginx:1.17 \
        --record -o yaml --dry-run
    ```

    ```yaml
    # nginx:1.17-deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      annotations:
        deployment.kubernetes.io/revision: "2"
        kubernetes.io/change-cause: kubectl set image deployment/nginx-deploy nginx=nginx:1.17
          --record=true --output=yaml --dry-run=true
      generation: 2
      labels:
        app: nginx-deploy  
        name: nginx-deploy
      namespace: default
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx-deploy
      strategy:
        rollingUpdate:
          maxSurge: 25%
          maxUnavailable: 25%
        type: RollingUpdate
      template:
        metadata:
          labels:
            app: nginx-deploy
        spec:
          containers:
          - image: nginx:1.17
            imagePullPolicy: IfNotPresent
            name: nginx
    ```

    [Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config)

- ★★★네임 스페이스 `alpha` 에 `alpha-mysql` 라는 새로운 deployment가 배포되었습니다. 
그러나 포드가 실행되고 있지 않습니다. 문제를 해결하십시오. 
배포시 `alpha-pv`마운트 될 PV를 사용해야하고 빈 루트 암호를 `/var/lib/mysql`사용하려면 환경 변수 `MYSQL_ALLOW_EMPTY_PASSWORD=1`를 사용해야합니다.
중요 : PV를 변경하지 마십시오.
`Weight: 20`

    ```bash
    kubectl get all -n alpha # pod, rs, deploy 3개 존재

    # 아래 두 개 이상 로그 없음.
    kubectl describe deploy alpha-mysql -n alpha 
    kubectl describe rs alpha-mysql-57d78bdbc6 -n alpha

    # 이상 로그 존재
    kubectl describe pods alpha-mysql-57d78bdbc6-7f9s9 -n alpha 
    # "mysql-alpha-pvc" not found

    # 1. .spec.claimRef.name 확인 /name, mysql-alpha-pvc
    # mysql-alpha-pvc가 필요한데 없다. -> 만들어야 된다.
    # 2. .spec.containers.env 확인
    # name: MYSQL_ALLOW_EMPTY_PASSWORD
    # value: 1
    kubectl edit pods alpha-mysql-57d78bdbc6-7f9s9 -n alpha 

    # 3. CAPACITY 확인, 1Gi (pvc보다 높아야 됨.)
    # 4. ACCESS MODES 확인, RWO (pvc와 동일해야 됨.)
    # 5. STORAGECLASS 확인, slow (pvc와 일치해야 됨.)
    kubectl get pv -n alpha
    kubectl edit pv alpha-pv -n alpha # 두 개 확인해야 됨.

    kubectl get pvc -n alpha
    kubectl describe pvc alpha-claim -n alpha
    # storageclass.storage.k8s.io "slow-storage" not found
    # 위 에러는 STORAGECLASS가 일치하지 않는다는 말임.

    kubectl get pvc -n alpha -o yaml > mysql-alpha-pvc.yaml
    vim mysql-alpha-pvc.yaml
    ```

    ```yaml
    # 1, 2, 3, 4, 5를 토대로 yaml 파일 작성
    # mysql-alpha-pvc.yaml
    # 필요 없는 거 다 지우면 됨.
     
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-alpha-pvc
      namespace: alpha
    spec:
      accessModes:
    # - ReadWriteMany에서 수정
      - ReadWriteOnce
      resources:
        requests:
    #     storage: 2Gi에서 수정
          storage: 1Gi
    # storageClassName: slow-storage에서 수정
      storageClassName: slow
    ```

    ```bash
    # environment variable MYSQL_ALLOW_EMPTY_PASSWORD=1
    # /var/lib/mysql
    # 위 두 개 확인해야 됨.

    kubectl get deploy -n alpha
    kubectl edit deploy alpha-mysql -n alpha

    # .spec.containers.volumeMounts[*].mountPath 
    # /var/lib/mysql인지 확인

    # .spec.containers.env[*].name 
    # MYSQL_ALLOW_EMPTY_PASSWORD인지 확인

    # .spec.containers.env[*].value 
    # 1인지 확인
    ```

- `/opt/etcd-backup.db` 마스터 노드의 위치 에서 ETCD 백업을 수행 하십시오.
`Weight: 10`

    ```bash
    # API 버전 확인
    etcdctl --version

    ETCDCTL_API=3 etcdctl snapshot save -h

    kubectl describe pod etcd-master -n kube-system | grep -i etcd

    # --cacert
    # --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

    # --cert
    # --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

    # --key
    # --key-file=/etc/kubernetes/pki/etcd/server.key

    # endpoints
    kubectl describe pods etcd-master -n kube-system | grep -i listen-client-urls
    ```

    ```bash
    ETCDCTL_API=3 etcdctl member list \
        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
        --cert=/etc/kubernetes/pki/etcd/server.crt \
        --key=/etc/kubernetes/pki/etcd/server.key \
        --endpoints=127.0.0.1:2379 \
        /opt/etcd-backup.db

    ETCDCTL_API=3 etcdctl snapshot save \
        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
        --cert=/etc/kubernetes/pki/etcd/server.crt \
        --key=/etc/kubernetes/pki/etcd/server.key \
        --endpoints=127.0.0.1:2379 \
        /opt/etcd-backup.db

    # 확인
    ETCDCTL_API=3 etcdctl --write-out=table snapshot status /opt/etcd-backup.db
    ```

- ★★★`busybox` 이미지를 사용하여 네임 스페이스 `admin1401` 에 이름이 `secret-1401`이고 파드를 만듭니다 . 
포드 내의 컨테이너 이름은 `secret-admin` 로 하며 `4800`초 동안 sleep 이여야 합니다 . 
컨테이너는 경로가 `/etc/secret-volume` 이고 `read-only` 로 마운트 시켜야 하며 이름이  `secret-volume`이다. 
시크릿은 이미 `dotfile-secret` 생성되었다.
`Weight: 20`

    ```bash
    kubectl get secret -n admin1401
    kubectl get all -n admin1401 --show-labels
    ```

    ```yaml
    # busybox-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
    #  labels:
    #    run: secret-1401
      name: secret-1401
      namespace: admin1401
    spec:
      containers:
      - name: secret-admin
        image: busybox
        command: ["sleep", "4800"]
        volumeMounts:
        - name: secret-volume
          readOnly: true
          mountPath: "/etc/secret-volume"
      volumes:
      - name: secret-volume
        secret:
          secretName: dotfile-secret
    ```

    [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

    [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)