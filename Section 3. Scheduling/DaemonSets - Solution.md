# DaemonSets - Solution

# KEYWORD:
kubectl get ds / 
데몬셋 설정은 레플리카셋 설정과 같다.

- 모든 네임스페이스에서 Daemonset의 개수

    ```bash
    kubectl get ds --all-namespaces

    # 또는 kubectl get daemonsets --all-namespaces
    # 2개
    ```

- 데몬셋은 어느 네임스페이스에 만들어졌나?

    ```bash
    kubectl get ds --all-namespaces

    # 또는 kubectl get daemonsets --all-namespaces

    # kube-system
    ```

- 다음 중 데몬셋인 것은?

    ```bash
    kubectl get -n kube-system
    ```

- kube-proxy에 데몬셋에서 예약한 파드 수

    ```bash
    kubectl get ds --all-namespaces
    kubectl get ds kube-proxy -n kube-system

    # 또는
    # kubectl describe ds kube-proxy -n kube-system | grep -i -A5 -B5 pod
    # kubectl describe ds kube-proxy --namespace=kube-system | grep -i -A5 -B5 pod

    # 2개
    ```

- weave-net 데몬셋에 사용한 image

    ```bash
    kubectl get ds --all-namespaces
    kubectl describe ds weave-net -n kube-system | grep -i image

    # 또는 kubectl describe ds weave-net --namespace=kube-system | grep -i image

    ```

- fluentD 로깅을 위한 데몬셋 배포
    - Name: elasticsearch
    - Namespace: kube-system
    - Image: k8s.gcr.io/fluentd-elasticsearch:1.20

    ```yaml
    # fluentd-ds.yaml
    # 레플리카셋에서 kind만 DaemonSet으로 바꿔주면 됨

    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: elasticsearch
      namespace: kube-system
      labels:
        k8s-app: fluentd-logging
    spec:
      selector:
        matchLabels:
          name: fluentd-elasticsearch
      template:
        metadata:
          labels:
            name: fluentd-elasticsearch
        spec:
          containers:
          - name: fluentd-elasticsearch
            image: k8s.gcr.io/fluentd-elasticsearch:1.20
    ```

    ```bash
    kubectl create -f fluentd-ds.yaml
    ```

    [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)