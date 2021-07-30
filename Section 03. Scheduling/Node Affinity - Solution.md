# Node Affinity - Solution

# KEYWORD:
kubectl get nodes node01 --show-labels /
kubectl label nodes [node] [label-key=label-value] /
kubectl

- node01의 labels의 개수는?

    ```bash
    kubectl get nodes node01 --show-labels
    ```

- label beta.kubernetes.io/arch의 value는?

    ```bash
    kubectl get nodes node01 --show-labels | grep beta.kubernetes.io/arch
    ```

- color가 blue인 labels 만들기

    ```bash
    kubectl label nodes node01 color=blue
    # kubectl label nodes <node-name> <label-key>=<label-value>

    # 또는
    kubectl edit nodes node01
    > color: blue 추가

    # 라벨 삭제
    # kubectl label node <nodename> <labelname>-
    # kubectl label node node01 color-
    ```

- 이름이 blue고, replicas가 6개인 deployment를 NGINX 이미지로 만들기

    ```bash
    # deployment는 kubectl run --generator 안 됨.

    kubectl create deploy blue --image=nginx
    kubectl scale deploy blue --replicas=6
    ```

- pod는 어디에 배치되는가?

    ```bash
    kubectl get nodes -o wide

    # master & node01
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

- node affinity를 통해 node01에만 배치되게 할 것.

    ```bash
    # 위에서 만든 deployment 삭제
    kubectl delete deploy blue

    # operator을 위한 옵션 확인
    kubectl get nodes --show-labels
    ```

    ```yaml
    # blue-deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: blue
    spec:
      replicas: 6
      selector:
        matchLabels:  
          run: nginx
      template:
        metadata:
          labels:
            run: nginx
        spec:
          containers:
          - image: nginx     
            name: nginx
          # Pod spec에 있어야 한다.

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
                    values:
                    - blue
    ```

    ```bash
    kubectl create -f blue-deployment.yaml
    ```

    [Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)

- 다음을 충족하는 deplyment를 만드시오.
    - Name: red
    - Replicas: 3
    - Image: nginx
    - NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
    - Key: node-role.kubernetes.io/master
    - Use the right operator

    ```bash
    # operator을 위한 옵션 확인
    kubectl get nodes --show-labels \
        | grep -i node-role.kubernetes.io/master
    ```

    ```yaml
    # red-deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:  
      name: red       
    spec:       
      replicas: 3     
      selector:
        matchLabels:
          run: nginx
      template:
        metadata:
          labels:
            run: nginx
        spec:
          containers:
          - image: nginx
            name: nginx
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

    [노드에 파드 할당하기](https://kubernetes.io/ko/docs/concepts/configuration/assign-pod-node/)