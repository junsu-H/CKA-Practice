# Backup and Restore Methods - Solution

# KEYWORD:
ps -aux | grep -i config /
ETCDCTL_API=3 etcdctl snapshot save -h

- 몇 개의 deployment가 있는가?

    ```bash
    kubectl get deploy

    # 2개
    ```

- etcd 버전은?

    ```bash
    etcdctl --version

    # 또는
    kubectl get all --all-namespaces | grep -i etcd
    kubectl describe pods etcd-master -n kube-system | grep -i etcd

    # 또는
    kubectl logs etcd-master -n kube-system | grep -i version
    ```

- 마스터 노드에서 어떤 주소로 ETCD 클러스터에 도달합니까?

    ```bash
    # listen-client-urls이 연결 주소

    kubectl describe pods etcd-master -n kube-system | grep -i listen-client-urls

    # --listen-client-urls=https://127.0.0.1:2379,https://172.17.0.51:2379
    ```

- ETCD 서버 인증서 파일은 어디에 있습니까?

    ```bash
    ps -aux | grep -i etcd | grep -i server.crt

    # 또는
    kubectl describe pod etcd-master -n kube-system | grep -i -A3 cert

    # /etc/kubernetes/pki/etcd/server.crt
    ```

- ETCD CA 인증서 파일은 어디에 있습니까?

    ```bash
    ps -aux | grep -i etcd | grep -i ca.crt

    # 또는
    kubectl describe pod etcd-master -n kube-system | grep -i ca

    # /etc/kubernetes/pki/etcd/ca.crt
    ```

- 내장 스냅샷 기능을 사용하여 ETCD 데이터베이스의 스냅샷을 작성하십시오 . 백업 파일은 /tmp/snapshot-pre-boot.db 위치에 저장하십시오.
    - Backup ETCD to /tmp/snapshot-pre-boot.db
    - Name: ingress-space

    ```bash
    etcdctl --version

    ETCDCTL_API=3 etcdctl snapshot save -h

    # --dry-run과 유사한 기능
    ETCDCTL_API=3 etcdctl member list \
        --name=ingrespp-space
        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
        --cert=/etc/kubernetes/pki/etcd/server.crt \
        --key=/etc/kubernetes/pki/etcd/server.key \
        --endpoints=https://127.0.0.1:2379
    # 4529f081e0aa996d, started, master, https://172.17.0.56:2380, https://172.17.0.56:2379

    # 정답
    ETCDCTL_API=3 etcdctl snapshot save /tmp/snapshot-pre-boot.db \
         --cacert=/etc/kubernetes/pki/etcd/ca.crt \
         --cert=/etc/kubernetes/pki/etcd/server.crt \
         --key=/etc/kubernetes/pki/etcd/server.key \
         --endpoints=https://127.0.0.1:2379

    # 상태 확인
    ETCDCTL_API=3 etcdctl --write-out=table snapshot \
        status /tmp/snapshot-pre-boot.db
    ```

    [mmumshad/kubernetes-the-hard-way](https://github.com/mmumshad/kubernetes-the-hard-way/blob/master/practice-questions-answers/cluster-maintenance/backup-etcd/etcd-backup-and-restore.md)

- 재부팅 후 마스터 노드가 온라인으로 돌아 왔지만 액세스 할 수있는 애플리케이션이 없습니다. 문제는?

    ```bash
    kubectl get pods,svc,deploy

    # All of the above 전부 다 문제
    ```

- 위의 백업 파일을 통해 복원
    - Deployments: 2
    - Services: 3

    ```bash
    ETCDCTL_API=3 etcdctl snapshot restore -h
    OPTIONS:
    # data-dir는 개발자 마음대로
    --data-dir=""                                             Path to the data directory

    # 특별한 거 없음. 
    --initial-advertise-peer-urls="http://localhost:2380"     List of this member's peer URLs to advertise to the rest of the cluster

    # default는 name과 일치시켜야 함.
    --initial-cluster="default=http://localhost:2380"         Initial cluster configuration for restore bootstrap

    # etcd-cluster-backup 이런 식으로 더 추가해야 됨. 
    --initial-cluster-token="etcd-cluster-backup"                    Initial cluster token for the etcd cluster during restore bootstrap

    # name 확인
    --name="default"                                          Human-readable name for this member

    # 옵션 확인을 위해서
    cd /etc/kubernetes/manifests
    vim etcd.yaml
    ```

    ```bash
    ETCDCTL_API=3 etcdctl snapshot restore /tmp/snapshot-pre-boot.db \
        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
        --cert=/etc/kubernetes/pki/etcd/server.crt \
        --key=/etc/kubernetes/pki/etcd/server.key \
        --endpoints=https://127.0.0.1:2379 \
        --data-dir="/var/lib/etcd-from-backup" \
        --initial-advertise-peer-urls="http://localhost:2380" \
        --initial-cluster="master=http://localhost:2380" \
        --initial-cluster-token="etcd-cluster-backup" \
        --name="master"
    ```

    ```bash
    cd /var/lib/etcd-from-backup/
    ls
    # member

    ps -aux | grep -i kubelet
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cd /etc/kubernetes/manifests
    vim etcd.yaml
    ```

    ```yaml
    # etcd.yaml

    apiVersion: v1
    kind: Pod
    metadata:  
      labels:
        component: etcd
        tier: control-plane
      name: etcd
      namespace: kube-system
    spec:
      containers:
      - command:
        - etcd
        - --advertise-client-urls=https://172.17.0.56:2379
        - --cert-file=/etc/kubernetes/pki/etcd/server.crt
        - --client-cert-auth=true
    #   - --data-dir=/var/lib/etcd
        - --data-dir=/var/lib/etcd-from-backup
        - --initial-cluster-token=etcd-cluster-backup 
        - --initial-advertise-peer-urls=https://172.17.0.56:2380
        - --initial-cluster=master=https://172.17.0.56:2380
        - --key-file=/etc/kubernetes/pki/etcd/server.key
        - --listen-client-urls=https://127.0.0.1:2379,https://172.17.0.56:2379
        - --listen-metrics-urls=http://127.0.0.1:2381
        - --listen-peer-urls=https://172.17.0.56:2380
        - --name=master
        - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
        - --peer-client-cert-auth=true
        - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
        - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
        - --snapshot-count=10000
        - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
        image: k8s.gcr.io/etcd:3.3.15-0
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 8
          httpGet:
            host: 127.0.0.1
            path: /health
            port: 2381
            scheme: HTTP
          initialDelaySeconds: 15
          timeoutSeconds: 15
        name: etcd
        resources: {}
        volumeMounts:
        - mountPath: /var/lib/etcd-from-backup
          name: etcd-data
        - mountPath: /etc/kubernetes/pki/etcd
          name: etcd-certs
      hostNetwork: true
      priorityClassName: system-cluster-critical
      volumes:
      - hostPath:
          path: /etc/kubernetes/pki/etcd
          type: DirectoryOrCreate
        name: etcd-certs
      - hostPath:
          path: /var/lib/etcd-from-backup
          type: DirectoryOrCreate
        name: etcd-data
    status: {}
    ```

    ```yaml
    watch "docker ps -a | grep -i etcd"
    # Up 될 때까지 대기
    ```

    ```yaml
    ETCDCTL_API=3 etcdctl member list \
         --cacert=/etc/kubernetes/pki/etcd/ca.crt \
         --cert=/etc/kubernetes/pki/etcd/server.crt \
         --key=/etc/kubernetes/pki/etcd/server.key \
         --endpoints=https://127.0.0.1:2379
    # 제대로 재생성 됐는지 확인

    kubectl get pods,svc,deploy
    ```

    [mmumshad/kubernetes-the-hard-way](https://github.com/mmumshad/kubernetes-the-hard-way/blob/master/practice-questions-answers/cluster-maintenance/backup-etcd/etcd-backup-and-restore.md)