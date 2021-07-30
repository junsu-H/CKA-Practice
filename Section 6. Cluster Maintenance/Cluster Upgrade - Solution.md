# Cluster Upgrade - Solution

# KEYWARD: 
kubeadm upgrade plan /
kubeadm upgrade apply

- 현재 클러스터 버전은?

- 클러스터에 몇 개의 노드가 있나?

    ```bash
    kubectl get nodes
    ```

- 몇 개의 노드가 작업 부하를 호스팅 할 수 있나?

    ```bash
    kubectl get pods -o wide

    # master, node02 2개
    ```

- 클러스터에서 몇 개의 응용 프로그램이 호스팅됩니까? deployment

    ```bash
    kubectl get deploy

    # 1개
    ```

- pod가 어느 nodes에 할당되는가?

    ```bash
    kubectl get pods -o wide

    # master, node01
    ```

- 클러스터를 업그레이드해야합니다. 사용자의 응용 프로그램 액세스가 영향을 받아서는 안 됩니다. 그리고 새로운 VM을 프로비저닝 할 수 없습니다. 클러스터를 업그레이드하기 위해 어떤 전략을 사용 하시겠습니까?

    ```bash
    워크로드를 다른 워크로드로 이동하면서 한 번에 한 노드씩 업그레이드
    ```

- 업그레이드 가능한 최신 안정 버전은 무엇입니까? kubeadm

    ```bash
    kubeadm upgrade plan

    # 가장 아래에 있는 거
    # kubeadm upgrade apply v1.17.4
    ```

    [kubeadm upgrade](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/)

- 현재 버전에서 최신 안정 버전으로 업그레이드하려면 어떻게 해야 합니까?

    ```bash
    최신 버전까지 한 번에 하나씩 버전 업그레이드
    ```

- 먼저 마스터 노드를 업그레이드 할 것입니다. 워크로드의 마스터 노드를 비우고 표시 UnSchedulable

    ```bash
    # kubectl drain은 노드 관리를 위해서 
    # 지정된 노드에 있는 포드들을 다른곳으로 이동시키고 
    # UnSchedulable 시킵니다.

    # kubectl cordon은 지정된 노드에 더 이상 포드들이 
    # UnSchedulable 시킵니다.

    # drain, cordon 둘 다 스케줄링 안 됨.

    kubectl drain master --ignore-daemonsets

    # master에 할당된 pod 있는지 확인
    kubectl get pods -o wide

    kubectl get nodes -o wide   
    # master status가 Ready,SchedulingDisabled인지 확인
    ```

- 마스터 구성 요소를 정확한 버전으로 업그레이드 v1.17.0
    - Master Upgraded to v1.17.0
    - Master Kubelet Upgraded to v1.17.0

    ```bash
    # 패키지를 우선 전부 다운로드한 후에 upgrade 적용

    # kubeadm version 확인
    kubeadm version

    # master node unschedulable
    kubectl drain master --ignore-daemonsets

    # 업그레이드 가능한 최신 안정 버전 확인
    kubeadm upgrade plan

    # kubeadm 패키지 다운로드
    apt-mark unhold kubeadm && \
    apt-get update && apt-get install -y kubeadm=1.17.0-00 && \
    apt-mark hold kubeadm

    # kubelet 패키지 다운로드
    apt-mark unhold kubelet kubectl && \
    apt-get update && apt-get install -y kubelet=1.17.0-00 kubectl=1.17.0-00 && \
    apt-mark hold kubelet kubectl

    # 업그레이드 적용, 좀 오래 기다려야 됨.
    kubeadm upgrade apply v1.17.0
    # [upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.17.0". Enjoy!

    # 위 명령어 안 되면
    kubeadm upgrade apply v1.17.0 \
        --ignore-preflight-errors=ControlPlaneNodesReady

    kubectl get nodes
    ```

    [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

- 마스터 노드 Schedulable 표시

    ```bash
    kubectl uncordon master
    kubectl get nodes -o wide
    ```

- node01 Drain작업 부하의 작업자 노드 및 표시UnSchedulable

    ```bash
    # kubectl drain은 노드 관리를 위해서 
    # 지정된 노드에 있는 포드들을 다른곳으로 이동시키는 명령입니다.

    # kubectl cordon은 지정된 노드에 더 이상 포드들이 
    # 스케쥴링되서 실행되지 않도록 합니다.
    # drain, cordon 둘 다 실행하면 스케줄링 안 됨.

    kubectl drain node01 --ignore-daemonsets

    # kubectl get nodes -o wide   node01 status 확인
    # kubectl cordon node01
    ```

- 작업자 노드를 정확한 버전으로 업그레이드 v1.17.0
    - Worker Node Upgraded to v1.17.0
    - Worker Node Ready

    ```bash
    # ssh 연결
    ssh node01

    # kubeadm version 확인
    kubeadm version

    # 업그레이드 판단 여부 확인
    kubeadm upgrade plan

    # kubeadm 패키지 다운로드
    apt-mark unhold kubeadm && \
    apt-get update && apt-get install -y kubeadm=1.17.0-00 && \
    apt-mark hold kubeadm
    
    # kubelet, kubectl 패키지 다운로드
    apt-mark unhold kubelet kubectl && \
    apt-get update && apt-get install -y kubelet=1.17.0-00 kubectl=1.17.0-00 && \
    apt-mark hold kubelet kubectl

    # 업그레이드 방법 찾기
    kubeadm upgrade -h
    kubeadm upgrade node -h
    kubeadm upgrade node --kubelet-version -h
    # [upgrade] Using kubelet config version -h, while kubernetes-version is v1.17.0

    # 업그레이드 적용
    kubeadm upgrade node --kubelet-version v1.17.0

    # 또는
    kubeadm upgrade node config --kubelet-version v1.17.0
    
    # 또는
    kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2)
    ```

    [kubeadm upgrade](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/)

- node01 Schedulable 상태로

    ```bash
    # kubectl cordon은 지정된 노드에 더 이상 포드들이 
    # 스케쥴링되서 실행되지 않도록 합니다.

    kubectl uncordon node01
    ```