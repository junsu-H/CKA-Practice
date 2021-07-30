# Worker Node Failure - Soultion

# KEYWORD:
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf /
systemctl status kubelet.service -l /
journalctl -u kubelet /
systemctl daemon-reload /
systemctl restart kubelet /
kubectl cluster-info

- 손상된 클러스터 수정

    ```bash
    kubectl get nodes

    ssh node01

    journalctl -u kubelet
    # -u 특정 유닛의 로그 확인

    # 아무 것도 없음. 즉 kubelet이 실행되어있지 않다.
    ps -aux | grep -i kubelet

    systemctl status kubelet.service -l
    # Active: inactive (dead)

    systemctl restart kubelet
    systemctl status kubelet.service -l

    kubectl get nodes
    ```

    [Troubleshoot Clusters](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)

- 클러스터가 다시 끊어졌습니다. 문제를 조사하고 수정하십시오.

    ```bash
    kubectl get nodes
    kubectl describe node nodes # node01 또 문제
    ssh node01

    systemctl status kubelet.service -l # 문제 없음.

    journalctl -u kubelet | grep -i unable
    # 스페이스 누르면 다음으로
    ```

    ```bash
    # journalctl -u kubelet 결과
    unable to load client CA file /etc/kubernetes/pki/WRONG-CA-FILE.crt: open /etc/kubernetes/pki/WRONG-CA-FILE.crt: no such file or directory
    ```

    ```bash
    # systemctl status kubelet.service
    # Drop-In: /etc/systemd/system/kubelet.service.d
    #          └─10-kubeadm.conf
    cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
    # Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
    # 경로는 아래 문서 참고

    vim /var/lib/kubelet/config.yaml
    >
    x509:
      clientCAFile: /etc/kubernetes/pki/ca.crt 수정
    ```

    ```bash
    systemctl daemon-reload
    systemctl restart kubelet
    exit
    kubectl get nodes
    ```

    [Configuring each kubelet in your cluster using kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/kubelet-integration/)

- 클러스터가 다시 끊어졌습니다. 문제를 조사하고 수정하십시오.

    ```bash
    kubectl get nodes # node01 NotReady
    ssh node01

    systemctl status kubelet.service -l
    # dial tcp 172.17.0.9:6553: connect: connection refused

    journalctl -u kubelet | grep -i unable
    # dial tcp 172.17.0.9:6553: connect: connection refused
    # journalctl -u kubelet -f

    exit
    kubectl cluster-info
    # kubectl cluster-info 결과
    # Kubernetes master is running at https://172.17.0.82:6443
    # KubeDNS is running at https://172.17.0.82:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    # To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

    ssh node01
    service kubelet status
    cd /etc/systemd/system/kubelet.service.d
    cat 10-kubeadm.conf
    # --kubeconfig=/etc/kubernetes/kubelet.conf

    vim /etc/kubernetes/kubelet.conf
    # server: https://172.17.0.82:6553 ---> server: https://172.17.0.82:6443 수정
    ```

    ```bash
    systemctl daemon-reload
    systemctl restart kubelet
    service kubelet status
    exit
    kubectl get nodes
    ```