# Practice Test - ReplicaSets - Solution

# KEYWORD:
kubectl get rs /
kubectl edit /
kubectl scale --replicas

- pod 개수 세기

    ```bash
    kubectl get pods
    ```

- ReplicaSet 개수 세기

    ```bash
    kubectl get rs
    kubectl get replicaset
    ```

- ReplicaSet에서 생성한 Pod 개수 세기

    ```bash
    kubectl describe rs
    kubectl get rs -o wide
    ```

- ReplicaSet에서 Pod를 생성할 때 사용한 image 확인

    ```bash
    kubectl describe rs [레플리카셋 이름]
    kubectl get rs -o wide
    ```

- ReplicaSet에서 READY 상태 확인

    ```bash
    kubectl describe rs [레플리카셋 이름]
    kubectl get rs -o wide
    ```

- Pod가 왜 wait 상태인지 확인

    ```bash
    kubectl get pods
    kubectl describe pods new-replica-set-5szch
    kubectl logs new-replica-set-5szch # logs는 pod만 가능
    ```

- ReplicaSet가 만든 Pod 중 아무거나 하나 삭제

    ```bash
    kubectl delete pods --all
    kubectl delete pods new-replica-set-4fhmn

    # kubectl delete rs [레플리카셋 이름]
    ```

- replicaset-definition-1.yaml 수정해서 생성하기

    ```yaml
    # replicaset-definition-1.yaml

    # apiVersion: **v1 아래와 같이 수정**
    apiVersion: **apps/v1**
    kind: ReplicaSet
    metadata:
      name: replicaset-1
    spec:
      replicas: 2
      selector:
        matchLabels:
          tier: frontend
      template:
        metadata:
          labels:
            tier: frontend
        spec:      
          containers:
          - name: nginx        
            image: nginx
    ```

    ```bash
    kubectl create -f replicaset-definition-1.yaml
    ```

- replicaset-definition-2.yaml 수정해서 생성하기

    ```yaml
    # replicaset-definition-2.yaml

    apiVersion: apps/v1
    kind: ReplicaSetmetadata:
      name: replicaset-2
    spec:
      replicas: 2
      selector:
        matchLabels:
          tier: frontend
      template:
        metadata:
          labels:
    #       tier: nginx 아래와 같이 수정
            **tier: frontend**
        spec:
          containers:
          - name: nginx        
            image: nginx
    ```

    ```bash
    kubectl create -f replicaset-definition-2.yaml
    ```

- replicaset-definition-1, replicaset-definition-2 삭제

    ```bash
    kubectl delete -f replicaset-definition-1.yaml
    kubectl delete -f replicaset-definition-2.yaml

    # 또는
    kubectl get rs
    kubectl delete rs replicaset-1 replicaset-2

    # 또는
    kubectl delete rs replicaset-1
    kubectl delete rs replicaset-2
    ```

- Pod image busybox777에서 busybox로 교체

    ```yaml
    kubectl edit rs new-replica-set # kubectl edit rs [rs 이름]

    # > image: busybox777
    > image: busybox

    kubectl get rs
    kubectl get pods
    kubectl describe pods []
    kubectl get rs new-replica-set -o yaml > new-replica-set.yaml
    kubectl delete rs new-replica-set
    kubectl create -f new-replica-set.yaml

    kubectl get rs
    kubectl get pods
    ```

- replicas=5로 바꾸기 ( kubectl edit 사용 )

    ```bash
    # kubectl edit으로 바꾸기
    kubectl edit rs new-replica-set

    # > replicas=4
    > replicas=5
    ```

    ```bash
    # scale로 바꾸기

    kubectl get rs
    kubectl scale rs new-replica-set --replicas=5
    ```

    ```bash
    # yaml 파일로 바꾸기

    kubectl get rs new-replica-set > new-replica-set.yaml
    vim new-replica-set.yaml

    > replicas=5 
    변경 후

    kubectl replace -f new-replica-set.yaml
    ```

- replicas=2로 바꾸기 ( kubectl scale 사용 )

    ```bash
    # kubectl scale로 바꾸기
    kubectl scale --replicas=2 -f new-replica-set.yaml
    ```

    ```bash
    # type, name으로 바꾸기
    kubectl scale [type] [name] --replicas=2

    ex) kubectl scale --replicas=2 replicaset new-replica-set
    ex) kubectl scale rs new-replica-set --replicas=2
    ```

# scale URL

[Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#scale)

[kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)