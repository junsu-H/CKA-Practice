# Service Networking - Solution

# KEYWORD:
클러스터 네트워크 범위 -o wide /
파드 네트워크 범위 weave-net log /
서비스 네트워크 범위 kube-api

- 클러스터의 노드는 어떤 네트워크 범위입니까?

    ```bash
    kubectl get nodes -o wide
    # 172.17.0.55

    ip addr | grep 172.17.0.55

    # 172.17.0.55/16
    ```

- 이 클러스터에서 POD에 대해 구성된 IP 주소 범위는 무엇입니까?

    ```bash
    kubectl get pods --all-namespaces
    kubectl describe pods weave-net-dg548 -n kube-system
    kubectl logs weave-net-dg548 -c weave -n kube-system  | grep -i ip

    # ipalloc-range:10.32.0.0/12
    ```

- 클러스터 내 서비스에 대해 구성된 IP 범위는 무엇입니까?

    ```bash
    kubectl get svc --all-namespaces

    ps -aux | grep kube-api | grep -i ip
    # --service-cluster-ip-range=10.96.0.0/12
    ```

- 이 클러스터에 배포 된 kube-proxy 포드 수

    ```bash
    kubectl get pods -n kube-system | grep -i kube-proxy

    # 2개
    ```

- kube-proxy는 어떤 유형의 프록시를 사용하도록 구성되어 있습니까?

    ```bash
    kubectl logs kube-proxy-bwhn5 -n kube-system

    # iptables
    ```

- 이 Kubernetes 클러스터는 kube-proxy 포드가 클러스터의 모든 노드에서 실행되도록 어떻게 보장합니까? kube-proxy 포드를 검사하고 배치 방법을 식별하십시오.

    ```bash
    kubectl get pods -n kube-system
    kubectl describe pods kube-proxy-bvn5f -n kube-system | grep -i daemon

    # Controlled By:  DaemonSet/kube-proxy
    # using a daemonset
    ```