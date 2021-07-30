# Explore Kubernetes Environment - Solution

# KEYWORD: 
ip link /
arp /
route /
netstat -nplt

- 이 클러스터에 몇 개의 노드가 있습니까? 마스터 및 작업자 노드 포함

    ```bash
    kubectl get nodes --no-headers | wc -l

    # 4개
    ```

- 마스터 노드에서 클러스터 연결을 위해 구성된 네트워크 인터페이스는 무엇입니까? 노드 간 통신

    ```bash
    # master node ip 확인 172.17.0.63
    kubectl get nodes master -o wide

    ip addr | grep 172.17.0.63
    # ens3

    # ip link // hint인데 도움이 하나도 안 됨.
    ```

- 이 인터페이스에서 마스터 노드에 할당 된 IP 주소는 무엇입니까?

    ```bash
     kubectl get nodes master -o wide
    ```

- 마스터 노드에서 인터페이스의 MAC 주소는 무엇입니까?

    ```bash
    ip link show ens3
    ```

- node02에 할당 된 IP 주소는 무엇입니까?

    ```bash
    kubectl get nodes node02 -o wide

    # 172.17.0.87
    ```

- node02에 할당 된 MAC 주소는 무엇입니까?

    ```bash
    arp # IP 주소로 MAC 주소 아는 방법
    arp node02

    # 02:42:ac:11:00:57
    ```

- 컨테이너 런타임으로 Docker를 사용합니다. 이 호스트에서 Docker가 생성 한 인터페이스 / 브릿지는 무엇입니까?

    ```bash
    ip link

    # 또는
    brctl show

    # docker0
    ```

- 인터페이스 docker0의 상태는 무엇입니까?

    ```bash
    ip link show docker0 | grep -i state

    # DOWN
    ```

- 마스터 노드에서 Google을 핑하려면 어떤 경로를 사용합니까? 기본 게이트웨이의 IP 주소는 무엇입니까?

    ```bash
    route

    # 또는
    ip route

    # 또는
    ip route show default

    # default gateway 보면 됨.
    ```

- 마스터 노드에서 kube-scheduler가 수신 대기하는 포트는 무엇입니까?

    ```bash
    netstat -nlpt | grep -i kube-scheduler

    # -n 주소나 포트 형식을 숫자로 표현
    # -l LISTEN 하고 있는 포트를 보여줌.
    # -p 해당 프로세스를 사용하고 있는 프로그램 이름을 보여줌
    # -t TCP로 연결된 포트를 보여줌
    ```

- ETCD는 두 개의 포트에서 청취하고 있습니다. 이 중 더 많은 클라이언트 연결이 설정되어 있습니까?

    ```bash
    netstat -anop | grep etcd

    # -a 모든 연결 및 수신 대기 포트를 표시
    # -n 주소나 포트 형식을 숫자로 표현
    # -o PID를 표시
    # -p 해당 프로세스를 사용하고 있는 프로그램 이름을 보여줌
    ```

    2379는 모든 컨트롤 플레인 구성 요소가 연결되는 ETCD의 포트이기 때문입니다. 2380은 etcd 피어 투 피어 연결 전용입니다. 여러 마스터 노드가 있는 경우 이 경우에는 그렇지 않습니다.