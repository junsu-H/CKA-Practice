# Networking Weave - Solution

# KEYWORD:
/etc/cni/net.d/ /
ip route

- 이 클러스터에 몇 개의 노드가 있습니까? 마스터 및 작업자 노드 포함

    ```bash
    kubectl get nodes --no-headers | wc -l

    # 4개
    ```

- 이 클러스터에서 사용되는 네트워킹 솔루션은 무엇입니까?

    ```bash
    cat /etc/cni/net.d/10-weave.conflist

    # /etc/cni/net.d default 경로임.
    # name: weave
    ```

    [Network Plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)

- 이 클러스터에 몇 개의 weave agents/peers가 배치되어 있습니까?

    ```bash
    kubectl get all --all-namespaces | grep -i weave
    kubectl get pods -n kube-system | grep -i weave | wc -l

    # 4개
    ```

- weave agents/peers는 어느 노드에 있습니까?

    ```bash
    kubectl get pods -n kube-system -o wide | grep -i weave

    # 모든 노드에 하나씩
    ```

- 각 노드에서 weave로 생성된 브리지 네트워크/인터페이스 이름은?

    ```bash
    ip link
    brctl show
    # weave
    ```

- weave로 구성된 POD IP 주소 범위는 무엇입니까?

    ```bash
    ip addr show weave

    ip addr | grep -i weave
    ```

- node03에 예약된 POD에 구성된 기본 게이트웨이는 무엇입니까? node03에서 포드를 예약하고 ip route을 확인하십시오.

    ```yaml
    # busybox-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: busybox
      labels:
        app: busybox
    spec:
      containers:
      - image: busybox
        command:
          - sleep
          - "3600"
        name: busybox
      nodeName: node03
    ```

    ```bash
    kubectl create -f busybox-pod.yaml
    kubectl get pods
    kubectl exec -it busybox -- ip route

    # 10.36.0.0
    ```