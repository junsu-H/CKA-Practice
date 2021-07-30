# OS Upgrades - Solution

# KEYWORD: 
kubectl drain /
kubectl cordon /
kubectl uncordon

```bash
# kubectl drain은 노드 관리를 위해 스케줄링이 안 되게 하고 
# 지정된 노드에 있는 포드들을 다른곳으로 이동시키는 명령

# kubectl cordon은 지정된 노드에 더 이상 파드들이 
# 스케줄링되지 않도록 하는 명령

# cordon은 기존에 있던 pod를 삭제하지 않고 노드 제한만 하고, 
# drain은 기존에 있던 pod를 다른 노드로 이동하고 삭제한 후 노드 제한

# drain은 rs로 만들어진 pod만 다른 노드로 이동한다.
# rs를 통해 만들어지지 않은 pod는 영구 삭제된다.
```

- 노드 개수는?

    ```bash
    kubectl get nodes

    # 4개
    ```

- deployment 개수는?

    ```bash
    kubectl get deploy

    # 2개
    ```

- pod는 어느 노드에 할당되어 있나?

    ```bash
    kubectl get pods -o wide

    # node01, node02, node03
    ```

- node01에 모든 응용 프로그램의 노드를 비우고 예약 할 수 없도록 표시하십시오.

    ```bash
    kubectl drain node01 --ignore-daemonsets
    ```

- 현재 pod가 할당되어 있는 노드

    ```bash
    kubectl get pods -o wide
    # node02, node03
    ```

- node01에 다시 예약 가능하게 설정

    ```bash
    kubectl uncordon node01
    ```

- 현재 node01에 있는 pod

    ```bash
    kubectl get pods -o wide

    # 0개
    # 포드가 없는 이유는 새 포드가 생성 될 때만 예약된다.
    ```

- 마스터 노드에 pod가 없는 이유

    ```bash
    kubectl describe nodes master
    kubectl describe nodes master | grep -i taints

    # taint이 설정되어 있다.
    ```

- node02에 모든 응용 프로그램의 노드를 비우고 예약 할 수 없도록 표시하십시오.

    ```bash
    kubectl drain node02 --ignore-daemonsets

    # 안 됨
    ```

- node02에 왜 force가 필요한가?

    ```bash
    kubectl get pods -o wide
    kubectl get rs

    kubectl describe pods blue-68df9fb9d9-j67jc
    kubectl describe pods blue-68df9fb9d9-zfxpd
    kubectl describe pods hr-app # 얘
    kubectl describe pods red-5dc59c9654-ft8hd

    # kubectl describe pods hr-app | grep -i Controlled By:
    # 위 명령어로 확인 가능, 아무 것도 안 뜨면 기본 pod
    # 레플리카셋으로 만들어진 pod 이외에 pod가 존재한다.
    ```

- pod에 hr-app force를 적용하면 어떻게 되나?

    ```bash
    hr-app은 영원히 사라진다.
    ```

- node02에 drain, unschedulable을 하라

    ```bash
    # drain 노드를 비우도록 지시

    kubectl get nodes
    kubectl drain node02 --ignore-daemonsets --force
    ```

- pod를 삭제하지 않고 node03을 unschedulable하게 하라

    ```bash
    kubectl cordon node03
    ```