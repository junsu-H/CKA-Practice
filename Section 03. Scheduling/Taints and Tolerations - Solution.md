# Taints and Tolerations - Solution

# KEYWORD:
kubectl taint nodes [node] key=value:NoSchedule /
kubectl taint /
kubectl taint nodes [node] key:NoSchedule- /
조건부 할당

```bash
# NoSchedule: 노드가 taint를 허용하지 않는 경우,
# 파드가 노드에 스케줄되지 않음을 의미한다.

# PreferNoSchedule: NoSchedule의 소프트 버전이다. 즉, 스케줄러가
# 노드 스케줄을 피하려고 하지만 다른 곳에서 스케줄할 수 없는 경우,
# 노드를 스케줄한다.

# NoExecute: 스케줄링에만 영향을 미치는 NoSchedule나 PreferNoSchedule과
# 달리, 노드에서 실행 중인 파드에도 영향을 미친다.
# NoExecute taint를 노드에 추가하면 해당 노드에서 이미 실행 중인
# 파드와 NoExecute taint를 tolerations하지 않는 파드가 노드에서 제거된다.
```

```bash
# requiredDuringSchedulingIgnoredDuringExecution  
# 반드시 규칙을 만족해야 한다.

# preferredDuringSchedulingIgnoredDuringExecution
# 우선적으로 규칙을 만족시키는 곳에 스케줄링, 
# 규칙 불만족시 아무 곳에나 스케줄링한다.

operator는 In, NotIn, Exists, DoesNotExist, Gt, Lt가 있다.

# In은 color=[blue가 있냐?]
# Notin은 color=[blue가 없냐?]

      # 레이블 기반으로 선호       
      affinity:
        # 노드 선호                                  
        nodeAffinity:
          # 반드시 규칙을 만족해야 한다.
          requiredDuringSchedulingIgnoredDuringExecution:
            # nodeAffinity와 한 세트, 해당 부분을 만족해야 스케줄링
            nodeSelectorTerms:
              # matchSelector와 유사
            - matchExpressions:
              - key: color
                # key에 대한 value 설정
                operator: In
                operator: NotIn
                values:
                - blue

# Exists와 DoesNotExists는 value값이 없다.
# Exists는 key인 node-role.kubernetes.io/master가 존재하냐?
# DoesNotExists는 key인 node-role.kubernetes.io/master가 존재 안하냐?

      # 레이블 기반으로 선호       
      affinity:
        # 노드 선호                                  
        nodeAffinity:
          # 반드시 규칙을 만족해야 한다.
          requiredDuringSchedulingIgnoredDuringExecution:
            # nodeAffinity와 한 세트, 해당 부분을 만족해야 스케줄링
            nodeSelectorTerms:
              # matchSelector와 유사
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                # key에 대한 value 설정
                operator: Exists
```

- 시스템에 몇 개의 노드가 있습니까?

    ```bash
    kubectl get nodes

    # 2개
    ```

- node01에는 taints가 있나요?

    ```bash
    kubectl describe nodes node01 | grep -i taints 

    # grep -i는 대소문자 무시
    # no
    ```

- key='spray', value='mortein' effect='NoSchedule'가 조건인 taints를node01에 만들어라.

    ```bash
    # Base
    kubectl taint nodes [node] key=value:NoSchedule

    kubectl taint nodes node01 spray=mortein:NoSchedule
    ```

    [Taints and Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)

- nginx image로 pod mosquito 생성

    ```bash

    # --dry-run으로 yaml 파일 확인
    kubectl run mosquito --generator=run-pod/v1 --image=nginx --dry-run

    kubectl run mosquito --generator=run-pod/v1 --image=nginx
    ```

    ```yaml
    # mosquito-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: mosquito
    spec:
      containers:
      - image: nginx
        name: mosquito
    ```

- pod가 pending인 이유

    ```bash
    kubectl describe pod mosquito | grep -i warning

    # grep -A100 "KeyWord" xxxx.txt
    # => keyword를 포함한 이후 100라인을 추출

    # grep -B100 "KeyWord" xxxx.txt
    # => keyword를 포함한 이전 100라인을 추출
    ```

- nginx image로 pod bee 생성

    ```bash
    vim bee.yaml
    ```

    ```yaml
    # bee-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: bee
    spec:
      containers:
      - image: nginx
        name: bee
      tolerations:
      - key: spray
        value: mortein
        effect: NoSchedule    
        operator: Equal
    ```

    ```bash
    kubectl create -f bee.yaml
    ```

    [Taints and Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)

- master node에 Taints가 있는가?

    ```bash
    kubectl describe nodes master | grep -i taints

    # output
    Taints:             node-role.kubernetes.io/master:NoSchedule
    ```

- master node에 있는 Taints를 제거하시오.

    ```bash
    kubectl describe nodes master | grep -i taints
    # Taints: node-role.kubernetes.io/master:NoSchedule

    kubectl taint nodes master \
        node-role.kubernetes.io/master:NoSchedule-

    # 또는
    kubectl edit nodes master #에서 taints 항목 삭제

    kubectl describe nodes master | grep -i taints
    # Taints:             <none>
    ```

- Taints를 제거 후 현재 pod mosquito의 상태는?

    ```bash
    kubectl get pods
    ```

- pod mosquito는 어느 node에 배치되어 있나?

    ```bash
    kubectl get pods mosquito -o wide

    # master
    ```